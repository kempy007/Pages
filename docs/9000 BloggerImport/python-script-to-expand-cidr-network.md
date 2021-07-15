---
title: 'Python script to expand CIDR network list to full IP range for mapping intangible data against vulnerabilities.'
date: 2016-03-22T17:43:00.001Z
draft: false
aliases: [ "/2016/03/python-script-to-expand-cidr-network.html" ]
parent: Blogger
---
#### date: 2016-03-22

\# use python 3.4

\# purpose of this file is to write out IP address list for mapping intangible data against  
\# vulnerabilaties from a CIDR list of your networks.  
import sys  
if not sys.version\_info\[:2\] >= (3, 4):  
    print("Ensure you use python 3.4") #todo add check for env  
    sys.exit(1)  
     
print("Expected input format should be .csv with columns in following order;")  
print("CIDR, ZONE, DEPARTMENT, FUNCTION")  
print("")  
import ipaddress  
import unittest, time, re, os  
import csv

\# Set test variables here  
netblock = "192.168.0.0/24" # only takes cidr notation, cannot use range  
dept = "State"  
zone = "external"  
nomFunction = "servers"  
write2csvPath = "z:\\output.csv"  
read2csvPath = "z:\\TestInput.csv"

\# Check for files we don't want to trip over  
if os.path.isfile(write2csvPath):  
            os.remove(write2csvPath)  
\# set read and write files  
cw = csv.writer(open(write2csvPath, "wt", newline="\\n", encoding="utf-8" )) # had empty lines with newline & encoding  
cr = csv.reader(open(read2csvPath, "rt"))  
\# write out our header here  
#print('IP, CIDR, ZONE, DEPARTMENT, FUNCTION, IPRANGESIZE, NETWORKADDR, BROADCASTADDR, NETMASK, HOSTMASK, FIRSTIP, LASTIP, USEABLEHOSTS')  
cw.writerow(\['IP', 'CIDR', 'ZONE', 'DEPARTMENT', 'FUNCTION', 'IPRANGESIZE', 'NETWORKADDR', 'BROADCASTADDR', 'IPRANGE', 'NETMASK', 'HOSTMASK', 'FIRSTIP', 'LASTIP', 'USEABLEHOSTS'\])

#Begin reading in master seed list file  
for row in cr:  
    print (row\[0\], row\[1\], row\[2\], row\[3\])  
    if row\[0\] == 'CIDR':  
        print('header found')  
    else:  
        netblock = row\[0\]  
        zone = row\[1\]  
        dept = row\[2\]  
        nomFunction = row\[3\]

        tmpNet = ipaddress.ip\_network(netblock, strict=False) # got an errors on some lines with strict setting 'error has host bits set'

        IPRangeSize = tmpNet.num\_addresses  
        networkAddr = tmpNet.network\_address  
        BroadcastAddr = tmpNet.broadcast\_address  
        NetMask = tmpNet.netmask  
        HostMask = tmpNet.hostmask

        FirstIP = None  
        LastIP = None  
        HostArray = \[\] # for useable hosts  
        if IPRangeSize > 2: # /32 pointless adding to array as is /31 as it's not useable  
            TmpHostArray = tmpNet.hosts()  
            for b in TmpHostArray:  
                #print(b)  
                HostArray.append(b) # /32 will not populate array and useableHost = 0  
        useableHost = 0  
        if NetMask == ipaddress.IPv4Address('255.255.255.255'): # handle /32 differently  
            useableHost = 1  
            FirstIP = networkAddr  
            LastIP = networkAddr  
        else:  
            useableHost = HostArray.\_\_len\_\_()  
            FirstIP = HostArray\[0\]  
            LastIP = HostArray\[useableHost-1\]

        for a in tmpNet: # write out every IP for network  
            cw.writerow(\[str(a), netblock, dept, zone, nomFunction, IPRangeSize, networkAddr, BroadcastAddr, str(networkAddr)+'-'+str(BroadcastAddr), NetMask, HostMask, FirstIP, LastIP, useableHost\])

print('End')