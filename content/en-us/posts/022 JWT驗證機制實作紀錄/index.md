---
title: "JWT Authentication Mechanism Principles and Implementation Examples (C#) "
date: 2023-03-06T16:05:44+08:00
draft: False
slug: "tutorials/jwt-authentication-mechanism"
thumbnail: "/img/thumbnail/022en.png" # Thumbnail image
description: ""
categories:
  - "Program"
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
JWT (JSON Web Token) is an open standard used for securely transmitting information between parties. JWT uses JSON objects to represent the messages to be transmitted and uses digital signatures or encryption to protect these messages.
<!--more-->
JWT is commonly used for quickly setting up login authentication for lightweight websites. Here is a Record the usage of implementing JWT in C#.
{{< br 3  1>}}
### Introduction
[JWT](https://jwt.io/  "JWT")   

Encoded 

---
`{{< color "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."  "#EA0000" >}}{{< color "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ."  "#BE77FF" >}}{{< color "SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"  "#80FFFF" >}}`

---
Consisting of three parts, namely Header, Payload, and Signature.  
Using the example provided on the official website directly:
> Header part includes data such as the encryption algorithm and type used by JWT.
```JSON
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Header is a JSON object that contains the following two properties:  
- alg： Indicates the encryption algorithm used, such as HS256, RS256, etc.
- typ：Indicates the type, which is usually set to JWT.
{{< br 3  2>}}
> Payload part contains the information to be transmitted.   

Payload is generally also a JSON object and can contain custom properties. Customer information, account passwords, etc. are typically stored in the Payload part.  
```JSON
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```
Standard properties that can be included in the Payload are:  
- iss: The issuer of the JWT, indicating who issued the JWT token.
- sub: The subject of the JWT, indicating the entity (usually the user) that the JWT - token represents.
- aud: The audience of the JWT, indicating who can use the JWT token.
- exp: The expiration time of the JWT, indicating when the JWT token expires and can no longer be used.
- nbf: The time when the JWT becomes valid, indicating when the JWT token becomes effective.
- iat: The time when the JWT was issued, indicating when the JWT token was issued.
- jti: The unique identifier of the JWT, used to prevent the JWT token from being reused.

The properties listed above are some standards defined in the JWT specification, but custom names can also be defined to represent specific information in the application. 
{{< br 3  3>}}
> Signature part is the result of digitally signing the Header and Payload.

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
'your-256-bit-secret'
)
```
The signature can ensure that the JWT has not been tampered with. The calculation method of the signature varies depending on the encryption algorithm used, with commonly used algorithms including HMAC and RSA.  
{{< br 3  4>}}
### Principle
In simple terms, a string is generated using the Header and Payload, with each string separated by a `.`. Then, a private key is used to sign the string, generating a JWT that includes the Header, Payload, and Signature.  

When verifying the JWT, the receiver can decode the Header and Payload, and then use the same key to verify the Signature to ensure that the JWT has not been tampered with. If the verification is successful, the information contained in the JWT can be trusted.
{{< br 3  5>}}
### Creating JWT TOKEN Example (.NET 6)
```C#  {linenos=inline}
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

// Set the signing key for JWT
var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("MySuperSecretKey"));
var signingCredentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

var claims = new[]
{
            new Claim(JwtRegisteredClaimNames.Sub, "user123"), // Set the username
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()) //  Set the JWT ID
        };

// Set the expiration time of JWT to 1 hour
var expires = DateTime.UtcNow.AddHours(1);

// Create JWT token object
var token = new JwtSecurityToken(
    issuer: "MyApp",
    audience: "MyClient",
    claims: claims,
    expires: expires,
    signingCredentials: signingCredentials);

// Convert the object to a JWT string
var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

// Print the JWT string
Console.WriteLine(tokenString);
```
{{< br 3  6>}}
### Validate JWT Token Example (.NET 6)
```C#  {linenos=inline}
//  Get the JWT token carried from the front-end
string tokenString = "{insert JWT token string here}";

// Set the validation parameters for JWT
var validationParameters = new TokenValidationParameters
{
    // Set the issuer and audience of JWT
    ValidIssuer = "MyApp",
    ValidAudience = "MyClient",
    // Set the signing key and encryption algorithm for JWT validation
    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("MySuperSecretKey")),
    ValidateIssuerSigningKey = true,
    ValidateLifetime = true,
    ClockSkew = TimeSpan.Zero
};

try
{
    //  Validate the JWT token
    var jwtHandler = new JwtSecurityTokenHandler();
    var principal = jwtHandler.ValidateToken(tokenString, validationParameters, out var validatedToken);
    var jwtToken = validatedToken as JwtSecurityToken;

    //  Get the username and JWTID
    var username = jwtToken.Claims.First(x => x.Type == JwtRegisteredClaimNames.Sub).Value;
    var jwtId = jwtToken.Claims.First(x => x.Type == JwtRegisteredClaimNames.Jti).Value;

    // Additional validation can be performed here, 
    //such as checking if the user exists in the database or 
    //if the user has permission to access resources, etc.

    // Validation succeeded
    Console.WriteLine($"Token validated. Username: {username}, JWT ID: {jwtId}");
}
catch (SecurityTokenException e)
{
    // Validation failed
    Console.WriteLine($"Token validation failed: {e.Message}");
}

```
> The issuer and receiver of a JWT token.
- Issuer (iss): Used to indicate who issued the JWT token, typically the name or domain of the organization or application that identifies the JWT token. When verifying the JWT token, we can check if the issuer of the JWT token matches the expected value to ensure that the JWT token was issued by the correct organization or application.

- Audience (aud): Used to indicate who can use the JWT token, typically the name or domain of the application or API that identifies the JWT token user. When verifying the JWT token, we can check if the audience of the JWT token matches the expected value to ensure that the JWT token can be used by the correct application or API.
{{< br 3  7>}}
### Conclusion
I currently used JWT only for lightweight website login authentication, but in fact, JWT has other uses such as resource authorization, single sign-on (SSO), timed services, inter-application communication, etc. By distributing time-limited JWTs and verifying them each time whenever resources are accessed, its application can be more widely extended.  




