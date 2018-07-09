---
id: 74
title: Nagios SMS Notificaties
date: 2010-12-29T14:36:43+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=74
permalink: /nagios-sms-notificaties/
hits:
  - "3558"
categories:
  - Tech Tips
---
Sending SMS notifications from Nagios is not as difficult as it might seem. It has the advanced of being able to send alerts even if the internet connection is down.

Nagios sends notifications via external scripts. The Nagios macro&#8217;s can be used to call this script with arguments.

All that is required is a compatible SMS device. In the example below a dongle is used.

<!--more-->

From the systems perspective the dongle is a modem. The device used presented itself to the system as /dev/ttyUSB and is a &#8216;Huawei Technologies Co., Ltd. E220 HSDPA Modem / E270 HSDPA/HSUPA Modem&#8217; which is compatible with SMS Server Tools 3.

The config file of SMS Tools, /etc/smsd.conf, looks like this:

<pre>devices = GSM1
loglevel = 7
autosplit = 3
incoming = /var/spool/sms/incoming
outgoing = /var/spool/sms/outgoing
checked = /var/spool/sms/checked
failed = /var/spool/sms/failed
sent = /var/spool/sms/sent
logfile = /var/log/smsd.log

[GSM1]
device = /dev/ttyUSB0
incoming = no
init2=AT+CSQ
pin = 0000</pre>

Make sure that all the referenced directories do exist and make sure that SMS Tools services are started at boot time.

<pre>chkconfig smsd on</pre>

Start smsd:

<pre>service smsd start</pre>

Check that it is working with a command-line command:

<pre># smssend “Message…”</pre>

The next step is to write a script that is called by Nagios. It should format the SMS and send it via SMS Tools. The name of the script is /usr/local/bin/nagios-send-sms and looks like this:

<pre>#!/bin/bash
#
# This script takes input from STDIN and queues it in the outgoing
# directory of the sms tools
#
# The message has to be formatted in a specific way:
# To:  &lt;destination&gt;\n\n&lt;message&gt;
## (c) 2010 Marco Verleun, marco at marcoach.nl
# Get the message from STDIN
MESSAGE=$(cat /dev/stdin)
echo $MESSAGE
# Create a file in the outging queue
SMSFILE=$(mktemp /var/spool/sms/outgoing/send_XXXXXX)
# Fill it with the contents of the message
cat &gt; $SMSFILE&lt;&lt; EOT
$MESSAGE
EOT</pre>

With this script in place it is time to configure a Nagios command in the Nagios config files:

<pre>define command {
   command_namenotify-host-by-sms
   command_line/usr/bin/printf "%b" "To: $CONTACTPAGER$nn***** Nagios *****nnNotification Type: $NOTIFICATIONTYPE$nHost: $HOSTNAME$nState: $HOSTSTATE$nAddress: $HOSTADDRESS$nInfo: $HOSTOUTPUT$nnDate/Time: $LONGDATETIME$n** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **"  | /usr/local/bin/nagios-send-sms
}</pre> Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.