---
title: "C# Enum Usage Record"
slug: "tutorials/Csharp Enum"
date: 2023-01-15T08:49:07+08:00
draft: false
thumbnail: "" # Thumbnail image
description: "C# Enum (enumeration) usage and use of extension methods to obtain description detailed records"
categories:
  - "Program"
tags:
  - "Csharp"
# Theme-Defined params
lead: "" # Lead text
toc: false

---
C#-Enum (enumeration) uses and uses the extension method to obtain detailed description records.
<!--more-->
Enumeration is to name and classify some sets of constants. Taking the example of life as a week is a kind of enumeration.  
|  week  | int  |
|  :----:  | :----:  |
| Monday  | 1 |
| Tuesday  | 2 |
| Wednesday  | 3 |
| Thursday  | 4 |
| Friday  | 5 |
| Saturday  | 6 |
| Sunday  | 7 |

If there is such a fixed set that needs to be used frequently, it can be implemented by enumeration in the program. In addition to being easy to manage, if you want to use a string as a key in a program, you can clearly know the purpose, and there will be no trouble with capitalization or spelling mistakes.

### Example of use
If there is no preset corresponding number, the value is given in order   
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
The way to get enum parameters
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


### Practical application example
The backend needs to add an extensibility object to each returned API(```List<dynamic>```), you can add objects according to the situation, but you need to add specific parameters that let front-end know properties of the objects.  

It can be easily managed by using enumeration. If you want to return the specific parameter name of the object, other people can directly use it. There will be no artificial string misspelling or capitalization errors. You can also directly add extensions if you need other specific parameters later.  
```C# {linenos=inline}
public enum prop
{
    [Description("Synchronize record information")]
    SyncInfo,
    [Description("Version information")]
    VersionInfo
}

```

If you want to get the content of Description, you can use this method to expand, if there is no content of Description, will return the original name.

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
>> Synchronize record information

```


>References  
[Extension Methods](https://learn.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)  
[C# enum 列舉 同時儲存內容值和中文說明 教學](https://www.ruyut.com/2022/09/csharp-enum-description.html)
