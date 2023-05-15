---
title: "MongoDB Atlas 資料庫基礎觀念、操作流程與使用 Compass 連線"
date: 2023-04-24T15:33:44+08:00
draft: false
slug: "tutorials/mongodb-atlas-compass-tips"
thumbnail: "img/thumbnail/026.webp" # Thumbnail image
description: ""
categories:
  - "程式語言"
tags:
  - "NoSQL"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
紀錄學習MongoDB Atlas過程幾個重點觀念與詳細連線操作教學。
<!--more-->


### 1. 前言

平常使用慣了RDBMS(關聯式資料庫)，但因為在做個人專案時，若要部署上線資料庫的花費總是不小，偶然發現了MongoDB的雲端服務Atlas竟然有提供免費的學習版，所以來學習使用一下這種屬於document database，NoSQL（Not Only SQL）的資料庫 MongoDB。

以下是官方介紹  
> MongoDB Atlas 是 MongoDB 官方提供的全托管雲資料庫服務，它提供了一個簡單、安全和可靠的方式來運行 MongoDB，無需進行任何基礎架構的配置和管理。MongoDB Atlas 運行在 AWS、Azure 和 Google Cloud 等雲平台上，讓用戶可以輕鬆地在全球範圍內運行、擴展和管理 MongoDB。它提供了自動化的部署和管理，包括備份和恢復、監控、性能優化、安全性等。使用 MongoDB Atlas，您可以輕鬆地配置、部署和管理 MongoDB 集群，並使用它提供的各種工具和服務來簡化 MongoDB 開發和運營。例如，它提供了一個簡單易用的管理控制台，讓您可以輕鬆地管理您的 MongoDB 集群、用戶和權限。此外，MongoDB Atlas 還提供了一些高級功能，例如全文搜索、圖形搜索、視圖、聚合管道等。這些功能可以幫助您更輕鬆地處理海量數據和複雜查詢。  

### 2. NoSQL 使用時機
MongoDB設計目的是處理大量非結構化數據，上網找到了一些適合使用MongoDB的情況。  
1. 非結構化數據：如果今天需要儲存的資料格式有著不確定性，就可以選擇，例如日誌、多媒體、網頁內容、社交媒體數據等。
2. 可擴展性：MongoDB可以輕鬆擴展，通過添加新的節點或分片，以處理更多的數據。
3. 高性能：MongoDB是一個高性能的資料庫，它可以處理高速的讀寫操作，通常比關聯式資料庫更快。
4. 靈活的數據模型：MongoDB的文檔數據模型非常靈活，可以根據應用程序的需要進行更改，而不需要改變整個資料庫的結構。  

目前所理解的，如果在資料上不確定資料結構，或是想保留擴充性，之後預期不太會需要使用join各種表，且需要較快的讀寫操作，就可以考慮使用NoSQL類型的資料庫。  

### 3. Atlas 註冊
因為是直接使用雲服務的資料庫，過程很簡單，從官網 [MongoDB](https://www.mongodb.com/  "MongoDB")，直接註冊帳號後，在產品的地方選擇Atlas，照著步驟建立基本資料、連線權限就能直接使用。

### 4. 權限管理
在 MongoDB Atlas 中，可以建立多個組織（Organization），每個組織下可以建立多個專案（Project），每個專案下可以建立多個集群（Cluster）。以下是它們之間的差異：

1. 組織（Organization）：組織是 MongoDB Atlas 中的最高級別。您可以使用一個電子郵件地址註冊並建立一個組織，並在該組織中邀請其他使用者加入。您可以將不同的專案分配給不同的團隊或使用者，管理組織中的所有資源和設定。  

2. 專案（Project）：專案是一個或多個 MongoDB 集群的集合。在專案中，您可以設定相關的安全性、監控和自動化功能，也可以設定存取權限和通知規則等。一個專案可以被一個團隊或一個使用者所擁有和管理。  

3. 集群（Cluster）：集群是 MongoDB 的實例，它是由一組 MongoDB 伺服器組成的分佈式系統。每個集群都有自己的存儲空間、計算資源和設定，並由 MongoDB Atlas 提供管理和監控。您可以在每個集群中設定各種選項，例如可用區域、資料庫版本、存儲引擎和安全設定等。  

簡單來說，就是由這三個層級，由上到下去管理規範每個資料庫的使用權限。

### 5. 資料庫操作
我的話習慣使用GUI來操作資料庫，MongoDB也有提供GUI供使用者操作資料庫，可以去 [MongoDB Compass Download (GUI)](https://www.mongodb.com/try/download/compass  "MongoDB Compass Download (GUI)") 在Tools的地方，選擇下載MongoDB Compass。
安裝好後，執行會出現提示要求你輸入用於連線資料庫的字串。連線字串可以在Atlas的Cluster管理畫面上右側有個connect，點選就能找到各種的連線方式。  
{{< figure src="pics/connect.webp" width="80%" alt="" title="">}}  

因為是使用Compass 連線，所以就點選Compass，畫面就會出現對於Compass的連線字串。
```
mongodb+srv://<user>:<password>@cluster0.swvkd15.mongodb.net/test
```
`<user>` `<password>`就是當初建立Cluster時建立的帳號與密碼。

再複製連線字串並修改`<user>` `<password>`後，**記得要在Network Access的地方設定可以連線的IP位置**，
{{< figure src="pics/Network Access.webp" height="10%" alt="" title="">}}  

設定好後，在 MongoDB Compass 貼上連線字串後就能連線成功了。
### 6. 資料庫結構  
在 MongoDB 中，資料是以文件（document）的形式儲存的。一個集合（collection）則是一個文件的集合。換句話說，一個 MongoDB 資料庫可以包含多個集合，而每個集合裡面包含多個文件。

與關聯式資料庫比較，我覺得是這種感覺 :
|  MySQL   | MongoDB  |
|  :----:  | :----:  |
| database  | database |
| table  | collection |
| per row  | document |

每一行資料就是一個文件。和關聯式資料庫不同的是，MongoDB 的文件可以包含非常彈性的結構和內容，而且不需要先定義欄位。

簡單來說，集合是存儲文件的地方，而文件則是實際存儲資料的容器。如果把 MongoDB 視為一個文件數據庫系統，那麼集合就是這個系統中的表格，而文件就是這個表格中的行。

### 7. 結語  
大致瞭解了 MongoDB 的數據結構，與如何連線至 Atlas 的雲資料庫，接下來要學習如何使用node.js進行 MongoDB 的讀寫操作。



