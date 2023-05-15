---
title: "Leetcode(Medium) 146. LRU Cache (Python)"
slug: "tutorials/leetcode-lru-cache-medium-python"
date: 2022-12-04T19:44:38+08:00
draft: flase
thumbnail: "" # Thumbnail image
categories:
  - "程式語言"
tags:
  - "Python"
  - "LeetCode"
# Theme-Defined params
lead: "" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page

---
LRU Cache算是我遇過覺得蠻有趣生活化的題目，感覺也是在很多底層系統會需要用到的觀念。  
可惜的是目前工作還沒使用到... 
<!--more-->

   
**純粹紀錄解題方法**


### 題目 
設計一個資料結構符合 Least Recently Used (LRU) cache 機制。

### 觀念
快取機制主要就是會保留較常使用的資料，淘汰不常使用的資料，以讓使用者在執行程式時，可以更加快速取得自己想要的資訊。  

利用HashMap與DoubleLinkedList搭配實作，  
每個事件代表HashMap的key與value，value則是以DoubleLinkedList產生的node，  
這樣就能知道node實際的順序。   

為了在呼叫 get 與 put 同時更新node的狀態，需要額外有兩個functions輔助  
```def insertInToHead(self, node):```  
插入此節點至LinkedList的第一個位置(head的後方、原本第一個節點的前方)

```def removeNode(self, node):```  
移除此節點(前後節點相連)  

後續不管是get 與 put，在更改node的狀態時都是使用這兩個functions。  

### 細節
1. LRUCache 初始資料結構需包含dummyHead與dummyTail，這樣才能知道最常使用的位置(dummyHead的後一個)與最不常使用的位置(dummyTail的前一個) 
2. 呼叫functions put 時，若最終HashMap長度大於capacity，需先刪除HashMap中的值(key:value)，key的value為LinkedList的最後一個節點(dummyTail的前方節點)，再刪除此節點。  

  
  

### 程式碼  
  
<!-- {{< gist zhongyousun 275c45834a9305318d656c29a2df4734 >}} -->
```Python {linenos=inline}
class LinkListed:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.size = capacity
        self.dic = {}
        self.dummyHead = LinkListed(0, 0)
        self.dummyTail = LinkListed(0, 0)
        self.dummyHead.next = self.dummyTail
        self.dummyTail.prev = self.dummyHead.next

    def get(self, key: int) -> int:
        if key not in self.dic: return -1
        node = self.dic[key]
        self.removeNode(node)
        self.insertInToHead(node)
        return node.val
        
    def put(self, key: int, value: int) -> None:
        if key in self.dic:
            node = self.dic[key]
            node.val = value
            self.removeNode(node)
            self.insertInToHead(node)
        else:
            node = LinkListed(key, value)
            self.dic[key] = node
            self.insertInToHead(node)
            
            if len(self.dic) > self.size:
                del self.dic[self.dummyTail.prev.key]
                self.removeNode(self.dummyTail.prev)
        
    def insertInToHead(self, node):
        node.next = self.dummyHead.next
        self.dummyHead.next.prev = node
        self.dummyHead.next = node
        node.prev = self.dummyHead
        return
        
    def removeNode(self, node):
        node.next.prev = node.prev
        node.prev.next = node.next
        return

```
<!-- 
### 程式碼

```python
class LinkListed:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None
class LRUCache:

    def __init__(self, capacity: int):
        self.size = capacity
        self.dic = {}
        self.dummyHead = LinkListed(0, 0)
        self.dummyTail = LinkListed(0, 0)
        self.dummyHead.next = self.dummyTail
        self.dummyTail.prev = self.dummyHead.next

    def get(self, key: int) -> int:
        if key not in self.dic: return -1
        node = self.dic[key]
        self.removeNode(node)
        self.insertInToHead(node)
        return node.val
        

    def put(self, key: int, value: int) -> None:
        if key in self.dic:
            node = self.dic[key]
            node.val = value
            self.removeNode(node)
            self.insertInToHead(node)
        else:
            node = LinkListed(key, value)
            self.dic[key] = node
            self.insertInToHead(node)
            
            if len(self.dic) > self.size:
                del self.dic[self.dummyTail.prev.key]
                self.removeNode(self.dummyTail.prev)
        
    def insertInToHead(self, node):
        node.next = self.dummyHead.next
        self.dummyHead.next.prev = node
        self.dummyHead.next = node
        node.prev = self.dummyHead
        return
    def removeNode(self, node):
        node.next.prev = node.prev
        node.prev.next = node.next
        return


```
-->