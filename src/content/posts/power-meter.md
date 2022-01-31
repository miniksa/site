---
title: "Home Power Monitoring with PSE"
date: 2022-01-30T20:04:00-08:00
draft: false
tags: ["home automation"]
---

In the last few months, [Home Assistant](https://www.home-assistant.io/) added a new feature to display *Energy* usage in a pretty dashboard. However, it wasn't clear to me how to get the data from my house into the tool.

I spent some time evaluating the [energy add-ons](https://www.home-assistant.io/integrations/#energy) trying to see what I could use to do it. I basically came up with *CT Clamp* sensors (little devices that snap around the wires and measure current to calculate total power).

My first thought was [Sense](https://sense.com/) which is probably the sleekest and most streamlined solution. However it's $300+ and it appears to rely entirely on their cloud system to collect all the data. Reviews I read also seemed to suggest that their interface hasn't improved at all since launch. If their "AI" doesn't recognize your device by its power signature, there's no manual way of teaching it about new devices. Still. After years on the market. This gives me serious *Nest Thermostat* vibes where it's a good product, but it's languished on the vine with no improvements/updates.

My next thought was [Shelly 3PM](https://shelly.cloud/products/shelly-3em-smart-home-automation-energy-meter/). I have used *Shelly* relays for other things in my house before. They're definitely not as sleek or polished as other solutions, but they make up for it in how open their software and hardware is combined with the strong community. However, where I got stuck here was trying to figure out how to apply this to a US panel. *Shelly* is a Europe-first company and while their devices generally work for the US as well... it can take some thinking.

For either of these solutions, though, I'd have to open up the main electrical panel and install *CT Clamps* around the 100 amp main wires... as well as potentially add a new breaker and circuit for it to get a reference voltage so the math was correct and it could power the device. Yuck. I only like electrical work when I can turn the breaker off and that doesn't really apply to the mains wires. Well, I mean, or I could hire an electrician to do it. Doesn't sound like fun.

But then... then I had the brain wave. I walked past my new *smart meter* after Puget Sound Energy's [Meter Upgrade Project](https://www.pse.com/pages/meter-upgrade) and noticed that it said "Zigbee" on it. WELL. Zigbee I can do.

Of course... it wasn't that easy. Turns out that the Zigbee in the smart meters isn't like the "push a button on it and it pairs with your smart home" type that is usually available. Or maybe there is a button on it behind the locked door that pairs it to PSE's network and it's not for me. A look around the internet seemed to suggest this is a special encrypted Zigbee called "Zigbee Smart Energy Profile" that consumers normally can't see. Well that's annoying.

I found [Rainforest Automation](https://www.rainforestautomation.com/homeowners/), a company that seemed to provide little modules that could read the "Zigbee Smart Energy Profile", but they didn't list a partnership with PSE.

I emailed PSE trying to ask if I could get access to it and nothing. They responded back with seemingly not even understand what I was asking for or what a smart meter even was. Big surprise.

So I waited a few months. And on yet another day of that little *Energy* icon torturing me in the Home Assistant interface, being unconnected to anything, I emailed *Rainforest Automation* and asked them if they could partner with PSE.

Lo and behold, a few days later, they told me they **were** partnering and had a trial program available at [PSE's site](https://www.pse.com/rebates/home-energy-display).

And here's the solution for folks who have a smart meter and can get one of these *Rainforest Automation* devices (in specific an *EMU-2* device that operates locally only and requires no cloud service):

1. Buy it. It's paired by the utility and Rainforest Automation to the specific encryption code or whatnot that is a part of the meter before shipping.
1. Unbox it. It's a little pod powered over a Micro USB connector. It comes with a USB power brick.
1. Let it turn on. And it works. It'll just start showing you, within a few seconds, the real time usage of the meter. The little up/down buttons on the right scroll through pages about history and the absolute meter read value and the date/time it sees from the meter and so on.
1. *Bonus:* Hold both buttons then release to get into a menu. It's a very clunky menu but you can set the price per kwh and a few other things. It technically lets you set the price to an accuracy of 1/100 of a cent, but since that fluctuates every few months anyway around here... I just guessed 11 cents per kWh.

Ta-da. It shows the real-time (well... about 8 seconds delayed) usage and price all day every day! Now go flip stuff on and off and watch what happens!

![A photo of the EMU-2 monitor on my desk reading out the current usage at 0.787kW and estimating I'm using 8.6 cents per hour of power right now. The red-yellow-green stop light to the left is showing yellow.](/2022-01-30-emu-2.png)

I've noticed that the little stop-light on the left shows green for 0.5kWh or less, yellow for up 2kWh, and red beyond that.

But wait, you say, how does this have anything to do with Home Assistant? Well...

1. Instead of powering it off the brick, power it off a USB port on whatever device hosts Home Assistant.
1. Using [HACS](https://hacs.xyz/), the Home Assistant Community Store tool (which I don't remember how I installed in the first place, maybe I'll write that up later), add the [ha-rainforest-emu2](https://github.com/damienheiser/ha-rainforest-emu2) repository. *NOTE:* the Rainforest integration from the regular [Home Assistant integrations page](https://www.home-assistant.io/integrations/rainforest_eagle/) only seems to support the other product, the *Eagle-200* not the smaller *EMU-2* one.
1. To the `configuration.yaml` for *Home Assistant*, add this (that I stole from somewhere online, if I ever find where, I'll credit it):

```YAML
sensor:
  - platform: rainforest
    port: "/dev/ttyACM0"
  - platform: template
    sensors:
      grid_energy_delivered:
        unique_id: delivered_kWh
        friendly_name: "Delivered kWh"
        unit_of_measurement: "kWh"
        value_template: "{{ state_attr('sensor.rainforest_energy_monitoring_unit','Delivered kWh') }}"
        device_class: energy

utility_meter:
  grid_energy_consumed:
    source: sensor.grid_energy_delivered
    name: "Grid Energy Consumed"
```

4. Reboot. As you do when messing with *HACS* and the `configuration.yaml`.
1. Go to **Configuration** > **Energy**. Then under the *Electricity Grid* heading, do **Add Consumption** and you should be able to choose `sensor.grid_energy_consumed`. Set a static price (which I did at 0.11 USD/kWh). And **Save**. Then wait a bit and it will start showing on the *Energy* dashboard! ![Screenshot of the Energy dashboard in Home Assistant displaying a bar graph of historical hourly data and an estimated total price for my home's power usage today.](/2022-01-30-energy-dashboard.png)
1. For fun, I also added an *Entity* card to my usual dashboard set to `sensor.rainforest_energy_monitoring_unit` to get a little live view like this: ![Screenshot of entity card in Home Assistant interface reading out the current kWh consumption, just like the physical EMU-2 device.](/2022-01-30-entity.png)

So that's that. Now I see the energy usage in Home Assistant aggregated over the day/week/month or as real time on the card on my usual dashboard. Or for funsies, I walk past the read-out on my desk when flipping things on and off through the day.

Of course... now I'm wondering if I can break it down per device... a thought for another day.
