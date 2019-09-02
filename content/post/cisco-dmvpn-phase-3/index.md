---
title: "Cisco DMVPN Phase 3"
date: 2015-03-21
draft: false
url: "2015/03/21/cisco-dmvpn-phase-3"
tags: [ "remote access", "Cisco", "routing", "security", "vrf" ]
categories: [ "remote access", "routing", "security", "data center" ]
---

### The Request ###

I have a client with a data center, a headquarters/DR site, and a lot of branches spread out all over the world with Internet connectivity.

They are currently using static IPSEC Internet facing VPNs to connect to their data center and HQ environemts, but the company is hitting a growth spurt and they are quickly realizing this solution is becoming difficult to scale and manage with their limited in-house IT staff.

<!--more-->

The client wanted to stick with Internet based VPNs for connectivity. They also wanted a solution that would allow them to easily stand up a new remote site quickly with a template configuration and provide tunnel redundancy between the data center and HQ locations. They also wanted direct site-to-site communications when necessary.

## The Solution:

**_The examples below are from my lab. They are working in production with different crypto algorithms._** 

Since this client has an existing Cisco routing environment and plans to continue to use Cisco routers it was decided that a DMVPN setup would work best to met their needs. To provide the type of connectivity they desired a Phase 3 dual-hub setup would be the best bet.

The client also had an existing EIGRP setup that ran between the HQ and data center over an existing GRE tunnel. This GRE tunnel will be removed and replaced with the DMVPN as well. There is nothing special with the EIGRP configuration used on the routers when using this DMVPN setup besides the EIGRP aggregate statements we will place on the hub tunnel interfaces. 

First lets look at the standard crypto config that goes on all routers, both hub and spoke:

    crypto isakmp policy 1
    encr 3des
    authentication pre-share
    group 2
    crypto isakmp key t3$tk3y address 0.0.0.0        
    crypto isakmp keepalive 10
    !
    !
    crypto ipsec transform-set TRANSSET ah-sha-hmac esp-aes 256 esp-sha-hmac 
     mode transport
    !
    crypto ipsec profile VPNPROF
     set transform-set TRANSSET

For the following tunnel interface examples we need to lay out an example IP scheme so everything makes sense:

*   EIGRP AS: 1
*   Data Center
    *   WAN IP: 1.1.1.1
    *   Tunnel100: 10.100.0.1
    *   LAN 1: 192.168.10.0/24
    *   LAN 2: 192.168.11.0/24
*   HQ
    *   WAN IP: 2.2.2.2
    *   Tunnel100: 10.100.0.10
    *   LAN 1: 192.168.20.0/24
    *   LAN 2: 192.168.21.0/24
*   SPOKE 1
    *   WAN IP: 3.3.3.3
    *   Tunnel100: 10.100.0.20
    *   LAN: 192.168.41.0/24
*   SPOKE 2
    *   WAN IP: 4.4.4.4
    *   Tunnel100: 10.100.0.30
    *   LAN: 192.168.51.0/24

Lets look at the configuration for our first hub…the Data Center:

    interface Tunnel100
    ip address 10.100.0.1 255.255.255.0
    no ip redirects
    no ip split-horizon eigrp 1
    ip pim dr-priority 100
    ip pim sparse-dense-mode
    ip nhrp authentication t3$tk3y
    ip nhrp map multicast dynamic
    ip nhrp map multicast 2.2.2.2
    ip nhrp map 10.100.0.10 2.2.2.2
    ip nhrp network-id 100
    ip nhrp holdtime 360
    ip nhrp nhs 10.100.0.10
    ip nhrp shortcut
    ip nhrp redirect
    ip summary-address eigrp 1 192.168.0.0 255.255.0.0
    ip summary-address eigrp 1 192.168.10.0 255.255.255.0
    ip summary-address eigrp 1 192.168.11.0 255.255.255.0
    ip tcp adjust-mss 1416
    tunnel source Ethernet0/0
    tunnel mode gre multipoint
    tunnel key 100
    tunnel protection ipsec profile VPNPROF

For this hub config you will see a few different things.

First you will see that in addition to the standard DMVPN dynamic statements you will also see a static config that is pointing to the second hub, in this case the HQ site.

You will also see the command **_ip nhrp shortcut_** and **_ip nhrp redirect_**. These commands enable the smooth creation of spoke-to-spoke tunnels and are additions in Phase 3.

Now for the summary addresses…

First we want to cover the entire private IP space that could be used inside the organization. Since this organization uses strictly 192.168.X.X space we place the 192.168.0.0/16 aggregate on the hub tunnel. But, since this is a dual-hub design, we also need to place the site specific aggregates on the tunnel as well. Since this sites design does not allow us to cleanly roll up the subnets each /24 was added as a separate statement.

The reason behind these aggregates is because the spoke will only see routes from the hub. In DMVPN Phase 3 the EIGRP relationship only exists between the spoke and hub. When a spoke tries to route to the IP space of another spoke the hub will pass the more specific route via an NHRP message and inject it into the spoke as an H designated route. The more specifics allow the traffic to flow directly to the hub that possesses that IP space. If the more specific aggregates were not configured both hubs would only advertise the /16 aggregate and this could lead to less than optimal routing.

Now lets look at the HQ tunnel remembering that HQ is the second hub:

    interface Tunnel100
    ip address 10.100.0.10 255.255.255.0
    no ip redirects
    no ip split-horizon eigrp 1
    ip pim dr-priority 95
    ip pim sparse-dense-mode
    ip nhrp authentication t3$tk3y
    ip nhrp map multicast dynamic
    ip nhrp map multicast 1.1.1.1
    ip nhrp map 10.100.0.1 1.1.1.1
    ip nhrp network-id 100
    ip nhrp holdtime 360
    ip nhrp nhs 10.100.0.1
    ip nhrp shortcut
    ip nhrp redirect
    ip summary-address eigrp 1 192.168.0.0 255.255.0.0
    ip summary-address eigrp 1 192.168.20.0 255.255.255.0
    ip summary-address eigrp 1 192.168.21.0 255.255.255.0
    ip tcp adjust-mss 1416
    tunnel source Ethernet0/0
    tunnel mode gre multipoint
    tunnel key 100
    tunnel protection ipsec profile VPNPROF

As you can see the configuration is close to identical except for the static NHRP configuration, the EIGRP aggregates, and the PIM DR priority.

Now that we have our hubs configured we can put together the configuration for our spokes. This tunnel config will be used over and over again as new spokes are rolled out. Since the HQ and Data Center IPs remain static the only thing that needs to be changed for each spoke would be the tunnel interface IP address and the source interface. 

Remember…the spokes use the same crypto config as the hubs:

    interface Tunnel100
     ip address 10.100.0.20 255.255.255.0
     no ip redirects
     ip pim sparse-dense-mode
     ip nhrp authentication t3$tk3y
     ip nhrp map multicast 1.1.1.1
     ip nhrp map multicast 2.2.2.2
     ip nhrp map 10.100.0.1 1.1.1.1
     ip nhrp map 10.100.0.10 2.2.2.2
     ip nhrp network-id 100
     ip nhrp holdtime 360
     ip nhrp nhs 10.100.0.1
     ip nhrp nhs 10.100.0.10
     ip nhrp registration no-unique
     ip nhrp shortcut
     ip nhrp redirect
     ip tcp adjust-mss 1416
     ip ospf network point-to-multipoint
     tunnel source Ethernet0/0
     tunnel mode gre multipoint
     tunnel key 100
     tunnel protection ipsec profile VPNPROF

Pretty basic right? This config will create static DMVPN tunnels to both hubs. When you run a **_show dmvpn_** command you will see the following output on the spoke:

    SPOKE1#sh dmvpn
    Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
    N - NATed, L - Local, X - No Socket
    # Ent --> Number of NHRP entries with same NBMA peer
    NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
    UpDn Time --> Up or Down Time for a Tunnel
    ==========================================================================

    Interface: Tunnel100, IPv4 NHRP Details 
    Type:Spoke, NHRP Peers:2, 
    # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
    ----- --------------- --------------- ----- -------- -----
        1  1.1.1.1         10.100.0.1       UP   01:20:18    S
        1  2.2.2.2         10.100.0.10      UP   01:20:08    S

Great! The spoke is now up and connect to the hubs! Lets take a look at the routing table: **_Routing table truncated to show relevant pieces_**

    SPOKE1#sh ip route

     10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
    C        10.100.0.0/24 is directly connected, Tunnel100
    L        10.100.0.20/32 is directly connected, Tunnel100
    D        10.254.254.1/32  [90/27008000] via 10.100.0.1, 01:23:00, Tunnel100
    D        10.254.254.10/32 [90/27008000] via 10.100.0.10, 01:23:00, Tunnel100
    D        10.254.254.30/32 [90/28288000] via 10.100.0.10, 01:23:00, Tunnel100
    D     192.168.0.0/16 [90/27008000] via 10.100.0.10, 01:23:00, Tunnel100
                         [90/27008000] via 10.100.0.1, 01:23:00, Tunnel100
      192.168.41.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.41.0/24 is directly connected, Loopback1
    L        192.168.41.1/32 is directly connected, Loopback1
    D     192.168.10.0/24 [90/27008000] via 10.100.0.1, 01:23:00, Tunnel100
    D     192.168.11.0/24 [90/27008000] via 10.100.0.1, 01:23:00, Tunnel100
    D     192.168.20.0/24 [90/27008000] via 10.100.0.10, 01:23:00, Tunnel100
    D     192.168.21.0/24 [90/27008000] via 10.100.0.10, 01:23:09, Tunnel100

As you see we only have the aggregates in the spoke routing table. Now lets try to ping the LAN IP space on SPOKE 2\. Lets look at the **_show dmvpn_** once again:

    SPOKE1#ping 192.168.30.1 source Lo1
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.114.1, timeout is 2 seconds:
    Packet sent with a source address of 192.168.30.1 
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms
    SPOKE1#sh dmvpn                     
    Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
    N - NATed, L - Local, X - No Socket
    # Ent --> Number of NHRP entries with same NBMA peer
    NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
    UpDn Time --> Up or Down Time for a Tunnel
    ==========================================================================

    Interface: Tunnel100, IPv4 NHRP Details 
    Type:Spoke, NHRP Peers:3, 

     # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
     ----- --------------- --------------- ----- -------- -----
          1 1.1.1.1         10.100.0.1        UP 01:30:11     S
          1 2.2.2.2         10.100.0.10       UP 01:30:01     S
          2 4.4.4.4         10.100.0.30       UP 00:00:29     D
                            10.100.0.30       UP 00:00:29   DT1

Nice! The dynamic tunnel popped up without a problem. The following route was added to the routing table:

    H     192.168.30.0/24 [250/1] via 10.100.0.30, 00:04:38, Tunnel100

Just like we expected…NHRP has injected the more specific with the H designator. Now, if either of the hubs were to fail the other hub would continue to act as the NHRP server and the dynamic environment would continue to function.

## Conclusion:

DMVPN is a very useful tool in a Cisco routed environment. It can make rolling out new spokes very easy. There are also many ways to customize this environment. I see a lot of clients that will place the routers Internet interface and Internet default route into its own VRF and then have the tunnel passing routes into the global table. Then the hub will pass a default to the spoke and force all traffic through the hub. This allows for filtering and other security measures to be taken centrally.

## THANKS!

Thanks for taking the time to read this. I hope you find it helpful! Please feel free to leave comments or contact me via twitter ([@dancwilliams](https://twitter.com/dancwilliams)) if you have any questions or feedback.