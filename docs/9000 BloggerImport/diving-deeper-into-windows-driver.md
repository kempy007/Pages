---
title: 'Diving deeper into windows - Driver development - Security in mind, Part 1'
date: 2015-05-12T21:09:00.000+01:00
draft: false
aliases: [ "/2015/05/diving-deeper-into-windows-driver.html" ]
parent: Blogger
---
#### date: 2015-05-12

Why would anyone want to bother writing drivers for Windows, I mean it's hard enough writing applications that work as intended and are free of logic flaws that cannot be compromised by cyber hackers right?  
  
Well I have a problem, I'm just too inquisitive, I need to find out how programming drivers work, as I have learned that a driver is the only place I can do special things with windows.  
  
I wan't to teach myself how to check every process/thread that starts and runs on a windows computer, my aim is to enable myself to identify the code wanting to run, and say hey do I know you, have you got a reputation? From there on in I want to hash the digital world one bit at a time. From this knowledge I can derive the power to decide if I want to allow this process/thread to continue or if I should terminate it. At this stage I suppose I am essentially looking to write my own anti-virus engine looking for signatures, however I am coming at this problem from a different angle, really I want to be able to assure the integrity of any code looking to execute. I am hoping I can hoover up many sources on the internet to aid in delivering a verdict. Well now that the back ground for my drive and reason to even be thinking this, is out there I can begin to document how I go about achieving this.  
  
  
If driver code goes wrong, you will blue screen! For that reason driver development is done across two computers. The host contains your development tools and is where you write the code, the target has just enough tools to run debug and deploy the packages to. They communicate by serial port if older than windows 7, or from windows 8 onwards they can do this over a network. either can be physical or virtual.  
  
My host OS is 64bit.  
I happen to use virtual box 4.3.26-98 so I have to add in serial ports that use a pipe :) \\\\.\\pipe\\vmcom#  
I find virtualbox a bit finicky with com ports, sometimes it will add an lpt port! so I add both com ports that it will allow.  
For the target OS I am trying out windows 10 preview, I've stuck to windows 7 so far as I've really disliked windows 8 & 8.1, so version 10 is a real positive change for us poor desktop users, it installs without effort, I think even my Gramps wouldn't struggle with it.  
My IDE is visual studio 2013 community/express.  
Additionally you need to install the WDK 8.1 update 1.  
  
Install windows 10 in the background on virtualbox, checking often to progress through the dialogs.  
  
So starting on the host, create a new project > installed > templates > visual C++ > windows driver > WDF > Kernel mode driver(kmdf).  
This gives you a skeleton driver. Change your debugging profile to win8.1 debug - x64 on the toolbar.  
Then open the solutions properties and go to 'configuration properties' and again change all to win8.1 debug - x64.  
Now open properties on the package project. go to 'configuration properties' > 'driver install' > deployment. then 'enable deployment' which defaults to 'install and verify'. you will have to click the icon with three dots under 'target computer name' and run through the wizard before it will let you apply these options. We'll leave this open and come back to it later.  
  
Back to the windows 10 install, once finished, setup an account with same name and password as what your using on the host. Change the computer name to something like driver-dev01 and reboot.  
from the target browse to the host over samba share, I changed the vbox network to host only and then disabled the windows firewall. the file the target needs is at c:\\Program Files (x86)\\Windows Kits\\8.1\\Remote\\x64\\WDK Test Target Setup x64-x64\_en-us.msi. Once this file is installed we can finally deploy our driver package!  
I also change the com ports speed to 128000 baud instead of 9600.  
  
Back to the IDE we need to add the deployment computer. we can let it be provisioned automatically but that will default to the network for communications. So we'll try it and see if it works ok. During this process you will see colourful command prompts open on the target and it should reboot a few times. I find the network option tends to fail on creating a system restore point. If this is going to work properly we should be able to break and the target will pause. this allows us to enter commands into the remote debugger, if this does not happen then something is wrong!  
I ended up switching to serial instead.  
Also to bear in mind on the driver project properties > configuration properties > debugging > set the remote computer!  
  
So now hopefully hitting F5 should package up the driver, deploy and connect to remote debugging.  
I ran into inf2cat error, so opened properties on the driver project and went to 'configuration properties' > 'driver signing' and set the test certificate to 'create' which generates a self signed cert.  
Also had to correct some incorrect filename issues.  
  
One other problem I ran into were no symbols loading for my driver, to resolve goto tools > options, > debugging > symbols, and add the path to your drivers build directory with the .pdb located in it.  
  
The next problem I faced was getting the debugger to hit a break point and actually step through the code without it showing assembly.  
The trick that worked is to reboot the target and request a break before boot up really begins pre your driver loading.  
Then at this break after...  
  
Loaded module 'mup.sys', Symbol status: <Deferred>  
Loaded module 'mup.sys', Symbol status: <Deferred>  
Loaded module 'intelpep.sys', Symbol status: <Deferred>  
Loaded module 'intelpep.sys', Symbol status: <Deferred>  
Loaded module 'hwpolicy.sys', Symbol status: <Deferred>  
Loaded module 'hwpolicy.sys', Symbol status: <Deferred>  
Loaded module 'disk.sys', Symbol status: <Deferred>  
Loaded module 'disk.sys', Symbol status: <Deferred>  
Loaded module 'CLASSPNP.SYS', Symbol status: <Deferred>  
Loaded module 'CLASSPNP.SYS', Symbol status: <Deferred>  
  
.. in the debuggers immediate window use;  
kd> bp KMDFDriver1!DriverEntry  
  
to check you have a break point set use;  
kd> bl  
  
now continue either with f5 or enter;  
kd> g  
  
After sometime of waiting, all drivers will have loaded along with your symbols, and you will have hit your breakpoint and are able to step through like any other app you've written.  
  
Well I think that's all there is to it, I'm off to write some code. May redo this post in an easier to understand manner when I have time.  
  
I spent about 5 days figuring this out as crawling the internet does not yield lots of articles, and the book I bought did not simplify it for the amateur like me!  
  
Well I hope that helps someone.