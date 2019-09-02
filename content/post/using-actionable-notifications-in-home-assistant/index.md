---
title: "Using Actionable Notifications in Home Assistant"
date: 2018-01-11T19:02:48-06:00
draft: false
tags: [ "home assistant", "iOS", "notifications" ]
categories: [ "home assistant" ]
---

In this post I will cover how I use [actionable notifications](https://home-assistant.io/docs/ecosystem/ios/notifications/actions/) within [Home Assistant](https://home-assistant.io).  Using [push notifications](https://home-assistant.io/docs/ecosystem/ios/notifications/) with the [Home Assistant iOS App](https://home-assistant.io/docs/ecosystem/ios/) you can setup some really cool triggers within the system.

<!--more-->

_Below is a list of the technology used at the time of writing:_

* [_Home Assistant_](https://home-assistant.io) _- Version 0.60.1_
* [_Apple iPhone X and iPhone 7_](https://www.apple.com/iphone/)
* [_Apple iOS 11.2.2_](https://support.apple.com/en-us/HT208401)

I have two actionable notifications that get used most often:

1. At 6AM I receive a push notification that asks if I am working from home today.
   * `Yes` and `No` options are presented
   * If I answer `Yes`, it will turn on the AC in my office and set the proper temperature based on the temperature outside.

2. On days when my wife and I go into our "_real_" offices we will receive a notification when we leave in the evening and the house is set to away.  This notification asks if you are headed home.
   * `Yes` and `No` options are presented
   * If `Yes` is selected, it will set the air conditioner to the home setting to cool/heat the house in advance.

We will use my "Are you working from home today?" notification as the example[^1].

#### Step One:

I have an automation configured to send me a message asking if I am working from home on `weekdays` at `6AM` if the house is in `sleep` mode:

```yaml
- alias: Ask if Dan is working from home in the morning - Weekday
  id: ask_if_dan_is_working_from_home_in_morning_weekday
  condition:
    condition: and
    conditions:
      - condition: state
        entity_id: input_select.home_state
        state: sleep
      - condition: time
        weekday:
          - mon
          - tue
          - wed
          - thu
          - fri
  trigger:
    platform: time
    at: '06:00:00'
  action:
    service: notify.ios_dan_iphone
    data:
      message: "Are you working from home today?"
      data:
        push:
          category: "working_from_home"
```

The thing to key on here is the `push category` used in the action.  This will be used later...

#### Step Two:

Now we need to configure the iOS push category that corresponds with the category used in the automation.

I keep my push categories in a separate file named `ios_push_categories.yaml`.  Here is the excerpt from my `configuration.yaml`:

```yaml
ios:
  push:
    categories: !include ios_push_categories.yaml
```

And here is the appropriate section fomr `ios_push_categories.yaml`:

**NOTE: The `identifier` must be lower-case!**

```yaml
- name: Working From Home
  identifier: 'working_from_home'
  actions:
    - identifier: 'WORKING_FROM_HOME_YES'
      title: 'Yes'
      activationMode: 'background'
      authenticationRequired: no
      destructive: yes
    - identifier: 'WORKING_FROM_HOME_NO'
      title: 'No'
      activationMode: 'background'
      authenticationRequired: no
      destructive: no
```

This will provide the options that are presented with the notification and tie those options with an `identifier`.  This will be used later...

#### Step Three:

This is where the rubber meets the road!

In this case the `No` option does nothing.  The notification will close on the phone and that is all.  But, 'Yes' will trigger an automation in Home Assistant.

Here is the automation that is triggered when `Yes` is selected:

```yaml
- alias: iOS Action - Working From Home Yes
  id: ios_action_working_from_home_yes
  trigger:
    platform: event
    event_type: ios.notification_action_fired
    event_data:
      actionName: WORKING_FROM_HOME_YES
  action:
    - service: input_select.select_option
      data:
        entity_id: input_select.office_ac_power
        option: 'on'
    - service: input_select.select_option
      data_template:
        entity_id: input_select.office_ac_mode
        option: >
          {% if (states("sensor.dark_sky_temperature") | int) < 65 %}
          heat
          {% else %}
          cool
          {% endif %}
    - service: input_number.set_value
      data:
        entity_id: input_number.office_ac_temperature
        value: 72
```

Under the trigger `event_data` you will see that the `actionName` corresponds to the `identifier` used in Step Two.  When `Yes` was selected it kicked off the automation to turn on the AC in my office and set the mode based on the outside temperature.  It has worked great!

## IMPORTANT NOTES!

### Creating and Updating iOS Push Notifications:

Each time you create or update an iOS push notification within home assistant you must update oush settings within the iOS app. 

To do this go into the settings (gear icon) within the iOS app and navigate to Notification Settings > Update push notifications.

### Interacting with Push Notifications:

When you receive a notification on you iPhone, **do not just tap on it!**  <mark>This will open the Home Assistant app and not present your options!</mark>

You can, however, tap notifications on your Apple Watch.  See explanations below:

#### On iPhone:

* **While phone is locked**:

    * Your received notification will look like this:

        {{< img src="images/notification_while_locked*" >}}

    * Swipe left on the notification and tap "View":

        {{< img src="images/notification_swipe_left*" >}}

    * You will then be presented with your options:

        {{< img src="images/locked_notification_options*" >}}

* **While phone is unlocked**:

    * Your received notification will look like this:

        {{< img src="images/notification_while_using*" >}}

    * Swipe down on the notification and you will be presented with your options:

        {{< img src="images/unlocked_notification_options*" >}}

#### On Apple Watch:

* Your received notification will look like this:

  {{< img src="images/watch_notification*" >}}

* Tap the notification to receive your options:

  {{< img src="images/watch_tap*" >}}

### Summary

This is covered often in the [Home Assistant Community](https://community.home-assistant.io) and I just wanted to get my configuration out in hopes that others will find it useful.  If you have any questions feel free to leave a comment.

Thanks for reading!

[^1]: _You can find all of my configuration on [GitHub](https://github.com/dancwilliams/networkhobo) if you are curious about other actionable notification automations._
