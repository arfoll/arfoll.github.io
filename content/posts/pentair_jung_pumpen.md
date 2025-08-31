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

# Hacking the Pentair Jung Pumpen AGR device

My house came with a strange little device called a **Pentair Jung Pumpen AGR**. Honestly, it might be the most overpriced gadget I’ve ever seen — essentially a passthrough plug with a small leak sensor that triggers a loud alarm.  

There’s no configuration, no smart features, and it’s only IP20 rated. Yet somehow, this thing sells for over **€300(!)**.  

I’d never paid much attention to it — until recently, when a small leak (thankfully no real damage) set it off. The alarm woke us up in time to clean up before anything serious happened. Nice!  

But then I wondered: what if we hadn’t been home? Or if the power had been out?  

![box](/img/pentair/box.png)  

---

## Overpriced Accessories  

Of course, there are official accessories — also hilariously overpriced.  
- A **€50 9V Ni-MH backup battery**  
- A **~€200 EnOcean automation module**  

At that point I was convinced this thing must be hiding some complicated tech inside.  

---

## Inside the Belly of the Beast  

When I cracked it open, the reality was… hilariously simple.  

It turns out you can just drop in a regular 9V Ni-MH battery. I picked up an Energizer for €7, slotted it in, closed the plastic cover, and voilà — now the AGR runs during power cuts.  

![battery](/img/pentair/battery.jpg)  

Even better, it turns out those mysterious terminals labeled **40 & 41** are just a **dry contact relay**. When a leak is detected, the pins are simply shorted together.  

Which means… perfect for a Shelly!  

---

## Adding Smart Monitoring  

My finished setup looks something like this:

![finished](/img/pentair/final.jpg)  

### Wiring  

Here’s all it takes to connect the Pentair AGR to a Shelly i4 DC:  

```text
       +-------------------+
       |   Shelly i4 DC    |
       |                   |
       |   [ GND ]---------+----------[40] Pentair AGR Alarm
       |                   |
       |   [ SW1 ]---------+----------[41] Pentair AGR Alarm
       |                   |
       +-------------------+
```

That’s it: when water is detected, the AGR shorts 40 & 41, and the Shelly sees it as a button press on SW1.  

Realistically, anything that can detect a button press would work here — a **Shelly 1** would be more than enough. I just went with the i4 for the extra inputs and add-on flexibility. The Pentair device provides gromets and cable relief so wiring is pretty easy, I used cat6 flexible ethernet wire because that's what I had around.

---

## Further Work  

I also added an **XKC-Y25 capacitive water level sensor** to monitor water outside the pump’s plastic tank. Hooking it up was easy using a Shelly add-on (details [here](https://kb.shelly.cloud/knowledge-base/using-shelly-plus-add-on-with-a-capacitive-liquid-)).  

Shelly also recently started with the [Flood 4 device](https://www.shelly.com/products/shelly-flood-gen4) that would probably be worth adding as additional protection and seems reasonably priced — I just wish there was a 12V DC version.  

