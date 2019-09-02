---
title: "GoDaddy: Delegate Subdomain to Different Nameserver"
date: 2014-07-22
draft: false
url: "2014/07/22/godaddy-delegate-subdomain-to-different-nameserver"
---

1.  Access your GoDaddy domain manager
2.  Select your domain
3.  Select the “DNS Zone File” Tab
4.  Select “Add Record”
5.  Create a new “Nameserver” entry. See capture below. The “host” will be the subdomain you want to point to the new server. Use the proper nameserver naming format or GoDaddy will kick an error.

<!--more-->

{{< img src="images/5.png" alt="add fqdn initial pointer" >}}

6.  If you do not have a FQDN for your nameserver you will want to create an A record pointing to its IP:

{{< img src="images/6.png" alt="add pointer to dns server" >}}

### THANKS!

I would like to that [Glen Kemp](https://twitter.com/ssl_boy) who began discussing this topic with me. I thought I should put this up even though GoDaddy clearly supports this procedure in their [Support Forums](http://support.godaddy.com/help/article/680/managing-dns-for-your-domain-names?pc_split_value=4).