---
title: 'Debugging Lsass.exe'
date: 2016-11-21T23:02:00.001Z
draft: false
aliases: [ "/2016/11/debugging-lsassexe.html" ]
parent: Blogger
---
#### date: 2016-11-21


You need two host for this one!

I copied over from my wdk 8.1 install from C:\\Program Files (x86)\\Windows Kits\\8.1\\Debuggers\\x64 to remote host c:\\x64  

on remote host run

> C:\\Users\\Administrator>cd c:\\x64  
> c:\\x64>dbgsrv.exe -t tcp:port=1234,password=letmeinlol

on local host run

> C:\\Program Files (x86)\\Windows Kits\\8.1\\Debuggers\\x64>windbg.exe -premote tcp:server=192.168.56.10,port=1234,password=letmeinlol -pn lsass.exe -y "C:\\Users\\user\\AppData\\Local\\Temp\\SymbolCache"

>   
> .sympath  
> .reload

When you want to quit – make sure you use ‘qd’ to  ‘quit and detach’ so you don’t kill LSA  on the target machine.

to set a breakpoint

> bp PasswordFilter!DllMain

to show breakpoints

> bl

to continue

> g