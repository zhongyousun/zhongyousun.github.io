---
title: ".NET 6 Web API 使用SignalR實作WebSocket技術實現即時通訊"
date: 2023-05-03T16:08:13+08:00
draft: true
slug: "tutorials/aspnetcore-6.0-signalr-websocket"
thumbnail: "img/thumbnail/XXXX.jpg" # Thumbnail image
description: ""
categories:
  - "程式語言"
tags:
  - "Web"
  - "Csharp"
  - "JavaScript"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
使用.NET 6 Web API 建立 WebSocket 通訊，本篇紀錄藉由簡易前端模擬全推播、群組推播、單一推播範例。
<!--more-->


### 前言Foreword
我所待的組舊專案，若需要即時通訊功能，原本都是串接其他同仁已完成的 Flask-SocketIO，但我們組後端語言都是使用 C#，主管想要自己弄很久了，這次新專案就被指派要自己弄一個 C# 的 WebSocke 通訊。其實對平常只負責生 API 的我，要瞭解整個通訊的過程(包含前端)還蠻艱困的，所以就來記錄一下自己測試的過程。

### 原理Principle
WebSocket 是一種協議與平常前後端彼此串聯所使用的 API 不同之處在於它可以建立一個持久性的、雙向通訊的連接。而在傳統的 API 請求中，**每次客戶端需要向伺服器獲取資料時，都需要發送一個 HTTP 請求，然後等待伺服器的回應**。這樣的請求和回應是無狀態的，也就是說每次請求和回應之間都是相互獨立的。  
WebSocket 則可以在**建立連接後一直保持通訊狀態**，客戶端和伺服器可以直接互相傳送數據，而不需要重新建立連接。這樣可以大大減少通訊的開銷，提高通訊效率，並且可以支援一些即時性要求較高的應用場景，例如多人遊戲、聊天室、實時監控等。

### SignalR
SignalR 是一個 ASP.NET Core 框架的庫，用於建立即時 Web 應用程序。SignalR 允許伺服器端代碼推送新資訊到與客戶端建立的 WebSocket 連線，這樣客戶端就可以即時地接收到這些資訊。而 ASP.NET Core 也是有 WebSocket 的庫，但根據微軟建議若要實現即時通訊可以選則 SignalR，因為 SignalR 在 WebSocket 連線的基礎上實現了許多進階功能，例如自動重新連線、分組傳送、廣播、序列化等等。透過 SignalR 可以更加輕鬆地建立高效的即時 Web 應用程序，並且減少開發的複雜性。所以這次的專案就選擇 SignalR 吧。  

### 測試
一開始被指派要弄出一個即時通訊功能時其實蠻頭大的，畢竟跟傳統資料傳送的模式不一樣。對於後端工程師，如果想要測試API，可以使用 swagger 來進行測試。但對於即時通訊，則需要測試連接、訊息傳遞、斷連，因為前端還蠻忙的，所以就只好自己來😗。  

測試除了**連線與斷線**，在訊息傳遞的部分，因為連線是都是連線到同一個伺服器的 Hub，(Hub 是一種抽象概念，它用來描述客戶端和伺服器之間的一個邏輯上的連線點)，通常同一種功能都是連線到同一個 Hub，所以對於互相傳送資訊，要從同個 Hub 傳到指定的客戶端就非常重要，傳錯訊息就尷尬了😂。  
接下來訊息傳遞會測試 **全部傳遞、群組傳遞與指定傳遞**。


### 建立 Web API 專案
使用 Visual studio 點選 Create a new project > ASP.NET Core Web API > 取好專案名稱 > 選擇 .Net 6.0 > Create  

專案結構如下:  
{{< figure src="pics/Project-Structure.webp" width="50%" alt="" title="">}}  

```
SignalR-Example/
├── Connected Services/
├── Dependencies/
├── Properties/
│   ├── launchSettings.json
├── Controllers/
│   ├── WeatherForecastController.cs
├── appsettings.json
│   ├── appsettings.Development.json
├── Program.cs
├── WeatherForecast.cs

```
接下來要新增 SignalR 的包
點選 Visual studio 上方 Tools > NuGet Package Manager > Manage NuGet Package for Solution   
下載 SignalR NuGet package  
{{< figure src="pics/Visual-studio-SignalR.webp" width="80%" alt="" title="">}}  

### 建立 SignalR Hub
新增 Hub 資料夾 > 在 Hub 底下新增 IMessageHub.cs 介面 > 新增一個 sendToAllConnections 方法 用於傳送訊息給所有已經連線的使用者  

```C# {linenos=inline}
namespace SignalR_Example.Hub
{
    public interface IMessageHub
    {
        Task sendToAllConnections(List<string> message);
    }
}

```

在 Hub 資料夾底下新增  MessageHub.cs  
```C# {linenos=inline}
using Microsoft.AspNetCore.SignalR;
namespace SignalR_Example.Hub
{
    public class MessageHub: Hub<IMessageHub>
    {
        public async Task sendToAllConnections(List<string> message)
        {
            await Clients.All.sendToAllConnections(message);
        }
    }
}
```

接下來新增一個提供推播功能的 Controller， 選擇新增 API Controller - Empty : MsgController.cs  
```C# {linenos=inline}
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.SignalR;
using SignalR_Example.Hub;

namespace SignalR_Example.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class MsgController : ControllerBase
    {
        private IHubContext<MessageHub, IMessageHub> messageHub;
        public MsgController(IHubContext<MessageHub, IMessageHub> _messageHub)
        {
            messageHub = _messageHub;
        }
        [HttpPost]
        [Route("toAll")]
        public string ToAll()
        {
            List<string> msgs = new List<string>();
            msgs.Add("Don't forget, the deadline for submitting your expense reports is this Friday.");
            msgs.Add("Friendly reminder, please refrain from using the conference room for personal calls or meetings without prior approval.");
            messageHub.Clients.All.sendToAllConnections(msgs);
            return "Msg sent successfully to all users!";
        }
    }
}

```
新增了一個 API 且注入 SignalR 包含的 IHubContext，提供發送消息給所有人的功能。    

為了方便確認前端是否有連接上 Hub 與後續測試訊息傳傳遞，需要在專案新增一個html頁面(還能直接避免CORS):
在專案底下新增 wwwroot，並在底下新增一個html檔。  
`wwwroot 是 ASP.NET Core 中的一個特殊目錄，用於存放靜態資源文件，如 HTML、CSS、JavaScript、圖像等等。在 ASP.NET Core 應用程式中，使用者能夠直接從瀏覽器輸入 URL 訪問該目錄下的文件，因為這些文件可以直接從網頁請求中載入。` 

新增 htmlpage.html，因為主要是實現後端功能，前端的訊息顯示一率顯示在console上。  
```Html {linenos=inline}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>SignalR TEST </title>
    <script src="https://cdn.jsdelivr.net/npm/@microsoft/signalr@5.0.7/dist/browser/signalr.min.js"></script>
    <script>
        // 建立 SignalR Hub 連線
        const hubConnection = new signalR.HubConnectionBuilder()
            .withUrl("https://localhost:7013/messageHub/")
            .build();

        hubConnection.start()
            .then(() => {
                console.log("Connection started");
            });
        // 註冊 MessageHub 的 sendToAllConnections 事件
        hubConnection.on("sendToAllConnections", function (msgs) {
            console.log(msgs);
        });
    </script>
</head>
<body>
    SignalR TEST
</body>
</html>
```
接下來就是調整 SignalR 跟 CORS policy 還有讀取靜態資料在 Program.cs 的一些設定。   
`加上註解的為新增的項目`  
```C# {linenos=inline}
using SignalR_Example.Hub;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddSignalR();   // 註冊 SignalR 服務

builder.Services.AddCors(options =>    // 註冊 CORS 服務，允許跨來源請求
{
    options.AddPolicy("CorsPolicy", builder =>
    {
        builder.AllowAnyOrigin()
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});     
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseCors("CORSPolicy");   // 使用上面定義的 CORS 策略
app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();
app.UseFileServer();    // 使用內建的中介軟體提供靜態檔案
app.MapHub<MessageHub>("/messageHub");   // 映射 SignalR Hub 到 "/messageHub"(為前端的連線字串)
app.Run();

```


{{< figure src="pics/Project-Structure1.webp" width="50%" alt="" title="最終專案結構">}}  

### 結果
啟動專案，開啟 swagger 與 `https://localhost:7013/htmlpage.html`，用來確認呼叫API時，前端是否有收到訊息。

{{< figure src="pics/swagger.webp" width="80%" alt="" title="">}}  


{{< figure src="pics/console.webp" width="80%" alt="" title="進入網站，就會跳出連線已成功。">}}   


{{< figure src="pics/console1.webp" width="80%" alt="" title="呼叫 toALL 後，就會顯示訊息，可以打開多個畫面確認都會收到訊息">}}    

以上實作了對所有連線傳遞訊息的功能。  


> Reference

- [SignalR introduction and implementation using the.NET Core 6 Web API and Angular 14](https://medium.com/@jaydeepvpatil225/signalr-introduction-and-implementation-using-the-net-core-6-web-api-and-angular-14-b4cfd51a6fac  "SignalR introduction and implementation using the.NET Core 6 Web API and Angular 14")