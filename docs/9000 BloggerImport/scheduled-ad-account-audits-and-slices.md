---
title: 'Scheduled AD account audits and slices with Talend Studio'
date: 2017-03-14T21:31:00.001Z
draft: false
aliases: [ "/2017/03/scheduled-ad-account-audits-and-slices.html" ]
parent: Blogger
---
#### date: 2017-03-14

It was decided to take the above two scripts further and take out the manual collation of data, and to offer slices of the dataset to highlight ‘High privileged users’, ‘60+days inactive/idle’, ‘90+days PW not changed, ‘180+days disabled’.

To accomplish this I decided to use ‘Talend Open Studio’, A tool I had used in the past to process Qualys data into KPI’s for vulnerability management. It is free software and open source, based upon eclipse ide.

The task involved running the two scripts ‘ADPrivilegeAccounts-Audit-v3.1.ps1’ & ‘AD-User-Last-Logon.ps1’.

Stitching the two output files together, like a DB join, normalising the date fields, creating new columns that calculate the date difference from current date to that of either ‘last logon’ or ‘password last set’, outputting the dataset and slices.

Attaching output files to email and sending.

Cleaning up temp files.

The directory structure to run this was as follows;

C:\\TS-Job

Data < temp files wrote here

Scripts < powershell scripts invoked here

Etc < various folders from the talend job

I setup the scheduled job on the PC in the comm room XYZ to run Monday at 08:00 weekly

I extracted zipfile to the root of C: drive like c:\\TS-Job so that command paths do not break.

Requires Java as job was built in talend studio.

For Windows 7 install the RSAT tools, and add all features in program files.

I set the task to run as NETWORK SERVICE and to my surprise it works with AD.

The scheduled task triggers C:\\TS-Job\\ProcessReport\\ProcessReport\_run.bat

I had to edit above script to include an updated java path before the java command;

set PATH=%PATH%;C:\\TS-Job\\jre1.8.0\_111\\bin

I also copied from my pc the JRE folder to C:\\TS-Job\\jre1.8.0\_111 as the version 6 was too old to run the job.

Also had to run as administrator from cmd.exe

c:\\windows\\syswow64\\WindowsPowerShell\\v1.0\\powershell.exe -command set-executionpolicy unrestricted

Zipfile can be obtained from [https://github.com/kempy007/AD-Reports](https://github.com/kempy007/AD-Reports "https://github.com/kempy007/AD-Reports") 

Talend Job looks like below;

[![image](https://lh3.googleusercontent.com/-fdbnI9h5Tw8/WMhhIq93l6I/AAAAAAAAAGQ/vfvpUrYk-ZQ/image_thumb%25255B2%25255D.png?imgmax=800 "image")](https://lh3.googleusercontent.com/--JpVxDMTpgI/WMhhIDkotzI/AAAAAAAAAGM/Tw4iXtuVVSo/s1600-h/image%25255B4%25255D.png)