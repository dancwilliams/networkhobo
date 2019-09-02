---
title: "Cisco Translation Rules, CDRs, and Unity Connection"
date: 2013-12-19
draft: false
url: "2013/12/19/cisco-translation-rules-cdrs-and-unity-connection"
tags: [ "callmanager", "cisco", "Cisco Unified Communications Manager", "troubleshooting", "Unity Connection", "voice", "voip" ]
categories: [ "voice" ]
---

### The Request:

I had a new client contact me requesting that I investigate a call that came in over the weekend where an external customer was able to leave a voicemail on an internal users extension, using the corporate IVR, instead of being routed to the weekend service. This behavior was unacceptable and caused much concern throughout the leadership. The client provided me the internal extension where the voicemail was left and a timeframe. Since I was relatively new to the system I was excited to get my hands dirty and really investigate the routing. As all voice engineers know, coming into a system someone else built can always be an adventure…

<!--more-->

### The Investigation:

The system in question is a Cisco CUCM[^1] and CUC[^2] 9.0 with SCCP[^3] trunking. Since this client is closed on the weekends I thought the best place to start would be by looking at the weekend CDRs[^4] from the time frame provided to see if anything stood out. I pulled the records and noticed that some calls were being translated directly to a certain extension although the client only had a small set of DIDs mapped to internal extensions. I stepped into the translation patterns and found that there were many DIDs being translated to the single extension I had noticed in the CDRs. I found this extension was a CTI[^5] Route Point and was forwarded to voicemail. Upon accessing the voicemail system I first went into forwarded routing rules and located the CTI Route Point extension I had collected from the CDRs. It was forwarded to a company standard IVR call handler with no schedule! This definitely looked like our culprit. I compiled a list of the DIDs using this extension and forwarded the list to my client. The client replied saying these extensions were from their pre-Cisco “legacy” days and were used for all hour access to the system but were not published to the public. Apparently someone on the outside of the organization had found or been giving one of these numbers. The question was…which one?

### Translation Patterns, CDR Analysis, and The Solution:

To cut to the chase…translation patterns do not show up in CDR records. It is sad but true. You will see the Calling No. field as the external caller’s phone number and the Called No. as the called number AFTER translation. So, since all of these legacy DIDs were translating to a single internal extension I was unable to say with certainty exactly which DID was being used. To solve this problem I changed the translation patterns for these DIDs into CTI Route Points. CTI Route Points do produce CDRs unlike the translation patterns. I then forwarded these CTI Route Patterns to voicemail. I created a new [voicemail profile](http://www.cisco.com/en/US/docs/voice_ip_comm/cucm/admin/7_1_2/ccmcfg/b05vmprf.html) for masking these numbers with some arbitrary mask (99999XXXXXXXXXX for example) so I can setup a forwarded routing rule within Unity Connection to catch this mask and hand the call off to the call handler. This way we will see the DID used in all of the CDRs from now on. The solutions was tested and the CDRs were checked and everything looks great! Has anyone else done this a different way or recommend a different approach?

### THANKS!

First I would like to thank [David](http://protocol41.wordpress.com/about/) over at [protocol41](http://protocol41.wordpress.com/2013/04/23/call-routing-rules/). His entry had a quick blurb concerning Unity Connection order of operations that I needed to verify to make sure I was thinking straight! A few other links I used were [DID configuration from Cisco](http://www.cisco.com/en/US/tech/tk652/tk653/technologies_tech_note09186a00801c43f6.shtml). I was just investigating the Cisco “best practice” for DID configuration. I also used the [Unity Connection Administration Guide](http://www.cisco.com/en/US/docs/voice_ip_comm/connection/9x/administration/guide/9xcucsag090.pdf) which is always a ton of info.


[^1]: Cisco Unified Communications System [ ↩](1 "return to article")
[^2]: Cisco Unity Connection [ ↩](2 "return to article")
[^3]: Skinny Call Control Protocol [ ↩](3 "return to article")
[^4]: Call Detail Records [ ↩](4 "return to article")
[^5]: Computer Telephony Integration [ ↩](5 "return to article")