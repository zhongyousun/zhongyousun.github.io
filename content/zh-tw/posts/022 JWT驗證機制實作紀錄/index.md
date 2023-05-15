---
title: "JWT 驗證機制原理與實作範例 (C#)"
date: 2023-03-06T16:05:44+08:00
draft: False
slug: "tutorials/jwt-authentication-mechanism"
thumbnail: "img/thumbnail/022.png" # Thumbnail image
description: ""
categories:
  - "程式語言"
tags:
  - "Csharp"
  - "Web"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
JWT（JSON Web Token）是一種開放標準，用於在各方之間安全地傳遞資訊。JWT使用JSON物件表示要傳遞的訊息，並使用數字簽名或加密來保護這些訊息。
<!--more-->
平常用於快速建立輕型網站的登入驗證，紀錄一下以 C# 實作 JWT 用法。
{{< br 3  1>}}
### 介紹
[JWT](https://jwt.io/  "JWT")   

Encoded 

---
`{{< color "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."  "#EA0000" >}}{{< color "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ."  "#BE77FF" >}}{{< color "SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"  "#80FFFF" >}}`

---
由三部分組成，分別是 Header、Payload 和 Signature。  
直接使用官網的範例:  
> Header 部分包含了JWT使用的加密算法、類型等數據。
```JSON
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Header是一個JSON物件，包含了以下兩個屬性：
- alg：表示使用的加密算法，例如HS256、RS256等。
- typ：表示類型，通常設置為JWT。
{{< br 3  2>}}
> Payload 部分包含了要傳輸的信息。  

Payload一般也是一個JSON物件，可以包含自定義的屬性。客戶資訊帳密等...就是放在Payload的部分。
```JSON
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```
Payload 中可以包含的標準屬性包括：
- iss：JWT的發行者，表示 JWT token 是由誰發行的。
- sub：JWT的主題，表示 JWT token 所代表的實體（通常是使用者）。
- aud：JWT的接收者，表示 JWT token 可以被誰使用。
- exp：JWT的過期時間，表示 JWT token 何時過期，過期的 JWT token 不能再使用。
- nbf：JWT的生效時間，表示 JWT token 何時生效。
- iat：JWT的發行時間，表示 JWT token 何時被發行。
- jti：JWT的唯一標識符，用於防止 JWT token 被重複使用。

以上列出的屬性是 JWT 規格中定義的一些標準，當然也可以自定義一些名稱，用於表示應用中的特定資訊。  
{{< br 3  3>}}
> Signature 部分則是對Header和Payload進行數字簽名的結果。  

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
'your-256-bit-secret'
)
```
簽名可以保證JWT沒有被篡改。簽名的計算方式根據不同的加密算法而不同，常見的算法包括HMAC和RSA。
{{< br 3  4>}}
### 原理
簡單來講就是用Header和Payload生成一個字串，字串之間以`.`連接，然後使用一個私密的金鑰對此字串進行簽名，最終生成一個包含Header、Payload和簽名的字符串(JWT)。
在驗證JWT時，接收方可以將Header和Payload解碼，然後使用相同的金鑰對簽名進行驗證，以確保JWT沒有被篡改。如果驗證成功，則可以信任JWT中包含的訊息。
{{< br 3  5>}}
### 建立JWT TOKEN 範例 (.NET 6)
```C#  {linenos=inline}
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

// 設定 JWT 的簽名金鑰
var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("MySuperSecretKey"));
var signingCredentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

var claims = new[]
{
            new Claim(JwtRegisteredClaimNames.Sub, "user123"), // 設定使用者名稱
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()) // 設定 JWT ID
        };

// 設定 JWT 的有效期限為 1 小時
var expires = DateTime.UtcNow.AddHours(1);

// 建立 JWT token 物件
var token = new JwtSecurityToken(
    issuer: "MyApp",
    audience: "MyClient",
    claims: claims,
    expires: expires,
    signingCredentials: signingCredentials);

// 把物件轉成 JWT 字串
var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

//印出 JWT 字串
Console.WriteLine(tokenString);
```
{{< br 3  6>}}
### 驗證JWT TOKEN 範例 (.NET 6)
```C#  {linenos=inline}
// 取得從前端攜帶的 JWT token 
string tokenString = "{insert JWT token string here}";

// 設定 JWT 的驗證參數
var validationParameters = new TokenValidationParameters
{
    // 設定 JWT 的發行者和接收者
    ValidIssuer = "MyApp",
    ValidAudience = "MyClient",
    // 設定驗證 JWT 的簽名金鑰和加密算法
    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("MySuperSecretKey")),
    ValidateIssuerSigningKey = true,
    ValidateLifetime = true,
    ClockSkew = TimeSpan.Zero
};

try
{
    // 驗證 JWT token
    var jwtHandler = new JwtSecurityTokenHandler();
    var principal = jwtHandler.ValidateToken(tokenString, validationParameters, out var validatedToken);
    var jwtToken = validatedToken as JwtSecurityToken;

    // 取得使用者名稱和 JWTID
    var username = jwtToken.Claims.First(x => x.Type == JwtRegisteredClaimNames.Sub).Value;
    var jwtId = jwtToken.Claims.First(x => x.Type == JwtRegisteredClaimNames.Jti).Value;

    // 在這裡可以執行額外的驗證，例如檢查使用者是否存在於資料庫中，以及檢查使用者是否有權限存取資源等等

    // 驗證成功
    Console.WriteLine($"Token validated. Username: {username}, JWT ID: {jwtId}");
}
catch (SecurityTokenException e)
{
    // 驗證失敗
    Console.WriteLine($"Token validation failed: {e.Message}");
}

```
> JWT token 的發行者和接收者  
- 發行者（issuer）：用於表示 JWT token 是由誰發行的，通常是一個識別 JWT token 的組織或應用程式的名稱或網域。當我們驗證 JWT token 的時候，可以檢查 JWT token 的發行者是否符合預期的值，以確保 JWT token 是由正確的組織或應用程式發行的。

- 接收者（audience）：用於表示 JWT token 可以被誰使用，通常是一個識別 JWT token 使用者的應用程式或 API 的名稱或網域。當我們驗證 JWT token 的時候，可以檢查 JWT token 的接收者是否符合預期的值，以確保 JWT token 可以被正確的應用程式或 API 使用。
{{< br 3  7>}}
### 總結
目前開發上只用在輕型網站的登入系統驗證機制，但其實JWT還有其他的用途像是資源權限授權、單點登錄(SSO)、時效性服務、應用程序間通訊等。藉由派發有時效性的JWT並在每次存取資源時驗證可以使其應用更廣泛。  