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

## Memory Analysis 
  
The used memory dump file name is memory.mem

### Dump Registery keys 

we can start by dumping All the Registery keys from ``` HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree ``` ,  ```HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks ```

```bash
vol -f memdump.mem windows.registry.printkey.PrintKey --key Microsoft\\Windows\ NT\\CurrentVersion\\Schedule\\TaskCache\\Tree
```

```bash
vol -f memdump.mem windows.registry.printkey.PrintKey --key Microsoft\\Windows\ NT\\CurrentVersion\\Schedule\\TaskCache\\Tasks
```

In the keys thet you extracted from Tree hive you can find value in this keys called ```Id``` this Id is in form of ```{00000000-0000-0000-0000-00000000}```  , You can use this id to correlate between the keys from the tree and tasks because you will find the corresponding key in tasks with the name of the tree id.
  
In the keys under taskes there is a value called URI. It's better to extract all the URIs to a file (it will take too much time).

for now we are done with this part.

### Get SVCHOST inctance 
  
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
 
### dump SVCHOST memory 

Time to dump process , but we will dump the process memory. Unfortunately, I didn't find a way in Volatility 3 to dump the process vads only.

```bash 
vol -f memdump.mem windows.memmap.Memmap --pid 2248 --dump
```

After dumping the processes' memory, we will have a huge file. I hope I can only extract the process vads.
  
Anyway, the file size will affect only the analysis time.
  
# SVCHOST Memory Analysis 
  
Tasks' data is represented in the form of objects in SVCHOST memory.

How is this object organized ?
  
you rember the URI name !!! 

Well, this object contains many sections, but the important part is the task start bytes. In Windows (10 , Server 2019; I tested those 2 versions only), a task object starts with 16 bytes. ```N\x00T\x00 \x00T\x00A\x00S\x00K\x00/\x00``` follwed by the task URI .
the task ends with ```\x44\x44\x44\x04``` 

[![2](/assets/images/scvh/2.png)](/assets/images/scvh/2.png)

The healthy schedule task object will start and end with those 2 sections.

There are many other sections inside of this object. We may talk about it in another blog post.
  
## Task Object parse 
  
you can write your own parser based on the start and end bytes 
  
or you can use my [parser](https://github.com/DetectiveStrings/svcHostTasksParser) , In large files, it will take a long time, but it will aggressively carve the memory. 
  
i used it on the extracted SVCHOST memory 
  
```bash 
python svcTaskExtractor.py pid.2248.dmp
```
After about 30 minutes, I found the execution (BTW, you will receive output on files during execution).
  
the imporatint part of the extracted data is the URI Names on the dump.csv file 
  
[![3](/assets/images/scvh/3.png)](/assets/images/scvh/3.png)
  
## Task URI Comparsion

Now you should compare the URI from SVCHOST memory and registery.
  
If any URI is found in SVCHOST memory but not found in the registry, then this task may be malicious and it must be investigated.  
  
In my case, there was a URI key that I couldn't find in the extracted registery. 
  
[![4](/assets/images/scvh/4.png)](/assets/images/scvh/4.png)

I found another object for this task in another location ```offset 0x2539d0``` .
  
## Task Investigation 
  
The tool will parese some important data which was found in the task object sections. Hopefully, if the task is healthy, you can find the task command.

[![5](/assets/images/scvh/5.png)](/assets/images/scvh/5.png)
  
Before checking the file, we can check information from the other task instance.  
  
[![6](/assets/images/scvh/6.png)](/assets/images/scvh/6.png)

We discovered something that can indicate the deleted key from/tree (I will add more functions to the tool to pares all the key information from this section), but you will not be able to find this information most of the time.   
  
We can now check the bat file on the desk and analyze it.
  
# Conclusion 

SVCHOST is an impostant windows process , so it's one of the best choices for attackers and malware to use it on their attack chane . As a result, you can find useful information on it. 
  
Thanks For your Time 
