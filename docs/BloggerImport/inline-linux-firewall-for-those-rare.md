---
title: 'Inline Linux Firewall For Those Rare Occasions'
date: 2014-08-29T15:00:00.001+01:00
draft: false
aliases: [ "/2014/08/inline-linux-firewall-for-those-rare.html" ]
nav_order: 9000
description: "blogger"
has_children: false
has_toc: false
permalink: docs/blogger
has_children: false
has_toc: false
---

[Inline Linux Firewall For Those Rare Occasions](https://www.blogger.com/null)
------------------------------------------------------------------------------

[Contents](https://www.blogger.com/null)

  

1.0 Purpose. 3

2.0 Installation and Configuration. 3

Details. 3

Install Ubuntu with SSH.. 3

Adding Scripts to clean up archived log files to prevent disk exhaustion. 5

Installing Tivoli storage manager  5

3.0 Administration using firewall builder  6

  

[Illustrations](https://www.blogger.com/null)

  

Figure 1 - Firewall Builder GUI. 7

Figure 2 - Push Policy Step1. 8

Figure 3 - Push Policy Step2 Compiling Rules. 8

Figure 4 - Policy Push Step3 Deploying Files. 9

  

[Tables](https://www.blogger.com/null)

  

**No table of figures entries found.**

  

  

  

  

[1.0 Purpose](https://www.blogger.com/null)
===========================================

The purpose of this document is to describe the steps taken to install the operating system and configuration of that operating system to the point that in can be put into service as an inline firewall. The implementation does rely on the use of rapid spanning tree protocol to prevent network loops since there are two nodes for the sake of redundancy, this can make it a bit finicky(be sure to connect network on one node at a time leaving an interval before proceeding with the other node).

[](https://www.blogger.com/null)[2.0](https://www.blogger.com/null) Installation and Configuration
==================================================================================================

[Details](https://www.blogger.com/null)
---------------------------------------

OS; Ubuntu 12.10

Server; IBM 3650 M4, 1gb ram

UFW1 IP; 192.168.52.241

UFW2 IP; 192.168.52.242

  

[Install Ubuntu with SSH](https://www.blogger.com/null)
-------------------------------------------------------

First Install Ubuntu with SSH from CD and set the local account to sadmin.

  

~# apt-get update

~# apt-get upgrade

~# apt-get install bridge-utils ethtool ssh traceroute conntrackd vrrpd snmp ipset ifenslave-2.6 vlan

~# nano /etc/network/interfaces

  

 This file describes the network interfaces available on your system

\# and how to activate them. For more information, see interfaces(5).

  

\# The loopback network interface

auto lo

iface lo inet loopback

  

\# The primary network interface

auto em2

iface em2 inet dhcp

  

auto em3

iface em3 inet dhcp

  

auto em4

iface em4 inet dhcp

  

\# Management IP

auto em5

iface em5 inet static

address 172.16.52.242

netmask 255.255.255.0

gateway 172.16.52.254

  

#setup of bridged ports

auto p1p1

iface p1p1 inet manual

iface p1p2 inet manual

iface p1p3 inet manual

iface p1p4 inet manual

  

auto p2p1

iface p2p1 inet manual

iface p2p2 inet manual

iface p2p3 inet manual

iface p2p4 inet manual

  

iface br0 inet manual

bridge\_ports p1p1 p2p1

  

iface br1 inet manual

bridge\_ports p1p2 p2p2

  

iface br2 inet manual

bridge\_ports p1p3 p2p3

  

iface br3 inet manual

bridge\_ports p1p4 p2p4

  

  

~# nano /etc/sysctl.conf

  

Search for this line and uncomment it so that it looks like the following

  

net.ipv4.ip\_forward=1

  

~# mkdir /etc/fw

~# nano /etc/init.d/firewall

  

\# Required-Start:    $network

\# Required-Stop:

\# Default-Start:     2 3 4 5

\# Default-Stop:      0 1 6

\# Short-Description: start and stop the Firewall

\### END INIT INFO

  

opts="start stop restart"

bin=/etc/fw/rc.firewall.local

trapped\_log=/var/log/trapped.log

traf\_log=/var/log/traffic.log

  

case "$1" in

  start)

        $bin

;;

  

  stop)

        /sbin/iptables --flush

        /sbin/iptables -t nat --flush

        /sbin/iptables -F -t mangle

        /sbin/iptables -P INPUT ACCEPT

        /sbin/iptables -P OUTPUT ACCEPT

        /sbin/iptables -P FORWARD ACCEPT

        /sbin/iptables -t nat -P POSTROUTING ACCEPT

        /sbin/iptables -t nat -P PREROUTING ACCEPT

        /sbin/iptables -t nat -P OUTPUT ACCEPT

;;

  

esac

  

exit 0

  

  

~# chmod +x /etc/init.d/firewall

~# update-rc.d firewall defaults

~# nano /etc/rsyslog.conf

  

append to end of file

\*.\* @172.16.52.156:514

  

~# sudo su

~# passwd

  

  

  

### [Adding Scripts to clean up archived log files to prevent disk exhaustion](https://www.blogger.com/null)

Ubuntu uses anacron so you can drop scripts into /etc/cron.\[hourly|daily|monthly\] folder.

I created a script called clean-archived-logs and chmod 777 this file. The contents are;

  

#!/bin/sh

  

cd /var/log

rm \*.gz

  

I then symlinked this to the hourly folder for testing

  

### [Installing Tivoli storage manager](https://www.blogger.com/null)

Original source = http://enterprisetechieblog.wordpress.com/2013/05/12/tsm-v6-4-client-on-ubuntu/#comments

  

Ibm do not officially support ubuntu but we can install a few extra packages to translate the install across. Run the following commands

  

\# apt-get install ksh libstdc++5 alien

  

Transfer across and Unpack the official download

  

\# tar –xvf 6.4.0.7-TIV-TSMBAC-LinuxX86.tar

  

Run Alien on the rpms which will create directories for each package.

  

alien -k gskcrypt64-8.0.14.14.linux.x86\_64.rpm  
alien -k gskssl64-8.0.14.14.linux.x86\_64.rpm  
alien -k TIVsm-API64.x86\_64.rpm  
alien -k TIVsm-BA.x86\_64.rpm  
dpkg -i \*.deb

  

Link the libraries

  

ln -s /opt/tivoli/tsm/client/api/bin64/libgpfs.so /lib/

ln -s /opt/tivoli/tsm/client/api/bin64/libdmapi.so /lib/

ln -s /usr/local/ibm/gsk8\_64/lib64/libgsk8cms\_64.so /lib/

ln -s /usr/local/ibm/gsk8\_64/lib64/libgsk8ssl\_64.so /lib/

ln -s /usr/local/ibm/gsk8\_64/lib64/libgsk8sys\_64.so /lib/

ln -s /usr/local/ibm/gsk8\_64/lib64/libgsk8iccs\_64.so /lib/

ln -s /opt/tivoli/tsm/client/lang/EN\_US /opt/tivoli/tsm/client/ba/bin/

  

Now you should be ready to set up TSM config files and proceed normally.

Configure your dsm.opt & dsm.sys + your scheduler and so forth – then you are

  

Modify /opt/tivoli/tsm/client/ba/bin/dsm.sys.smp and save as dsm.sys same for dsm.opt.smp remember to drop the smp

  

Dsm.sys

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\* Tivoli Storage Manager                                               \*

\*                                                                      \*

\* Sample Client System Options file for UNIX (dsm.sys.smp)             \*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

  

\*  This file contains the minimum options required to get started

\*  using TSM.  Copy dsm.sys.smp to dsm.sys.  In the dsm.sys file,

\*  enter the appropriate values for each option listed below and

\*  remove the leading asterisk (\*) for each one.

  

\*  If your client node communicates with multiple TSM servers, be

\*  sure to add a stanza, beginning with the SERVERNAME option, for

\*  each additional server.

  

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

  

SErvername  Site-A

   COMMMethod         TCPip

   TCPPort            1500

   TCPServeraddress   172.16.52.165

  

  

Dsm.opt

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\* Tivoli Storage Manager                                               \*

\*                                                                      \*

\* Sample Client User Options file for UNIX (dsm.opt.smp)               \*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

  

\*  This file contains an option you can use to specify the TSM

\*  server to contact if more than one is defined in your client

\*  system options file (dsm.sys).  Copy dsm.opt.smp to dsm.opt.

\*  If you enter a server name for the option below, remove the

\*  leading asterisk (\*).

  

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

  

SErvername      Site-A

  

\* A server name defined in the dsm.sys file

  

  

  

[3.0 Administration using firewall builder](https://www.blogger.com/null)
=========================================================================

  

Firstly install firewall builder 5.1.

  

You can find the policy at [\\\\nasserver2\\techsupport\\Network Infrastructure\\Inline Firewalls\\Policy CDS.fwb](file://nasserver2/techsupport/Network%20Infrastructure/Inline%20Firewalls/Policy%20CDS.fwb)

  

  

  

Once you have opened FWBuilder and have opened the policy (see Figure 1 - Firewall Builder GUI), you can begin to edit the rules by going to Clusters > UFW > Policy. If your familiar with checkpoint then you will be very comfortable with this interface.

Once you’ve finished editing you then need to push policy, go and click the ’Install’ icon.

When you click next it will begin compiling the rules, if no errors are detected then you can move on, compiling can take upto 5 mins. Click next once finished.

You will then be prompted for the password, for each firewall in turn. Enter the details and click Install. Note if you get the password wrong you will not be notified and it will sit there doing nothing. The process of deploying the files only takes a minute.

  

  

![](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image002.gif)

[](https://www.blogger.com/null)[Figure](https://www.blogger.com/null) 1 - Firewall Builder GUI

  

![](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image004.gif)

[Figure](https://www.blogger.com/null) 2 - Push Policy Step1

  

![](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image006.gif)

[Figure](https://www.blogger.com/null) 3 - Push Policy Step2 Compiling Rules

  

  

  

  

![](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image008.gif)

  

[Figure](https://www.blogger.com/null) 4 - Policy Push Step3 Deploying Files