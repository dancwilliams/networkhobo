---
title: "Cisco IOS SSL VPN with AD/RADIUS Authentication"
date: 2014-12-06
draft: false
url: "2014/12/06/cisco-ios-ssl-vpn-with-adradius-authentication"
tags: [ "remote access", "Cisco", "routing", "security" ]
categories: [ "remote access", "routing", "security" ]
---

### The Request:

Now that Cisco has included SSL VPN licensing as part of the 15.3(3)M IOS I have had multiple clients ask about turning on the capability and reaching back into Active Directory for authentication.

<!--more-->

## The Solution:

The equipment I used to lab this solution:

*   Cisco 881 w/ IOS 15.3(3)M3 (10.0.1.238)
*   Windows Server 2008 R2 (10.0.1.231)

First we will go through the steps to configure the RADIUS server on Windows so we have access to Active Directory for authentication. You must first ensure the “Network Policy and Access Services” role is installed on the server. Once this role is installed we will go into NPS (Local) > RADIUS Clients and Servers > RADIUS Clients. Here will will configure our router as a RADIUS Client. Be sure to make note of the key you specify here as you will need it when configuring the RADIUS server on the router.

[![Add RADIUS Client 1](/img/add-radius-client-1.png)](/img/add-radius-client-1.png)

[![RADIUS Client Config 1](/img/radius-client-config-1.png)](/img/radius-client-config-1.png)

[![RADIUS Client Config 2](/img/radius-client-config-2.png)](/img/radius-client-config-2.png)

Once our RADIUS client is configured we will move on to configuring the Network Policies in NPS (Local) > Policies > Network Policies and clicking NEW under Actions.

[![RADIUS Policy 1](/img/radius-policy-1.png)](/img/radius-policy-1.png)

Under the Conditions Tab you will want to add a Windows Group that contains your users that are allowed VPN access and a NAS IPv4 Address to specify the requesting router.

[![RADIUS Policy 2](/img/radius-policy-2.png)](/img/radius-policy-2.png)

Under the Constraints tab you will only select Unencrypted Authentication (PAP, SPAP).

[![RADIUS Policy 3](/img/radius-policy-3.png)](/img/radius-policy-3.png)

The Settings tab can be left at default. Make sure that you move your new policy to the top of the list!

[![RADIUS Policy 4](/img/radius-policy-4.png)](/img/radius-policy-4.png)

Now that we have the Windows Server piece configured we can move on to the configuration of the router. I have included the main configuration blocks below. Be sure to bind radius requests to the interface with the IP you specified in the Windows Server configuration or else requests may fail. Depending on the environment some people choose to use a loopback address for this. 

**Note**: _The only interface I have configured on this router is the Fa4 interface with the IP 10.0.1.238 which is plugged into my lab environment. Also, when you first issue the webvpn gateway NAME command and self-signed cert and trustpoint will be configured. I have included a reference doc at the bottom that goes through the SSL VPN config in more detail._

    aaa new-model
    !
    radius server RADIUS 
    address ipv4 10.0.1.231 auth-port 1645 acct-port 1646 
    key XXXXXXXXX
    !
    aaa group server radius TEST881
     server name RADIUS
    !
    ip radius source-interface FastEthernet4 
    !
    aaa authentication login SSL_VPN group TEST881 local
    !
    webvpn gateway SSLVPN_Gateway
        ip address 10.0.1.238 port 443  
        http-redirect port 80
        ssl trustpoint TP-self-signed-4045373729
        inservice
    !
    webvpn context SSLVPN_Context
        title "Network Hobo VPN"
        login-photo file flash:/Blog_LOGO.png
        logo file flash:/Blog_LOGO.png
        login-message "Secure Access"
        aaa authentication list SSL_VPN
        gateway SSLVPN_Gateway
        !
        ssl authenticate verify all
        !
        url-list "Internal Sites"
            heading "LAB"
            url-text "CACTI" url-value "http://10.0.1.241"
            url-text "IOU-WEB" url-value "http://10.0.1.34"
        inservice
        !
        policy group SSLVPN_DefaultPolicy
            url-list "Internal Sites"
        default-group-policy SSLVPN_DefaultPolicy

Once you have your RADIUS server and additional aaa config in place you can test RADIUS authentication using the following command:

    TEST_881#test aaa group radius dwilliams Test1Test1 legacy 
    Attempting authentication test to server-group radius using radius
    User was successfully authenticated.

Next you can navigate to your SSL VPN site and attempt to log in. Everthing should be good to go if you have followed the steps above.

[![VPN LOGIN](/img/vpn-login.png)](/img/vpn-login.png)

[![VPN LOGIN 2](/img/vpn-login-2.png)](/img/vpn-login-2.png)

## Conclusion:

The ability to implement the Cisco IOS SSL VPN and tie it back into AD without any additional cost or licensing is a big thing to many of my clients. This will give many existing organizations a new capability to lock down their edge and really enhance remote access capabilities with the investment of a little time and possibly some consulting dollars. While I mainly focused on authenticating through AD/RADIUS in this article there are many other capabilities of the SSL VPN that I did not cover. Maybe in a future write up…

## THANKS!

I would like to say a quick thank you to the following references while I was working through this:

*   [Clientless SSL VPN on Cisco IOS Router - Knowledge Base](https://sites.google.com/site/amitsciscozone/home/security/clientless-ssl-vpn-on-cisco-ios-router-with-sdm)
