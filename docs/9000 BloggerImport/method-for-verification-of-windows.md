---
title: 'Method for verification of windows system integrity'
date: 2017-03-14T20:29:00.001Z
draft: false
aliases: [ "/2017/03/method-for-verification-of-windows.html" ]
parent: Blogger
---
#### date: 2017-03-14

Sometimes you find things on systems that look a little odd, and you wonder if just maybe it got breached...  
Well I have good news for you, I have a method to deal with such a scenario.  
First you need to snapshot / take image of suspect system and do what you have to in order to preserve evidence.  
Next you need to index all the .exe .dll and .sys files and generate a hash digest sha1 or sha256.  
I now prefer to use sysinternals sigcheck tool like;  
    sigcheck.exe -a -h -c -e -s -nobanner c:\\ > HashDigest.csv

or you can use powershell to grab a simpler list like;

    # requires powershell 2.0 or greater run as admin  
    $date1 = Get-Date -Format ddMMyyyy

    $Dir = get-childitem C:\\windows\\system32 -recurse  
    $List = $Dir | where {$\_.extension -eq ".dll" -or $\_.extension -eq".exe" -or $\_.extension -eq".sys"}  
    #$list\[0\].FullName

    $sha1hash = @();  
    $sigcheckarray = @();  
    $i = 0;  
    DO  
    {  
        $sha1hash += Get-FileHash -Algorithm SHA1 $List\[$i\].FullName;  
        $i++;  
    } While($i -le $List.Count-1)

    $sha1hash |Export-Csv ".\\Integrity-$env:COMPUTERNAME-$date1.csv" -NoTypeInformation  
    #end of script

Now what I can do for you is save you some time and tell you that NIST's National Software Reference Library will only net you ~1% of hits.  
You are best placed to order the whitehat set from hashsets.com, this will net you ~50% of hits, the other ~49% happens to be in the \\Windows\\winsxs folder.  
I tend to ignore the winsxs folder for the time being.

To churn through the multi-gigabyte datasets I use Talend Open Studio, I cannot recommend this enough as a swiss army knife for any data job.  
I define a job with two csv inputs, the map component and a csv output. connect the output from the larger dataset first then the digest you created,  
map all the fields across to the output, and filter out null records from your digest set.  
Now define a second job in the same way as above, but connect your digest first and then the output from the first job.  
map across whichever fields you feel relevant but keep at least the hash and paths.  
The resulting output file should be less than 20mb and so you can handle it in excel to filter down the list, I tend to split out the path with macros.  
After filtering out the matches and winsxs folder, 99% should be ruled out and only a few thousand files are left for analysis.

My next favourite tool is selenium and writing a python script to drive a webscraper which I point at various anti-malware multivendor scanning services.  
I essentially lookup the hashes and capture the results (I have captcha to contend with and anti-abuse banning, so have to watch these scripts carefully).  
At the end i'm left with a few hundred items to investigate, and tend to submit the files that are not recognised.  
After ruling out the many false positives by performing reanalysis, I end up with either a clean system or one needing remediation.

You can of course build up your own hashset by installing virus free software into your lab and indexing, but this only net me 2000+ hits out of a 22k digest.  
so it's still valid for 3rd party software, better yet bake in the indexing as part of your golden image procedures.  
P.S. don't neglect case sensitivity else you won’t match anything.

Hope this helps

Kempy