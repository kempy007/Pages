---
title: 'Sysmon for a poor mans EDR'
date: 2016-03-29T20:58:00.001+01:00
draft: false
aliases: [ "/2016/03/sysmon-for-poor-mans-edr.html" ]
parent: Blogger
---
#### date: 2016-03-29

I began my driver project because I couldn’t find what I wanted, unbeknown to me that sysmon existed and is exactly what I wanted (well for the first stage). I wanted a kernel driver to like a blackbox record every process that is created and destroyed and tell me the hash along with the process lineage. you can grab it from [https://technet.microsoft.com/en-gb/sysinternals/sysmon](https://technet.microsoft.com/en-gb/sysinternals/sysmon "https://technet.microsoft.com/en-gb/sysinternals/sysmon")

I can then couple this with NxLog, to ship the events to a log platform of your choice eg (Splunk | Ossim | ELK) to name a few, for my toy i’ll use splunk simply because it has many addons available, and i’ll be keen to try out the virustotal checker with the hashes that sysmon generates, I can then also setup alerts for suspicious process lineage (call chains).  Plenty of reference material at any sandboxing sites like [http://malwr.com](http://malwr.com)

Searching for opensource edr lead me to this article [http://blogs.gartner.com/anton-chuvakin/2015/07/23/reality-check-on-edr-etdr/](http://blogs.gartner.com/anton-chuvakin/2015/07/23/reality-check-on-edr-etdr/ "http://blogs.gartner.com/anton-chuvakin/2015/07/23/reality-check-on-edr-etdr/") I have played with El Jefe before and was impressed, but it lacked stability and documentation, the others are new to me so i’ll be exploring them soon.

Sites like [http://www.hashsets.com/](http://www.hashsets.com/ "http://www.hashsets.com/") do charge but you get to rule out more known good files.

_Updated:17/05/2016_

Good reference for processing the data [http://www.crowdstrike.com/blog/sysmon-2/](http://www.crowdstrike.com/blog/sysmon-2/ "http://www.crowdstrike.com/blog/sysmon-2/") particularly using Microsoft Logparser and the tekcollect.py script for building up a hash set.

I installed logparser and copied the dll and exe from C:\\Program Files (x86)\\Log Parser 2.2 to C:\\Windows\\System32 and then ran the following commands

robocopy C:\\Windows\\system32\\winevt\\Logs c:\\temp \*sysmon\*.evtx

logparser -i:evt -o:csv "Select RecordNumber,TO\_UTCTIME(TimeGenerated),EventID,SourceName,ComputerName,SID,Strings from c:\\temp\\Microsoft-Windows-Sysmon%4Operational.evtx WHERE EventID in ('1';'2';'3';'4';'5';'6';'7';'8')" > c:\\temp\\sysmon\_parsed.txt

had to install package so cd c:\\python27\\scripts

pip install httplib2

fetched script from [https://raw.githubusercontent.com/1aN0rmus/TekDefense/master/tekCollect.py](https://raw.githubusercontent.com/1aN0rmus/TekDefense/master/tekCollect.py "https://raw.githubusercontent.com/1aN0rmus/TekDefense/master/tekCollect.py")

python c:\\temp\\tekcollect.py -f c:\\temp\\sysmon\_parsed.txt -t SHA1 > c:\\temp\\Sha1Hashes.txt

fetched virustotalchecker from [https://github.com/woanware/woanware.github.io/raw/master/downloads/virustotalchecker.v1.1.4.zip](https://github.com/woanware/woanware.github.io/raw/master/downloads/virustotalchecker.v1.1.4.zip "https://github.com/woanware/woanware.github.io/raw/master/downloads/virustotalchecker.v1.1.4.zip")

need to set the api key in settings.xml

cd c:\\temp\\vtchecker  
virustotalchecker.exe -m c -f c:\\temp\\sha1Hashes.txt -o c:\\temp\\

You should have a list of detection ratios, and maybe failed lookups however this failed lookups file does not like excel so open it with a text editor.

Updated 18/07/2016:
-------------------

Data sizing is estimated around 1GB per month, 3.08GB  (29 Mar 16:20 >> 18 Jul 10:51) 3,993,495 Events