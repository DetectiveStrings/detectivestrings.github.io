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
