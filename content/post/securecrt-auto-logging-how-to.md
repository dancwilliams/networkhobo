---
title: "SecureCRT Auto Logging How-To"
date: 2018-01-10T09:41:43-06:00
draft: false
tags: [ "SecureCRT", "logging", "network" ]
categories: [ "SecureCRT" ]
---

This is a quick post to document the process I use to auto-log [SecureCRT](https://www.vandyke.com/products/securecrt/) sessions.

I like to automatically log all sessions that I open, just so I have a record of all my keystrokes.  It also helps when doing discovery operations so that I do not have to repeat logins to collect information that I may not have felt was important previously.

<!--more-->

This post will lay out how to setup aut logging for all sessions globally.  This is also geared toward Windows, but the same logic can be used on any platform.  Only the folder structures will change.

*I am using SecureCRT Version 7.3.7 (build 1034).*

Open **SecureCRT** > **Options** > **Global Options** > **General** > ****Default Session > Select the **Edit Default Settings** button:

[![SecureCRT Global Option](/img/securecrt-auto-logging-how-to/secure-crt-screecap.png)](/img/securecrt-auto-logging-how-to/secure-crt-screecap.png)

Log File Name: `C:\Users\{{ YOUR USERNAME }}\{{ YOUR FOLDER LOCATION }}\%Y\%Y-%M\%Y-%M-%D\%S (%H) -- %Y-%M-%D_%h-%m.txt`

Upon Connect: `Start recording %S (%H) - %Y/%M/%D %h:%m:%s`

Upon Disconnect: `Stop recording %S (%H) - %Y/%M/%D %h:%m:%s`

On each line: `%h:%m:%s.%s §`