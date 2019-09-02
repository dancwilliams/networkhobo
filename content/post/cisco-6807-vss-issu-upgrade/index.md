---
title: "Cisco 6807 VSS ISSU Upgrade"
date: 2014-12-05
draft: false
url: "2014/12/05/cisco-6807-vss-issu-upgrade"
tags: [ "Catalyst", "Cisco", "data center", "routing", "switching", "upgrade" ]
categories: [ "data center", "routing", "switching" ]
---

### The Request:

I have a client with multiple 6807 VSS pairs that required an IOS upgrade. All of the pairs have a single SUP2-T in each chassis and were in the 15 code train. Although the ISSU process is very straight forward I wanted to put this quick process up as I had to search through multiple documents to gather all the pieces I needed to knock it out.

<!--more-->

### The Solution:

Since these switches were in the proper code train to utilize ISSU I decided that was the best route to go. It also helps that everything was already dual-homed. This process is for VSS pairs with only one SUP per chassis! If you have another configuration you can reference the Cisco document provided at the bottom of the post. **_Some example text was taken from the Cisco Document referenced below_** One of the first things you want to verify is that there is a current boot variable configured on the VSS pair pointing to the version of code that is running currently. Some devices only have one version of code on the bootdisk so there is not a boot variable configured. For the ISSU to perform properly you MUST configure the boot variable:

    Router(config)# boot system flash bootdisk:s72033-oldversion.v1

Next you will want to download the new image from your favorite file transfer spot to your bootdisk. I prefer to use FTP:

    Router# copy ftp: bootdisk:
    Address or name of remote host []? test.ftp.local
    Source filename []? s72033-newversion.v2
    Destination filename [s72033-newversion.v2]?
    Accessing ftp://test.ftp.local/s72033-newversion.v2...!!!!! Complete

Once the image is downloaded you will want to copy the image over to the slavebootdisk:

    Router# copy bootdisk: slavebootdisk:
    Source filename []? s72033-newversion.v2
    Destination filename [s72033-newversion.v2]?
    Copy in progress...CCCCCCCCCCCCCCC Complete

After you have the images on both bootdisks you can begin the ISSU process. The first step is to verify the VSS pair is ready for the ISSU upgrade:

    Router# show issu state detail
    Slot = 1/3
    RP State = Active
    ISSU State = Init
    Boot Variable = bootdisk:s72033-oldversion.v1,12;
    Operating Mode = sso
    Primary Version = N/A
    Secondary Version = N/A
    Current Version = bootdisk:s72033-oldversion.v1
    Variable Store = PrstVbl

    Slot = 2/3
    RP State = Standby
    ISSU State = Init
    Boot Variable = bootdisk:s72033-oldversion.v1,12;
    Operating Mode = sso
    Primary Version = N/A
    Secondary Version = N/A
    Current Version = bootdisk:s72033-oldversion.v1

    Router# show redundancy states
    my state = 13 -ACTIVE
    peer state = 8 -STANDBY HOT
    Mode = Duplex
    Unit = Secondary
    Unit ID = 18

    Redundancy Mode (Operational) = sso
    Redundancy Mode (Configured) = sso
    Redundancy State = sso
    Maintenance Mode = Disabled
    Communications = Up

    client count = 132
    client_notification_TMR = 30000 milliseconds
    keep_alive TMR = 9000 milliseconds
    keep_alive count = 0
    keep_alive threshold = 18
    RF debug mask = 0x0

Once the pair is verified to be good to go you will want to load the new image onto the standby chassis. This will load the new code on the standby and reload the chassis. If you have <span style="text-decoration:underline;">**EVERYTHING DUAL HOMED**</span> you will not see an interruption in traffic.

    Router# issu loadversion bootdisk:s72033-newversion.v2

    000133: Aug 6 16:17:44.486 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet1/3/4, changed state to down
    000134: Aug 6 16:17:43.507 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet2/3/4, changed state to down
    000135: Aug 6 16:17:43.563 PST: %LINK-3-UPDOWN: Interface TenGigabitEthernet2/3/4, changed state to down
    000136: Aug 6 16:17:44.919 PST: %LINK-3-UPDOWN: Interface TenGigabitEthernet1/3/4, changed state to down

    (Deleted many interface and protocol down messages)

    %issu loadversion executed successfully, Standby is being reloaded

    (Deleted many interface and protocol down messages, then interface and protocol up messages)

    0000148: Aug 6 16:27:54.154 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet1/3/5, changed state to up
    000149: Aug 6 16:27:54.174 PST: %LINK-3-UPDOWN: Interface TenGigabitEthernet2/3/5, changed state to up
    000150: Aug 6 16:27:54.186 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet2/3/5, changed state to up
    000151: Aug 6 16:32:58.030 PST: %HA_CONFIG_SYNC-6-BULK_CFGSYNC_SUCCEED: Bulk Sync succeeded

During the process you will want to run the following command until you see that the **ISSU** <span style="text-decoration:underline;">**Sub-State**</span> is **Load Version Complete**:

    Router# show issu state detail
    Slot = 1/3
    RP State = Active
    ISSU State = Load Version
    Boot Variable = bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = bootdisk:s72033-oldversion.v1
    Secondary Version = bootdisk:s72033-newversion.v2
    Current Version = bootdisk:s72033-oldversion.v1
    Variable Store = PrstVbl

    Slot = 2/3
    RP State = Standby
    ISSU State = Load Version
    Boot Variable = bootdisk:s72033-newversion.v2,12;bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = bootdisk:s72033-oldversion.v1
    Secondary Version = bootdisk:s72033-newversion.v2
    Current Version = bootdisk:s72033-newversion.v2

    Router# show redundancy status
    my state = 13 -ACTIVE
    peer state = 8 -STANDBY HOT
    Mode = Duplex
    Unit = Secondary
    Unit ID = 18

    Redundancy Mode (Operational) = sso
    Redundancy Mode (Configured) = sso
    Redundancy State = sso
    Maintenance Mode = Disabled
    Communications = Up

    client count = 132
    client_notification_TMR = 30000 milliseconds
    keep_alive TMR = 9000 milliseconds
    keep_alive count = 1
    keep_alive threshold = 18
    RF debug mask = 0x0

Once this state is reached you will want to go into the next step to force a switchover to the standby chassis that is running the new code and being upgrading the remaining chassis. Once you issue the runversion command it will start a rollback timer that is by default set to 45 minutes. **If you do not commit the version (next step) before the timer runs out the upgrade will be rolled back!**

    Router# issu runversion
    This command will reload the Active unit. Proceed ? [confirm]
    (Deleted many lines)

    Download Start
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    (Deleted many lines)
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    Download Completed! Booting the image.
    Self decompressing the image : ##########################################################################################
    (Deleted many lines)

###### ##########################################################################

    running startup....

    (Deleted many lines)

    000147: Aug 6 16:53:43.199 PST: %HA_CONFIG_SYNC-6-BULK_CFGSYNC_SUCCEED: Bulk Sync succeeded

Once the chassis has rebooted we will once again want to verify the ISSU state and redundancy state:

    Router# show issu state detail
    Slot = 2/3
    RP State = Active
    ISSU State = Run Version
    Boot Variable = bootdisk:s72033-newversion.v2,12;bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = bootdisk:s72033-newversion.v2
    Secondary Version = bootdisk:s72033-oldversion.v1
    Current Version = bootdisk:s72033-newversion.v2
    Variable Store = PrstVbl

    Slot = 1/3
    RP State = Standby
    ISSU State = Run Version
    Boot Variable = bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = bootdisk:s72033-newversion.v2
    Secondary Version = bootdisk:s72033-oldversion.v1
    Current Version = bootdisk:s72033-oldversion.v1

    Router# show redundancy status
    my state = 13 -ACTIVE
    peer state = 8 -STANDBY HOT
    Mode = Duplex
    Unit = Primary
    Unit ID = 39

    Redundancy Mode (Operational) = sso
    Redundancy Mode (Configured) = sso
    Redundancy State = sso
    Maintenance Mode = Disabled
    Communications = Up

    client count = 134
    client_notification_TMR = 30000 milliseconds
    keep_alive TMR = 9000 milliseconds
    keep_alive count = 1
    keep_alive threshold = 18
    RF debug mask = 0x0

You will now want to commit the new version to reload the standby chassis and have it run the new image:

    Router# issu commitversion
    Building configuration...
    [OK]
    000148: Aug 6 17:17:28.267 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet2/3/4, changed state to down
    000149: Aug 6 17:17:28.287 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet1/3/4, changed state to down

    (Deleted many interface and protocol down messages)

    %issu commitversion executed successfully

    (Deleted many interface and protocol down messages, then interface and protocol up messages)

    000181: Aug 6 17:41:51.086 PST: %LINEPROTO-5-UPDOWN: Line protocol on Interface TenGigabitEthernet1/3/5, changed state to up
    000182: Aug 6 17:42:52.290 PST: %HA_CONFIG_SYNC-6-BULK_CFGSYNC_SUCCEED: Bulk Sync succeeded

Once this has completed your entire VSS pair will be upgraded. You can verify the upgrade by once again checking the ISSU & redundancy state:

    Router# show issu state detail
    Slot = 2/3
    RP State = Active
    ISSU State = Init
    Boot Variable = bootdisk:s72033-newversion.v2,12;bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = N/A
    Secondary Version = N/A
    Current Version = bootdisk:s72033-newversion.v2
    Variable Store = PrstVbl

    Slot = 1/3
    RP State = Standby
    ISSU State = Init
    Boot Variable = bootdisk:s72033-newversion.v2,12;bootdisk:s72033-oldversion.v1,12
    Operating Mode = sso
    Primary Version = N/A
    Secondary Version = N/A
    Current Version = bootdisk:s72033-newversion.v2

    Router# show redundancy status
    my state = 13 -ACTIVE
    peer state = 8 -STANDBY HOT
    Mode = Duplex
    Unit = Primary
    Unit ID = 39

    Redundancy Mode (Operational) = sso
    Redundancy Mode (Configured) = sso
    Redundancy State = sso
    Maintenance Mode = Disabled
    Communications = Up

    client count = 134
    client_notification_TMR = 30000 milliseconds
    keep_alive TMR = 9000 milliseconds
    keep_alive count = 1
    keep_alive threshold = 18
    RF debug mask = 0x0

### Conclusion:

Once I found all of the information I needed this was a very easy process and actually did not take as long as I expected. The boot variable was one issue I ran into but once I figured out what it was asking for that was easy to fix. Another thing to be mindful of is that after the upgrade process your original active processor will be the standby processor.

### THANKS!

I would like to say a quick thank you to the following references while I was working through this:

*   [Cisco Release 15.1SY Supervisor Engine 2T Software Configuration Guide](http://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/15-1SY/config_guide/sup2T/15_1_sy_swcg_2T/virtual_switching_systems.html#14718)