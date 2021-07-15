---
title: 'Cuckoo Sandbox–BareMetal with SAN and IPMI'
date: 2016-06-19T21:25:00.001+01:00
draft: false
aliases: [ "/2016/06/cuckoo-sandboxbaremetal-with-san-and.html" ]
parent: Blogger
---
#### date: 2016-06-19

Ever since the stories of VM aware malware hit the news, I started thinking about how I could move from the virtual environment to the physical. I started playing with cuckoo back in 2014 and it helped when other online tools were down, or failed to give a correct verdict and classed it benign( eg I knew it was malware as it had trashed a users machine). It’s taken me too long to knuckle down and experiment with this mainly because of time but partly because I did not have the hardware (got some Sept 2015).

For the SAN I decided to use solaris because it has support for FC cards in target mode, bolt on Napp-It as a nice webgui to manage the storage on a surplus HP DL380 with 25 SFF HDD bays fully populated with 73gb drives and i’m happy as Larry.

I also acquired two Dell 2950’s with 8 cores and 32gb ram, used a usb stick to install vmware onto one host, setup a new VM with cuckoo sandbox configured, hacked a new machinery file and associated .conf file, by the way I had no idea how to implement this before I started, I had experience with wake on lan before, however the servers did not work with this, so I turned to the lights out management cards, I had IPMI down as a possible candidate but was expecting quite a fight to get working. turned out to be so trivial I was shocked, just install ipmitool on linux figure out the right commands and your in business for startup and shutdown. It took me a while to figure out how to snapshot and rollback on the Solaris host, but in the end it too seems trivial. Then it’s a case of setting the 2nd host to boot from SAN, install windows and configure for insecurity and monitoring (yeah we have to do opposite and make insecure OS for sandbox analysis lol)

I have tried twice using camstudio to record this setup in action however it appears to have corrupted itself, unsure why, I was able to rescue this screenshot;

[![Take2SS](https://lh3.googleusercontent.com/-MhuqutGXYZc/V2b_wvUYAQI/AAAAAAAAACs/Pa77_geSUtY/Take2SS_thumb%25255B1%25255D.jpg?imgmax=800 "Take2SS")](https://lh3.googleusercontent.com/-Ack3X6oluOo/V2b_wFCsXOI/AAAAAAAAACk/rIuyN9Ly4Ww/s1600-h/Take2SS%25255B3%25255D.jpg)

I still have to draw a diagram and write a document detailing howto replicate my efforts. I wonder what the estimate is for the FOG server based pysical machinery…

Anyways that’s all I have to quickly say for now.

Updated 20/06/2016
==================

Finally made a video old school way and subtitled it (took forever);

CuckooSB with IPMI + SAN

**Updated 23/06/2016**

[![BareMetalConcept](https://lh3.googleusercontent.com/-qgcSt8Cbqvs/V2xLzrw7xxI/AAAAAAAAADE/SD6tONr8tFQ/BareMetalConcept_thumb%25255B2%25255D.jpg?imgmax=800 "BareMetalConcept")](https://lh3.googleusercontent.com/-JoWnhpsqNFw/V2xLy05HjfI/AAAAAAAAAC8/-EbhcN_-2D0/s1600-h/BareMetalConcept%25255B4%25255D.jpg)