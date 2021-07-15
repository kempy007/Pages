---
title: 'undefined'
date: 2016-03-01T19:06:00.003Z
draft: false
aliases: [ "/2016/03/cuckoo-2.html" ]
parent: Blogger
---
#### date: 2016-03-01

Cuckoo 2.0 (Server virtualised) – Platform Vmware ESXi  
  

Date updated  23/02/2016

Notice this is not my own work but has been updated by myself to make it work with newer versions, original sources have been referenced at end of this document.

  

  

Cuckoo is a sandbox for automated Malware Analysis. The idea of a malware sandbox is to have a collection of virtual machines that you can setup up, run malware and when your finished reset to a clean state so you can start again without mixing samples.

Cuckoo takes it one step further and generates reports based on Dynamic and some static behaviours the malware exhibited while it was in the sandbox.

This installation process is specifically to install Cuckoo Sandbox as a virtual machine in to an ESXi hypervisor environment. The Analysis machines are network connected, but instead could use inetsim to emulate internet access, we have flexibility.

For the most part the Cuckoo Guides provided by Cuckoo are great and if your doing a standard install this should be enough to get you going. There are several steps missing from these guides if your installing in to ESXi.

The Host OS is an Ubuntu Server 14.04 Virtual Machine running on the ESXi. For my network i have given the VM 3 Network Interfaces. The configuration of these is covered in the Network Section.

We need to create another vswitch which we connect the victim VM's to and traffic is routed out via our Cuckoo Server VM because it bridges the default vswitch which has physical connectivity to the internet and the new sandbox vswitch.

**Network Components.**

Getting the network configuration correct is one of the most important aspects, if Cuckoo cant speak to the analysis machines then nothing is going to work.

As i mentioned earlier i have 3 network cards on my Cuckoo Controller as detailed here.

eth0 – This connects to my home environment and allows me to access the Cuckoo Interface. It is statically assigned to my 192.168.1.0 network.

eth1 – Connects to the Host Only Network in Promiscuous mode.

eth2 – Connects to a Host Only Network with all the Analysis VMs. It is statically assigned to my 192.168.100.0 network.

The following is my /etc/network/interfaces

\# This file describes the network interfaces available on your system

\# and how to activate them. For more information, see interfaces(5).

  

\# The loopback network interface

auto lo

iface lo inet loopback

  

\# The primary network interface

auto eth0

iface eth0 inet static

   address 192.168.1.103

   netmask 255.255.255.0

   gateway 192.168.1.1

   dns-nameservers 192.168.1.1 8.8.8.8

  

\# The Monitor network interface

auto eth1

iface eth1 inet manual

   up ip address add 0/0 dev $IFACE

   up ip link set $IFACE up

   up ip link set $IFACE promisc on

down ip link set $IFACE promisc off

down ip link set $IFACE down

  

\# The Analysis network interface

auto eth2

iface eth2 inet static

address 192.168.100.254

netmask 255.255.255.0

The final step is to add a line to /etc/rc.local to ensure our nic enters promisc mode on reboot.

\# By default this script does nothing.

ifconfig eth1 up

ifconfig eth1 promisc

exit 0

Enable non root access to tcpdump with the following command.

1. sudo setcap cap\_net\_raw,cap\_net\_admin=eip /usr/sbin/tcpdump

**Update the OS**

1. sudo apt-get update

2. sudo apt-get upgrade

3. sudo apt-get install ssh

4. shutdown now -r

  

**Core Dependencies**

Grab our core dependencies. This is all one long line, make sure you copy to the end.

1. sudo apt-get install python-sqlalchemy python-bson python-dpkt python-jinja2 python-magic python-pymongo python-gridfs python-libvirt python-bottle python-pefile python-chardet python-django

  

**Optional Elements**

These steps install Yara and SSDeep. They are optional but there’s no reason not to add them.

1. sudo apt-get install build-essential python-dev python-pip git automake libtool

https://github.com/plusvic/yara/archive/v3.1.0.tar.gz- Must be 2x or greater

1. tar zxf v3.1.0.tar.gz

2. cd yara-3.1.0/

3. bash build.sh

4. sudo make install

Now lets do the python bindings.

1. cd yara-python/

2. sudo python setup.py install

http://ssdeep.sourceforge.net/#download

grab the latest version and away we go. Tested with version 2.11.1

1. tar zxf ssdeep-2.11.1.tar.gz

2. cd ssdeep-2.11.1/

3. ./configure && make

4. sudo make install

Lets install the python bindings to go with it.

1. sudo pip install pydeep

**Libvert – ESXi**

This is the major missing point from the Cuckoo Docs, we need to build libvert and tell it to include ESXi support.

1. sudo apt-get install libpciaccess-dev libnl-dev pkg-config libxml2-dev libgnutls-dev libdevmapper-dev libcurl4-gnutls-dev w3c-dtd-xhtml

2. wget http://libvirt.org/sources/libvirt-1.3.1.tar.gz

3. tar zxf libvirt-1.3.1.tar.gz libvirt-1.3.1

4. cd libvirt-1.3.1/

5. ./configure --with-esx=yes

6. make

7. sudo make install

That should no longer completes without errors.(do ignore html .tmp errors)

\*\*\*build failed, had to >#  apt-get install w3c-dtd-xhtml

\>#  make clean

\># make 

#still fails though but ignore and continue virsh and libvirtd report 1.3.1

I somehow installed one from the repository, so had to remove it before runing make install, but after it would report version 1.3.1

  

**MySQl**

Cuckoo is capable of running multiple analysis tasks concurrently, this is not supported if your using the default SQLite Database options. We are going to install MySql to add this support.

1. sudo apt-get install mysql-server

2. sudo apt-get install python-mysqldb

3. sudo mysql\_install\_db

4. sudo mysql\_secure\_installation

After command 4.  Say no to changing the root password and YES to all other questions.

1. mysql -u root -p

2. **CREATE**  **DATABASE**cuckoo;

3. **GRANT**  ALL  **ON**cuckoo.\* **TO**  'cuckoo'@'localhost'IDENTIFIED **BY**  'newpassword';

4. FLUSH PRIVLIGES;

5. exit

Please don't use newpassword change it!

**Cuckoo**

Now lets grab the latest version of cuckoo

1. git clone https://github.com/cuckoobox/cuckoo

Before we can run cuckoo we need to make the configuration changes for our environment.

**conf/cuckoo.conf**

Line 20  machinery = esx

Line 62  ip = 192.168.0.210 #IP of vmware host

Line 82  resolve\_dns = on # Change as suits your needs.

Line 91 connection = mysql://cuckoo:newpassword@localhost/cuckoo # The same password you created in the mysql setup.

**conf/auxiliary.conf**

line 11 interface = eth1

conf/reporting.conf

Find \[mongodb\] and set

enabled = yes

**conf/esx.conf**

This config file is where we declare all our analysis Virtual Machines that cuckoo can use. The configuration of these Analysis machines is detailed later on.

The main section is where we setup our connection these are the details for the ESXi host NOT the cuckoo controller.

dsn = esx://127.0.0.1/?no\_verify=1  
username = username\_goes\_here  
password = password\_goes\_here

machines = analysis1

machines = CuckooVM1

interface = eth1

\[CuckooVM1\]

label = CuckooVM1

platform = windows

snapshot = name\_of\_snapshot

IP = 192.168.100.1

The machines line is a , seperated list of machines that are available for cuckoo to use. Each machine listed here must have its own config section.

The config sections contain a lot of comments to help you understand the sections. Here is an example of the relevant fields with the comments removed.

machines = Win7Adobe9, Win7Adobe10, Win7Adobe11

\[Win7Adobe9\]

label = Win7\_1

platform = windows

snapshot = SnapShotName

ip = 10.10.10.3

tags = Win7,Office07,Adobe9.1

  

\[Win7Adobe10\]

label = Win7\_2

platform = windows

snapshot = SnapShotName

ip = 10.10.10.4

tags = Win7,Office07,Adobe10.0

  

\[Win7Adobe11\]

label = Win7\_2

platform = windows

snapshot = SnapShotName

ip = 10.10.10.5

tags = Win7,Office07,Adobe11

**Community Modules**

There are several community contributions that are useful to have so we want to grab these as well. Some of the community modules can be noisy so remove them as you see fit. they can be found under /modules/signatures

Please update the community modules else you'll get errors when you start cuckoo.

1. cd utils

2. ./community.py -af

**Django Web Interface (installed packages but skipped this section)**

Cuckoo comes with two web interfaces. A basic Bottle interface and a more feature full Django interface. This guide will be using the latter of these options.

1. sudo apt-get install mongodb python-django

At the time of writing Cuckoo does not support Django versions > 1.5 the fix is easy to implelemt yourself and i prefer to run the latest version.

You only need to make the change if you see the following error when running.

"Please fix your settings." % setting)

django.core.exceptions.ImproperlyConfigured: The TEMPLATE\_DIRS setting must be a tuple. Please fix your settings.

edit settings.py and around line 102 Change:

TEMPLATE\_DIRS = (

"templates"

)

To:

TEMPLATE\_DIRS = (

("templates"),

)

We add the Cuckko Web and Cuckko API to upstart jobs, this means that they will autostart with the OS. You will loose visible logging on this

We could add the cuckkoo main thread as an upstart job as well but this could be problemtic when debugging issues.

As an example  
add this content to \\etc\\init\\cuckooapi.conf

description "Cuckoo Web API Service"

author "Kevin Breen @kevthehermit"

  

start on runlevel \[234\]

stop on runlevel \[0156\]

  

setuid thehermit

setgid thehermit

  

chdir /home/thehermit/cuckoo/utils

exec python api.py -H 0.0.0.0 -p 5556

respawn

add this content to \\etc\\init\\cuckooweb.conf

description "Cuckoo Web API Service"

author "Kevin Breen @kevthehermit"

start on runlevel \[234\]

stop on runlevel \[0156\]

setuid thehermit

setgid thehermit

chdir /home/thehermit/cuckoo/utils

exec python api.py -H 0.0.0.0 -p 5556

respawn

And dont forget to replace the uid and gid with your own. DO NOT LEAVE EMPTY AND DO NOT USE ROOT

And change the chdir line as required. -H 0.0.0.0 says listen on any IP address i have configured. If you use localhost no one external can use the api or web interface

```
pip install -r requirements.txt
``````
pip install python-_**dateutil**_
```

pip install pycrypto distorm3

apt-get install python-libvirt

git clone https://github.com/volatilityfoundation/volatility.git

cd volatility

make

make install

  

  

 Below is for reference only, it highlights the need to use updated libvirtd package and to avoid use of vmware express, it does work with esxi 6.
--------------------------------------------------------------------------------------------------------------------------------------------------

  

Cuckoo is an open-source malware analysis tool which utilizes sandbox technology. It allows you to break down malicious programs in not only isolated, but also controlled environment while giving the ability to gather artefacts to document your findings.

Generally, most Cuckoo installation guides are written for VirtualBox. However, there is also a way to utilize ESX driver to deploy Cuckoo host on ESX infrastructure. This enables you to have your Malware hosts in an isolated network on an ESXi infrastructure.

I’m going to point you to the best guide that I found to accomplish this and point out some gaps that you need to be aware of. This article is from TechAnarchy – https://techanarchy.net/lab/cuckoo-esxi/

Gaps in article:

1.  Libvert installation – Libvert is the driver used by Cuckoo to connect to hosts on ESXi or VirtualBox. There is special configuration required to enable ESXi driver within the article. But be aware that old Libvert 1.2.7 that was used in the article is outdated. You would need to download libvert 1.2.14 using: wget http://libvirt.org/sources/libvirt-1.2.7.tar.gz
2.  Second gap is that you will get an error message if you are using ESXi unlimited free license. That is because_ vSphere API functionality is not_ supported. I discovered this the hard way through insane troubleshooting and packet chasing.  
    This is required to allow Cuckoo to natively control the Malware hosts for analysis.The error message should look like this:

3. root@cuckoo:/etc/cuckoo# ./cuckoo.py\_

4. \_\_\_\_ \_ \_ \_\_\_\_| | \_ \_\_\_ \_\_\_

5. / \_\_\_) | | |/ \_\_\_) |\_/ ) \_ \\ / \_ \\

6. ( (\_\_\_| |\_| ( (\_\_\_| \_ ( |\_| | |\_| |

7. \\\_\_\_\_)\_\_\_\_/ \\\_\_\_\_)\_| \\\_)\_\_\_/ \\\_\_\_/

8. 

9. Cuckoo Sandbox 1.3-dev

10. www.cuckoosandbox.org

11. Copyright (c) 2010-2015

12. 

13. Checking for updates...

14. Good! You have the latest version available.

15. 

16. 2015-07-14 13:13:21,635 \[lib.cuckoo.core.scheduler\] INFO: Using "esx" machine manager

17. libvirt: ESX Driver error : internal error: HTTP response code 500 for call to 'PowerOffVM\_Task'. Fault: ServerFaultCode - Current license or ESXi version prohibits execution of the requested operation.

**2015-07-14 13:13:23,749 \[root\] CRITICAL: CuckooCriticalError: Please update your configuration. Unable to shut 'Windows10-Malware' down or find the machine in its proper state: Error stopping virtual machine Windows10-Malware: internal error: HTTP response code 500 for call to 'PowerOffVM\_Task'. Fault: ServerFaultCode - Current license or ESXi version prohibits execution of the requested operation.**

18.  Root cause of this problem is that ESX6 free license does not support vSphere API. The reason for this limitation in license is unknown but the issue initially surfaced in ESX5 and eventually fixed through patches. However, ESX6 seem to be plagued with the same problem that would prevent many developers from utilizing vSphere API interface.  
    

**Configure Cuckoo**

1.  Configure 2 Port Groups

1.  One for access to local network and Internet
2.  Isolated Port Group

3.  Configure Cuckoo on Ubuntu
4.  Assign it 3 interfaces to Cuckoo

1.  Interface 1 – To your Local Network and for access to the Internet
2.  Interface 2 should be to the isolated network – this will be configured in promiscuous mode (not ESXi promiscuous – this will be configured on the Linux server)
3.  Interface 3 will be to the isolated port group and assigned IP address

**Configure the Malware Host**

1.  Install Windows 10 Host on ESX
2.  Assign one interface to the host
3.  Assign an IP Address to the interface

The end result should look like this:

root@cuckoo:/etc/cuckoo# ./cuckoo.py

  

.:

::

.-. , : .-. ;;.-. .-. .-.

; ; ; ; ;; .' ; ;'; ;'

\`;;;;'.'\`..:;.\_\`;;;;'\_.'\` \`.\`;;' \`;;'

  

Cuckoo Sandbox 1.3-dev

www.cuckoosandbox.org

Copyright (c) 2010-2015

  

Checking for updates...

Good! You have the latest version available.

  

**2015-07-14 06:42:12,336 \[lib.cuckoo.core.scheduler\] INFO: Using "esx" machine manager**

**2015-07-14 06:42:16,105 \[lib.cuckoo.core.scheduler\] INFO: Loaded 1 machine/s**

**2015-07-14 06:42:16,108 \[lib.cuckoo.core.scheduler\] INFO: Waiting for analysis tasks.**

  

Sources//>

https://techanarchy.net/lab/cuckoo-esxi/

http://www.apexbits.net/integrating-cuckoo-with-esxi-6/

  
  

http://docs.cuckoosandbox.org/en/latest/installation