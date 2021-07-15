---
title: 'Kali Linux Install notes on VirtualBox'
date: 2014-08-29T14:41:00.001+01:00
draft: false
aliases: [ "/2014/08/kali-linux-install-notes-on-virtualbox.html" ]
parent: Blogger
---
#### date: 2014-08-29

Basically this is an install list for getting Kali Linux setup ready for some of the security projects I work on.
-----------------------------------------------------------------------------------------------------------------

1.  ```
    Install from ISO using default options all way through.
    ```
2.  ```
    apt-get clean && apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y
    ```
3.  ```
    shutdown now -r
    ```
4.  ```
    apt-get install linux-headers-$(uname -r)
    ```
5.  ```
    Insert Vbox-Additions Iso; Devices > Install Guest Additions CD Image...
    ```
6.  ```
    mkdir /root/vbox-additions && cp -r /media/cdrom/\* /root/vbox-additions/
    ```
7.  ```
    chmod 777 /root/vbox-additions/VBoxLinuxAdditions.run && cd /root/vbox-additions/
    ```
8.  ```
    ./VBoxLinuxAdditions.run
    ```