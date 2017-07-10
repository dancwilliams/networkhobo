---
title: "Configure TACACS+ Access on Nexus 7K"
date: 2014-01-23
draft: false
url: "2014/01/23/cconfigure-tacacs-access-on-nexus-7k"
tags: [ "7K", "data center", "Nexus", "security", "TACACS+" ]
---

### The Request:

Two new Nexus 7Ks have been installed at one of my client’s data centers. Management connectivity was brought up to the data center core and verified. I was given console access and told to configure TACACS+[^1] authentication and authorization on the F2 VDC[^2].

<!--more-->

### The Solution:

Configuring TACACS+ on the Nexus 7K is totally different than on IOS and even different than on the Nexus 5K equipment. It also requires a certain order of operations and there is one solid “gotcha” that most people run into. But, knowing these going in will make this a painless procedure. The first thing to remember is that you MUST enter the TACACS+ server key UNENCRYPTED. Most templates within many organizations I work with keep the TACACS+ key in its encrypted format within template documents. Entering it into a Nexus 7K in this format WILL NOT WORK. Been there…done that… First you will need to make sure the TACACS+ feature in enabled on the NEXUS 7K by entering the following command:

    config
    feature tacacs+

Now you will need to decide how to configure your TACACS+ server keys. You can either configure a global key for all servers or on a per-server basis: Global Key:

    tacacs-server key 0 TESTKEY

Per-Server Key:

    tacacs-server host X.X.X.X key 0 TESTKEY

Now you will need to list all of your TACACS+ hosts. Previously I showed you how to enter a host with a per-server key. If you use a global key you will use this command:

    tacacs-server host X.X.X.X

Now we need to configure a TACACS+ group to use for authentication, authorization, accounting, etc. Here is an example:

    aaa group server tacacs+ TESTNAME
        server X.X.X.X
        server X.X.X.X
        server X.X.X.X
        use-vrf VRFNAME

The servers you enter into the group must first be defined as tacacs-server hosts as shown in the previous configuration . If you know this fact going in it is a huge time saver! It is also recommended that you configure the VRF that you would like to use for TACACS+ access. If you have no VRFs configured just use the following code to use the default VRF:

    use-vrf default

Now you want to tell the Nexus 7K where to source the request from. For example if you were to use VLAN 2 for the TACACS+ source interface you would use the following code:

    ip tacacs source-interface vlan 2

Some organizations also like to use directed requests to allow certain groups point their logins toward certain authentication servers outside of the standard group configuration. The command that allows this to happen is:

    tacacs-server directed-request

After all of this has been configured you are ready to add your authentication strings and test. I always recommend ensuring authentication works before configuring anything further, especially authorization as it can definitely slow down the process. The aaa string you need to enter is as follows:

    aaa authentication login default group TESTNAME

Now you can test using the following command:

    test aaa group TESTNAME username password

This will allow you to verify TACACS+ is working properly. Once this is confirmed you can move on to the authorization and accounting configuration:

    aaa authentication login console group TESTNAME
    aaa authorization commands default group TESTNAME
    aaa accounting default group TESTNAME
    aaa authentication login error-enable

I have included the full config below. If commands are entered in this order you will be good to go!

    config
    feature tacacs+
    tacacs-server key 0 TESTKEY
    tacacs-server host X.X.X.X
    tacacs-server host X.X.X.X
    tacacs-server host X.X.X.X
    aaa group server tacacs+ TESTNAME
        server X.X.X.X
        server X.X.X.X
        server X.X.X.X
        use-vrf VRFNAME
    ip tacacs source-interface vlan 2
    tacacs-server directed-request
    aaa authentication login default group TESTNAME
    aaa authentication login console group TESTNAME
    aaa authorization commands default group TESTNAME
    aaa accounting default group TESTNAME
    aaa authentication login error-enable

### Conclusion:

I had a selfish motive for writing this post…I was tired of join through it over and over again. If the order of operations is followed properly, and the gotchas are avoided, this can be a fun and painless procedure. I hope this helps everyone and if you have any questions or improvements just let me know!

### THANKS!

I would like to say a quick thank you to the following references while I was working through this:

*   Josh O’Brien ([@joshobrien77](https://twitter.com/joshobrien77)) over at staticnat.com! You post on Nexus 7000 TACACS+ helped a TON. You can read it [here](http://www.staticnat.com/2010/11/07/tacacs-on-nexus-7000/).
*   [Cisco Nexus 7K Security Design Guide](http://www.cisco.com/en/US/docs/switches/datacenter/sw/6_x/nx-os/security/configuration/guide/b_Cisco_Nexus_7000_NX-OS_Security_Configuration_Guide__Release_6.x_chapter_0110.html)
*   [Cisco Nexus 7K TACACS+ Example](https://supportforums.cisco.com/docs/DOC-16435)


[^1]: Terminal Access Controller Access-Control System Plus
[^2]: Virtual Device Context