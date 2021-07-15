---
title: 'OSSIM custom plugin - Palo Alto Traps Endpoint Protection'
date: 2015-03-26T20:10:00.003Z
draft: false
aliases: [ "/2015/03/ossim-custom-plugin-palo-alto-traps.html" ]
parent: Blogger
---
#### date: 2015-03-26

I have been testing Palo Alto Traps Endpoint Protection (of which I'm extremely satisfied with) and today decided to forward the logs to OSSIM by alienvault.  
  
My sandbox was the following,  
OSSIM Installed; AlienVault\_OSSIM\_64bits\_4.15.2.iso  
VM Platform; VirtualBox 4.3.20 on Windows 7  
  
Prerequisite  
Install OSSIM with two host only interfaces, configure eth0 192.168.56.200/24 eth1 192.168.56.222/24  
  
Login to OSSIM Vm console with root, choose jail break from the menu option.  
Once you get a full shell run the following command  
  
\# adduser –s /bin/bash sadmin  
  
Change the password  
  
\# passwd sadmin  
  
Now add this user to the /etc/sudoers file under; root    ALL=(ALL) ALL  

> sadmin ALL=(ALL) ALL

Now on your windows host open putty and ssh onto ossim.  
  
Syslog Config  
We will create a new config file to catch and redirect to specific logfile.  
  
\# sudo nano /etc/rsyslogd/192.168.56.140.conf  

> if $fromhost == '192.168.56.140' and $rawmsg contains 'Palo Alto Networks|EndpointSecurityManager' then /var/log/ossim/paloalto-endpoint.log  
> if $fromhost == '192.168.56.140' ~

\# service rsyslog restart  
  
Plugin config  
We now need to create a new plugin. I used http://www.rubular.com/ to help with the regex  
  
\# sudo nano /etc/ossim/agent/plugins/paloalto-endpoint.cfg  

> \# Alienvault plugin  
> \# Author: martyn from blogsploit.co.uk  
> \# Plugin paloalto-endpoint id:9007 version: 0.0.1  
> \# Last modification: 2015-03-26 12:37  
> #  
> \# Accepted products:  
> \# syslog - syslog -  
> \# Description:  
> #  
> #  
> #  
>   
> \[DEFAULT\]  
> plugin\_id=9007  
>   
> \[config\]  
> type=detector  
> enable=yes  
>   
> source=log  
> location=/var/log/ossim/paloalto-endpoint.log  
>   
> create\_file=true  
>   
> process=  
> start=no  
> stop=no  
>   
>   
> #;\[translation\]  
> #;Some thing=2  
> #;ACCEPT=1  
> #;REJECT=2  
> #;DROP=3  
> #;DENY=3  
> #;Inbound=4  
> #;Outbound=5  
> #;Terminate=3  
>   
> #;WARNING=3  
>   
> \# Sample Log  
> \# Mar 24 15:18:37 Host CEF: 0|Palo Alto Networks|EndpointSecurityManager|3.2.0.3328|7|Prevention from machine|9|dhost=USER-PC4 msg=Prevention details: User=USER-PC4\\user PreventionMode=Terminate ProcessName=AcroRd32 EPM= Cycode=S01 PreventionKey=74ff476a-6752-49a6-b750-12b11c1f6713 Arguments="C:\\Program Files (x86)\\Adobe\\Reader 9.0\\Reader\\AcroRd32.exe" "C:\\Users\\user\\Desktop\\adobe\_cooltype\_sing.pdf".  
>   
> \[paloalto - endpoint event\]  
> event\_type=event  
> regexp="^(?P<logline>(\\S+\\s+\\d+\\s+\\d\\d:\\d\\d:\\d\\d)\\s+.+dhost=(?P<host>\\S+).+User=(?P<username>\\S+)\\s+PreventionMode=(?P<action>\\S+)(?P<logged\_event>.+))$"  
> device={resolv($host)}  
> date={normalize\_date($1)}  
> plugin\_sid=3  
> username={$username}  
> src\_ip={resolv($host)}  
> dst\_ip={resolv($host)}  
> userdata1={md5sum($logline)}  
> userdata3={$host}  
> userdata4={$action}  
> userdata5={$logged\_event}

Back on the vm console in the alienvault menu goto option 1’configure sensor’ then to option 4 ‘configure data source plugins’ and select your new plugin.  
  
You may have to from putty  
\# /etc/init.d/ossim-server restart  
\# /etc/init.d/ossim-agent restart  
  
Login to your ossim web portal and goto ‘configuration’ > ‘deployment’ > ‘Alienvault center’ > double click ‘alienvault’ object > ‘sensor configuration’ > click ‘collection’  
Search for palo and add ‘paloalto-endpoint’ to the enabled plugins.  
  
Now to get your events into the dashboard we need to create a sql file and import it. Back on putty!  
\# sudo nano /usr/share/doc/ossim-mysql/contrib./plugins/paloalto-endpoint.sql  

> \-- paloalto endpoint  
> \-- plugin\_id: 9007  
>   
> DELETE FROM plugin WHERE id = "9007";  
> DELETE FROM plugin\_sid where plugin\_id = "9007";  
>   
> INSERT IGNORE INTO plugin (id, type, name, description) VALUES (9007, 1, 'paloalto-endpoint', 'Palo Alto Endpoint plugin with md5 checksum logging');  
>   
> INSERT IGNORE INTO plugin\_sid (plugin\_id, sid, category\_id, class\_id, name, priority, reliability) VALUES (9007, 3, NULL, NULL, 'PaloAlto-Endpoint: Terminate Event' , 5, 5);

Back on the vm console in a jail broken full shell;  
  
\# cat /usr/share/doc/ossim-mysql/contrib./plugins/paloalto-endpoint.sql | ossim-db  
  
Best to redo from putty  
\# /etc/init.d/ossim-server restart  
\# /etc/init.d/ossim-agent restart  
  
  
!troubleshooting in putty  
  
\# sudo cat /var/log/alienvault/agent/agent.log | grep 9007