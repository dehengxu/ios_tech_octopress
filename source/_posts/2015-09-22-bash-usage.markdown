---
layout: post
title: "bash usage"
date: 2015-09-22 09:48:42 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---

## 基础

### 变量定义

`var_name=var_value`，注意等号两边不能留下空格

### 输入参数

参数以空格分开分别是`$1`, `$2`, 到 `$9`

## 条件控制处理

### if 语句

```
if [ condition1 ]; then

elif [ condition2 ]; then

else

fi
```

1. 方括号内的条件和方括号之间至少有一个空格的距离
2. 每个方括号结束后通过`;`分隔
3. 不论 `if`, `else` 还是 `elif`，除最后一个条件外，都需要以 `then` 结尾
4. 最后以 `fi` 结尾。`fi` 是 `if` 的逆序写法，表示和`if`对应。

### 条件表达式

1. `[ -z var_name ]`，判断变量是否为空

## 字符串处理

## 文件夹处理

1. `if [ -d path ]; ` 路径是否存在，反之 `if [ ! -d path ];` 路径是否不存在

## 循环控制

## 数组

### 定义

array_name=(value1, value2, value3);

### 使用

${array_name[0]}
