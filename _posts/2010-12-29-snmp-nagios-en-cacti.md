---
id: 75
title: SNMP, Nagios en Cacti
date: 2010-12-29T23:00:00+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=75
permalink: /snmp-nagios-en-cacti/
hits:
  - "329"
xyz_fbap:
  - "1"
categories:
  - Training Portfolio
---
De open source gemeenschap heeft een paar geweldige tools ontwikkelt waarmee op eenvoudige wijze netwerken, systemen en diensten gemonitord kunnen worden:

<a href="http://www.cacti.net/" target="_blank">Cacti</a> wordt met name gebruikt om trends in kaart te brengen. <a href="http://www.nagios.org/" target="_blank">Nagios</a> wordt gebruikt om de status van hosts en services te bewaken en bij het uitvallen hiervan een notificatie te sturen. <a href="http://www.net-snmp.org/" target="_self">SNMP</a> wordt door veel devices gebruikt als beheer protocol.<!--more-->

Tijdens deze cursus komt de configuratie van SNMP op Linux hosts aan bod waarbij de authorizatie aan bod komt. Ook wordt behandeld hoe SNMP uitgebreid kan worden zodat er ook andere informatie via SNMP beschikbaar gesteld wordt voor bijvoorbeeld Cacti en/of Nagios.

Over Cacti leert de cursist hoe de diverse componenten samenwerken en hoe de cursist zelf eenvoudige scripts kan schrijven om daarmee niet SNMP gerelateerde trendgrafieken te maken.

Bij Nagios wordt de nadruk gelegd op de werking van Nagios, de interne configuratiebestand structuur en het uitbreiden van Nagios met eigen scripts die zowel host en service monitoring doen als ook eigen notificatie scripts waarmee de nagios meldingen gekoppeld kunnen worden aan een helpdesk/servicedesk systeem. De configuratie van Nagios wordt gedaan mbv. een webinterface (NagiosQL) Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.