---
title: 'Linux Clustering of Web Application Firewall'
date: 2014-08-29T14:54:00.002+01:00
draft: false
aliases: [ "/2014/08/linux-clustering-of-web-application.html" ]
parent: Blogger
---
#### date: 2014-08-29

[WAF Conversion to cluster](https://www.blogger.com/null)
=========================================================

Please bear in mind that # is the cli prompt and do not include ---bof--- and ---eof--- in your configs.

  

clone off vm and set network to disconnected, then start.

  

change the hostname with x being the node number

\# nano /etc/hostname

                WAFCNx

  

add under '127.0.0.1  localhost'

\# nano /etc/hosts

                192.168.10.218        WAFCN1

                192.168.10.219        WAFCN2

  

\# nano /etc/network/interfaces

update the IP.

  

reboot node, then check settings like IP etc.

enable and connect virtual nic.

  

at this point it is better to ssh onto the box rather than use vmware console.

  

nano /etc/apache2/mod-enabled/status.conf

find the line '#  allow from 192.0.2.0/24' and change to ' allow from 192.168.10.216/29'

  

\# apt-get install heartbeat pacemaker wget

  

\# nano /etc/cron.daily/clean-archived-logs

  

\--- bof -----

#!/bin/bash

cd /var/log

rm \*.gz

\---eof -----

  

\# chmod 777 /etc/cron.daily/clean-archived-logs

  

\# nano /etc/ha.d/ha.cf

  

\--- bof -----

#debugfile             /var/log/ha-debug

logfile                     /var/log/ha-log

logfacility              local0

keepalive               2

deadtime               30

warntime               10

initdead 120

udpport                  694

\# IP address of the other node (change it in every node)

ucast                      eth1        172.16.1.21x

#Tell what nodes are in the cluster, must match uname -n

node WAFCN1 WAFCN2

#Enable pacemaker

crm respawn

\---- eof -----

  

\# nano /etc/ha.d/authkeys

  

\---bof----

auth 1

1 crc

\---eof ----

  

\# chmod 600 /etc/ha.d/authkeys

  

\# service heartbeat restart

  

only need to run crm commands on a single node once cluster has had time to communicate, check with 

\# crm status

  

\# crm configure property stonith-enabled=false

\# crm configure property expected-quorum-votes="2"

\# crm configure property no-quorum-policy=ignore

  

Adding our virtual IP's here

\# crm configure primitive VIP61-www-site1-co-uk ocf:IPaddr2 params ip=192.168.10.61 cidr\_netmask=32 nic=eth0 op monitor interval=15s

\# crm configure primitive VIP59-wildcard-site2-co-uk ocf:IPaddr2 params ip=192.168.10.59 cidr\_netmask=32 nic=eth0 op monitor interval=15s

\# crm configure primitive VIP58-www-site3-co-uk ocf:IPaddr2 params ip=192.160.10.58 cidr\_netmask=32 nic=eth0 op monitor interval=15s

  

Adding our service

\# crm configure primitive SRV-apache-rproxy-dotDefender lsb::apache2 op monitor interval=15s

  

Binding our VIP to the Service

\# crm configure colocation SRV-apache-rproxy-dotDefender-VIP61 INFINITY: VIP61-www-site1-co-uk SRV-apache-rproxy-dotDefender

\# crm configure colocation SRV-apache-rproxy-dotDefender-VIP59 INFINITY: VIP59-wildcard-site2-co-uk SRV-apache-rproxy-dotDefender

\# crm configure colocation SRV-apache-rproxy-dotDefender-VIP58 INFINITY: VIP58-www-site3-co-uk SRV-apache-rproxy-dotDefender

  

Configure service startup order, ensure VIP's are started first

\# crm configure order ip-apache mandatory: VIP58-www-site3-co-uk VIP59-wildcard-site2-co-uk VIP61-www-site1-co-uk SRV-apache-rproxy-dotDefender

  

  

setup subversion

\# apt-get install subversion

  

Somescript i wrote using svn, to get stuff into svn run # svn import --username Some.Admin sourceDir  DestinationServer

you then need to checkout the folder before you can commit changes.

  

cat checkoutApacheConf.sh

#!/bin/bash

svn co --username Some.Admin --force https://vm-svn.somecompany.local/svn/Infrastructure/0WebApplicationFirewall/apache2/@head /etc/apache2/

  

  

 cat commitApacheConf.sh

#!/bin/bash

svn commit /etc/apache2/