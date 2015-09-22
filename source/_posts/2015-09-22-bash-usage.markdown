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

1. `[ -z var_name ]`，变量是否为空字符串
2. `[ -n var_name ]`，变量是否为空值

## 字符串处理

```
   -eq   等于
　　-ne    不等于
　　-gt    大于
　　-lt    小于
　　-le    小于等于
　　-ge   大于等于
　　-z    空串
　　=    两个字符相等
　　!=    两个字符不等
　　-n    非空串
```

## 文件，文件夹处理

1. `if [ -d $path ]; ` 目录是否存在，反之 `if [ ! -d path ];` 目录是否不存在
2. `if [ -f $file ]; ` 文件是否存在，同上 `if [ ! -f file ];`
3. `if [ -x $path ];` 是否有执行权限

```
   –b 当file存在并且是块文件时返回真
　　-c 当file存在并且是字符文件时返回真
　　-d 当pathname存在并且是一个目录时返回真
　　-e 当pathname指定的文件或目录存在时返回真
　　-f 当file存在并且是正规文件时返回真
　　-g 当由pathname指定的文件或目录存在并且设置了SGID位时返回为真
　　-h 当file存在并且是符号链接文件时返回真，该选项在一些老系统上无效
　　-k 当由pathname指定的文件或目录存在并且设置了“粘滞”位时返回真
　　-p 当file存在并且是命令管道时返回为真
　　-r 当由pathname指定的文件或目录存在并且可读时返回为真
　　-s 当file存在文件大小大于0时返回真
　　-u 当由pathname指定的文件或目录存在并且设置了SUID位时返回真
　　-w 当由pathname指定的文件或目录存在并且可执行时返回真。一个目录为了它的内容被访问必然是可执行的。
　　-o 当由pathname指定的文件或目录存在并且被子当前进程的有效用户ID所指定的用户拥有时返回真。
```

## 循环控制

## 数组

1. 声明数组: `declare -a array_name`
2. 定义: `array_name=(value1 value2 value3)`
3. 内部指令:
```
${name[i]}: Use element i of array name. i can be any arithmetic expression as described under let.
${name}: Use element 0 of array name.
${name[*]}: Use all elements of array name.
${name[@]}: Same as previous.
${#name[*]}: Use the number of elements in array name.
${#name[@]}: Same as previous.
```
4. 关联数组: `array_name=([key1]=value1 [key2]=value2 [key3]=value3)``

### 使用

${array_name[0]}
