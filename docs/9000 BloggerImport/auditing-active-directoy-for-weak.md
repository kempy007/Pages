---
title: 'Auditing Active Directoy for weak passwords.'
date: 2016-10-23T23:36:00.001+01:00
draft: false
aliases: [ "/2016/10/auditing-active-directoy-for-weak.html" ]
parent: Blogger
---
#### date: 2016-10-23

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\*\* Password auditing windows 2012R2 AD accounts \*\*\*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Summary: To audit AD for weak passwords, we need to use password cracking tools and techniques. An attacker would have to obtain three files from the DC in order to find the user accounts and hashed passwords, and would somehow have to elevate to domain admin privileges. these files are;  
    c:\\windows\\ntds\\ntds.dit  
    c:\\windows\\system32\\config\\SYSTEM  
    c:\\windows\\system32\\config\\SAM  
The problem is that these files are locked by the OS, so over the years various methods have been used, such as injecting into LSASS, Faking AD replication, Bypassing OS locks by reading raw disk, or using shadow copies(the method we'll be using). The SAM file is not necessary for our task to be completed but are captured just in case and are typically used to extract the keys to unlock one of the files. We will install all the tools onto a test VM so that we can compile some of them and test our procedure.

  
Process: we install github desktop tools so we can do git clone later, cygwin is used to install gnu make tools and ophcrack is the tools to compare rainbow tables to hased values to reveal what the passwords actually are, download the following files;

[https://github-windows.s3.amazonaws.com/GitHubSetup.exe](https://github-windows.s3.amazonaws.com/GitHubSetup.exe)

[https://cygwin.com/setup-x86\_64.exe](https://cygwin.com/setup-x86_64.exe)

[http://ophcrack.sourceforge.net/download.php?type=ophcrack](http://ophcrack.sourceforge.net/download.php?type=ophcrack)

[http://ophcrack.sourceforge.net/tables.php](http://ophcrack.sourceforge.net/tables.php)  
[http://sourceforge.net/projects/ophcrack/files/tables/XP%20free/tables\_xp\_free\_small.zip/download](http://sourceforge.net/projects/ophcrack/files/tables/XP%20free/tables_xp_free_small.zip/download)  
[http://sourceforge.net/projects/ophcrack/files/tables/XP%20free/tables\_xp\_free\_fast.zip/download](http://sourceforge.net/projects/ophcrack/files/tables/XP%20free/tables_xp_free_fast.zip/download)

[https://www.python.org/ftp/python/2.7.12/python-2.7.12.msi](https://www.python.org/ftp/python/2.7.12/python-2.7.12.msi)

[http://aka.ms/vcpython27](http://aka.ms/vcpython27)  
[https://www.microsoft.com/en-gb/download/confirmation.aspx?id=44266](https://www.microsoft.com/en-gb/download/confirmation.aspx?id=44266)

The process to install githubSetup.exe is straight forward next next and skip the account signup and close the app at the end. A reboot will be needed for the PATH changes to apply.

the cygwin install is a little more involved and the setup-x86\_64.exe is used to add packages too, so run this file, click 'Next', choose 'Install from internet' and click 'Next', leave the directory set as 'C:\\cygwin64' and 'All users' set and click 'Next', accept the default 'C:\\Users\\Administrator\\Desktop' and click 'Next', choose 'Direct Connection' and click 'Next', select the mirror at top of the list and click 'Next',  
You should now be at the package selection window.  
Now as per instructions from [https://github.com/libyal/libesedb/wiki/Building](https://github.com/libyal/libesedb/wiki/Building) we need to ensure we install the following packages;  
    autoconf  
    automake  
    binutils  
    gcc-core  
    gettext  
    libiconv  
    libtool  
    make  
    pkg-config  
I used the search box to filter down by each package mentioned above, and then click the circle symbol with opposing arrows drawn onto the circle to 'Install'. repeat for each package and once complete click 'Next'.  
It will now download all the relevant packages and takes a while. eventually you can click 'Finish'

Now is the time to reboot the server VM.

Log back in and open the cygwin shell. any line with a $ represents command to enter into this shell and the dollar symbol is not part of the command.  
$ git clone [https://github.com/libyal/libesedb.git](https://github.com/libyal/libesedb.git)  
$ cd libesedb\\  
$ ./configure --enable-static-executables=yes  --enable-shared=no  
$ make  
$ make install

If we tried to configure without the extra options then we'd run into the issue described at [https://github.com/libyal/libesedb/issues/15](https://github.com/libyal/libesedb/issues/15)

the compiled tools are located at /usr/local/bin or in windows explorer at C:\\cygwin64\\usr\\local\\bin  
Copy esedbexport.exe & esedbinfo.exe to your desktop into a folder 'pkg-esedbtool'  
also copy C:\\cygwin64\\bin\\cygwin1.dll to above folder.

Great now were nearly at the fun stuff, but we have to write a powershell script to execute the volume shadow snapshot and collect the files of interest and then delete the snapshot.

########################  
\## Start of script #####  
########################  
\# snapfiles.ps1  
\# Author:Mkemp  
\# Date:23-10-16  
\# Ver:1.0

$state1 = vssadmin list shadows  
$snapshot = vssadmin create shadow /for=C:  
$state2 = vssadmin list shadows

ForEach($line in $snapshot)  
{  
    #$line  
    if($line.Contains("Shadow Copy ID:"))  
    {  
        $split = $line.Split(":")  
        $vssID = $split\[1\].Replace('{','').Replace('}','')  
        $vssID = $vssID.trim()  
    }  
    if($line.Contains("Shadow Copy Volume Name:"))  
    {  
        $split2 = $line.Split(":")  
        $volume = $split2\[1\]  
    }  
}

if( $state1.Contains("No items found that satisfy the query.") )  
{  
    #we can assume this is the first snapshot  
    cd .\\dumps  
    $cmd = @"  
    copy $volume\\windows\\ntds\\ntds.dit .  
"@  
    CMD /C $cmd  
    $cmd = @"  
    copy $volume\\windows\\system32\\config\\SYSTEM .  
"@  
    CMD /C $cmd     
    $cmd = @"  
    copy $volume\\windows\\system32\\config\\SAM .  
"@  
    CMD /C $cmd  
    reg SAVE HKLM\\SYSTEM .\\SYS  
}

if( $state.Contains("No items found that satisfy the query.") )  
{  
    $delete1 = vssadmin delete shadows /for=C: /All /Quiet  
}  
else{  
    #not working ????  
    $cmd = @"  
    vssadmin delete shadows /for=C: /shadow=$vssID  
"@  
    CMD /C $cmd  
}

$state3 = vssadmin list shadows

if($state1 = $state3)  
{ "States ok" }

########################  
\## End of script #######  
########################

########################  
\## Start of script #####  
########################  
\# dumphashes.ps1  
\# Author:Mkemp  
\# Date:23-10-16  
\# Ver:1.0  
cd dumps  
..\\esedbtool\\esedbexport.exe -m tables ntds.dit

..\\dsusers\\dsusers.exe ntds.dit.export\\datatable.4 ntds.dit.export\\link\_table.7 hashwork --syshive SYS --passwordhashes --lmoutfile lm-out.txt --ntoutfile nt-out.txt --pwdformat ophc  
########################  
\## End of script #######  
########################

  
########################  
\## Start of script #####  
########################  
\# clean.ps1  
\# Author:Mkemp  
\# Date:23-10-16  
\# Ver:1.0  
del .\\dumps\\hashwork\\\*  
del .\\dumps\\ntds.dit.export\\\*  
del .\\dumps\\ntds.dit.export  
del .\\dumps\\ntds.dit  
del .\\dumps\\SAM  
del .\\dumps\\SYS  
del .\\dumps\\SYSTEM  
########################  
\## End of script #######  
########################

Now we can use the compiled esedbexport tool to dump out the structure like below;

\> esedbexport -m tables ntds.dit

you should now have a folder called ntds.dit.export

you should install python now accepting defaults next next etc and also install the vcpython27 package.

\> c:\\Python27\\Scripts\\pip.exe install pyinstaller pycrypto  
\> cd c:\\user\\Administrator  
\> git clone [https://github.com/csababarta/ntdsxtract.git](https://github.com/csababarta/ntdsxtract.git)  
\> cd ntdsextract  
\> c:\\Python27\\Scripts\\pyinstaller.exe dsusers.py

example command to create the hashdump;  
\> dsusers.exe ntds.dit.export\\datatable.4 ntds.dit.export\\link\_table.7 hashdumpwork --syshive SYS --passwordhashes --lmoutfile lm-out.txt --ntoutfile nt-out.txt --pwdformat ophc

  
I build my distribution files on the desktop in the following structure;

Desktop  
    pkg-pw-audit  
        dsusers  
            dsusers.exe  
            ...  
        dumps  
            hashwork  
            ...  
        esedbtool  
            esedbexport.exe  
            ...  
        dumphashes.ps1  
        snapfiles.ps1  
        clean.ps1

I then add these to my github account so other people don't have to spend a day figuring out how to compile or meet the dependencies, they can just crack on with the job of auditing for weak passwords. you find under dump\\hashwork\\nt-out.txt is where you need to point ophcrack at.  
I will leave the output as sample files and have clean script.

you can find my work at [https://github.com/kempy007/pkg-adpw-audit](https://github.com/kempy007/pkg-adpw-audit)

now you can clone from github onto your domain controller and run the powershell scripts, then copy off the nt-out.txt to your ophcrack host.

I won't bother with instructions for ophcrack other than use the free table 'xp free fast', 'xp free small', 'vista free', 'vista probabilistic free' you could also generate your own tables too with other tools but you will need huge amounts of ram.

That's it for this blog post, I'm tired now :) so that mean all errors and omission excepted

  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\* end of guide> appendix starts here \*\*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
failures: tried to use nuitka to compile python to binary and it dependancy used with mingw64 without success, got it to compile with visual studio, however it still failed to work due a an unhandled exception in sys.stderror.out having 3 instead of one switches/overloads. ultimately pyinstaller was so much simpler and effective.

do not use the MinGW-W64 package because of a bug, install visual studio and then you can compile the python scripts into portable executables that not longer require python.

\> C:\\Users\\Administrator>c:\\Python27\\Scripts\\nuitka --standalone --recurse-all C:\\cygwin64\\home  
\\Administrator\\ntdsxtract\\dsusers.py

\--- VSS copy method --------  
vssadmin list shadows

vssadmin create shadow /for=C:

#\\\\?\\GLOBALROOT\\Device\\<SHADOWYCOPY DISK>\\windows\\<directory>\\<File> <where to put file>

copy \\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy\[X\]\\windows\\ntds\\ntds.dit .  
copy \\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy\[X\]\\windows\\system32\\config\\SYSTEM .  
copy \\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy\[X\]\\windows\\system32\\config\\SAM .

reg SAVE HKLM\\SYSTEM c:\\SYS

#vssadmin delete shadows /for= \[/oldest | /all | /shadow=\] \[/quiet\]

vssadmin delete shadows /for=C: /shadow=e8eb7931-5056-4f7d-a5d7-05c30da3e1b3

\---- Alt Method -----

[https://github.com/clymb3r/PowerShell/tree/master/Invoke-NinjaCopy](https://github.com/clymb3r/PowerShell/tree/master/Invoke-NinjaCopy)

Invoke-NinjaCopy -Path “c:\\windows\\ntds\\ntds.dit” -ComputerName “RDLABDC02” -LocalDestination “c:\\temp\\ntds.dit”

  
\--- what to do with files ----

most successful> [https://blog.joelj.org/windows-password-audit-with-kali-linux/](https://blog.joelj.org/windows-password-audit-with-kali-linux/)  
had to use latest kali and files from [https://github.com/libyal/libesedb/archive/master.zip](https://github.com/libyal/libesedb/archive/master.zip) instead.

  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\*\*\*\*\*  references  \*\*\*\*\*\*\*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

[https://github.com/PowerShellMafia/PowerSploit/tree/master/Exfiltration/NTFSParser/NTFSParser](https://github.com/PowerShellMafia/PowerSploit/tree/master/Exfiltration/NTFSParser/NTFSParser)  
[https://github.com/clymb3r/PowerShell/tree/master/Invoke-NinjaCopy](https://github.com/clymb3r/PowerShell/tree/master/Invoke-NinjaCopy)  
[https://clymb3r.wordpress.com/2013/06/13/using-powershell-to-copy-ntds-dit-registry-hives-bypass-sacls-dacls-file-locks/](https://clymb3r.wordpress.com/2013/06/13/using-powershell-to-copy-ntds-dit-registry-hives-bypass-sacls-dacls-file-locks/)  
[https://adsecurity.org/?p=2398](https://adsecurity.org/?p=2398)  
[https://github.com/quarkslab/quarkspwdump](https://github.com/quarkslab/quarkspwdump)  
[http://security.stackexchange.com/questions/48307/where-does-active-directory-store-user-hashes](http://security.stackexchange.com/questions/48307/where-does-active-directory-store-user-hashes)  
[https://adsecurity.org/?p=451](https://adsecurity.org/?p=451)  
[https://www.swordshield.com/2015/07/getting-hashes-from-ntds-dit-file/](https://www.swordshield.com/2015/07/getting-hashes-from-ntds-dit-file/)  
[https://www.dsinternals.com/en/dumping-ntds-dit-files-using-powershell/](https://www.dsinternals.com/en/dumping-ntds-dit-files-using-powershell/)  
https://blog.joelj.org/windows-password-audit-with-kali-linux/