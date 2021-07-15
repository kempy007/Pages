---
title: 'Password filter for windows with entropy and banned dictionary words'
date: 2017-03-14T21:03:00.001Z
draft: false
aliases: [ "/2017/03/password-filter-for-windows-with_14.html" ]
parent: Blogger
---
#### date: 2017-03-14

You can find my password filter over at [https://github.com/kempy007/password\_filter\_dll](https://github.com/kempy007/password_filter_dll)

### Install

Extract the PasswordFilter.dll from the archive and copy to the Windows installation directory on all domain controller’s or the local computer.

On standard installations, the default folder is %SYSTEMROOT%\\System32 and %SYSTEMROOT%\\sysWOW64

The filter has been compiled for 64bit operating systems.

Extract the banned.dic to %SYSTEMROOT%

To register the password filter, update the following system registry key ‘REG MULTI\_SZ’: ‘HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Lsa\\Notification Packages’

If the ‘Notification Packages’ subkey exists, add the file name of your PasswordFilter.dll to the existing value data at the end. Do not overwrite the existing values, and do not include the .dll extension of the filename.

If the ‘Notification Packages’ subkey does not exist, add it as new ‘REG MULTI\_SZ’, and then specify the name of the DLL for the value data. Do not include the .dll extension of the filename.

The ‘Notification Packages’ subkey can have multiple packages specified on each new line.

[![clip_image002[4]_thumb[1]](https://lh3.googleusercontent.com/-6hpaiDGskIQ/WMhaluCgAbI/AAAAAAAAAE8/eZwhcYeOdpI/clip_image002%25255B4%25255D_thumb%25255B1%25255D_thumb.jpg?imgmax=800 "clip_image002[4]_thumb[1]")](https://lh3.googleusercontent.com/-d9lCkTqTKk4/WMhalSZPZAI/AAAAAAAAAE4/InESK2jvepQ/s1600-h/clip_image002%25255B4%25255D_thumb%25255B1%25255D%25255B2%25255D.jpg)

Figure 1 - RegEdit

NOTE!

On 64 Bit systems, you must copy the dynamic link library to both the "sysWOW64" and "System32" directories. In other words, both directories need the "PasswordFilter.dll" library. May result in System Event ID 16953_._

Reboot the system for the changes to take effect.

### Removal

To unregister the password filter, update the following system registry key ‘REG MULTI\_SZ’: ‘HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Lsa\\Notification Packages’

Remove the line ‘PasswordFilter’

Or if this is temporary then append something like Null so the line reads ‘PasswordFilterNull’ which means the file won’t exist and subsequent library loading will fail generating event ID 16953 in the system logs.

Reboot the system for changes to take effect.

### Maintaining Banned Words

The current dictionary list was created from the files available at [https://www.karamasoft.com/ultimatespell/dictionary.aspx](https://www.karamasoft.com/ultimatespell/dictionary.aspx)

Using the English United Kingdom set [https://www.karamasoft.com/UltimateSpell/Dictionary/English%20(United%20Kingdom)/en-GB.zip](https://www.karamasoft.com/UltimateSpell/Dictionary/English%20(United%20Kingdom)/en-GB.zip)

Using notepad++

This list was then edited with find+replace to remove plurals/apostrophe’s lines. ^.\*'s

Reprocessed to change all to lowercase. TextFx > TextFx Characters > lower case.

Final processing via regex search to remove lines with 6 or less characters. ^.{1,6}$

We need to ensure this file has each word on its own line and that the file is free from apostrophe and quotation mark and slash symbols ‘ “ \\ / as these are used for strings and escaping in native code and will cause problems when we try to load these into an array for use by the password filter, we also expect these to be lowercase otherwise our checks will fail to match words.

#### Software Functionality

For the most basic skeleton password filter dynamic link library to compile, then some prerequisite functions need to be implemented and are in effect call backs that are registered into the LSA module at boot time.

On boot the OS or the LSA module reads the Notification Packages value and loads all DLLs listed in the registry multiline string key from the system32 and syswow64 directories.

The first 3 functions in ‘Figure 2 - Skeleton Password Filter’ are formalities to satisfy various check by the LSA module and the dynamic link library itself, the last function ‘PasswordFilter’ is where one can apply their own logic to validate any password before it is set.

The functional goals for this project are to;

Stop low entropy passwords such as

Aa00000000000

Stop use of single dictionary words even with common substitution such as

P@ssword1

Allow the use of pass phrases such as

The quick brown fox

Load banned words from text file %systemroot%\\banned.dic

[![clip_image002_thumb[1]](https://lh3.googleusercontent.com/-c-wul96HmCw/WMhanXGKdwI/AAAAAAAAAFE/g4jkAYhOobg/clip_image002_thumb%25255B1%25255D_thumb.jpg?imgmax=800 "clip_image002_thumb[1]")](https://lh3.googleusercontent.com/-BN6G2jf0Ogs/WMham8gd7mI/AAAAAAAAAFA/bqxyDi3Yego/s1600-h/clip_image002_thumb%25255B1%25255D%25255B2%25255D.jpg)

Figure 2 - Skeleton Password Filter

To verify if the DLL is successfully loaded run msinfo32, See Computer Configuration + Software Environment + Loaded Modules.

The DLL is only called during a pw change and only if the pw meets the windows requirements (min length, no min age issue, not in history buffer etc).

The LSA first checks windows requirements (in 2K3 and 2K8) and then calls the PasswordFilter() function for each DLL listed in the Notification Packages in the order listed.

If our DLL says pw is OK and no other filters reject it the pw is committed to AD/SAM and then the LSA goes through all DLLs listed there again to call PasswordChangeNotify()

Now a DLL can do pw sync, Good example might be sync to some linux directory service in a heterogeneous environment (sync is OK since pw was committed to AD, but never try to sync of PasswordFilter() call because it may get rejected giving you sync issues).

If you need to modify update the DLL you have to remove the entry from Notification Packages, reboot, copy the new one to system32 and syswow64, update Notification Packages again and reboot.

Kernel debugging is painful. So if you debug it is probably easier to have it write to a file.

Solving the entropy issue is straight forward using Shannon equations, we get a float value returned after sending a string into our function and anything with a score lower than 3.00 is pretty poor so we define our logic to return a fail if the value is lower than 3.00.

Looking at the way password cracking programs seed in a dictionary list and then generate all permutations with common substitutions and prepended and appended values, this quickly generates huge volumes of data and storage quickly becomes a limiting factor, look at rainbow tables for an example.

Whilst the initial train of thought was stuck with how to generate the banned list with all possible combinations it was fundamentally flawed whilst I considered only half of the problem.

[![clip_image004_thumb[2]](https://lh3.googleusercontent.com/-7h8Y9q-LrCI/WMhaoVKwAgI/AAAAAAAAAFM/eizL9BEF3Ro/clip_image004_thumb%25255B2%25255D_thumb.jpg?imgmax=800 "clip_image004_thumb[2]")](https://lh3.googleusercontent.com/-SiSAwa1JooU/WMhaoKjP6AI/AAAAAAAAAFI/mBWerLVrZNg/s1600-h/clip_image004_thumb%25255B2%25255D%25255B2%25255D.jpg)

Figure 3 - Password List Generation

After looking at the problem as in ‘Figure 3 - Password List Generation’ the solution became quite obvious. We need to reverse the techniques used in password cracking.

We build our function to copy the password and then change it all to lowercase.

We then iterate over each character matching any common substitution back into a letter.

Now we compare this string against the array of dictionary words.

In order to allow pass phrases we had to add another check this time against the length of the original password string divide by 2 and compare greater than or equal to the length of the matched dictionary word.

If the dictionary word accounts for less than half, then we fail the password change.

Ultimately you end up with a solution flow as in the right half of ‘Figure 4 - Password List to Filter Funnel Flow’.

[![clip_image006_thumb[2]](https://lh3.googleusercontent.com/-s62E8u789Uo/WMhapTlVNnI/AAAAAAAAAFU/IuYifSoUv4I/clip_image006_thumb%25255B2%25255D_thumb.jpg?imgmax=800 "clip_image006_thumb[2]")](https://lh3.googleusercontent.com/-wX21DJX6nls/WMhao8lm5kI/AAAAAAAAAFQ/D0bDDX__sCU/s1600-h/clip_image006_thumb%25255B2%25255D%25255B2%25255D.jpg)

Figure 4 - Filter Funnel Flow