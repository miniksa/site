---
title: "M5Stack Atom Echo Setup into ESPHome"
date: 2024-01-19T19:05:00-08:00
draft: false
tags: ["home assistant"]
---

## Background

So I want voice assistants, but I don't want to send all my data to either Google or Amazon. Fortunately, Home Assistant called 2023 their *Year of Voice* and worked a lot on voice last year. Instead of dealing with it being half-baked all year, now that it's 2024... I'm going to try it out.

I'm following [Home Asssistant Blog's $13 assistant blog post](https://www.home-assistant.io/voice_control/thirteen-usd-voice-remote/).

I went and bought one of these things from M5Stack. I believe they charged shipping and it came directly from China so like AliExpress... it took a while and I probably should have ordered a handful other things at once. If these work well, I guess I'll make up for it by buying a pile more in one batch.

## Notes
- Following [the blog post](https://www.home-assistant.io/voice_control/thirteen-usd-voice-remote/) works fairly well for everything except the Install.
- If you install how they say with the button on the page, it will show up for automatic discovery in the **Integrations** section, but it won't be adoptable or show up in the **ESPHome** extension. Boo. It seems I cannot push updates to it unless I do it this way.
- To remedy this, install from **ESPHome** instead.
- I basically copied my [ratgdo](https://ratgdo.github.io/esphome-ratgdo/) configuration. I have no idea how I ended up with this configuration as that site has a button too. Maybe it did this.
    1. Click `+ New Device` in the **ESPHome** panel
    1. Throw down this:
    ```yaml
    substitutions:
    name: m5stack-atom-echo
    friendly_name: m5stack-atom-echo
    packages:
    m5stack.atom-echo-voice-assistant: github://esphome/firmware/voice-assistant/m5stack-atom-echo.yaml@main
    esphome:
    name: ${name}
    friendly_name: ${friendly_name}
    api:
    encryption:
        key: <I don't remember what I put here, but it was something and might have even been whatever I stole from ratgdo... doesn't matter... I think it's base64 something...>
    wifi:
    ssid: !secret wifi_ssid
    password: !secret wifi_password
    ```
    3. Flash it over USB to the Echo.
    4. When it comes online, it'll be discovered and adoptable in **ESPHome** but ask to do another installation to set an API key. I think it erases whatever gunk I put above and makes a new one at this point.
    5. After that whole installation occurs again, it shows up just fine in **ESPHome**. You can delete the other non-suffixed one now.
    - This is what the config looked like after it reflashed it:
    ```yaml
    substitutions:
        name: m5stack-atom-echo-b82cec
        friendly_name: m5stack-atom-echo b82cec
    packages:
        m5stack.atom-echo-voice-assistant: github://esphome/firmware/voice-assistant/m5stack-atom-echo.yaml@main
    esphome:
        name: ${name}
        name_add_mac_suffix: false
        friendly_name: ${friendly_name}
    api:
        encryption:
        key: <something else that looks like base64>
    wifi:
        ssid: !secret wifi_ssid
        password: !secret wifi_password
    ```
    6. Then pop back over to Home Assistant's **Integrations** and it shows up as discoverable. Add it there too. All good now.
        - If it's complaining that the API key doesn't match when you copy and paste it over... just go back to **ESPHome** and hit `Edit` then `Install` and flash it again... and it'll probably work the next time.

Now to figure out how this thing works...
