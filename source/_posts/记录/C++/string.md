---
title: String专题
description: String 和 经典算法
date: 2021-05-05 22:02:00
author: zhangyuanes
categories: C++
tags:
  - C++
  - string
---

# String概念

## String API （C++11）

`string`是在C++中才封装的内容，在C中只能使用`char* []`或者`const char* []`,但是操作繁琐很容易出错。

String 的一些比较重要的API：

- c_str ： 返回字符串的不可修改的 C 字符数组版本。该指针有范围 [c_str(); c_str() + size()] 为合法，且其中的值对应存储于字符串的值，且在最后位置有个附加的空终止字符。

### 字符查找

- find：于字符串中寻找字符
- rfind：寻找子串的最后一次出现
- find_first_of：寻找字符的首次出现
- find_first_not_of: 寻找字符的首次缺失
- find_last_of: 寻找字符的最后一次出现
- find_last_not_of:寻找字符的最后一次缺失

### 数值转换

> 转换字符串为有符号整数
stoi
stol
stoll
  
> 转换字符串为无符号整数
stoul
stoull

> 转换字符串为浮点值
stof
stod
stold


> 转换整数或浮点值为 string,wstring
to_string
to_wstring

## String 基本考察类型

### 顺序 —— 翻转，反转

### 匹配 —— 子串，子序列，模式匹配

### 比较 —— 字典序

### 回文

### 统计 —— 出现次数，位置，条件，包含

## 典型题目

> 最长回文子串 DP

> 最长不重复子串 滑动窗口

> 和不小于K的最短子串 前缀叔祖+单调双端队列