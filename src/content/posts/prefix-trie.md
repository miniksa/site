---
title: "Prefix Trees"
date: 2024-01-01T15:32:00-08:00
draft: false
tags: ["code"]
---

Okay, so I know the entire point of taking a vacation is to not work. But I couldn't get this bug of an idea out of my head the entire time. Or at least for the first week or so until I got some code out of it.

## Background

At work, I've had this issue where we're trying to keep track of a TON of file system paths in a relational database. The existing method has them all in one gigantic string table (to eliminate duplicates, they do not insert the string if it already exists and instead return the ID of the existing string... so basic string interning stuff.) At runtime, in memory, it's one gigantic block of strings. All null terminated. So completely uncompressed. Which makes it super easy to hand out pointers to each of those strings to other elements and have them refer directly to them when they need to look at that information. 

Downside, however, is that there is also a bunch of time spent measuring the length of those strings... over and over... because every access has to count the whole way through to know that it's not going to fail. I mean, technically they wouldn't have to and could just search forward until reaching a null. But that doesn't work for things like suffix matching which need to start from the end. And it leads to patterns where because we force "safe string" functions like the CRT ones ending in `_s`... folks will measure them all first and then feed the max lengths in before doing any other operation. So. Measuring. Over and over. And over.

There is one good part though. At rest, the strings are stored compressed. Somewhat. They use LZNT1 through the [RTLCompressBuffer](https://learn.microsoft.com/windows-hardware/drivers/ddi/ntifs/nf-ntifs-rtlcompressbuffer) function. In 1MB chunks. Which is substantially worse than compressing it solid with anything more modern, especially like zstd or 7-zip's LZMA or whatnot given it's just a bunch of text and has so much duplication as paths.

The problem comes that when all this text is in memory... we're starting to run into troubles with how many gigs it takes. And when people are doing prefix and suffix searches (for "files in this directory" or "files with this extension"), it can often take longer than I really think it ought to.

## Thoughts

A few of my co-workers have solved this problem in their similar tool through using a Prefix Tree (or a "trie"). Though I was told "don't use our implementation; it's not great."

So I went looking for ones that I could drop in and maybe solve both of our problems.

## Research

The main one that I found that looked awesome was [hat-trie](https://github.com/Tessil/hat-trie). Diagrams, benchmarks, research papers... that indicates usually a pretty damned good library to use. Then I can let someone much better at algorithms than me (what I affectionately refer to as "a bag of PhDs" since I couldn't ever brain hard enough to go get one) write the nitty gritty performant details and I'll go off and write the glue goop like I'm good at. This strategy worked great when I identified a problem in this same code that was solved with a custom bitmap logic and found the "bag of PhDs" solution in [RoaringBitmap](https://github.com/RoaringBitmap/RoaringBitmap) which was much faster and much better memory efficiency than our custom existing one.

There's also a C implementation out there. And then I found a handful of C# implementations of assorted tries in libraries that just didn't look quite right to me. I wrote down all the ones I looked at when I uploaded the [README](https://github.com/miniksa/trie/blob/main/README.md).

## Debate

Okay so the issue here is... I wanted it to be in .NET Core C# code. And this is frustrating to me because I spent the last few months championing how easy it is to mix and match C++ and C# code by using P/Invokes as they're not as scary as everyone thinks, they can be damned good performing when you're not marshalling strings all day, and they technically work on Windows and Linux. This means that I should probably just be able to wrap the hat-trie with P/Invokes and be good to go. Build it for all targets, pack it next to the .NET Core, and we're golden.

Yeah okay. Except in the last few weeks, one of my attempts at removing custom crap in the native side went horrifically wrong when I tried to introduce [mimalloc](https://github.com/microsoft/mimalloc) and I'm thinking it's probably wisest long-term to just get everything into the C# half of the fence. When building two langauges... now you have two complete toolchains worth of problems. 

So I want a .NET Core solution. And I want it to hopefully be efficient as hell in terms of allocations (not new up everything all over the place and leave it to the garbage collector to clean up the mess as too many allocations is a perf problem both in managed C# and native C++). And I want to generally understand what's going on with it. And I want it to operate on compressed tries because I have paths (that is, each subpath should be one node instead of each individual character being a node like a traditional trie.) And I'd like it to use `byte` for the characters so it's UTF-8 friendly instead of having to transit back and forth from ASCII to UTF-16 to UTF-8 and around and around. And I'd really like for it to be able to do partial matches (Contains) or prefix matches (StartsWith) very quickly. And ideally I'd like to teach it some amount of suffix behavior too so extensions (more or less EndsWith) or just the file names (the leaf nodes) can also be referenced quickly.

And that basically left me with doing it myself until proven otherwise. I hate it, but sometimes it is what it is. And if it's anything like it was working on [Terminal](https://github.com/microsoft/terminal), someone will prove me wrong a few years down the road and this will work until then and we'll erase my code in favor of it.

## Solution

I came up with this bugger on [Github at miniksa/trie](https://github.com/miniksa/trie/tree/main). 

Basically, you start with the `Trie` and add UTF-8 strings as sequences of `byte`. They'll be split on the separator character for paths on Windows, backslash (`\`), (and yes I know I'll have to make it account for forward slash (`/`) also for Linux but baby steps). And each subpath will be stored in a `Node`.

`Node`s contain 4 32-bit integers that refer to all the information they need to move around. So each one is 128-bits big. I wanted it to be a nice round multiple of 32-bits. I give each one an ID for which string is its piece of the path, one ID for which other `Node` is its parent, and then two IDs for any nodes that are its child `Node`s as an index and length pair (why will become apparent when I talk about the children more soon.)

Then I made other Block classes to track all the other pieces of information which are the substrings themselves representing parts of the path (`StringBlock`), the children of any given node which can be 0 to N of them (or an array of them... `ChildrenBlock`), and all of the storage for the Nodes themselves (`NodeBlock`). These are "blocks" because I want to line up all this information in one big gigantic contiguous array to avoid allocations and enable very fast access as they'll be offsets from a pointer instead of sprayed all over a heap. Theoretically it also means I should be able to serialize and deserialize this very quickly or even memory-map file them as it's just raw dumping of bits from memory to disk and back again.

`NodeBlock` is just all of the nodes currently allocated. Every block has an ID to it which is implicit on its position in the array. 0 is the root node which represents nothing and just has children. -1 here and everywhere else is used as an indicator for "I don't have one". So the root node has -1 as its parent. So it's 128-bits per block and they're all one big array in a row. Since I use a `List<T>` for this, they'll just be reallocated larger geometrically (multiplying by 2) every time I reach the bounds and need to add another one so they'll move in memory when that happens, but still be contiguous and numbered in the same way.

```
0      1      2
NODE | NODE | NODE | ....
```

`StringBlock` is all of the substrings currently allocated WITHOUT null terminators in one giant array. Each string ID is just the index to its first character in the giant array memory block. To avoid the "measuring the strings over and over" problem and the "have I already seen this string" problem, the lengths and [XXH3](https://github.com/Cyan4973/xxHash) hashes (which for .NET can be found in [System.IO.Hashing](https://www.nuget.org/packages/System.IO.Hashing/)) of all the strings are in two dictionaries inside of here as well matching string IDs to lengths and string hashes to IDs. ID to length should just be 64-bits per record... and hash to ID is... 64+32bits which is 3 32-bit words and an odd number which could be waste... but I might worry about that later. It's important to keep this metadata here because multiple nodes might refer to the same string.

```
0         10        20
Iamstring1Iamstring2Iamstring3...
```

`ChildrenBlock` is a bunch of arrays all lined up in a row in one gigantic array representing the children. The IDs are the beginning of the array and the Lengths were stored/known by the nodes for how many child IDs are in that array. As no two nodes should have the same children, I don't keep the metadata about the array length here and instead let each Node hold it. That makes it weird to know when we're running up to the capacity of a given "allocation" out of the mega array, but I avoid that by rounding to powers of 2 from whatever I was told to get that similar geometric expansion pattern of `List<T>`. I also have a little internal dictionary that recycles smaller arrays when a batch of children move up and into a bigger space to mildly avoid fragmentation.

```
0 2  6  9
123645897...
```

And that's about what I came up with.

I got to the point where I wrote a crappy amount of unit tests mostly to exercise the whole thing, running it through a performance profiler over and over and tweaking (which was the inner-loop by which I developed the block tables strategies above), and then got a result where I got about 500,000 paths from my laptop which was about 60MB on disk loaded entirely into memory in about 1 second and consuming about 20-25MB of RAM (so less than half of before). And that seemed pretty darn good.

I'm sure the one at work has a lot more paths than this but cutting it down to 30-50% of its existing memory usage is amazing. And I haven't implemented all the starts/ends/contains/extension searches yet but those will probably be pretty quick as well (gut is speaking here from past profiling experience.)

## Conclusion

With all that settled and proving that a trie would indeed do the job here and could be pretty damned fast and memory efficient and allocation efficient in C#... I parked it. The bug was now out of my head and I enjoyed the rest of my holidays.

I threw it up on GitHub so I could easily pull it back down at work if I want to use it as a reference or to extend it. Or so the next goof who goes searching the internet for something like this can stumble across my process and solution and maybe it will help them. Or if I get lucky, they'll read it and it'll be the wrong answer and by [Cunningham's Law](https://meta.wikimedia.org/wiki/Cunningham%27s_Law) I'll get an even better solution than this one in the future.

