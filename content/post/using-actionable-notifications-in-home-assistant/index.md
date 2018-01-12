---
title: "Using Actionable Notifications in Home Assistant"
date: 2018-01-11T19:02:48-06:00
draft: true
tags: [ "home assistant", "iOS", "notifications" ]
categories: [ "home assistant" ]
---

In this post I will cover how I use [actionable notifications](https://home-assistant.io/docs/ecosystem/ios/notifications/actions/) within [Home Assistant](https://home-assistant.io).  Using [push notifications](https://home-assistant.io/docs/ecosystem/ios/notifications/) with the [Home Assistant iOS App](https://home-assistant.io/docs/ecosystem/ios/) you can setup some really cool triggers within the system.

<!--more-->

I have two actionable notifications that get used most often:

1. At 6AM I receive a push notification that asks if I am working from home today.
   * Yes and No options are presented
   * If I answer yes, it will turn on the AC in my office and set the proper temperature based on the temperature outside.

2. On days when my wife and I go into our "_real_" offices we will receive a notification when we leave in the evening and the house is set to away.  This notification asks if you are headed home.
   * Yes and No options are presented
   * If yes is selected, it will set the air conditioner to the home setting to cool/heat the house in advance.

**NOTE!**

When you receive a notification **do not just tap on it!**  This will open the Home Assistant app and not present your options!

If you receive the notification while your phone is locked:

{{< imgproc notification_while_locked Resize "300x" >}}