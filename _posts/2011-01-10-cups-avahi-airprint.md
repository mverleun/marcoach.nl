---
id: 88
title: Linux Airprint Server
date: 2011-01-10T21:37:26+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=88
permalink: /cups-avahi-airprint/
hits:
  - "895"
xyz_fbap:
  - "1"
categories:
  - Tech Tips
---
AirPrint is de nieuwe techniek van Apple waarmee het mogelijk is om vanaf een IOS 4.2 device te printen zonder drivers te installeren. De printers worden via het <a href="http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/NetServices/Introduction.html%23//apple_ref/doc/uid/TP40002445-SW1" target="_blank">bonjour</a> protocol (ook bekend als mDNS of multicast DNS) op het netwerk geadverteerd en herkend door het IOS device. Hiermee kunnen iPhone en iPad gebruikers zonder extra handelingen afdrukken maken.

Verschillende leveranciers ondersteunen de AirPrint functionaliteit in (een aantal van) hun printers. Oudere modellen ondersteunen dit helaas niet.<!--more-->

Voor printers die via CUPS op een Linux server worden aangestuurd is het op eenvoudige wijze mogelijk om als AirPrint compatibele printer geconfigureerd te worden. Het enige dat nog ontbreekt bij een printer die via CUPS gedeeld wordt is dat die niet via het bonjour protocol wordt geadverteerd.

Bij de meeste Linux distributies wordt Avahi gebruikt om services via het bonjour protocol te adverteren. Avahi leest hiervoor informatie uit XML bestanden in /etc/avahi/services. Door zelf een service bestand aan deze directory toe te voegen kunnen bestaande CUPS printers nu ook worden gevonden door de IOS devices en is het mogelijk om te printen vanaf een iPhone of iPad.

Een voorbeeld van zo&#8217;n bestand ziet er als volgt uit:<?xml version=&#8221;1.0&#8243; standalone=&#8217;no&#8217;?>

<pre><em id="__mceDel">&lt;!--*-nxml-*--&gt;
 &lt;!DOCTYPE service-group SYSTEM "avahi-service.dtd"&gt; &lt;service-group&gt;
 &lt;name&gt;Cups Printer via AirPrint&lt;/name&gt;
 &lt;service&gt;
   &lt;type&gt;_ipp._tcp&lt;/type&gt;
   &lt;subtype&gt;_universal._sub._ipp._tcp&lt;/subtype&gt;
   &lt;port&gt;631&lt;/port&gt; &lt;txt-record&gt;txtver=1&lt;/txt-record&gt;
   &lt;txt-record&gt;qtotal=1&lt;/txt-record&gt;
   &lt;txt-record&gt;rp=printers/Cups_Printer_Name&lt;/txt-record&gt;
   &lt;txt-record&gt;ty=Play Printer&lt;/txt-record&gt;
   &lt;txt-record&gt;adminurl=http://www.example.com:631/printers/Cups_Printer_Name&lt;/txt-record&gt;
   &lt;txt-record&gt;note=Cups Printer via AirPrint&lt;/txt-record&gt; &lt;txt-record&gt;priority=0&lt;/txt-record&gt;
   &lt;txt-record&gt;product=virtual Printer&lt;/txt-record&gt;
   &lt;txt-record&gt;printer-state=3&lt;/txt-record&gt;
   &lt;txt-record&gt;printer-type=0x801046&lt;/txt-record&gt;
   &lt;txt-record&gt;Transparent=T&lt;/txt-record&gt;
   &lt;txt-record&gt;Binary=T&lt;/txt-record&gt;
   &lt;txt-record&gt;Fax=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Color=T&lt;/txt-record&gt;
   &lt;txt-record&gt;Duplex=T&lt;/txt-record&gt;
   &lt;txt-record&gt;Staple=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Copies=T&lt;/txt-record&gt;
   &lt;txt-record&gt;Collate=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Punch=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Bind=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Sort=F&lt;/txt-record&gt;
   &lt;txt-record&gt;Scan=F&lt;/txt-record&gt;
   &lt;txt-record&gt;pdl=application/octet-stream,application/pdf,application/postscript,image/jpeg,image/png,image/urf&lt;/txt-record&gt;
   &lt;txt-record&gt;URF=W8,SRGB24,CP1,RS600&lt;/txt-record&gt;
 &lt;/service&gt;
&lt;/service-group&gt;</em></pre>

Tussen <name> en </name> (regel 4) staat de naam waaronder de printer gevonden wordt door de iPhone en/of iPad. Het textrecord <txt-record>rp=printers/Cups\_Printer\_Name</txt-record> (regel 11) is de naam van de printer waaronder deze in CUPS bekend is. Het is hetzelfde als het laatste deel van de URL die gebruikt wordt als de printer als IPP printer wordt aangesproken.

De <txt-record>adminurl</txt-record> regel (regel 13) bevat de URL die in CUPS gebruikt wordt om de printer te beheren.

Geef via T (True) of F (False) aan welke functies de printer ondersteunt zoals duplex printen en kleur op regel 19 t/m 30.

Zorg ervoor dat het bestand wordt opgeslagen onder een naam met de extensie .service, bijvoorbeeld AirPrint\_CUPS\_Printer_Name.service en herstart Avahi: /etc/init.d/avahi-daemon restart

Herhaal bovenstaande voor iedere gedeelde printer die via bonjour geadverteerd moet worden.

Bovengenoemde oplossing werkt in de meeste gevallen. De printerstatus wordt echter niet bijgewerkt en het kan voorkomen dat de printer offline is zonder dat het IOS device dit doorheeft.

Het is wachten op een nieuwe versie van CUPS die zelf de services adverteert en dynamisch de geadverteerde gegevens aanpast.

Als er veel printers via AirPrint beschikbaar moeten worden gemaakt dan is dit script wellicht een uitkomst?

Klik <a href="http://developer.apple.com/networking/bonjour/BonjourPrinting.pdf" target="_blank">hier</a> om de bonjour printing specificatie te lezen waarin in detail wordt ingegaan op de diverse instellingen. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.