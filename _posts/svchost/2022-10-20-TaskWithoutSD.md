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

When the task condition is true , An svchost instance will execute the command which is represented by the task Action value from ``` HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\{task triger}```

What will happen if the task SD value is removed from the registery ?

Nothing will happen, and SVChost will continue with the task execution, despite the fact that the task condition is ture. 

In this case, Svchost depends on its own memory data to execute the task command.

you can read more detailes from here [2](https://www.microsoft.com/en-us/security/blog/2022/04/12/tarrask-malware-uses-scheduled-tasks-for-defense-evasion/)

So we will investigate this Svchost process memory to collect information that can help us detect the malicious task.

# Lab Desription 

In this blog post, I will not provide the used dump.

Machine os : windows 10 

Instead of dumping svchost process memory only , I did a full memory dump to simulate what would happen in a real case.

Machine memory size : 10 GB 

Imaging Tool : [FTkimager](https://accessdata.com/product-download/ftk-imager-version-4-5) 

Memory Analysis Tool : [volatility 3](https://github.com/volatilityfoundation/volatility3) 

Svchost Memory Analysis Tool : [My own svchost tasks extractor](https://github.com/DetectiveStrings/svcHostTasksParser) ```The full version will be available in days ```

To simulate a case, you need to schedule a new task , then remove the task registry from (Tasks/{} , Tree/TaskName ) and remove the XML file from system32/Tasks/<TaskName>. 

You can follow this [video](https://www.youtube.com/watch?v=xrd0w505aS8) 
  
