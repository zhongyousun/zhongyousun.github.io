---
title: "SQL SCOPE_IDENTITY() 與 @@IDENTITY 取得自動增量值差異"
date: 2023-03-20T10:44:19+08:00
draft: false
slug: "tutorials/sql-scopeidentity-identity-diff"
thumbnail: "img/thumbnail/023.png" # Thumbnail image
description: "SQL取得最近新增一筆的資料"
categories:
  - "程式語言"
tags:
  - "SQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
SQL Server使用 SCOPE_IDENTITY() 取得最近新增一筆的資料，並比較 SCOPE_IDENTITY() 與 @@IDENTITY 取值差異。  
<!--more-->
{{< br 3  1>}}

### 需求
前端畫面在新增資料時，很常需要對此資料做後續的處理，需要此次新增資料的IDENTITY。或是後端資料庫若有資料變動時常會有觸發器進而更改或是新增資料，也需要當下新增的資料的IDENTITY做後續處理的需求。此篇記錄一下SQL Server使用 SCOPE_IDENTITY() 與 @@IDENTITY 的實際差異。
{{< br 3  2>}}

### 原理
`SCOPE_IDENTITY()` 函數是返回當前作用域中最近插入的自動增量值。也就是說，它只會返回當前執行 INSERT 陳述式時所產生的自動增量值，並不會受到任何觸發器所產生的自動增量值的影響。

`@@IDENTITY` 函數是返回最後一個 INSERT 陳述式所產生的自動增量值，而不管是在哪個作用域內產生的。所以`如果INSERT 陳述式中包含了觸發器`，則 `@@IDENTITY` 函數可能會返回`不是預期`的自動增量值。

講起來有點複雜，直接實作範例。  
{{< br 3  3>}}

### 範例

1.建立範例資料表
```SQL {linenos=inline}
CREATE TABLE ExampleTable (
  ID INT PRIMARY KEY IDENTITY(1,1),
  Name NVARCHAR(50) NOT NULL
);
```

2.建立觸發器，若有資料新增，則再複製一筆相同資料
```SQL {linenos=inline}
CREATE TRIGGER ExampleTrigger
ON ExampleTable
FOR INSERT
AS
BEGIN
  INSERT INTO ExampleTable (Name)
  SELECT Name FROM INSERTED
END;
```
3.新增 Name 為 Alvin 的資料，同時會啟動觸發器，資料表會有兩筆Alvin。
```SQL {linenos=inline}
INSERT INTO ExampleTable (Name)
VALUES ('Alvin');

SELECT SCOPE_IDENTITY()
SELECT @@IDENTITY

DROP TABLE ExampleTable;
```
資料表內容如下
|  ID   | Name  |
|  ----  | ----  |
| 1  | Alvin |
| 2  | Alvin |

`SELECT SCOPE_IDENTITY()` 結果為 1  
`SELECT SELECT @@IDENTITY`  結果為 2  

可以很明顯看出，SCOPE_IDENTITY()就是取得當下 insert 的ID，而 @@IDENTITY 則是會取得觸發器新增資料的ID。
{{< br 3  4>}}

### 結論
在取得自動增量值時，如果是要對此次新增的資料做後續處理，請使用 SCOPE_IDENTITY()，使用 @@IDENTITY 有可能會取到因為此次新增而額外新增的值。





