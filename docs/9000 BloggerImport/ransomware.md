---
title: 'Ransomware'
date: 2016-07-10T13:42:00.001+01:00
draft: false
aliases: [ "/2016/07/ransomware.html" ]
parent: Blogger
---
#### date: 2016-07-10

Ransomware is easy money for criminals, let's face it a few lines of code to enumerate the file system, byte stream read the file pass to encryption function and then write the returned bytes back to file stream renaming during process is not a hard day's work.

If that's too much like hard work then a nice Turkish gent wrote hidden-tear just for educational purposes mind you... He published his work here [https://github.com/utkusen/hidden-tear](https://github.com/utkusen/hidden-tear) but as you can see he changed his mind and redacted his work... but this is github, and he got forked 434 times in fact. It wasn't difficult to find another repository a good one is here [https://github.com/goliate/hidden-tear](https://github.com/goliate/hidden-tear) you will set of anti-virus alerts with the compiled binaries.

I checked out which AV were detecting this file, with previous analysis here [http://nodistribute.com/result/tcQF3uda2iwq](http://nodistribute.com/result/tcQF3uda2iwq) and my reanalysis here [http://nodistribute.com/result/tcQF3uda2iwq](http://nodistribute.com/result/tcQF3uda2iwq) the interesting thing to note is the name of the strain the AV vendors are naming it.

So to demonstrate how ineffective AV is these days, I tweaked the code, and my sample which will be very handy for demonstrating to people the attack and how to mitigate has the analysis results here [http://nodistribute.com/result/vG7APQVI459Oo1zRbN8U](http://nodistribute.com/result/vG7APQVI459Oo1zRbN8U) with a nice zero detections, Think ZERO DAY, any potential target could be in a world of trouble if they have not taken effective counter measures.

Concerned about ransomware? seek advice, professionals like myself can help!