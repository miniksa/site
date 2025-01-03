---
title: "Ikea Tradfri Shortcut Button"
date: 2025-01-02T15:28:00-08:00
draft: false
tags: ["home automation", "zigbee", "ikea", "home assistant"]
---

# Getting the Ikea Tradfri Shortcut Button to work with Home Assistant

## Background

[Reference images](#reference-images) of the button, its box, and the Zigbee radio can be found below.

### Ikea Button
- Ikea Part Number 203.563.82
- E1812 Shortcut Button (per the manual inserted)
- Input: 3V DC, CR2032 Battery

### Nortek GoControl Z-Wave/Zigbee Quick Stick Combo
- See also: https://www.homecontrols.com/GoControl-Z-Wave-Zigbee-QuickStick-LNHUSBZB1
- Never done a firmware update  
- Channel 15
- Connected via a 6 foot USB extension cable to the Home Assistant to avoid USB port interference

### Home Assistant
- [Home Assistant Blue](https://www.home-assistant.io/blue/) host device
- HAOS version 13.2
- HA Core version 2024.12.5
- Zigbee Home Automation (ZHA) integration in use

## Setup

[Reference images](#home-assistant-ui) of the UI available below.

1. Open Home Assistant Dashboard
2. Go to `Settings` > `Devices & Services` > `Zigbee Home Automation`
3. Press `CONFIGURE` on the integration
4. Open the back of the Ikea Tradfri button with a small Phillips head screwdriver
5. Insert a fresh CR2032 battery
6. Hop back to Home Assistant and press `+ ADD DEVICE`
7. Click the pairing button 4-5 times
8. Observe a red LED will blink a few times on the front side of the button then pulse. This takes maybe 10 seconds.
9. Observe Home Assistant will transition to `Configuring` then `The device is ready to use`. This takes about 10 more seconds.
10. Fill out a device name and an area
11. Go back to devices and find the device
12. Click the triple dot near `Device Info` then choose `Manage zigbee device`
13. In the `Bindings` tab, use the `Bindable devices` drop down and choose the Zigbee hub. Mine was `HubZ ZigBee HUSBZB-1`. Press `BIND` and wait 30-60 seconds. Perhaps click the button once or twice to wake the button while you wait.
14. Confirm it's working. Use `Developer Tools` then the `Events` tab then say `Listen to events` on the `zha_event` channel and click `Start Listening`. If it worked, you will see events at the bottom on each button press event.
15. If not, try pairing again and starting from there. I found it worked better when paired near the hub stick than near where its final location was supposed to be.
16. When you confirm it works, make an automation triggering on the `Turn on` or the `Dim up` events. Note that `Turn on double clicked` never reported correctly for me.

## Other advice I saw when searching
- Upgrade firmware on radio. 
    - Didn't do this. Didn't want to upset existing network.
- Upgrade firmware on button. 
    - Couldn't figure out how. Some suggested using an Ikea Dirgiera hub or Tradfri gateway. Don't have one. Others suggested using zigbee2mqtt... I don't use that. Even more others suggested doing strange incantations with ZHA to give it a firmware file and/or dispatch manual Zigbee commands to the device. The thought gave me a headache.
- Hold the pairing button for 30-60 seconds without clicking the front. 
    - This didn't do anything for me.

## Conclusion
- These buttons are not good. They're extremely finnicky to set up with Home Assistant. I would not buy them again. Still on the hunt for good ones as the Aqara ones have different flaws (dropping off network occasionally and having to try pairing mutiple times until it decides to play nice).

## Reference Images

### Ikea Tradfri Button
#### Front
![Photo of the front side of the Tradfri Shortcut Button. It is a rounded corner square and mostly white. A small logo of a ceiling mounted lamp in a very basic style is printed in the center.](../../static/2025-01-02-Tradfri-Button-Front.jpg)

#### Back
![Photo of the reverse side of the Tradfri Shortcut Button. Model numbers, names, voltages, FCC IDs, and other regulatory information is visible including an IP44 rating.](../../static/2025-01-02-Tradfri-Button-Back.jpg)

#### Inside
![Photo of the inside reverse of the button. A Duracell CR 2032 battery is in place for most of the device. The top right corner shows a small levered button with a logo of a chain beside it as the pairing button. A screw hole is visible at the bottom where the back cover attaches.](../../static/2025-01-02-Tradfri-Button-Inside.jpg)


### Ikea Tradfri Button Box
#### Front
![Photo of the front side of the Tradfri button box. A picture of the device, its name, marketing information for the app, a Zigbee logo, and an Ikea product quality symbol are visible as well as a sample usage. Also lists what is included in the box (button, stickers) and not included (gateway).](../../static/2025-01-02-Tradfri-Button-Box-Front.jpg)
#### Back
![Photo of the reverse side of the Tradfri button box. Regulatory information, Ikea stock numbers, barcodes, and a message that the Ikea Gateway is Required in over a dozen languages is printed.](../../static/2025-01-02-Tradfri-Button-Box-Back.jpg)

### Nortek Stick
#### Front
![Photo of Nortek Security & Control USB dongle model HUSBZB-1 from the front. Nortek Security and Control logo is visible.](../../static/2025-01-02-Nortek-Stick-Front.jpg)

#### Back
![Photo of Nortek Security & Control USB dongle model HUSBZB-1 from the back. Serial, model number, and regulatory information is visible as is the end of an AmazonBasics USB extension cable plugging into the device.](../../static/2025-01-02-Nortek-Stick-Back.jpg)

### Home Assistant UI

#### ZHA UI Navigation

ZHA Integration page featuring *CONFIGURE* link:

![The Home Assistant Zigbee Home Automation integration page. One instance of the integration is visible with a blue CONFIGURE link to the top right](../../static/2025-01-02-ZHA-Configure.png)

ZHA Integration Configuration page featuring *+ ADD DEVICE* button:

![The Home Assistant Zigbee Home Automation integration configuration page. In the lower right corner is the Add Device button](../../static/2025-01-02-ZHA-AddDevice.png)

ZHA Integration searching for devices:

![A Searching for Zigbee devices page from the ZHA integration after Add Device was clicked. Some instructions and documentation are listed.](../../static/2025-01-02-ZHA-Searching.png)

ZHA Integration found device and configuring:

![Searching for Zigbee devices page from the ZHA integration when the button is found and configuring. A blue rounded rectangle with model and address information is visible.](../../static/2025-01-02-ZHA-Tradfri-Configuring.png)

ZHA Integration found device and complete; allows name and location setup:

![Searching for Zigbee devices page from the ZHA integration when the button is found and configuring. A green rounded rectangle with the device title, some icons, and two fields: "change device name" with a text box and "area" with a drop down are visible.](../../static/2025-01-02-ZHA-Tradfri-Ready.png)

Specific ZHA device page featuring the *...* link with pop-up menu and the *Manage zigbee device* field:

![The specific device page for the Tradfri button inside Home Assistant. Many regions are visible, but the focus is to the top-left most region of Device Information and then the lower right corner of that where the `...` link is opened to show a pop-up menu with 4 options. The top one entitled "Manage zigbee device" is the important one.](../../static/2025-01-02-ZHA-Manage-Zigbee-Device.png)

*Manage zigbee device* dialog featuring the *Bindings* tab where the **BIND** button can be triggered against the Zigbee hub:

![Screenshot of Manage Zigbee Device page in Home Assistant. The Bindings tab in the center of the top row is selected. The focus area is the first group region with a dropdown titled "Bindable devices" with "HubZ ZigBee HUSBZB-1" option selected. Two buttons are active below this field, BIND and UNBIND. Another group region with a table of checkboxes, names, and IDs for Bindable groups is below.](../../static/2025-01-02-tradfri-bind-hub.png)

#### zha_event Debugging

How to set up event listening:

![Screenshot of the Developer Tools, Events page in Home Assistant. The top region allows firing an event manually. The next region down says Listen to Events. `zha_event` is in the field and is grayed out as active. The button says `Stop Listening` and `Clear Events` and below is one event's output](../../static/2025-01-02-zha_event_listen.png)

Sample event received:

![Screenshot of a zha_event arriving in the Home Assisstant developer tools page. Many fields of JSON information are visible including the timestamp of the event, addressing information for the message, and a command string of "on"](../../static/2025-01-02-zha_event_confirmed.png)





