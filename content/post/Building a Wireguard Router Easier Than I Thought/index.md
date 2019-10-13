---
title: Building a Wireguard Router...Easier Than I Thought!
url: "building-a-wireguard-router"
date: "2019-10-12T15:51:00-0500"
draft: false
tags:
  - Wireguard
  - VPN
---

I have been seeing a lot of buzz about Wireguard.  I had considered setting up a server at home for external access just for fun, but all of the examples I saw used NAT behind the Wireguard box and I wanted to route entire subnets without NATing.  After I finally took some time and realized that Wireguard was just an interface and I would just be leveraging some `iptables` it all came together rather quickly.

<!--more-->

First, I have to give some credit to all of the sites that helped me along the way:

  * https://wiki.archlinux.org/index.php/WireGuard
  * https://securityespresso.org/tutorials/2019/03/22/vpn-server-using-wireguard-on-ubuntu/
  * https://emanuelduss.ch/2018/09/wireguard-vpn-road-warrior-setup/
  * https://restoreprivacy.com/wireguard/
  * https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/
  * https://github.com/pirate/wireguard-docs

Without these write ups I would have had a much longer journey.

In this post I will not be going into any theory.  This is just a run down of my setup and mostly notes to myself before I forget everything!

I am running this on a NUC that I use for [HASS.IO](https://www.home-assistant.io/hassio/).  It has a lot of horsepower and is already in my DMZ so I thought, why not.  This NUC is running Ubuntu 18.04 and was up to date when I started.  After making sure my box was up to date I performed the following steps:

```
sudo -s
add-apt-repository ppa:wireguard/wireguard
apt install qrencode
apt install wireguard
modprobe wireguard
lsmod | grep wireguard # This lets you know that the wireguard module is good to go.

cat << EOF >> /etc/sysctl.conf  # These lines add forwarding capability to your server
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF

sysctl -p
```

This should get the system ready to go.  Now set the wireguard ser to start at bootup:

`systemctl enable wg-quick@wg0`

Now we move on to configuring the server.

Run the following commands to lay the groundwork:

```
cd /etc/wireguard
umask 077
wg genkey | sudo tee privatekey | wg pubkey | sudo tee publickey
```

Next use you favorite text editor to create the `wg0.conf` file.  Here is an example of mine:

```
[Interface]
Address = 172.24.1.1/24
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -i eno1.11 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -i eno1.11 -j ACCEPT
ListenPort = 51820
PrivateKey = < use privatekey generated in previous step >
```

_You will need to adjust the interface in the `iptables` statement to reflect your interface name.  I used `eno1.11` on my server._

I have been asked about locking down the server some more using `iptables`.  It is definitely possible!  [Here is a good example post](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/).  I have my server behind pfSense which helps me keep it locked down and that is why I used the `iptables` config above.

In this setup you will not perform any NAT.  You will ahve to ensure routing is setup properly in your environment since this will route your VPN user IPs straight through.  I use `pfSense` and have static routing setup to allow the proper flow of traffic.  It also helps me lock down access.

Now you can run `wg-quick up wg0` to bring up your wireguard interface.  Too easy!

Now to configure a client.  I use the wireguard app on my iPad and iPhone, so I will go through an example of how I configure a client for that.

First, I create a directory for my device.  This is not required, I do it for my own sanity.  Since we are still in `/etc/wireguard` i run `mkdir dan_iphone` then switch into that directory.  Next things look familiar:

```
cd dan_iphone/
umask 077
wg genkey | sudo tee privatekey | wg pubkey | sudo tee publickey
```

Now I create a file named `dan_iphone.conf`.  See example below:

```
[Interface]
Address = 172.24.1.2/24
PrivateKey = < use privatekey generated in previous step >

[Peer]
Endpoint = < Your server name or IP >:51820
PublicKey = < Your server's public key >
AllowedIPs = 172.24.1.1/32, 192.168.0.0/24
PersistentKeepalive = 25
```

For allowed IPs use whatever you need for access to your network or `0.0.0.0/0` for a full tunnel experience.

You will need to run the following command on the server to add this peer:

`wg set wg0 peer < peer public key > allowed-ips 172.24.1.2/32 persistent-keepalive 0`

Now generate a QR code to configure the Wireguard App on your iPhone:

`qrencode -t ansiutf8 < dan_iphone.conf`

You can also output this to a file if you need to ship it off for some reason:

`qrencode -t ansiutf8 -o dan_iphone.jpg < dan_iphone.conf`

Once your phone is configured you will be good to go!

To delete a peer do the following (while the `wg0` interface is up):

`wg set wg0 peer <peer_pubkey> remove`

I also added the piece pointed out in the [Archlinux Wireguard Page](https://wiki.archlinux.org/index.php/WireGuard#Endpoint_with_changing_IP) to deal with changing IPs.  Since I am using this on phones and mobile hotspots, this could be an issue.

Here are the two files I created:

`/etc/systemd/system/wireguard_reresolve-dns.timer`
```
[Unit]
Description=Periodically reresolve DNS of all WireGuard endpoints

[Timer]
OnCalendar=*:*:0/30

[Install]
WantedBy=timers.target
```

`/etc/systemd/system/wireguard_reresolve-dns.service`
```
[Unit]
Description=Reresolve DNS of all WireGuard endpoints
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'for i in /etc/wireguard/*.conf; do /usr/share/doc/wireguard-tools/examples/reresolve-dns/reresolve-dns.sh "$i"; done'
```

Then you setup the timer service to run:

`systemctl enable --now wireguard_reresolve-dns.timer`

Now you should be good to go!  If you have any questions leave a comment.  Thanks for reading!