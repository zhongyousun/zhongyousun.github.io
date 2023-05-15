---
title: "SQL-partition by 實現資料分組"
date: 2023-02-07
draft: false
slug: "tutorials/sql-partition-by"
thumbnail: "" # Thumbnail image
description: "使用SQL partition by 實現資料分割，比較LINQ與partition by效能"
categories:
  - "程式語言"
tags:
  - "SQL"
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
mathjax: true # Enable MathJax for specific page

---
使用SQL partition by 實現資料分割，比較LINQ與partition by效能。
<!--more-->


### 前言
這次拿到的是貨車GPS定位的資料，包含貨車相關資訊、經緯度、行進方向、當下車速之類的。因為資料更新頻率每15分鐘一次，所以資料量算是非常龐大，而我只需要給出所有貨車當下的最新資訊，讓前端能顯示在地圖上。

### 使用LINQ
直覺就是直接撈取所有資料，使用```GroupBy```對車號分組，再對更新時間做降序排列然後選擇每一組中的第一項 解決。  
LINQ好好用!!😂
```C# {linenos=inline}
var vehicles = dbContext.VehicleData.ToList();

var latestVehicleData = vehicles
    .GroupBy(v => v.LicensePlate)
    .Select(g => g.OrderByDescending(v => v.Timestamp).First());

```

**但因為數據量龐大，直接撈取所有資料效能當然不會好**    

只好使用SQL查詢來提高效率了，分割資料表的話就是使用```partition by```來做分組並使用```row_number()```來取得排序後的number，取第一組就能得到跟LINQ一樣的結果。  



### 使用Partition by 
```sql {linenos=inline}
SELECT *
-- Create a subquery to get the latest direction for each vehicle
FROM (
  SELECT license_plate, direction, 
         -- Use the row_number() function to assign a unique row number to each row, 
         -- partitioned by license plate and ordered by timestamp in descending order
         row_number() over (partition by license_plate order by timestamp desc) as row_num
  FROM VehicleData
) subquery
-- Only return the rows where the row number is equal to 1, which corresponds to the latest data
WHERE row_num = 1;


```

### 結語   
LINQ雖然好用，但在一開始取資料時需要考慮到整體的資料量，跟最終要使用那些資料。若需要進行其他複雜的數據運算再考慮LINQ才能增加效率。  

>小知識  

**看到數據才知道聯結車，前面的車體跟後面的拖車都分別有自己的licence。 😎**
