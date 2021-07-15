---
title: 'Buspirate arduino where usbtinyisp will no longer suffice'
date: 2016-03-27T23:17:00.001+01:00
draft: false
aliases: [ "/2016/03/buspirate-arduino-where-usbtinyisp-will.html" ]
parent: Blogger
---
#### date: 2016-03-27

Having acquired a sparkfun buspirate recently to check out a router that no longer works, which is still a brick by the way...

I’ve turned my attention back towards my 3d printers and I've a little problem in that I'm unsure which firmware and which settings I compiled and uploaded. So I have a plan which is to backup the firmware from my working device, hash the hex files and compare to the three or four other options that I may have flashed to it.

I have some spare boards that I will be testing methods against first before I dare upset the printer. When I started my adventure into the painful world of 3d printing, the chipsets were at mega128 and 644p. the usb tiny isp was quite happy to burn the boot loaders for these fella's. but today we have more than 64k to play with and so I need something else.

Thankfully I can leverage my new buspirate to talk SPI. One pain in the arse is that the sparkfun edition has its colour theme backwards. Therefore I will have the details in this blog for my own future reference.

GND Black

+3.3v White

+5v Grey

ADC Purple

VPU Blue

AUX Green

CLK Yellow

MOSI Orange

CS Red

MISO Brown

So by hooking up the bus pirate to the arduino 6 pin header like below;

Black/GND > GND Bottom Left

Red/CS > Reset Top Left

Orange/Mosi > Mosi Bottom Middle

Brown/Miso > Miso Top Right

Yellow/SCL|CLK > SCK Top Middle

I power the board by its USB.

I change directory to c:\\arduino-1.0.1\\hardware\\tools\\avr

and then run this command;

bin\\avrdude.exe -C etc\\avrdude.conf -p m1284px -c buspirate -P com11

because for some reason my board has a different ID, I edited my conf file by copying the existing m1284p section and tweak to my boards ID. So now to read the from the device I use the following command;

bin\\avrdude.exe -C etc\\avrdude.conf -p m1284px -c buspirate -P com11 -U flash:r:c:\\factory\_sanguino.hex:i

Its slow, like half an hour slow... 70minutes actually, that's painful, and i never got the file so something was wrong with the path. perhaps try %temp% in future i suspect the : maybe cause.

In addition to flash after the -U switch, there is also eeprom, hfuse, lfuse and efuse data that can be read, the next section 'r' can be changed for w to write, the next section is the path to the file, and the 'i' I'm not sure about. arduino ide 1.05 seems to have buspirate support builtin.

Ref: [http://forum.arduino.cc/index.php?topic=182849.0](http://forum.arduino.cc/index.php?topic=182849.0)

Update… no make sure to update all software and firmware. Well the buspirate firmware was on 5.1 and it’s now at 6.1, but that made no difference to the speed using avrdude in the arduino ide 1.01 package. So I updated to arduino ide 1.6.8 and now it takes 154seconds to read the flash.

Ref: [https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/dangerous-prototypes-open-hardware/BusPirate.package.v6.1.zip](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/dangerous-prototypes-open-hardware/BusPirate.package.v6.1.zip "https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/dangerous-prototypes-open-hardware/BusPirate.package.v6.1.zip")

Just pull out relevant package, my hardware is v3 so I extracted BPv3-firmware. Console into the buspirate and put into bootloader mode using dollar symbol $, close the console and run BPv3-update-v6.1.bat.

Ref: [https://www.arduino.cc/download\_handler.php](https://www.arduino.cc/download_handler.php "https://www.arduino.cc/download_handler.php")

So back to the problem with my sanguinololu boards (I wouldn’t recommend these boards anymore because support seems to be non existant past arduino ide 1.01 buy ramps kit instead) So thanks to the post at [http://www.dragonflydiy.com/p/installing-arduino-software.html](http://www.dragonflydiy.com/p/installing-arduino-software.html "http://www.dragonflydiy.com/p/installing-arduino-software.html") which linked me to [https://github.com/jmgiacalone/sanguino1284p](https://github.com/jmgiacalone/sanguino1284p "https://github.com/jmgiacalone/sanguino1284p") So I can grab the bootloader hex file, the post here [http://reprap.org/wiki/Gen7\_Arduino\_IDE\_Support](http://reprap.org/wiki/Gen7_Arduino_IDE_Support "http://reprap.org/wiki/Gen7_Arduino_IDE_Support") show how to use avrdude on the cli to write the fuses and bootloader.

I’m still getting an error when attempting to backup these devices, so i’m going to continue to the writing stage to see if it helps.

> c:\\arduino-1.6.8\\hardware\\tools\\avr>bin\\avrdude.exe -C etc\\avrdude.conf -p m1284p -c buspirate -P com11 -U flash:r:backup\_sanguino.hex:i
> 
> Attempting to initiate BusPirate binary mode...  
> avrdude.exe: AVR device initialized and ready to accept instructions
> 
> Reading | ################################################## | 100% 0.09s
> 
> avrdude.exe: Device signature = 0x1e9705  
> avrdude.exe: reading flash memory:
> 
> Reading | ################################################## | 99% 154.10sBusPirate: Paged Read command returned zero.  

The two spare sanguinololu boards are both surface mounted boards, however they both have different fuses.

> c:\\arduino-1.6.8\\hardware\\tools\\avr>bin\\avrdude.exe -C etc\\avrdude.conf -p m1284p -c buspirate -P com11 ?
> 
> Attempting to initiate BusPirate binary mode...  
> avrdude.exe: AVR device initialized and ready to accept instructions
> 
> Reading | ################################################## | 100% 0.09s
> 
> avrdude.exe: Device signature = 0x1e960a
> 
> avrdude.exe: safemode: Fuses OK (H:FD, E:DC, L:FF)
> 
> avrdude.exe done.  Thank you.
> 
>   
> c:\\arduino-1.6.8\\hardware\\tools\\avr>bin\\avrdude.exe -C etc\\avrdude.conf -p m1284p -c buspirate -P com11 ?
> 
> Attempting to initiate BusPirate binary mode...  
> avrdude.exe: AVR device initialized and ready to accept instructions
> 
> Reading | ################################################## | 100% 0.09s
> 
> avrdude.exe: Device signature = 0x1e9705
> 
> avrdude.exe: safemode: Fuses OK (H:FF, E:9A, L:FF)
> 
> avrdude.exe done.  Thank you.

Set fuses as

```
Low 0xD6
  
High 0xDC
  
Ext  0xFD
``````
\# write fuses
  
bin\\avrdude -C etc\\avrdude.conf –c buspirate -p m1284p -P com11 \\
  
   -B 5 -U lfuse:w:0xD6:m -U hfuse:w:0xDC:m -U efuse:w:0xFD:m
  

``````
\# upload bootloader from [https://github.com/maniacbug/mighty-1284p/blob/master/bootloaders/optiboot/optiboot\_atmega1284p.hex](https://github.com/maniacbug/mighty-1284p/blob/master/bootloaders/optiboot/optiboot_atmega1284p.hex "https://github.com/maniacbug/mighty-1284p/blob/master/bootloaders/optiboot/optiboot_atmega1284p.hex")
  
bin\\avrdude -C etc\\avrdude.conf -c buspirate -p m1284p -P com11 \\
  
    -B 1 -U flash:w:optiboot\_atmega1284p.hex 
``````
\# lock the bootloader (omitted this part)
  
\# this gives an expected "verification error 0xcf != 0x0f"
  
bin\\avrdude -C etc\\avrdude.conf -c buspirate -p m1284p -P com11 \\
  
    -B 1 -U lock:w:0xCF:m 
  

``````
Seems that did not help, still get paged read zero. frustrating given that I can write a firmware and verify it ok, but not just plain read the flash, seems like a bug to me. Guess that means I cannot verify what is flashed to a device. I suspect the board with the bad signature is bricked.
``````
Well after spending all day it seems the solution is simple, compile what you think is installed on the device and use the verify option.
```

c:\\arduino-1.6.8\\hardware\\tools\\avr>bin\\avrdude -C etc\\avrdude.conf -p m1284p -P com11 -U flash:v:Sprinter.hex:i -c buspirate

Well thats enough meandering for today.