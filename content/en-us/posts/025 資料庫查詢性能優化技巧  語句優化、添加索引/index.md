---
title: "Database Query Performance Optimization Tips : Statement Optimization, Adding Indexes"
date: 2023-03-29T09:38:14+08:00
draft: false
slug: "tutorials/database-optimization-index-tips"
thumbnail: "img/thumbnail/025en.webp" # Thumbnail image
description: ""
categories:
  - "Program"
tags:
  - "SQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
As the system data grows increasingly large, it's common to encounter situations where database queries take too long. This article provides several methods for optimizing database query performance.

<!--more-->
---
Mainly records several techniques that I have used, including optimizing query statements, optimizing indexes, and reducing the amount of data in the database.

{{< br 5 2 >}}

## 1. Optimizing query statements
Here are several tips that we need to pay attention to when writing SQL statements.

**Avoid using `*` in SELECT queries**  

* Performance: SELECT * will query all columns in the entire table, including unnecessary ones. This increases the workload on the database and reduces query efficiency. Only querying the required columns can reduce the amount of data queried, thereby improving query efficiency.  

* Readability: SELECT * does not clearly indicate which columns are being queried, which can make the code difficult to understand and maintain. Explicitly specifying the required columns can make the code clearer and easier to understand.  

* Conflicts: Using SELECT * may lead to conflicts if there are columns with the same name in the table, such as when using JOIN. Using explicit column names can avoid such problems.  


**Avoid using `!=` or `<>` in the WHERE clause**   

Using this statement is equivalent to querying the entire table, which leads to a decrease in performance.  

**Avoid using `OR` in the WHERE clause**  

The `OR` operator also requires searching the entire table to find data that matches either of the conditions, leading to a decrease in performance. Instead, you can use `UNION ALL` to retrieve data that matches each condition separately and then combine them.  

**Avoid using expressions and functions in the WHERE clause**  

Ex:   
```
select name from table where age-10 = 20
``` 

Basically, this kind of writing will first subtract 10 from the entire "age" column of the table and then compare whether it is equal to 20, so it should be changed to: 

```
select name from table where age = 30
``` 

Of course, this example is a bit extreme and not normally written like this. The main point is to **avoid using expressions or functions on the left side of the equal sign to modify column data, and instead modify it on the right side**.









## 2. Adding Indexes 

An index is a data structure used in database management systems to speed up data queries. By sorting a column or a combination of columns, an index creates a data structure that can accelerate the speed of queries and sorting. In a database, an index can be seen as a pointer or address that points to the actual data.      

Simply put, adding an index for a specific column (e.g., the column containing data A, B, C, D, etc.) can help speed up queries searching for data related to a specific value (e.g., querying for data where the column equals B). This is because the index allows the database to quickly locate the block of data related to the value B, rather than searching through all blocks containing A, B, C, D, etc., which can increase query speed.  

### ● Recommendations situation for adding indexes 

Indexes can be created on a single column or multiple columns. The following are some situations where adding an index is recommended:    

* `The primary key and foreign key` : Both are one of the most important columns in a database table, and they are typically used as a linking condition for queries and table relationships. Creating indexes on these columns can make querying and relationship operations faster.

* `Frequently queried columns`: If a column is frequently used as a query condition, creating an index can significantly improve query efficiency.   
`!!`**Be aware of uniqueness and distribution issues.**  
Ex : The gender column is not suitable for indexing because it only has two values, male and female, and does not have uniqueness, so it cannot be used as a unique key or primary key column, and indexing it will not speed up queries. The distribution of gender values in the column is relatively even and does not cause a large amount of data to be concentrated on a particular value, so not creating an index has a smaller impact on query efficiency.  

* `Combinatorial query conditions` : When multiple columns are used together as query conditions, a composite index can be created. A composite index can help the database system locate data that matches multiple conditions more quickly.  

*  `Columns used for sorting` : If a column is frequently used for sorting, creating an index can significantly improve the efficiency of sorting operations.  

*  `Columns used for grouping` : An index on grouping columns can greatly improve the efficiency of grouping operations if a column is frequently used for grouping.  
---
### ● Note considerations

Although it may seem that many situations can benefit from adding indexes, **more indexes do not necessarily mean better performance**. Too many indexes can increase query complexity and result in lower efficiency. Therefore, determining which situations require adding indexes often requires experience and judgement.  

While indexes can speed up query performance, in reality, they trade space for time. In the example of a column with data such as A, B, C, D, etc., adding an index to this column means that there will be additional data tables storing parts of the data (such as A, B, C, D, etc.) for faster querying. Therefore, the server's hardware environment and capacity to handle this additional data also need to be considered.   

## 3. Reducing the amount of data in the database   

Currently, the main methods for reducing the amount of data in a database through querying include normalization, compressing the database, and deleting data.   

* `Normalization` : Is a process in database design that involves dividing data into smaller, more related parts to reduce data duplication, improve database efficiency, and enhance flexibility.

* `Database compression` : Compressing database files is a technique that can increase performance depending on the compression method and ratio used. Generally, if the compression ratio is high, it may take longer to decompress data, which could slow down queries. If the compression ratio is low, it can reduce disk I/O operations and improve query performance. It is recommended to seek assistance from a professional DBA for this task.

* `Deleting data`: Delete redundant or duplicate data as needed.   

I think the part about deleting data should be paid more attention to at the beginning, when designing the schema and deciding what data to store. It is better to consider it thoroughly beforehand, rather than waiting for a large amount of data to accumulate before figuring out how to delete it. (Of course, if there is a need for system merging, there may be no other option.)  

{{< br 5 1 >}}

The above are the techniques that I have actually implemented, and I will supplement the others once I have tried them.😎


