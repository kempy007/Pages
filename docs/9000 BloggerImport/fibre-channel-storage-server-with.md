---
title: 'Fibre Channel Storage Server with Solaris 11.3 and napp-it'
date: 2015-09-04T23:18:00.002+01:00
draft: false
aliases: [ "/2015/09/fibre-channel-storage-server-with.html" ]
parent: Blogger
---
#### date: 2015-09-04

Wanting to do many things in life I've finally got around to building my home server lab, and one of the things I wanted out of this is to setup clusters of which require shared storage, and also to play with boot from san with snapshot support.  
  
Given that server hardware can be had for around £100 a server for a xeon quad with 32gb ram and maybe a hard drive or two, i've chosen to grab a few bargains.  
Overall I have splashed out approx 580 to nab 2 compute nodes with 32gb ram and dual quad xenon chips with a 25bay HP fully populated with 73gb sas drives, of which the storage accounts for more than half the cost, but still 1.4tb on 25 spindles for £327 is a bargain.  
  
Anyway back onto the main topic, what software do I choose to give me the flexibility that I need at zero cost. having looked at openfiler and nexenta stor, both had paywall for FC support which put me off, sure I can hack it on but the point is it's a waste of time as these vendors want to force you onto the paid version.  
  
Thankfully Oracle Solaris 11.3 has FC support out of the box, with the addition of napp-it.org I can get a swish web interface to manage the storage just like the other distro's mentioned above.  
  
the built in documentation was actually quite useful and in no time I was able to reconfigure the drivers into target mode.  
  
I had split 3 drives off into a raid 5 for the os install, with the rest in mind for a large array to dole out file based luns, bu ti haven't settled on a plan as of yet.  
  
at the shell, # svcadm enable stmf  
(you may have to online-lu)  
  
within the napp-it webconsole I did  
  
Pools > create pool (selected hardware raid volume)  
  
Comstar > volumes > create volume (called it volume01 with 200gb)  
  
Comstar > Logical Units > create volume lu (used above volume01)  
  
Comstar > view > add view (used above lun, allowed any to connect)  
  
anyhow time to go i'm afraid.  
  
  
Update:14/07/2017 Added text only notes for Solaris only simple implementation below;  
  
#Install from latest .iso 11.3  
  
#following command may take a while  
pkg refresh  
  
#Configuring Fibre Channel Devices with COMSTAR  
pkg install --accept storage-server  
  
mdb -k  
::devbindings -q qlc (note pciex instance and driver qlc)  
$q  
update\_drv -d -i ‘pciex1077,2532′ qlc  
update\_drv -a -i ‘pciex1077,2532′ qlt  
reboot  
  
mdb -k  
::devbindings -q qlc (note driver is now qlt)  
$q  
  
svcadm enable stmf  
  
stmfadm list-target -v  
  
fcinfo hba-port (optional)  
  
zpool create datapool c0t0d0s0 c0t0d1s0  
  
zfs create -b 64k -V 10G datapool/fclun01  
sbdadm create-lu /dev/zvol/rdsk/datapool/fclun01  
  
sbdadm list-lu  
stmfadm list-lu -v  
stmfadm add-view 600144F0B152CC0000004B080F230004  
stmfadm list-view -l 600144F0B152CC0000004B080F230004  
  
  
other refs;  
http://www.oracle.com/technetwork/server-storage/solaris11/documentation/ips-one-liners-032011-337775.pdf  
http://www.oracle.com/technetwork/articles/servers-storage-admin/comstar-zfs-virtualized-storage-2159053.html  
https://docs.oracle.com/cd/E53394\_01/html/E54782/fnnoq.html  
http://www.c0t0d0s0.org/archives/6140-Less-known-Solaris-Features-iSCSI-with-COMSTAR.html  
http://thegeekdiary.com/solaris-zfs-command-line-reference-cheat-sheet/