+++
title = "Pentair Jung Pumpen automation"
date = "2025-08-31T21:53:02+02:00"
author = ""
authorTwitter = ""
cover = ""
tags = ["pentair", "pump", "jung", "automation"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++
My house always had this strange device called a PENTAIR Jung Pumpen AGR. This may just be the most overpriced thing I have ever seen, it's basically a passthrough plug with a small sensor that can detect a leak and sound a relatively loud alarm. There is no configuration and is IP20 rated only. And this thing seems to cost > 300eur. I've never payed much attention to it, but recently since I got a small leak (luckily no real damage) this device managed to wake us up and meant we could clean up before any real damage occured. Nice! But what if we had not been there? Or the power been out?

![box](/img/pentair/box.png)

Well, there are a few accesories you can buy for other crazy prices, like a 50eur 9V battery NI-MH backup and a ~200eur automation system using EnOcean. I decided this pricing was so insane this device must be complicated...

## Inside the belly of the beast

So this device is hilariously simple, so you can easily add a 9V NI-MH battery from your favourite vendor, i picked up this energizer for 7eur.
![battery](/img/pentair/battery.jpg)
Even better, turns out this energiser battery is all plastic like the original, just throw the plastic door trap back on and voila, your device is now working during power cuts!

What is more interesting are the two pins labelled 40 & 41 which is what the automation device [here](https://www.jung-pumpen.de/en/products/control-units/radio-transmitter?t=0#_Funktransmitter_2_1TL4_range_advantages) uses. This hilariously is just a dry relay, a leak is just registered as a short on those pins. So this means we can just use a shelly!

I actually tried two setups, initially i used an old shelly uni I had lying around, powered it using 12V and used one of the inputs. This works very well but the old shelly unis have some issues so I decided to instead buy a Shelly i4 DC which is really a nice device. I also printed a small case for it from [this](https://www.printables.com/model/612514-shelly-case-v2-addon) design.
![finished](/img/pentair/final.jpg).

Essentially this is it:
       +-------------------+
       |   Shelly          |
       |                   |
       |   [ GND ]---------+----------[40] Pentair AGR Alarm
       |                   |
       |   [ SW1 ]---------+----------[41] Pentair AGR Alarm
       |                   |
       +-------------------+

Anything that can monitor a button press will manage this, a shelly 1 would also have been more than practical enough, I just wanted the possibility to monitor more inputs for other usecases and use the addon capability.

And voila, now when a leak is detected, it will trigger the button press, and you can automate off of this.

## Further work

I went a little further and got a sensor XKC-Y25 sensor to check the level of the water outside the plastic tank where the pump is housed, it's really easy to hook this up using a shelly addon detailed [here](https://kb.shelly.cloud/knowledge-base/using-shelly-plus-add-on-with-a-capacitive-liquid-).

Shelly also recently started with the [Flood 4 device](https://www.shelly.com/products/shelly-flood-gen4) that would probably be worth adding as additional protection and seems reasonbly priced, I just wish there was a 12V DC version.
