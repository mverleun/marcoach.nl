---
id: 102
title: Mountain Lion 10.8 DHCP Server
date: 2012-08-11T15:57:58+00:00
author: Marco Verleun
excerpt: |
  <p><strong>NOTE: This article is no longer applicable if you updated Server.app</strong></p>
  <p><strong>The new version of Server.app allows DHCP management again.</strong></p>
  <p> </p>
  <p><span style="background-color: #ffffff; color: #565656; font-family: Helvetica, Arial, Geneva, sans-serif; font-size: 13px; line-height: 18px;">Mountain Lion Server continues to work as a DHCP server if the upgraded Lion Server was running as a DHCP server. If you have a small network it could be an option to introduce another DHCP server via an Airport Extreme or other network device. However, if you have to service multiple subnets this is not an option. Imagine a training facility with different classrooms that are all different VLAN's. Or a business that uses VLAN's to isolate manufacturing from sales networks. Do they need to invest in multiple DHCP servers? No they don't.</span></p>
  <p style="margin: 11px 0px 0px; padding: 0px; border: 0px; outline: 0px; font-size: 13px; font-family: Helvetica, Arial, Geneva, sans-serif; vertical-align: baseline; color: #565656; line-height: 18px; background-color: #ffffff;">The Netinstall Service introduced in Server.app is basically an extension of the DHCP protocol and that is why Mountain Lion can still be configured as a DHCP server using the commandline by editing a plist file, /etc/bootpd.plist. Inspection of this file reveals the DHCP network settings for the various subnets. The only thing that can't be found in this file are the static mappings between mac addresses and IP addresses.</p>
layout: post
guid: http://www.marcoach.nl/wordpress/?p=102
permalink: /mountain-lion-dhcp-server/
hits:
  - "2185"
categories:
  - Tech Tips
---
**Note**: This article no longer applies if you have updated Server.app. Apple has reintroduced the ability to configure DHCP from a GUI.

Mountain Lion Server continues to work as a DHCP server if the upgraded Lion Server was running as a DHCP server. If you have a small network it could be an option to introduce another DHCP server via an Airport Extreme or other network device. However, if you have to service multiple subnets this is not an option. Imagine a training facility with different classrooms that are all different VLAN&#8217;s. Or a business that uses VLAN&#8217;s to isolate manufacturing from sales networks. Do they need to invest in multiple DHCP servers? No they don&#8217;t.<!--more-->

The Netinstall Service introduced in Server.app is basically an extension of the DHCP protocol and that is why Mountain Lion can still be configured as a DHCP server using the commandline by editing a plist file, /etc/bootpd.plist. Inspection of this file reveals the DHCP network settings for the various subnets. The only thing that can&#8217;t be found in this file are the static mappings between mac addresses and IP addresses.

These are stored in Open Directory on the server in /Local/Default, underneath Computers. These mappings can be managed using Workgroup Manager. Just login as a local administrator instead of a directory administrator and have a look.

<div class="block wrapchrome image file" id="image-block-view-c557dce5-5519-5519-d7c5-6905ea0d3b2e" style="margin: 12px 0px; padding: 0px; border: 0px; outline: 0px; font-size: 13px; font-family: Helvetica, Arial, Geneva, sans-serif; vertical-align: baseline; display: table; width: 696px; color: #565656; line-height: 18px; background-color: #ffffff;">
  <div class="wrapper wrapchrome" id="image-block-view-c557dce5-5519-5519-d7c5-6905ea0d3b2e-wrapper" style="margin: 0px 1px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; display: table-cell; max-width: 696px; text-align: center;">
    <div class="inner wrapchrome" id="image-block-view-c557dce5-5519-5519-d7c5-6905ea0d3b2e-inner" style="margin: 0px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: top; position: relative; max-width: 100%; display: inline-block;">
      <div class="content selectable wrapchrome" style="margin: 0px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; -webkit-user-select: text !important;">
        <div class="container wrapchrome" style="margin: 0px; padding: 0px; border: 1px solid #ffffff; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; background-color: #f0f0f0; cursor: default; min-height: 20px; min-width: 20px; position: relative; border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; box-shadow: rgba(0, 0, 0, 0.496094) 0px 0px 6px;">
          <div class="image wrapchrome" style="margin: 0px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; overflow: hidden; border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; cursor: default;">
            <img class="clickable cc-routeable" style="margin: 0px; padding: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; display: block; max-width: 100%;" alt="" src="https://mac-mini.marcoach.nl/wiki/files/download/e7ef220d-107f-44bb-b333-99d793fe45fa" border="0" />
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

It is easy to add aditional computers to provide static-mappings if you need them. Just click on the plus sign (New Computer) and fill in the fields. Alternative you can create a file /etc/bootptab with contains the above mappings.

These are the static mappings, let&#8217;s just have a look at /etc/bootpd.plist to figure out how to setup and/or maintain this file in order to be able to add networks, change DHCP parameters etc.

Relevant comments are placed between <&#8211; and &#8211;> to indicate important directives which are also printed in bold. For a full explanation you can always read the manpage (man bootpd).

<pre>$ cat bootpd.plist</pre>

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "<a href="http://www.apple.com/DTDs/PropertyList-1.0.dtd" target="_blank">http://www.apple.com/DTDs/PropertyList-1.0.dtd</a>"&gt;
&lt;plist version="1.0"&gt;
&lt;dict&gt;
 &lt;key&gt;NetBoot&lt;/key&gt;
 &lt;dict/&gt;
 &lt;key&gt;<strong>Subnets</strong>&lt;/key&gt; &lt;-- Create an entry for each subnet --&gt;
 &lt;array&gt;
 &lt;dict&gt;
 &lt;key&gt;allocate&lt;/key&gt;
 &lt;true/&gt;
 &lt;key&gt;dhcp_bootfile_name&lt;/key&gt;
 &lt;string&gt;pxelinux/pxelinux.0&lt;/string&gt;
 &lt;key&gt;dhcp_domain_name&lt;/key&gt;
 &lt;string&gt;example.com&lt;/string&gt;
 &lt;key&gt;<strong>dhcp_domain_name_server</strong>&lt;/key&gt; &lt;-- DNS Servers offered to clients --&gt;
 &lt;array&gt;
 &lt;string&gt;192.168.15.2&lt;/string&gt;
 &lt;string&gt;8.8.8.8&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;<strong>dhcp_domain_search</strong>&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;example.com&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;dhcp_ldap_url&lt;/key&gt; &lt;-- important to get static mappings from Open Directory --&gt;
 &lt;array&gt;
 &lt;string&gt;ldap://mac-mini.example.com/dc=mac-mini,dc=example,dc=com&lt;/string&gt; &lt;-- hostname of DHCP server followed by hostname written in LDAP dc notation --&gt;
 &lt;/array&gt;
 &lt;key&gt;dhcp_nb_over_tcpip_name_server&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;192.168.15.2&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;dhcp_nb_over_tcpip_node_type&lt;/key&gt;
 &lt;string&gt;8&lt;/string&gt;
 &lt;key&gt;<strong>dhcp_router</strong>&lt;/key&gt; &lt;-- Default gateway for clients --&gt;
 &lt;string&gt;192.168.15.1&lt;/string&gt;
 &lt;key&gt;lease_max&lt;/key&gt;
 &lt;integer&gt;86400&lt;/integer&gt;
 &lt;key&gt;lease_min&lt;/key&gt;
 &lt;integer&gt;86400&lt;/integer&gt;
 &lt;key&gt;name&lt;/key&gt;
 &lt;string&gt;192.168.15 Local network&lt;/string&gt;
 &lt;key&gt;net_address&lt;/key&gt;
 &lt;string&gt;192.168.15.0&lt;/string&gt; &lt;-- Subnet for DHCP requests --&gt;
 &lt;key&gt;<strong>net_mask</strong>&lt;/key&gt;
 &lt;string&gt;255.255.255.0&lt;/string&gt;
 &lt;key&gt;<strong>net_range</strong>&lt;/key&gt; &lt;-- Dynamic address for clients --&gt;
 &lt;array&gt;
 &lt;string&gt;192.168.15.150&lt;/string&gt;
 &lt;string&gt;192.168.15.199&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;selected_port_name&lt;/key&gt;
 &lt;string&gt;en0&lt;/string&gt;
 &lt;key&gt;uuid&lt;/key&gt;
 &lt;string&gt;ECD03D13-AAB4-46D5-AA80-8D727D78BFA4&lt;/string&gt;
 &lt;/dict&gt;
 &lt;/array&gt;
 &lt;key&gt;allow_disabled&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;bootp_enabled&lt;/key&gt; &lt;-- Keep disabled of you want to use DHCP --&gt;
 &lt;false/&gt;
 &lt;key&gt;deny_disabled&lt;/key&gt;
 &lt;array&gt;
 &lt;/array&gt;
 &lt;key&gt;detect_other_dhcp_server&lt;/key&gt;
 &lt;false/&gt;
 &lt;key&gt;<strong>dhcp_enabled</strong>&lt;/key&gt; &lt;-- Enables DHCP --&gt;
 &lt;array&gt;
 &lt;string&gt;en0&lt;/string&gt; &lt;-- Interfaces on which DHCP is confituren --&gt;
 &lt;string&gt;vlan1&lt;/string&gt;
 &lt;array/&gt;
 &lt;key&gt;netboot_enabled&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;en0&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;old_netboot_enabled&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;vlan1&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;relay_enabled&lt;/key&gt;
 &lt;false/&gt;
 &lt;key&gt;relay_ip_list&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;reply_threshold_seconds&lt;/key&gt;
 &lt;integer&gt;0&lt;/integer&gt;
 &lt;key&gt;startTime&lt;/key&gt;
 &lt;string&gt;2012-07-18 19:59:55 +0000&lt;/string&gt;
 &lt;key&gt;timeServiceStarted&lt;/key&gt;
 &lt;string&gt;2012-07-17 18:20:58 +0000&lt;/string&gt;
&lt;/dict&gt;
&lt;/plist&gt;
 &lt;array/&gt;
 &lt;key&gt;netboot_enabled&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;en0&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;old_netboot_enabled&lt;/key&gt;
 &lt;array&gt;
 &lt;string&gt;vlan1&lt;/string&gt;
 &lt;/array&gt;
 &lt;key&gt;relay_enabled&lt;/key&gt;
 &lt;false/&gt;
 &lt;key&gt;relay_ip_list&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;reply_threshold_seconds&lt;/key&gt;
 &lt;integer&gt;0&lt;/integer&gt;
 &lt;key&gt;startTime&lt;/key&gt;
 &lt;string&gt;2012-07-18 19:59:55 +0000&lt;/string&gt;
 &lt;key&gt;timeServiceStarted&lt;/key&gt;
 &lt;string&gt;2012-07-17 18:20:58 +0000&lt;/string&gt;
&lt;/dict&gt;
&lt;/plist&gt;</pre>

&nbsp;

&nbsp;

(This page is copied from our Wiki: [https://mac-mini.marcoach.nl/wiki/pages/T61421N/5\_Managing\_the\_DHCP\_Service.html](https://mac-mini.marcoach.nl/wiki/pages/T61421N/5_Managing_the_DHCP_Service.html)#) Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.