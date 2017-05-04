---
title: C# 操作符问号和双问号的释义
date: 2016-03-21 21:50
categories: CSharp 
tags: [CSharp]
description: CSharp 操作符问号和双问号的释义
---
##  1. ? 运算符
``` cs
stirng name;
name = name == null ?"未命名" : name;
``` 

上面的 **?** 在C#中是一个三元运算符
是先将 **?** 左边的表达之进行运算，如果结果是True那么返回 **=** 左侧的变量否则返回右侧的变量。  

## 2. ?? 运算符
``` cs
string Id;
int UserId = Id as int ?? 0 
``` 

此处的 **??** 运算则是判断 **??** 左侧的运算结果是否为null，如果不是null 返回左侧的结果（`将 ?? 左侧的运算结果赋值给UserId`），否则返回右侧的结果