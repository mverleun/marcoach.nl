---
id: 1030
title: pfSense UTM firewall, installing and configuring
date: 2014-09-28T21:08:02+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=1030
permalink: /pfsense-utm-firewall/
hits:
  - "11564"
categories:
  - Tech Tips
---
## Unified Threat Management (UTM)

As the owner of a small company I do have a few servers directly connected to the internet. All are updated regularly and the security of each server is as tight as reasonably possible. Configuring the firewall on these servers is not a big deal. Remote login via ssh is configured to be done with keys instead of passwords. <a href="http://www.fail2ban.org/" target="_blank">Fail2ban</a> is quite responsive to anomalies in the log files etc.

<!--more-->On the open ports there still is a lot going on. Despite the fact that keys are required for ssh and that my servers would not relay SPAM they still have to deal with all the attempts. Maintaining these servers takes some time, but what I didn&#8217;t like is that they all were battling these bad folks on the internet by themselves.  Over time they seemed to be increasingly busy doing this, time to take the next step.

## pfSense UTM Firewall

I wanted to install a <a href="http://en.wikipedia.org/wiki/Unified_threat_management" target="_blank">UTM</a> firewall.  The UTM part is more important. UTM goes further that a firewall. It has the ability to inspect the network traffic and respond according to certain patterns and/or frequencies.

The firewall has to support bridging. Using a bridged interface I can connect it between my ISP&#8217;s router and my own hosts without changing the network topology any further. In essence my direct connected servers are now in a <a href="http://en.wikipedia.org/wiki/Demilitarized_zone" target="_blank">DMZ</a>.

There are many commercial products that can do this job, but they are all quite expensive. <a href="https://www.google.nl/?gws_rd=ssl#q=opnsense" target="_blank">OPNSense</a> has an affordable module but it has very little memory. And to run a UTM you need memory to keep track of what is going on.

OPNSense is based on <a href="https://www.pfsense.org" target="_blank">pfSense</a>, an open source firewall and UTM based on BSD Linux. Feature wise I liked it more than <a href="http://m0n0.ch/" target="_blank">m0n0wall</a>, <a href="http://www.endian.com" target="_blank">Endian Community Edition</a> or <a href="http://www.smoothwall.org" target="_blank">Smoothwall</a>. All I needed was suitable hardware to install it on. I found it in France. The folks are <a href="http://www.gooze.eu" target="_blank">Gooze</a> are into security and also supply decent hardware to build your own firewall/UTM solution.

I ordered an APU board with 4Gb of RAM and a 16 Gb mSata drive with an enclose and power supply. This is a 64 bit machine with 3 1 Gb ethernet ports with still a low power consumption.

## Installation

Installing the system requires the use of a null modem cable. I used the 64 bit LiveCD/Installer of pfSense that can be downloaded <a href="https://www.pfsense.org/about-pfsense/versions.html" target="_blank">here</a>. When installing make sure that you choose the correct kernel. The default kernel requires a VGA adapter, but the other kernel uses the serial port as a console.

## Configuration

Once the installation is completed the system will start for the first time. This is where the interfaces are assigned. I use the first interface (WAN) to connect to the internet. The second interface (LAN) connects to my servers and functions as a DMZ. The third interface (OPT1) is connected to my office network and can be used to access the website of pfSense.

Once the basic configuration was done I connected to the website and changed the network configuration. Port 1 still connects to the internet but has no ip configuration. Port 2 has no ip configuration either. I created a bridge consisting of port 1 and port 2 and gave it an official IP address that I have available. This is not necessary but allows me to run additional services that require an IP address. Without an IP address the bridge still passes traffic back and forth between the two ports and fire walling is still possible. Management is done via the OPT1 interface after all.

## Additional software packages

By it self pfSense is a firewall. It filters network traffic based on ip addresses, protocols, port numbers etc. pfSense also has a repository with additional packages that make it a true UTM device. I installed <a href="https://www.snort.org" target="_blank">Snort</a> and <a href="https://doc.pfsense.org/index.php/Pfblocker" target="_blank">pfBlocker</a> to control the traffic.

LADVD is also installed. This allows me to better manage the local network topology with <a href="http://en.wikipedia.org/wiki/Link_Layer_Discovery_Protocol" target="_blank">LLDP</a> and is something I install on all servers. Squid3 is installed to be able to provide proxy services and OpenVPN Client Export Utility helps me to easily configure VPN clients.

## Snort

In my setup Snort is configured to monitor traffic on the bridge interface only. The challenge is to not enable all rulesets. This will generate many falls positives and probably locks out quite some good guys. My advice is to have Snort running for a couple of days before you enable the Block Offenders function. And even then, start of with a short Blocked Hosts Interval.

My favourite rulesets are:

  * **Snort VRT** rules
  * **Snort Community **rules
  * **Emerging Threats **rules

They are updated regularly making sure that my protection level is good.

## Custom Snort rule

With the above settings I was lacking one feature: Blocking malicious mail servers that are blacklisted by Spamhaus. I&#8217;ve written a custom rule that will detect a rejection from a server. As soon as the rejection is detected Snort will block this mail server. The custom rule is as follows:

<pre class="lang:default decode:true" title="Snort Custom rule">alert tcp $SMTP_SERVERS 25 -&gt; $EXTERNAL_NET any (msg:"CUSTOM Spamhaus block relay access denied"; flow:from_server,established; content:"blocked using zen.spamhaus.org"; reference:url,http://www.spamhaus.org/query/bl; classtype:misc-activity; sid:4000013; rev:1;)</pre>

## pfBlocker

Snort downloads <a href="http://www.spamhaus.org/drop/" target="_blank">DROP</a> lists as part of the Emerging Threats ruleset.  pfBlocker does a similar job. It can download lists of known evil sources and automatically block them at the firewall level. This saves  performance since it doesn&#8217;t require as much CPU power as Snort requires.

## OpenVPN

Is easy to install. It has a client for Windows, Linux, Android, OS X and iOS. The OpenVPN client export Utility makes it easy to generate configurations specific for these platforms.

## Postfix Relay

The reason I install the Postix Relay package is that it&#8217;s always good to have a backup mail server that can hold the incoming mail temporary if the main mail server is unavailable. Key here is to avoid it being used by spammers to bypass the SPAM and virus filtering on the main server. In my setup the mail that&#8217;s being relayed still has to pass all checks on the main mail server. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.