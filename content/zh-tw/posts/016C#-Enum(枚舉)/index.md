---
title: "C# Enum(枚舉) 用法紀錄"
slug: "tutorials/Csharp Enum"
date: 2023-01-15T08:49:07+08:00
draft: false
thumbnail: "" # Thumbnail image
description: "C# Enum(枚舉)用途與使用擴充方法取得description詳細紀錄"
categories:
  - "程式語言"
tags:
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
toc: false

---
C#-Enum(枚舉)用途與使用擴充方法取得description詳細紀錄。
<!--more-->

枚舉就是把一些常數的集合命名分類，以生活上的例子星期就是一種枚舉。  
|  星期  | int  |
|  :----:  | :----:  |
| 星期一  | 1 |
| 星期二  | 2 |
| 星期三  | 3 |
| 星期四  | 4 |
| 星期五  | 5 |
| 星期六  | 6 |
| 星期日  | 7 |

如果有這種固定好的集合需頻繁使用，在程式上就可以以枚舉來實現。除了可方便管理，想在程式中用字串形式當key時，可以很明確知道用途，也不會有大小寫或是拼字錯誤的困擾。

### 使用範例
如果沒有預設對應的數字，就是照順序給值  
Veiw = 0  
Add = 1  
Edit = 2...
```C# {linenos=inline}
public enum LogBehavior
    {
        Veiw, Add, Edit, Delete, Search, Login, Send
    }

```
or
```C# {linenos=inline}
public enum Month
    {
        Jan = 1, Feb = 2, Mar = 3, Apr = 4, May = 5, Jun = 6, 
        Jul = 7, Aug = 8, Sep = 9, Oct = 10, Nov = 11, Dec = 12
    }

```
取用enum參數的方式  
```C# {linenos=inline}
Console.WriteLine(LogBehavior.Veiw);
>>  Veiw

Console.WriteLine(LogBehavior.Veiw.GetHashCode());
>>  0

string month = Month.Jan.ToString(); 
Console.WriteLine(month);
>> Jan

int monthValue = (int)Month.Jan;  
Console.WriteLine(monthValue);
>> 1
```


### 實際運用例子
後端需要在每個回傳的API裡增加一個可擴充性的物件(```List<dynamic>```)，裡面可以依據情況增加物件，但需加上特定參數告訴前端物件的資料性質。  

利用枚舉就能輕鬆管理，要回傳物件的特定參數名稱，其他人也能直接照用，不會有人為字串拼錯或大小寫的錯誤，在之後需要其他特定參數也能直接往下增加擴充。  
```C# {linenos=inline}
public enum prop
{
    [Description("同步紀錄資訊")]
    SyncInfo,
    [Description("版本資訊")]
    VersionInfo
}

```

若要取得Description的內容，可使用此方法擴充，若無Description內容則回傳原始名稱。

```C# {linenos=inline}
public static string GetDescription(this Enum value)
    {
        var field = value.GetType().GetField(value.ToString());
        var attribute = field?.GetCustomAttributes(typeof(DescriptionAttribute), false).FirstOrDefault() as DescriptionAttribute;
        return attribute == null ? value.ToString() : attribute.Description;
    }
```
```C# {linenos=inline}
Console.WriteLine(prop.SyncInfo.GetDescription());
>> 同步紀錄資訊

```


>參考資料  
[Extension Methods](https://learn.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)  
[C# enum 列舉 同時儲存內容值和中文說明 教學](https://www.ruyut.com/2022/09/csharp-enum-description.html)
