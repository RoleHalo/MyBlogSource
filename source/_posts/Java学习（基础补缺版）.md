---
title: java基础学习
date: 2019-08-06 08:18:12
tags: java
toc: true
---
# 目录
- [目录](#目录)
- [第一章 java语言概述](#第一章-java语言概述)
    - [文档注释](#文档注释)
    - [API文档的说明](#api文档的说明)
    - [Java的语言特点](#java的语言特点)
    - [JVM](#jvm)
- [第二章 变量与运算符](#第二章-变量与运算符)
    - [关键字](#关键字)
    - [标识符](#标识符)
    - [变量](#变量)
    - [基本数据类型变量间的转化](#基本数据类型变量间的转化)
    - [String类型](#string类型)
    - [运算符](#运算符)
- [第三章 流量控制](#第三章-流量控制)
# 第一章 java语言概述
#### 文档注释
1. 例如：
    ```
    /**
    	这是文本注释(java特有的)
    	可以被 javadoc 所解析生成一套以网页文件形式体现的该程序的说明文档
    	@author loumeng
    	@version 1.0
    */
        
    ```
2. 使用命令：` javadoc -d mkdir -author -version 文件名.java`  创建文件夹为mkdir的网页文件夹
 <!-- more -->   
#### API文档的说明
1. API（Application programming interface，应用程序编程接口）

#### Java的语言特点
1. 跨平台
2. 面向对象
3. 健壮性
4. 安全性高
5. 简单性
6. 高性能

#### JVM
1. 实现java的`跨平台性`
2. 自动内存管理：`GC自动回收`
3. 内存溢出 内存泄露


# 第二章 变量与运算符

#### 关键字
1. 全是小写
2. goto contes 保留字

#### 标识符
1. 可以自定义的地方、
2. 类名，包名。。。
3. 命名规则 （否则编译会报错） =="自洽"==
   1. 26字母大小写开头，0-9，_ 或$ 组成
   2. 数字不可以开头
   3. 不可以使用但可以包含关键字或者保留字
   4. `java严格区分大小写` 长度无限制
   5. 标识符不能包含空格
4. 命名规范
   1. 包名：多单词组成时字母都小写
   2. 类名、接口名：多单词组成时，所有单词的首字母大写
   3. 变量名、方法名：多单词组成时，第一个的单词首字母小写，第二个及以后的每个单词首字母大写
   4. 常量名：所有字母都大写，多单词时每个单词用下划线连接
   

#### 变量
1. 整型变量
    1. byte   1字节 = 8 bit
    2. short  2字节
    3. int(==常量默认为int==)    4字节
    4. long   8字节 。赋值最后用小写或者大写的`L`结尾。例如：`long l1 = 12138L` 或者 `long l1 = 12138l`
2. 浮点类型变量
    1. float 4字节  赋值最后用小写或者大写的`F`结尾。例如：`float f1 = 12138F` 或者 `float f1 = 12138f`
    2. double 8字节
    3. 通过测试 浮点型数据精度不高 如果需要极高的精度 使用`BigDecimal类`替换浮点型变量
3. bool类型
4. 字符类型 char


#### 基本数据类型变量间的转化
注：不包含boolean类型
    
```
    运算规则
        1. 自动类型提高
        2. 强制类型转换
```

1. 自动类型提升
    1. byte < short < int < long < float < double
    2. ==调用方法时候传参是自动类型转换==，而不是强制类型转换
    3. 规则：当容量小的变量与容量大的变量做运算时，结果自动转换为容量大的变量数据类型（==大转小编译不通过==）
2. 强制类型提升
    1. 例如：
        ```
            long l1  = 123L;
            short s1 = (short)l1;
            
            
            是可以通过编译的
        ```
    2. 规则：当容量小(或者大)的变量与容量大(或者小)的变量做运算时，结果强制转换为容量大(或者小)的变量数据类型（==主要用于大转小==，但精度可能会丢失）

#### String类型
1. 属于引用类型
2. 可以使用双引号的方式进行复制
3. 可以包含0或者1个或者多个字符
4. 包括Boolean在内的8种都可以转
5. 可以直接通过"+"进行运算
6. 运算结果肯定是 String类型
7. 无法转换为基本数据类型    只能转换为其他引用数据类型


#### 运算符
1. 按功能
    1. 算数运算符(7个)： +、-、*、/、%、++（前/后）、--（前/后）
    2. 赋值运算符(12个)： =、+=、-=、/=、%=、>>=、<<=、>>>=、&=、|=、^=等
    3. 比较（关系）运算符(6个)： >、>=、<、<=、==、||
    4. 逻辑运算符(6个)： &、|、^、！、&&、||
        1. &和| 两个条件无论第一个条件是否通过都需要运行第二个条件
        2. &&和|| 两个条件第一个条件已经不符合则直接跳过，不再对第二个条件进行运行判断
    5. 位运算符(7个)：&、|、^、~、<<、>>、>>>
    6. 条件运算符(1个)：==(条件表达式)？结果1：结果2==
        * 如果表达式结果为true，则执行结果表达式1，否则执行结果表达式2
        * 凡是可以使用条件运算符的位置，都可以使用if-else，反之则不然
        * ==建议：二者都能使用的时候使用条件运算符，执行效率稍快==
    7. Lambda运算符(1个)：->
2. 按操作数个数
    1. 一元运算符（单目运算符）: +、-、++、--、！、~
    2. 二元运算符（双目运算符）
    3. 三元运算符（三目运算符）: (条件表达式)？结果1：结果2

# 第三章 流量控制

1. if-else结构
2. Scanner类
3. switch-case
4. for循环
5. while循环和do-while循环
6. 无限循环结构和嵌套循环的使用

