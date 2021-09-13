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

# Note
BSOD is kind of hint on the challenge , to find the kernel function which change in the shared memory with usermod , during the comption i asked the palyers to do the step which will case BSOD (%50 to see there reaction , %50 to give them the hint).

i will not follow this path on this writeup , ill do the basec steps to follow driver entery 

# Setup Challenge 
at the first look at the challenge files, you will find exe called usermod and sys file which is the kernel driver. 

we can guess that the kernel is communicating with the usermod to do something. 

once we are working with the kernel driver, we have to set the testing machine environment.

start the cmd as admin and enable testing mod by running .

```bash 
bcdedit /set testsigning on
```

then restart the machine.

start cmd as admin again and create the driver 

```bash 
sc create <driver_name> type= kernel binpath= <path_to_.sys_file>
```
[![1](/assets/images/ASCWG/k1.png)](/assets/images/ASCWG/k1.png)

now we have the driver created . 

# Usermod 
usermod app is not very important in this challenge you can skip it . 

we will start by checking the application behavur befor and after strting the driver .

quickly run the app without starting the driver . 

[![2](/assets/images/ASCWG/k2.png)](/assets/images/ASCWG/k2.png)

