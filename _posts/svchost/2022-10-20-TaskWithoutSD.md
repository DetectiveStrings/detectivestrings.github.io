---
title: "May svchosts guid you"
classes: wide
header:  
  teaser: /assets/images/svch/svchosts1.gif
ribbon: red
description: "Malware can use hidden scaduald tasks to be persistent on your system and evade your defences. If you want to know how to investigate this case, just follow me."
categories:
  - Memory analysis 
  - Svchost Analysis
  - DFIR 
toc: true
---

# Introduction .
Windows scheduled tasks are essential for automating many important tasks, such as Windows updates, checking Windows services, backing up data, and so on.

you can read more about scadule task creation from here [1](https://www.windowscentral.com/how-create-automated-task-using-task-scheduler-windows-10) 

after the task creation a new registey keys will be created on  
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree 
``` 
and 
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks 
```
A new XML configuration file with the task name will be added to the directory ```windows/system32/Tasks``` as well.

When the task condition is true , An SVCHOST instance will execute the command which is represented by the task Action value from ``` HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\{task triger}```

What will happen if the task SD value is removed from the registery ?

Nothing will happen, and SVCHOST will continue with the task execution, despite the fact that the task condition is ture. 

In this case, SVCHOST depends on its own memory data to execute the task command.

you can read more detailes from here [2](https://www.microsoft.com/en-us/security/blog/2022/04/12/tarrask-malware-uses-scheduled-tasks-for-defense-evasion/)

So we will investigate this SVCHOST process memory to collect information that can help us detect the malicious task.

# Lab Desription 

Machine os : windows 10 

Instead of dumping SVCHOST process memory only , I did a full memory dump to simulate what would happen in a real case.

Machine memory size : 10 GB 

Imaging Tool : [FTkimager](https://accessdata.com/product-download/ftk-imager-version-4-5) 

Memory Analysis Tool : [volatility 3](https://github.com/volatilityfoundation/volatility3) 

SVCHOST Memory Analysis Tool : [My own SVCHOST tasks extractor](https://github.com/DetectiveStrings/svcHostTasksParser) ```The full version will be available in days ```

To simulate a case, you need to schedule a new task , then remove the task registry from (Tasks/{} , Tree/TaskName ) and remove the XML file from system32/Tasks/<TaskName>. 

You can follow this [video](https://www.youtube.com/watch?v=xrd0w505aS8) 
  
Then you can image the memory using FTKimager and start working on the dump.
  
In this blog post, I will not provide the used dump.

# Investegation    

The used memory dump file name is memory.mem
  
## Memory 
  
We are not working with a normal process, Scheduled tasks are being run in a thread of the SVCHOST process . 
  
However, on a typical machine, there may be more than ten instances of SVCHOST.exe running, so we must determine which one is in charge of running scheduled tasks. 
  
How does SVCHOST start the task ? 
  
It should load a module/DLL that can help run the task. Task Scheduler Service (schedsvc.dll) is the most well-known module for this mission. 
  
SVCHOST loaded this module, so we need to figure out which SVCHOST loaded it, fortunately only one instance. 

using Volatility 3 (volatility 2 will not work in the new Windows version) 
  
```bash 
vol -f memdump.mem windows.dlllist.DllList  | grep -i schedsvc.dll
```

Now we have the SVCHOST inctance pid 
  
[![1](/assets/images/scvh/1.png)](/assets/images/scvh/1.png)
  
| ![space-1.jpg](/assets/images/scvh/1.png) ||:--:|| <b>Image Credits - Fig.2 - 4K Mountains Wallpaper</b>|
