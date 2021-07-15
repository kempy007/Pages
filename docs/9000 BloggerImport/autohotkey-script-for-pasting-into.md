---
title: 'AutoHotKey script for pasting into clipboard disabled sessions'
date: 2017-07-16T12:57:00.001+01:00
draft: false
aliases: [ "/2017/07/autohotkey-script-for-pasting-into.html" ]
parent: Blogger
---
#### date: 2017-07-16

```
; Author: MKEMP 26-04-17 
; Requires AutoHotKey V1.1 file must have .ahk extension, can be compiled into standalone exe ~1mb. 
; This script allows data on clipboard to be typed into RDP secure desktop where password manager fails. 
; It does this by emulating keyboard behaviour but works just like paste in apps that block paste. 
; Don't forget to either adjust the X sec timeout longer in your PW Manager or to ctrl+c the password again for the 2nd entry prompt in Secure RDP. 
; Ensure you change rdp > Options > local resources > keyboard > Apply Windows Key combinations = on this computer. 
; failure to do so will give effect of stuck shift key after first capital letter is entered and results in incorrect password entry and possible lockout. 
; double tap numlock to type out contents of clipboard after copying password from manager app. 
SetWinDelay 100 
SetKeyDelay 50 

~NumLock:: 
If (A_PriorHotKey = "~NumLock" AND A_TimeSincePriorHotkey < 500 AND A_TimeSincePriorHotkey > 50) 
{ 
Sleep, 100 
SendRaw %clipboard% 
} 
Return 
; End of AHK script.
```