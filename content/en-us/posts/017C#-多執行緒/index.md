---
title: "C# Multi-threading principle, asynchronous usage and parameter with custom objects"
slug: "tutorials/Csharp-multithreading"
date: 2023-01-19T11:49:15+08:00
draft: false
thumbnail: "" # Thumbnail image
description: "C# multi-threading principle, asynchronous usage and parameter with custom objects records"
categories:
  - "Program"
tags:
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
toc: ture
sidebar: right
---
The principle of C# multi-threading, asynchronous usage and parameter with custom objects records.  
None of the projects encountered so far have a hard requirement to use multi-threading. This article is to record my own understanding and learning process.
<!--more-->

### Foreword
I had a project that has a file pre-store function, because there are new files to be updated every day, it needs to be executed pre-store function every day. The original way of programming is to download part A and then download part B. I would like to try new way with multi-threaded. The main reason why I want to try jsut it is too annoying when I test part B every time in the process, I always need to wait a lot of time (the amount of data is too large) until part A to finish.    

### Principle
First you must understand the difference between Program /Process/ Thread.

1. **Program** : Program code that has not yet been loaded into memory.   
2. **Process** : For a Program that has been executed and loaded into the memory, each line of code in the program may be executed by the CPU at any time.
3. **Thread** : There will be many Threads in the same Process, and each Thread is responsible for a certain function.  

Take the factory as an example:

**Program** = Detailed manufacturing drawings and staffing of the factory  
**Process** = factory already in operation  
**Thread** = Every worker in the factory  

When programs are executed, they are executed line by line in order, similar to the way workers in a factory complete their tasks in order. This will cause the problem that the program is executed for too long and inefficiency. If the workers can perform their own tasks at the same time (multiple threads), the efficiency will be much faster, but the disadvantage is that when the hardware is insufficient (similar to factory space too small), too many workers performing tasks together will rob each other of resources and cause the program **Deadlock**.  

### Use
Situations where multithreading is commonly used include:  

1. Multiple tasks need to be executed at the same time, and there is no relationship between these tasks. For example, downloading multiple files at the same time or calculating multiple values.
2. Need to wait for a long-running task, but don't want to make the interface uninteractive. For example, executing a database query in another execution line running at the same time.
3. Need to perform a task, but do not want to block other tasks because of it. For example, executing a network connection in another execution line running at the same time.
4. Tasks need to be executed on multiple cores to take advantage of the performance of multi-core CPUs.

### Example
```C# {linenos=inline}
using System.Threading;

Thread thread1 = new Thread(Task1); 
thread1.Start();

Thread thread2 = new Thread(Task2);
thread2.Start();

Console.WriteLine("Completed");
Console.ReadKey();

static void Task1()
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine("Task 1");
        Thread.Sleep(1000);
    }
}

static void Task2()
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine("Task 2");
        Thread.Sleep(1000);
    }
}

```
Result:
```
Completed
Task 1
Task 2
Task 1
Task 2
Task 1
Task 2
Task 1
Task 2
Task 2
Task 1

```
**Why does "Completed" appear first?**  
This is because in the above code, two threads are started at the same time, and the main thread does not wait for either thread to complete, but immediately writes "Completed" and ends the program. Therefore, no matter which thread completes first, the program will write "Completed" first.  

If you want to wait for thread2 to complete, you can use thread2.Join() to make the main thread wait for thread2 to complete, and then execute ```Console.WriteLine("Completed");```.  

### Pass parameters with custom objects
In actual use, the thread often needs to be executed with parameters. You can use ParameterizedThreadStart. ParameterizedThreadStart is a delegate, which points to a method with an object parameter and no return value. So when using ParameterizedThreadStart delegation, you need to pass in an object type parameter.  
But usually the data type is not object, most of them are JObject or self-defined types, the following is the implementation method.   

Create corresponding objects for different execution threads  
```C# {linenos=inline}
// Create an object to hold the list
class ListContainer1
    {
        public List<dynamic> TheList { get; set; }
    }
class ListContainer2
    {
        public List<customize> TheList { get; set; }
    }


```
Pass ```List<dynamic>``` customize_obj to Task1, get customize_obj in Task1 function and execute

```C# {linenos=inline}
List<dynamic> customize_obj  = new List<dynamic>();
var listContainer1 = new ListContainer1 { TheList = customize_obj };
// create new thread
Thread t1 = new Thread(new ParameterizedThreadStart(Task1));
// pass parameters and start thread
t1.Start(listContainer1);


void Task1(Object listContainer)
{
    // get customize_obj
    ListContainer1 container = (ListContainer1)listContainer;
    List<dynamic> customize_obj = container.TheList;
    //...
    //code
    //...
}

```  
  


### Asynchronous programming with async and await(to be continued)