---
title: 'To run CMD by local SYSTEM account'
date: 2017-12-29T12:06:00.001Z
draft: false
aliases: [ "/2017/12/to-run-cmd-by-local-system-account.html" ]
parent: Blogger
---
#### date: 2017-12-29

To run CMD by local SYSTEM account, you can use the following method: 1. Run CMD as administrator and then type in the followling command lines: sc create testsvc binpath= "cmd /K start" type= own type= interact sc start testsvc 2. You will get a popup window from "Interactive Services Detection". Click View the message. Then you will receive a cmd prompt running by SYSTEM.