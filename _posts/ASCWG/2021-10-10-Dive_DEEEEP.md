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
BSOD is kind of hint on the challenge, to find the kernel function which changes in the shared memory with usermod, during the competition I asked the players to do the step which will cause BSOD (%50 to see my friends reaction, %50 to give them the hint)

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
Usermod app is not very important in this challenge you can skip it. 

We will start by checking the application behaviour before and after starting the driver.

## general look 

Quickly run the app without starting the driver. 

[![2](/assets/images/ASCWG/k2.png)](/assets/images/ASCWG/k2.png)

the app just show a message and nothing else , it seams to be wating some input from the kernel. 

so lets try again after starting the driver . 
 
```bash 
sc start <driver_name>
```
[![3](/assets/images/ASCWG/k3.png)](/assets/images/ASCWG/k3.png)

run the app 

[![4](/assets/images/ASCWG/k4.png)](/assets/images/ASCWG/k4.png)

The app outputs the process id of any newly spawned process.

you  can check this by strting new processes and compair its id to the program output .

close the driver .
 
```bash 
sc stop <driver_name>
```

## close look
lets load the app to ida to check if it can do any auther functions . 

to get main function follow start function return . 

[![5](/assets/images/ASCWG/k5.png)](/assets/images/ASCWG/k5.png)

you can fund function takes environment argv, argc as parameters, this function should be the main function. 

[![6](/assets/images/ASCWG/k6.png)](/assets/images/ASCWG/k6.png)

the function start by seting some varuables , and creat shared file handle called **HelloLabib** (Just forgot to rename the registry ) . 

[![7](/assets/images/ASCWG/k7.png)](/assets/images/ASCWG/k7.png)

after loading the handle , it the app checks the shared memory , then load the received data which is being sent using the kernel control function (so in kernel er need to find the control function )

[![8](/assets/images/ASCWG/k8.png)](/assets/images/ASCWG/k8.png)

the variable inbuffer is the process id, you can know by following the std::cout parameter, you will see v2 is the process id in usermod, and v2 gets its value from inbuffer which is the received data from the kernel. 

[![9](/assets/images/ASCWG/k10.png)](/assets/images/ASCWG/k10.png)

before printing the PID, the app checks if the new PID is 0 or equal to the previous PID, by saving v2 in v1 after the if block. 

[![10](/assets/images/ASCWG/k11.png)](/assets/images/ASCWG/k11.png)

[![11](/assets/images/ASCWG/k9.png)](/assets/images/ASCWG/k9.png)

Then we move to another if block that checks 2 bytes received from kernel, the condition returns true that means the flag will be decrypted and mixed using received bytes from the kernel and then it will be printed after decryption.

[![12](/assets/images/ASCWG/k12.png)](/assets/images/ASCWG/k12.png)

the last thing we can get from this exe is the condetion that checks some recived handles bytes , debending on this bytes it will terminate the recived process pid.

[![13](/assets/images/ASCWG/k13.png)](/assets/images/ASCWG/k13.png)

but from the output message, we can expect a blue screen of death if this condition is true, so we need to be careful will working with the driver. 

now we don't need anything else from the usermod, let's move to the kernel driver. 

# Kernel 

just load the driver to IDA without starting it, we almost can do the analysis statically.

like the usermod , ida will not land you to the read driver entry location , just follow the function return . 

you will land in the driver entry loader function.

[![14](/assets/images/ASCWG/k14.png)](/assets/images/ASCWG/k14.png)

jump to driver entry .

[![15](/assets/images/ASCWG/k15.png)](/assets/images/ASCWG/k15.png)

The driver starts by creating a registry device and shared memory. 

also it set 3 driver objects to point to 3 functions , create call , clos call and control.

we can jump to the control function to findout what happens . 

[![16](/assets/images/ASCWG/k16.png)](/assets/images/ASCWG/k16.png)

the control function takes IRP object as permitted, then changes its structure objects values before completing the request and send it.

so we need where is this struct is being changed or set.

back to the driver entry function , we can find that the api **PsSetCreateProcessNotifyRoutine** is very intersting .

[![17](/assets/images/ASCWG/k17.png)](/assets/images/ASCWG/k17.png)

according to Microsoft documentation : The PsSetCreateProcessNotifyRoutine routine adds a driver-supplied callback routine to, or removes it from, a list of routines to be called whenever a process is created or deleted. <sub><sup>[1]</sup></sub>

```c++
NTSTATUS PsSetCreateProcessNotifyRoutine(
  PCREATE_PROCESS_NOTIFY_ROUTINE NotifyRoutine,
  BOOLEAN                        Remove
);
```

also according to Microsoft documentation about **PCREATE_PROCESS_NOTIFY_ROUTINE** : Process-creation callback implemented by a driver to track the system-wide creation and deletion of processes against the driver's internal state.<sub><sup>[2]</sup></sub>

```c++
PCREATE_PROCESS_NOTIFY_ROUTINE PcreateProcessNotifyRoutine;

void PcreateProcessNotifyRoutine(
  HANDLE ParentId,
  HANDLE ProcessId,
  BOOLEAN Create
)
{...}
);
```

now we know that PsSetCreateProcessNotifyRoutine will throw ppid , pid and creatFlag to PCREATE_PROCESS_NOTIFY_ROUTINE function .

so this is the API we were searching for, according to the usermod output it shows process id. 

jump to the PCREATE_PROCESS_NOTIFY_ROUTINE function . 

[![18](/assets/images/ASCWG/k18.png)](/assets/images/ASCWG/k18.png)

the function start by checking if the process is created or terminated by checking the creat flag. 

[![19](/assets/images/ASCWG/k19.png)](/assets/images/ASCWG/k19.png)

then it gets the eprocess object location using the PID and use this eprocess object to get the image name .

[![20](/assets/images/ASCWG/k20.png)](/assets/images/ASCWG/k20.png)

the next step do a check on parent id , if the ppid is 65537 then it will set byte from the shared struct to 1 ,
then it gets its eprocess object location .

[![21](/assets/images/ASCWG/k21.png)](/assets/images/ASCWG/k21.png)

then it starts to decrypt some data and write it in a struct to be shared .

befor doing any thing image name it cast the result from RtlunicodeString to char array and save it in dest . 

[![22](/assets/images/ASCWG/k22.png)](/assets/images/ASCWG/k22.png)

then it call function and thraw dest and ather string .

this function is just **strstr** 

[![23](/assets/images/ASCWG/k25.png)](/assets/images/ASCWG/k25.png)

So it searches for the word ida or ghidra in the process image name.

[![24](/assets/images/ASCWG/k23.png)](/assets/images/ASCWG/k23.png)

if its found it will set a byte in shared struct to 1 , and call a function 
