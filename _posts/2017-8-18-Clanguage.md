---
layout: post
title: C语言小计
categories: C
description: 一些C语言的基本用法 
keywords: jekll,theme
---
# *C语言小计*
## string.h:一般 字符串/数组 方法
### 拷贝:
```C
void* memcpy (void* dest, void* src, size_t n):拷贝n bit,字符串src到dest中.  
char* strcpy(char* dest, char* src):拷贝src字符串到dest中,返回dest.
```
### 连接：
```C
char * strcat (char * dest, char* src):添加src的拷贝到dest的末尾,返回dest
```

### 比较：
```C
int strcmp (char * str1, char * str2):通过比较str1,和str2成员(基于ASCII码值的每个成员比较,之后是字符串长度),返回比较结果
str1 < str2: -1, 
str1 == str2: 0, 
str1 > str2: 1
```
### 搜索:
```C
char* strstr (char * str1, char * str2):返回str2在str1中第一个匹配的地址指针,不存在怎返回NULL.
char* strtok (char * str, char * delimiters): 
```
### 其他:
```C
size_t strlen ( const char * str ):返回字符串的长度(不包括最后一位'\0')
void * memset (void* ptr, int val, size_t n ): 设置第一个地址从ptr到ptr+val的长度为n字节的内存块(只用来设置字节,不用来设置整型数组或其他)
```

## stdlib.h:一般的函数库
### 动态内存分配:
```C
malloc, calloc, free
```
### 字符串转换:
```C
int atoi(char* str): 把字符串解析成整型值(不能解析返回为零)
```
### 系统调用:
```C
void exit(int status): 终止调用进程,返回状态给调用它的父进程
void abort(): 不正常的中止进程
```
### 搜索/排序:
```C
提供数组,数组大小,元素大小,比较(函数指针)
bsearch:返回数组中匹配元素的指针
qsort:破坏性的排序数组(顺序打乱)
```
### 整型算数:
```C
int abs(int n): 返回n的绝对值
```
### 类型:
```C
size_t:无符号整型类型(存储任意对象的大小)
```

## stdio.h:通用的I/O方法(input/output)
```C
FILE* fopen (char* filename, char* mode): 使用特定的文件名,特定的模式打开文件(read,write,append,等),通过返回的文件指针将它与流定义联系在一起
int fscanf (FILE* stream, char* format, ...): 从流中读取数据,根据参数format在内存位置指向以及其他参数来存放.
int fclose (FILE* stream): 关闭文件有关的流
int fprintf (FILE* stream, char* format, ... ): 把C字符串按照format的格式写到流中,并且可以使用其他额外的参数来填补特定的格式.
```

## 注意一些库函数:
```C
malloc 可能会失败
文件可能打开失败
字符串可能不会正确的解析
要时常检查错误:
有些错误可能会中断程序(eg.malloc 失败)
有些会因为错误的输入导致错误.

```