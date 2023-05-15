---
title: "example"
date: 2023-01-10T10:11:38+08:00
lastmod: 2023-01-11T10:11:38+08:00
draft: true
thumbnail: "" # Thumbnail image
description: "Example article description"
categories:
  - "Program"
tags:
  - "SQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: false # Enable sidebar (on the right side) per page

---
<!--more-->
> Store information 

[La Vie Parisienne](https://www.facebook.com/lavieparisienneincebu)   
{{< googlemap "https://www.google.com/maps/embed?pb=!1m14!1m8!1m3!1d15700.78304988281!2d123.89817!3d10.3262129!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x63bfa697c164fc78!2sLa%20Vie%20Parisienne!5e0!3m2!1szh-TW!2stw!4v1672390038684!5m2!1szh-TW!2stw" >}}   
{{< figure src="pics/IMG_1253.jpeg" width="80%" >}} 

### Code  
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

{{< gist zhongyousun 275c45834a9305318d656c29a2df4734 >}}