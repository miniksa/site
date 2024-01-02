---
title: "Kittens Game"
date: 2024-01-01T16:32:00-08:00
draft: false
tags: ["games"]
---

I don't know why I'm doing this again. I'm a sucker for idle games I guess.

I decided to play Kittens Game again. I've been playing other crappy idle games on my phone on and off for a while. But I remember this one with a fondness. I played the [Paperclips one](https://www.decisionproblem.com/paperclips/index2.html) again earlier this year but it lasts like a day. I think maybe it's just a resurgance of having some stupid low-stakes fun that I used to do before the pandemic.

Not too much else to say about it other than we'll see how long this lasts.

### Notes:

#### Play the game
https://kittensgame.com/web/#

#### Add some progress bars
Make bookmark link in the bookmarks bar with target:
```javascript
javascript: (function () { var kgp = document.createElement('script'); kgp.src = 'https://cdn.jsdelivr.net/gh/vl20100/KGProgressBars@0.0.1.d/dist/KGP.js'; kgp.id = 'kpgscript_TriggerNotify'; document.head.appendChild(kgp); })();
```
(Source: https://github.com/vl20100/KGProgressBars)

#### Automate Stuff
Make bookmark link in the bookmarks bar with target:
```javascript
javascript:(function(){var d=document,s=d.createElement('script');s.src='https://kitten-science.com/stable.js';d.body.appendChild(s);})();
```
(Source: https://kitten-science.com/installation/)

#### Speed it up because you're impatient and don't have literal MONTHS or YEARS to grind out the first few resets like when you were younger
Open F12 developer prompt
Run:
```javascript
gamePage.ticksPerSecond = 50; // Or 30 or 80 or whatever
gamePage.start()
```
(Source: https://www.reddit.com/r/kittensgame/comments/8nwy9s/increasing_game_speed/)
