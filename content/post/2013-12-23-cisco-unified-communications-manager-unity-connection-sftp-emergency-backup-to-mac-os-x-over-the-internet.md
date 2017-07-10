---
title: "Cisco Unified Communications Manager & Unity Connection SFTP Emergency Backup to Mac OS X over the Internet"
date: 2013-12-23
draft: false
url: "2013/12/23/cisco-unified-communications-manager-unity-connection-sftp-emergency-backup-to-mac-os-x-over-the-internet"
tags: [ "callmanager", "cisco", "Cisco Unified Communications Manager", "backups", "Unity Connection", "voice", "voip" ]
categories: [ "voice" ]
---

### The Request:

I was engaged by a long time client who was having an issue with their local SFTP[^1] server. After some upgrades to their server infrastructure they noticed their phone/voicemail system had not been backed up in MONTHS! They asked if there was a way for me to perform an “emergency” backup to my system if they gave the voice VLAN access to the Internet temporarily while they fixed their SFTP issues.

<!--more-->

### The Solution:

It just so happened that when the client called me I was out of the office and working from the family home for the holidays. Here is the equipment I had available to me:

*   Apple Airport Express MC414LL/A (Internet Gateway)
*   MacBook Pro w/ OS X Mavericks 10.9.1

The Cisco phone system consists of the following components:

*   Cisco CUCM[^2] 9.0
*   CUC[^3] 9.0

This process is the same on both the CUCM and CUC.  There are additional services to backup under CUC but the backup system is identical. The first thing I needed to do was access the phone system and verify Internet connectivity. I used RDP[^4] to access the client’s management server. Once logged in I accessed the phone system to verify Internet connectivity. This is done by logging into Cisco Unified OS Administration web page then going to Services > Ping and trying to ping an outside address (4.2.2.2 in this example):

[![Screenshot 2013-12-23 08.47.19](/img/screenshot-2013-12-23-08-47-19.png?w=300)](/img/screenshot-2013-12-23-08-47-19.png)

Once Internet connectivity from the phone system was verified I moved on to configuring my local system to accept the transfer. First I configured NAT on the Apple Airport Express to pass SSH (port 22) from the Internet to my laptop. Below are the screenshots: First we access the Airport Utility and click on the Internet access router.  Make note of the IP Address as you will need it later:

[![Screenshot 2013-12-23 08.44.04](/img/screenshot-2013-12-23-08-44-04.png?w=300)](/img/screenshot-2013-12-23-08-44-04.png)

Next we click "Edit" and go to the "Network" tab.  Then click the "+" below the "Port Settings" field:

[![Screenshot 2013-12-23 08.44.35](/img/screenshot-2013-12-23-08-44-35.png?w=300)](/img/screenshot-2013-12-23-08-44-35.png)

Next use the drop down to select the "Remote Login - SSH" service and point it to the IP of your laptop.  Click "Save" and then apply the configuration to the Airport Express:

[![Screenshot 2013-12-23 08.44.53](/img/screenshot-2013-12-23-08-44-53.png?w=300)](/img/screenshot-2013-12-23-08-44-53.png)

Once this was configured I moved on to configuring my laptop to accept SSH connections.  This process is pretty straight forward and I have included screenshots below:

[![Screenshot 2013-12-23 08.41.10](/img/screenshot-2013-12-23-08-41-10.png?w=300)](/img/screenshot-2013-12-23-08-41-10.png)

[![Screenshot 2013-12-23 08.41.39](/img/screenshot-2013-12-23-08-41-39.png?w=300)](/img/screenshot-2013-12-23-08-41-39.png)

Once SSH was configured on the laptop it was time to configure a backup device on the phone system and perform a manual backup. Below are the screenshots: First I accessed the Disaster Recovery System > Backup > Backup Device and configured a new device.  You would replace the X.X.X.X with you Internet/Public IP address that you got off of your Airport Express.  Point the path to where you would like the backup files to be saved (I created a folder on my desktop to collect the files).  Then use your laptop username and password.  When you save the backup device it will test connectivity before declaring a successful save.

[![Screenshot 2013-12-23 08.49.13](/img/screenshot-2013-12-23-08-49-13.png?w=300)](/img/screenshot-2013-12-23-08-49-13.png)

Next you will go into Backup > Manual and start a manual backup of all available services.  You will select the services by checking the box beside each one.  In Unity Connection you may receive popups concerning dependencies.  This will be ok since you will be selecting all services to back up.

[![Screenshot 2013-12-23 09.33.11](/img/screenshot-2013-12-23-09-33-11.png?w=300)](/img/screenshot-2013-12-23-09-33-11.png)

Once I verified the backups (Backup > History) were successful I loaded them to Dropbox and shared them out to my client. Too easy!

[![Screenshot 2013-12-23 09.33.32](/img/screenshot-2013-12-23-09-33-32.png?w=300)](/img/screenshot-2013-12-23-09-33-32.png)

### Conclusion:

This is a short post but the process definitely came through in a pinch so I wanted to get it documented. Luckily I was at a location with high enough bandwidth that the entire procedure was not TOO painful. I was able to help the client in their time of need and provide the level of service they are accustomed to receiving. That being said I will be setting up a full time SFTP server in the DMZ[^5] at the office to handle these issues in the future. If you have any questions or suggestions feel free to comment below!  Thanks!


[^1]: Secure File Transfer Protocol [ ↩](1 "return to article")
[^2]: Cisco Unified Communications System [ ↩](2 "return to article")
[^3]: Cisco Unity Connection [ ↩](3 "return to article")
[^4]: Remote Desktop Protocol [ ↩](4 "return to article")
[^5]: Demilitarized Zone [ ↩](5 "return to article")