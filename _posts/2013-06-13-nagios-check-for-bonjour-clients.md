---
id: 746
title: Nagios check for bonjour clients
date: 2013-06-13T12:03:24+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=746
permalink: /nagios-check-for-bonjour-clients/
hits:
  - "901"
categories:
  - Tech Tips
---
# Nagios and bonjour

[Nagios](http://www.nagios.org/) is great to determine the status of hosts, clients, printers etc. as long as this node has a fixed ip address.

OS X and iOS are two operating systems that strive, very successful, to eliminate the dependency of a fixed IP address by discovering automatically when needed. This is done with the bonjour protocol. Changes in IP address do not really matter for these OS&#8217;es and changes occur quite frequently if you are using different technologies to connect to the network, e.g. switch from wireless to a cable or vice versa.<!--more-->

In order to allow Nagios to monitor such hosts I&#8217;ve written a small check script that works with the bonjour name (ending in .local) when checking a node. It requires that the package avahi-tools is installed on the system (CentOS 6). If you are not familiar with writing plugins for Nagios you should visit this [site](http://nagiosplug.sourceforge.net/developer-guidelines.html).

The name of the check script is check_bonjour and for bonjour nodes you&#8217;ll have to use this one in stead of the default check-host-alive in your host configuration.

The script is placed in a sub directory of the default Nagios plugins directory.

The content of the script is:

<pre>#!/bin/bash
BonjourName="$1" 
AVAHI_RESULT=$(avahi-resolve -4 -n "$BonjourName" 2&gt;/dev/null )

IP=$(echo $AVAHI_RESULT | awk '{ print $2 }')
if [ -z $IP ]
then
 echo "Bonjour host not resolved"
 exit 2
fi
PING_RESULT=$(ping -c 3 $IP 2&gt;/dev/null | tail -n 1)
if [ $? -ne 0 ]
then
 echo "Bonjour host not responding"
 exit 2
fi
echo "Current ip address: $IP| $PING_RESULT"
exit 0</pre>

The corresponding command definition for Nagios is:

<pre>define command {
 command_name check_bonjour
 command_line $USER1$/local/check_bonjour "$HOSTNAME$" 
 register 1
}</pre>

You might consider disabling flap detection and alerting&#8230;

When configuring the host you can&#8217;t use IP addresses obviously. Use the bonjour name (ending in .local) for both the hostname and the address. The script uses the host name and doesn&#8217;t require the address. Nagios however doesn&#8217;t like it if you leave it empty&#8230; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.