---
title: "Controlling Velux blinds using the KIG 300"
date: 2022-09-18T23:08:11+02:00
tags: [iot, homeassistant]
categories: [iot]
---
If you have some Velux blinds (SSL, MSL) or windows which use the velux ACTIVE branding you likely can hook them up to control them remotely. In order to do this you need to buy the velux gateway KIG300 which is often sold as a kit with some temp/humidity sensor which I didn't get.

The annoying thing with this setup is that it's using the apple homekit integration as well as requiring a smartphone app for the initial setup, pairing this with a network that has no internet is also a little tricky, so this is by far not my favourite solution. But there seemed to be not much information on these devices and how you can use them so I figured I would jot down my notes. I'm not too unhappy about it but for ~90eur I think it's quite expensive for what it does.

## My setup

I have two Velux MSL blinds with KLI300 controllers. The gateway pairs with the controllers via the [VELUX ACTIVE with NETATMO application](https://play.google.com/store/apps/details?id=com.velux.active&hl=en&gl=UShttps://play.google.com/store/apps/details?id=com.velux.active&hl=en&gl=US). Essentially you press a magic pair button on the remote and then add the blind to be controlled.

## Home assistant

Home assistant can use the [homekit integration](https://www.home-assistant.io/integrations/homekit/) and will report multiple covers. Currently it seems the position indicator is not working but it works in the VELUX app so I assume if I kicked it a little more I could make it work.

### Alternatives

With a shelly UNI someone made a really nice way to remote control the controller, I leave the post [here](https://www.reddit.com/r/homeassistant/comments/pzhkia/hack_velux_kli3xx_to_use_blinds_without_gateway/). I chose not to do this as the setup looked a little messy and would need quite some work to clean up but also there is then no way to get state/position information on the blind.

Also other Velux gateways like the older KLF200 are likely simpler and nicer to use but they are more expensive, ~150EUR at the time of writing.
