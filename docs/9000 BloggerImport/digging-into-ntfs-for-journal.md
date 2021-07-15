---
title: 'Digging into NTFS for the journal'
date: 2016-05-04T12:20:00.001+01:00
draft: false
aliases: [ "/2016/05/digging-into-ntfs-for-journal.html" ]
parent: Blogger
---
#### date: 2016-05-04

If you ever need to find out when a file was changed then NTFS keeps a record in the $Logfile and $UsnJrnl:$J  the journal will go back longer, and by default the 65mb log file will only last hours before being overwritten. So as part of your preparation you should increase the size of these files!

In order to extract these files you need to understand that they are strictly controlled by the windows api, the only way to show and extract them is by using a hex or forensic tool that can open the volume and recognise these files. So far I’ve only found [FTK Imager Lite](https://ad-zip.s3.amazonaws.com/Imager_Lite_3.1.1.zip)  and WinHex capable of this job, Encase has scripts but they have issue over 7 GB. I’ve elected to use FTK Imager.

Run ‘Ftk Imager.exe’ as admin.

Go to ‘file’ > ‘add all attached devices’

Expand the volume of interest for example + c:\\   + noname\[ntfs\]  >  select \[root\]

Scroll down and find and select both $MFT and $Logfile, right click and ‘export files’ > follow dialogs

Now scroll back up to $Extend open this and then $usnJrnl, select $J right click and ‘export files’ > follow dialogs

Now you can use a tool like [NTFS Log Tracker](https://sites.google.com/site/forensicnote/ntfs-log-tracker) to parse the extracted files and check through the events, I’ve not had much luck with exporting to csv, seems to crash.

You could also try this [python script](https://code.google.com/archive/p/parser-usnjrnl/downloads) but that took issue and crashed out.