---
title: "SQL-implement data partitioning (partition by)"
date: 2023-02-07
draft: false
slug: "tutorials/sql-partition-by"
thumbnail: "" # Thumbnail image
description: "Implement data partitioning using SQL partition by and compare the performance with LINQ."
categories:
  - "Program"
tags:
  - "SQL"
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
mathjax: true # Enable MathJax for specific page

---
Implement data partitioning using SQL partition by and compare the performance with LINQ.
<!--more-->


### Foreword
The data I received this time is the GPS positioning data of the trucks, including truck-related information, latitude and longitude, direction of travel, current speed, etc. The data volume is quite large because the data update frequency is once per 15 minutes. However, I only need to provide the latest information of all trucks, so that the front-end can display it on the map.  

### Using LINQ
Intuitively, it's just to directly retrieve all the data, use ```GroupBy``` to group the vehicles by license plate, sort the update time in descending order, and then choose the first item in each group.  
LINQ is very convenient!!😂
```C# {linenos=inline}
var vehicles = dbContext.VehicleData.ToList();

var latestVehicleData = vehicles
    .GroupBy(v => v.LicensePlate)
    .Select(g => g.OrderByDescending(v => v.Timestamp).First());

```

**But the data volume is large, the performance of directly retrieving all the data is certainly not good.**      

So I have to use SQL queries to improve efficiency. To partition the data table, then use the ```partition by``` to group and use ```row_number()``` to get the number after sorting. By taking the first group, you can get the same result as LINQ. 



### Using partition by 
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

### Conclusion  
Although LINQ is easy to use, one must consider the overall data volume and what data will be used when initially retrieving the data. If other complex data operations are required, then consider using LINQ to increase efficiency.  


>Fun fact   

**Upon seeing the data, I realized that if the vehicle is a trailer truck, it means that both the front and the back have their own separate license plates. 😎**
