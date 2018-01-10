---
title: "Intel NUC Memory and BIOS Gotcha"
date: 2018-01-10T09:33:42-06:00
draft: false
tags: [ "intel", "NUC", "BIOS", "memory" ]
categories: [ "intel", "NUC" ]
---

This is a quick post to cover a gotcha I ran into while preparing my *new to me* [Intel NUC DC3217IYE](https://ark.intel.com/products/71275/Intel-NUC-Kit-DC3217IYE) to take over has my primary [Home Assistant](https://home-assistant.io) box.

<!--more-->

The NUC I purchased had 4GB RAM, 64GB SSD, and wireless.  Before I began loading anyhting I wanted to make sure the NUC was updated with the latest and greatest BIOS version since I had heard of issues with previous versions.

When I logged into the machine I found it was running version 42.  The [Intel site](https://downloadcenter.intel.com/download/26602/NUCs-BIOS-Update-GKPPT10H-86A-?product=71275) showed that the latest version for this platform is 62.  So, I followed the instructions to perform a [Power Button Menu Update](https://www.intel.com/content/www/us/en/support/articles/000006030/mini-pcs.html).  This method looked to be very straightforward, but then I hit an issue.

When I selected `F7` to load the new BIOS from the USB stick I went through the image selection and hit `ENTER` to install, the screen went black, then the power button flashed twice and the NUC rebooted.  I hit `F2` to bring up the BIOS menu and noticed it was still running version 42.  No change...

I reloaded the USB, tried other methods, everything!  But, the same outcome occured each time.

I started researching and finally found some random forum entry somewhere (no link) that mentioned to check the RAM speed.  I check the RAM that was installed in the NUC and it was 2 x 2GB 1066MHz sticks.  Then I started digging up the version 62 [release notes](https://downloadmirror.intel.com/26602/eng/GK_0062_ReleaseNotes.pdf).  

There it was, 1066 MHz RAM was not supported by BIOS version 62:

**Note: The memory reference code in BIOS version 0046 was updated as a part of the changes made in the BIOS to meet Microsoft Windows 8.1 requirements. This new memory code no longer supports 1066 MHz memory modules.**

So, I went to the shop and found an old 4GB 1600MHz stick that I had and swapped out the RAM in the NUC.  I tried the Power Button Menu method again and it worked without issue!  Now I am up and running on the latest BIOS.

Hope this helps someone else that may run into this odd issue when the NUC provides very little feedback.

