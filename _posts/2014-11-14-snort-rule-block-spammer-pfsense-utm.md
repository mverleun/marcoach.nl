---
id: 1038
title: Snort rule to block spammer on pfSense UTM
date: 2014-11-14T11:15:34+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=1038
permalink: /snort-rule-block-spammer-pfsense-utm/
hits:
  - "2080"
panels_data:
  - 'a:0:{}'
categories:
  - Tech Tips
---
# pfSense UTM

In a previous [post](http://www.marcoach.nl/pfsense-utm-firewall/ "pfSense UTM firewall, installing and configuring") I wrote about installing pfSense to reduce the amount of undesired traffic to my servers. It turned out to be a great learning experience where I tried many of the possible rules available in [Snort](https://www.snort.org). Most of them don&#8217;t apply and some of them generate falls positives.

After some analysis it seems that over 95% of all the undesired traffic is blocked by the rules that use blacklists so I decided to keep things simple.

# Undesired traffic

Undesired traffic to my servers consists of two types of traffic. The 95% that is blocked using the blacklist rules and the other 5% is traffic that seems to be legitimate, e.g. a not blacklisted server is trying to deliver SPAM using a legitimate port, just like the mailservers that do deliver mail.

My mailserver does an excellent job to block those servers on its own but it still requires some effort from this server. I would like to block these sources of SPAM at the front door.

# Missing Snort rule

After quite some Google magic I concluded that there is no existing rule that would do this. So I went along and wrote a custom rule.

The rule itself is not really complex. When the mailserver decides to refuse a server it will send a &#8220;relay denied&#8221; message back to the &#8216;SPAMMER&#8217;. This message is picked up by Snort on the firewall and Snort will automatically block the sending host.

You might want to adjust the rule so it matches exactly what your mailserver is sending back if you implement something similar.

Here is my Snort rule:

<pre class="lang:default decode:true " title="Snort relay denied rule">alert tcp $SMTP_SERVERS 25 -&gt; $EXTERNAL_NET any (msg:"CUSTOM Spamhaus block relay access denied"; flow:from_server,established; content:"blocked using zen.spamhaus.org"; reference:url,http://www.spamhaus.org/query/bl; classtype:misc-activity; sid:4000013; rev:1;)</pre>

&nbsp; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.