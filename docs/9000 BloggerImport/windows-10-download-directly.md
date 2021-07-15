---
title: 'Windows 10 download directly for when the setup stub won''t play nicely'
date: 2015-12-31T13:32:00.002Z
draft: false
aliases: [ "/2015/12/windows-10-download-directly.html" ]
parent: Blogger
---
#### date: 2015-12-31

Given that I was stuck behind a proxy and the windows 10 stub didn't like authenticating I was unable to get the ISO file.  
  
So I fired up wireshark, retried and then checked the logs for the get request.  
  
http://fg.ds.b1.download.windowsupdate.com/c/updt/2015/11/10586.0.151029-1700.th2\_release\_clientmulti\_ret\_x64fre\_en-gb\_1411d5bb40764a6ebde4b536483c22aaa30b489c.esd  
  
Bingo the above link seems about right size at 2.6gb not sure about the format though.  
  
whilst waiting for 10mins I grab a coffee!  
  
Right so it looks like an encrypted .wim  
  
Ok so lets grab the files from https://github.com/gus33000/ESD-Decrypter  
Extract then runas administrator cmd.exe.  
run ESDDecrypter.bat and follow instructions using defaults, it handles the rest of the magic.  
  
After some time it will inflate to 3.67gb iso file in the ESD-Decrypter-master directory.  
  
Of course I don't expect the link above to last but at least you know this method to get stupid stub installers.