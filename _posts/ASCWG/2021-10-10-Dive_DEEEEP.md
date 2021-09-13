---
title: "Dive DEEEEP "
classes: wide
header:  
  teaser: /assets/images/ASCWG/kernel.jpg
ribbon: green
description: "the plain is very easy and straightforward, jump to the kernel, convince it to send the flag to the user, that's it ."
categories:
  - CTF Write_up
  - Reverse Engineering
  - DFIR 
toc: true
---
# About Challenge.

Name  : Dive DEEEEP.

Level : Hard.

Description : the plain is very easy and state forward, jump to the kernel, convince it to send the flag to the user, that's it . 

Files : Challenge cinatins 2 vresions for windows 7 and 10 , each version conatins 2 files , .sys & .exe .

# Setup Challenge 
once we are working with kernel driver, we have to set the testing machine invromint.

start the cmd as admin and enable testing mod by running 

```bash 
bcdedit /set testsigning on
```

then restart the machine .

start cmd as admin again and create the driver 

```bash 
sc create <driver_name> type= kernel binpath= <path_to_.sys_file>
```
[![1](/assets/images/ASCWG/k1.png)](/assets/images/ASCWG/k1.png)
