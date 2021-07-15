---
title: 'USB AT-Tiny85 HID Keyboard trojan keystrokes or password generator'
date: 2016-06-04T19:23:00.000+01:00
draft: false
aliases: [ "/2016/06/usb-at-tiny85-hid-keyboard-trojan.html" ]
parent: Blogger
---
#### date: 2016-06-04

Having used various microchips for printers and other jobs and hearing about bad-usb attacks, I though it was time to have a go and create something of dual purpose,

A. to trojan a host computer (if anyone finds this key, i’ll phone home with my trojan keystrokes)

B. to generate and output passwords because i’m lazy (i’ll have a sequence to fire it off)

The article at [http://codeandlife.com/2012/03/03/diy-usb-password-generator/](http://codeandlife.com/2012/03/03/diy-usb-password-generator/ "http://codeandlife.com/2012/03/03/diy-usb-password-generator/") was a useful starter.

I ordered one of these from amazon [http://digistump.com/products/1](http://digistump.com/products/1 "http://digistump.com/products/1") 

This page was useful to get going, I did not understand how the process worked initially! [http://digistump.com/wiki/digispark/tutorials/connectingpro](http://digistump.com/wiki/digispark/tutorials/connectingpro "http://digistump.com/wiki/digispark/tutorials/connectingpro") and even after reading this I missed off installing the driver part. So download the digistump drivers and install, plug in the usb device let windows install the drivers, then unplug, load up arduino sketch, add in the extra boards from above wiki, then upload and follow instructions (eg plug in device when asked).

Not had any luck getting code to compile from the first link, the digispark example works fine though.