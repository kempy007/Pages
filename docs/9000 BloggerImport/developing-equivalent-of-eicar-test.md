---
title: 'Developing the equivalent of an eicar test virus for dynamic behaviour only, whilst avoiding static detection, zero day code!'
date: 2014-12-16T03:43:00.000Z
draft: false
aliases: [ "/2015/02/developing-equivalent-of-eicar-test.html" ]
parent: Blogger
---
#### date: 2014-12-16

This topic has come about because I like to be thorough and mostly assure myself that something is working correctly as intended.  
  
Anyone dealing with malware incident response will know any code you capture and collect has already defeated the signature and heuristics engine of several vendors going into an enterprise, and that said enterprise is still using decade old techniques, propped up with reputation list's with cloud intelligence sharing and real time black lists and so on.  
  
Thankfully in the last few years sandbox dynamic behaviour detection techniques have finally made it out of the labs and into consumers hands.  
  

*   Opensource Cuckoo Sandbox Version 0.4.1 released Feb 2011.
*   Palo Alto Networks firewall, Version 4.1. introduced Wildfire in 2011.
*   FireEye formed in 2004, Virtual Execution (VX) engine was released in ?2011, maybe 2010.
*   Lastline formed in 2011, plus decades of research in academia

So 2011 was the year this got going, Now dynamic behaviour detection is no silver bullet by any means, you need to understand that successful detection relies on successful configuration of the virtual machine used to execute and record activities.  
  
What can go wrong, well a good example would be www.malwr.com, who have chosen to use the default Microsoft office trust centre settings, this blocks macros. Therefore you cannot use this online sandbox to detect if an office document contains malware or dropper macros, I've had similar issues with Wildfire. It's not because the sandbox detection software has failed, it's because the virtual machines configuration failed to give the malware the environment it needs in order to execute successfully.  
  
What else can go wrong, again this boils down to what the environment is lacking, or wouldn't normally contain.  
Lets look at what it would be lacking, Human interaction. What does this look like to a program. No mouse movements or clicks, no typing, no starting or closing apps, just no movement what so ever. How does a program do this, simply by logging the keys and pointer positions, in effect key logging.  
So what wouldn't a normal environment contain? We need to remember who is being targeted here, the weakest link being endpoints, primarily windows desktops. You wouldn't expect to find hyper-visor drivers or advanced debugging software here. It's easy for a program to enumerate this type of information and terminate on detection. I really want to cut short on the topic of anti-sandbox and anti-debugging as it's a really big topic on it's own.  
  
Hopefully that's enough of a grounding for most folks and we can come back to the main topic at hand.  
  
How can you test your shiny new 0-day protection security appliances are working effectively in today's climate of cyber crime?  
  
I don't think it's feasible for the average IT/Networks team to accomplish without degrading one layer or another to expose and test the dynamic detection layer.  
In short the only tool I can find that would allow me to test this feature with all the other layers untouched is Visual Studio, In effect I'm left with no choice but to write my own code in c++.  
  
My bread and butter is not programming, so my homework is to teach myself how to not only program in c++ but also how to perform many of the behaviours that malware use. I've set the bar high and it will lead to a positive development in my career, a deeper understanding of malware.  
  
I have seen Logrythm use such a tool, but they won't share, and for obvious reasons, If I were to share a tool, it would be a matter of hours before it gets detected and blocked. Therefore I will try to share what I have learnt by way of future blogs discussing the source code if I have time.