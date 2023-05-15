---
title: "SQL-merge into 語法紀錄"
slug: "tutorials/sql-merge-into-statement"
date: 2022-12-22T10:44:38+08:00
draft: flase
thumbnail: "" # Thumbnail image
categories:
  - "程式語言"
tags:
  - "SQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---


這次被指派實作不同資料庫同步的功能，公司現有一個ERP系統，  

資料來源一律以此為標準，讓客戶可在自家系統新增、編輯、寫入ERP、與ERP同步...  
<!--more-->
whatever~ 總之就是資料寫入EPR後，就是以ERP為準，也因為ERP會定期更動資料，  

在客戶自家系統已寫入ERP的資料需要頻繁同步。


### 解決方法
原本想法是直接撈取兩個資料庫資料，逐一做比對，比對有差異的就直接在專案裡生成update的sql語法，  

最後一起更新。(我是覺得蠻直觀的，反正就是硬幹)  

不過主管建議可以用 merge into 的做法，直接在資料庫裡實作就好，於是就要我去查一查使用方式。
  

### 程式碼  
Merge into語法如下：
```sql {linenos=inline}
--sql
MERGE INTO table_a target -- 需同步的資料(自家系統)
USING (select * from table_b ) source -- 基準資料(ERP)
ON (target.A=source.A and target.B=source.B) -- 兩張表對應的key 
WHEN MATCHED THEN      -- 若有找到對應key的資料，則執行以下動作
    UPDATE             -- 這裡我是對表做編輯
    SET target.E = source.E, 
        target.D = source.D 
WHEN NOT MATCHED THEN  -- 若沒有找到對應key的資料，則執行以下動作(可以省略)
    INSERT (target.E,target.D) VALUES (source.E,source.D); 

```

### 結論  
語法簡潔是蠻方便的，也不用同時執行多筆update語法。  

最後是沒比較兩種方法實際運行上的速度差異，但因為Merge into的做法不用逐一比對欄位是否有差異，   

而是只要有比對到對應的key就直接執行編輯的動作，又是直接在資料庫執行，應該是會快上不少。    


但缺點也是很明顯，因為不管如何有資料就是直接執行編輯的動作，對於有log紀錄的資料表，應該會增加蠻大的負荷跟資料量。
