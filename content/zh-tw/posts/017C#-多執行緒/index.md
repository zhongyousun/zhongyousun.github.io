---
title: "C# 多執行緒原理、非同步用法與帶入自定義物件"
slug: "tutorials/Csharp-multithreading"
date: 2023-01-19T11:49:15+08:00
draft: false
thumbnail: "" # Thumbnail image
description: "C# 多執行緒原理、非同步用法與帶入自定義物件紀錄"
categories:
  - "程式語言"
tags:
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
toc: ture
sidebar: right
---
C# 多執行緒原理、非同步用法與帶入自定義物件學習過程紀錄。  
目前遇到的專案都沒有硬性需求使用到多執行緒，此篇為紀錄自己的理解與學習的過程。
<!--more-->

### 前言
公司專案有個檔案預先儲存的功能，因為每天都有新檔案，需要每天跑排程執行。原本的寫法，是A的部分下載完，再下載B的部分。會想試試看改用多執行緒的方式寫的原因，主要只是覺得在測試時資料量太大，需要先等A跑完才能測試B的部分很煩而已。😎  

### 原理
得先了解Program(程式)/Process(程序)/Thread(執行緒)的差異。

1. **Program(程式)** : 還尚未載入記憶體的程式代碼。 
2. **Process(程序)** : 已經執行並且載入到記憶體中的 Program ，程序中的每一行程式碼隨時都有可能被CPU執行。  
3. **Thread(執行緒)** : 同一個 Process 中會有很多個 Thread ，每一個 Thread 負責某一項功能。  

以工廠來舉例:  

**Program(程式)** = 工廠的詳細製造圖與人員配置  
**Process(程序)** = 已經在運行的工廠  
**Thread(執行緒)** = 工廠裡的每位工作人員  

普遍程式執行時都是照著順序一行一行執行，相似於工廠裡的工人照著順序完成自己的任務。這就會產生程式執行過久，工人運用效率不高的問題，如果工人們可以同時執行各自的任務(多執行緒)，這樣效率就會變快很多，但缺點就是硬體不足時(工廠空間太小)，太多工人一起執行任務就會互搶資源導致程式**死結(Deadlock)**。

### 用途
使用多執行緒的情況通常包括：

1. 需要同時執行多個任務，而這些任務之間沒有關聯。例如，同時下載多個文件或計算多個值。
2. 需要等待一個長時間運行的任務，但不想讓界面變得不可互動。例如，在同時運行的另一個執行線中執行資料庫查詢。
3. 需要執行一個任務，但不想因為它阻塞其他任務。例如，在同時運行的另一個執行線中執行網路連接。
4. 需要在多個核心上執行任務，以利用多核 CPU 的性能。

### 範例
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
結果:
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
**為什麼會先出現 "Completed" ?**  
這是因為在上面的程式碼中，兩個執行緒被同時啟動，並且主執行緒不等待其中任何一個執行緒完成，而是立即寫出 "Completed" 並結束程式。因此，無論哪個執行緒先完成，程式都會先寫出 "Completed"。

如果想要等待 thread2 完成，可以加上 thread2.Join() 使主執行緒等待 thread2 執行完畢，再接著執行```Console.WriteLine("Completed");```。

### 帶入自定義參數
實際使用時執行緒常會有帶入參數的需求，可以使用ParameterizedThreadStart，ParameterizedThreadStart是一個委派，它指向一個object參數並沒有返回值的方法。所以在使用ParameterizedThreadStart 委派時，需要傳入一個object類型參數。  
但通常資料類型都不會是object，大部分是JObject或是自定義的類型，以下是實作方式。  

針對不同執行緒建立對應的物件
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
傳遞 ```List<dynamic>``` customize_obj 至 Task1，在 Task1 函式中取得 customize_obj 並執行
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
  


### 非同步方法(待續)