---
title: "SQL-merge into statement"
slug: "tutorials/sql-merge-into-statement"
date: 2022-12-22T10:44:38+08:00
draft: flase
thumbnail: "" # Thumbnail image
categories:
  - "Program"
tags:
  - "SQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---

I was assigned to implement the function of synchronizing from different databases.   
The company currently has an ERP system.  
All data sources are based on this standard, so that customers can create , update, write to ERP,   
and synchronize with ERP in their own systems...
<!--more-->
whatever ~ In short, after the data is written into the EPR, it is based on the ERP, and because the ERP will update the data regularly,  
The data that has been written into the ERP in the customer's own system needs to be synchronized frequently.


### Solution
The original idea was to directly fetch the data from the two databases and compare them one by one. If there is a difference in the comparison, directly generate the SQL statement of the update in the project.  
Final step executes the statement of update together. (I think it is quite intuitive, anyway, it is hard work) 

However, the colleague suggested that I can use the method of merge into and implement it directly in the database, so I check the usage method.
  

### Code  
Merge into statement is as follows：
```sql {linenos=inline}
--sql
MERGE INTO table_a target -- Data to be synchronized (own system)
USING (select * from table_b ) source -- Original data (ERP)
ON (target.A=source.A and target.B=source.B) -- join condition
-- If the data is found, execute the following statement
WHEN MATCHED THEN      
    UPDATE             -- execute update
    SET target.E = source.E, 
        target.D = source.D 
-- If no data, execute the following statement (can be omitted)
WHEN NOT MATCHED THEN  
    INSERT (target.E,target.D) VALUES (source.E,source.D); 
```

### Conclusion  
Concise statement is quite convenient, and there is no need to execute multiple update statement at the same time.  
However, there is no comparison of the actual program execution time difference between the two methods, but the Merge into method does not need to compare the fields one by one to check if there is a difference, 
Instead, as long as there is a comparison to the corresponding key, the update statement is directly executed, and it is executed directly in the database, which should be much faster.

But the disadvantages are also obvious, because no matter if data is changed, the update statement is executed directly. For the data table with log records, it should increase the load and data volume quite a lot.
