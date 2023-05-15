---
title: "MongoDB Atlas Basic Concepts & Using Compass to Connect MongoDB"
date: 2023-04-24T15:33:44+08:00
draft: false
slug: "tutorials/mongodb-atlas-compass-tips"
thumbnail: "img/thumbnail/026en.webp" # Thumbnail image
description: ""
categories:
  - "Program"
tags:
  - "NoSQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
Record the key concepts and detailed connection operation tutorial of learning MongoDB Atlas.
<!--more-->


### 1. Foreword

I'm used to working with RDBMS (relational databases), but when working on a personal project, deploying a database online always incurs significant costs. I stumbled upon MongoDB's cloud service, Atlas, which surprisingly provides a free learning version. Therefore, I decided to learn how to use MongoDB, which is a document database belonging to the NoSQL (Not Only SQL) category.  

Here is the official introduction:  
> MongoDB Atlas is a fully-managed cloud database service provided by MongoDB that offers a simple, secure, and reliable way to run MongoDB without any configuration or infrastructure management. MongoDB Atlas runs on cloud platforms such as AWS, Azure, and Google Cloud, enabling users to easily run, scale, and manage MongoDB globally. It provides automated deployment and management, including backup and recovery, monitoring, performance optimization, and security. With MongoDB Atlas, you can easily configure, deploy, and manage MongoDB clusters and use its various tools and services to simplify MongoDB development and operations. For example, it offers a user-friendly management console that allows you to easily manage your MongoDB clusters, users, and permissions. In addition, MongoDB Atlas provides advanced features such as full-text search, graph search, views, aggregation pipelines, and more, which can help you handle massive data and complex queries more easily.  

### 2. When to use NoSQL
Here are some scenarios where using MongoDB is appropriate:  

1. Unstructured data: If the data format to be stored is uncertain, MongoDB can be a suitable choice, such as logs, multimedia, web content, social media data, etc.

2. Scalability: MongoDB can easily scale to handle more data by adding new nodes or shards.

3. High performance: MongoDB is a high-performance database that can handle fast read/write operations, usually faster than relational databases.

4. Flexible data model: MongoDB's document data model is very flexible and can be changed according to the needs of the application without changing the structure of the entire database.

Based on these points, MongoDB is a good option when data structure is uncertain, scalability is important, and there is no need to use joins between tables. It is also a good choice when fast read/write operations are necessary.

### 3. Atlas sign-up
As it is a database service provided directly through cloud services, the process is very simple. After registering an account on the [MongoDB](https://www.mongodb.com/  "MongoDB")  official website, simply select Atlas in the product section and follow the steps to create basic information and connection permissions, allowing you to use it directly.  

### 4. Authority Management
In MongoDB Atlas, you can create multiple organizations, each of which can have multiple projects, and each project can have multiple clusters. Here are the differences between them:

1. Organization: An organization is the highest level in MongoDB Atlas. You can register and create an organization with an email address and invite other users to join. You can assign different projects to different teams or users, and manage all resources and settings in the organization.

2. Project: A project is a collection of one or more MongoDB clusters. In a project, you can configure related security, monitoring, and automation features, as well as access permissions and notification rules. A project can be owned and managed by a team or a user.

3. Cluster: A cluster is an instance of MongoDB, which is a distributed system composed of a group of MongoDB servers. Each cluster has its own storage space, computing resources, and settings, and is managed and monitored by MongoDB Atlas. You can configure various options in each cluster, such as available regions, database versions, storage engines, and security settings.

In summary, these three levels are used to manage and regulate the usage permissions of each database, from top to bottom.

### 5. Database Connection
I prefer using a GUI to operate databases, and MongoDB also provides a GUI for users to operate databases. You can go to [MongoDB Compass Download (GUI)](https://www.mongodb.com/try/download/compass  "MongoDB Compass Download (GUI)") and download MongoDB Compass in the Tools section.  

After installation, when you run it, you will be prompted to enter the string used to connect to the database. The connection string can be found in the Atlas Cluster management interface on the right side, where you can click on "Connect" to find various connection options.
{{< figure src="pics/connect.webp" width="80%" alt="" title="">}}  

Since we are using Compass to connect, we click on Compass and the screen will display the connection string for Compass.  
```
mongodb+srv://<user>:<password>@cluster0.swvkd15.mongodb.net/test
```
`<user>` `<password>` refer to the account username and password created when setting up the Cluster initially.  

After copying the connection string and modifying `<user>` and `<password>`, **remember to configure the allowed IP addresses in the Network Access settings**.  
{{< figure src="pics/Network Access.webp" height="10%" alt="" title="">}}  

After setting up the IP address for network access, paste the connection string in MongoDB Compass to establish a successful connection.  
### 6. Database Structure
In MongoDB, data is stored in the form of documents, and a collection is a collection of documents. In other words, a MongoDB database can contain multiple collections, and each collection contains multiple documents.  

Compared to relational databases, it like below
|  MySQL   | MongoDB  |
|  :----:  | :----:  |
| database  | database |
| table  | collection |
| per row  | document |

In MongoDB, data is stored in the form of documents. A collection is a collection of documents. In other words, a MongoDB database can contain multiple collections, and each collection contains multiple documents.  

Unlike relational databases, MongoDB documents can contain highly flexible structures and content, and there is no need to define fields beforehand.  

In simple terms, a collection is where documents are stored, and a document is the container that actually stores the data. If MongoDB is viewed as a document database system, then a collection is the table in this system, and a document is the row in this table.  
### 7. Conclusion 
Have a basic understanding of MongoDB's data structure and how to connect to an Atlas cloud database. The next step is to learn how to perform MongoDB read and write operations using node.js.



