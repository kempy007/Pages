---
title: 'Diving deeper into windows - Driver development - Differences between KMDF & UMDF2.0 - Update 3'
date: 2015-05-27T10:34:00.001+01:00
draft: false
aliases: [ "/2015/05/diving-deeper-into-windows-driver_27.html" ]
parent: Blogger
---
#### date: 2015-05-27

Turns out the subtle difference other than the privileges SYSTEM vs USER is in the libraries you can include, this drastically affects what you can achieve in each type of driver.  
  
With KMDF you can #include <ntddk.h> but not windows.h. Not certain what that ramifications are at this stage, but at first I thought it would stop me from hashing files, but I've since found ways around.  
  
ntddk.h allows you to register callbacks to;  
  

*   PsSetCreateProcessNotifyRoutine
*   PsSetCreateProcessNotifyRoutineEx
*   PsSetCreateThreadNotifyRoutine
*   PsSetLoadImageNotifyRoutine

The drawback to not being allowed to use windows.h is when I try to perform hashing of files, objects or strings etc, I am able to reference the likes of <bcrypt.h> however trying to use PBYTE gives definition not found, however I can reference <minwindef.h> which gets around the issue. I can use <stdio.h> and <stdlib.h>, so no real barrier has been encountered so far.

  

Ultimately the question is, should I do this in the KMDF as any error will result in a bluescreen, so really I should be looking to do bare minimum with very high level of error control to prevent BSOD. Will the delay in ferrying data between drivers be a problem?

  

Now looking back at UMDF2.0 you can #include <windows.h> but not ntddk.h, This means you cannot register callbacks for processes, threads and image creation/loading. We loose insight in to what is happening in the OS, but we can do more risky things safely in the fact that we won't blue screen the OS. 

  

So by using KMDF to monitor all activities, pass data to UMDF2.0 for hashing and reputation checks, wait for verdict, then block or allow activity. We can create our own application control which can later be expanded upon to perhaps perform multi-vendor sandbox submissions, or compare hash database against others sources such as www.nsrl.nist.gov (handy for verifying the integrity of your OS, often used in forensics for eliminating common and known files)

  

That's all for now