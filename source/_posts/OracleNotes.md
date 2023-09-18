---
title: Oracle学习笔记
date: 2019-06-06 11:15:12
tags: Oracle
toc: true
---
# 目录
- [目录](#目录)
- [第一章 ： 数据库的安装以及配置](#第一章--数据库的安装以及配置)
    - [登录方式：](#登录方式)
    - [修改密码：](#修改密码)
    - [解锁用户：](#解锁用户)
    - [锁定用户：](#锁定用户)
    - [在进行网络配置的时候出现：ORA-12514: TNS:监听程序当前无法识别连接描述符中请求的服务解决方案](#在进行网络配置的时候出现ora-12514-tns监听程序当前无法识别连接描述符中请求的服务解决方案)
    - [命令行用户登录：](#命令行用户登录)
  - [在SQL Plus中可以执行SQL语句和SQL Plus命令:](#在sql-plus中可以执行sql语句和sql-plus命令)
- [第二章 ： 系统结构：](#第二章--系统结构)
  - [物理结构](#物理结构)
  - [逻辑结构](#逻辑结构)
    - [数据块(多个)--\>区(多个)--\>段(多个)--\>表空间(多个)--\>数据库](#数据块多个--区多个--段多个--表空间多个--数据库)
  - [内存结构](#内存结构)
  - [进程结构](#进程结构)
    - [Oracle进程结构](#oracle进程结构)
    - [Oracle例程后台进程](#oracle例程后台进程)
    - [数据字典](#数据字典)
      - [数据字典的分类：](#数据字典的分类)
- [第五章  用户管理](#第五章--用户管理)
    - [物理存储结构的规划](#物理存储结构的规划)
    - [逻辑存储结构的规划](#逻辑存储结构的规划)
  - [表空间](#表空间)
    - [1.创建表空间](#1创建表空间)
    - [修改表空间](#修改表空间)
    - [表空间删除](#表空间删除)
    - [查询表空间信息](#查询表空间信息)
    - [数据文件设置与管理](#数据文件设置与管理)
    - [数据文件](#数据文件)
    - [创建数据文件](#创建数据文件)
    - [修改数据文件大小](#修改数据文件大小)
    - [改变数据文件的可用性](#改变数据文件的可用性)
    - [改变数据文件的名称和位置](#改变数据文件的名称和位置)
    - [查询数据文件信息](#查询数据文件信息)
    - [控制文件设置与管理](#控制文件设置与管理)
    - [补充：oracle的启动状态](#补充oracle的启动状态)
    - [实现多路复用控制文件](#实现多路复用控制文件)
    - [删除控制文件](#删除控制文件)
- [第六章](#第六章)
    - [模式](#模式)
  - [约束与参数](#约束与参数)
    - [PRIMARY KEY：主键约束](#primary-key主键约束)
    - [惟一性约束：UNIQUE](#惟一性约束unique)
      - [PRIMARY KEY与UNIQUE比较](#primary-key与unique比较)
    - [检查约束](#检查约束)
    - [外键约束：FOREIGN KEY](#外键约束foreign-key)
    - [空/非空约束：NULL/NOT NULL](#空非空约束nullnot-null)
  - [参数](#参数)
    - [参数（parameter\_list）](#参数parameter_list)
    - [索引](#索引)
    - [序列](#序列)
    - [分区](#分区)
      - [分区概念](#分区概念)
      - [对巨型表进行分区具有下列优点：](#对巨型表进行分区具有下列优点)
      - [范围分区：](#范围分区)
      - [列表分区](#列表分区)
      - [散列分区](#散列分区)
      - [复合分区](#复合分区)
- [第七章 数据操纵与事务处理](#第七章-数据操纵与事务处理)


# 第一章 ： 数据库的安装以及配置

### 登录方式：
1）普通用户：system/密码
2）系统管理员：conn sys/密码 as sysdba;

### 修改密码：
1. 先登录sys用户
2. alter user 用户名 identified by 密码    
在知道自己的密码情况下可以直接使用password命令直接修改密码

### 解锁用户：
1. 先登录sys用户
2. alter user 用户名 account unlock;

### 锁定用户：
1. 先登录sys用户
2. alter user 用户名 account lock;	

### 在进行网络配置的时候出现：ORA-12514: TNS:监听程序当前无法识别连接描述符中请求的服务解决方案

解决方案：
1. 配置监听器
2. 配置listener.ora 此目录下：F:\app\windows\product\11.2.0\dbhome_1\NETWORK\ADMIN
3. 在里面添加如下内容：
```
            (SID_LIST =
			(SID_DESC =
			(SID_NAME = CLRExtProc)
			(ORACLE_HOME = F:\app\windows\product\11.2.0\dbhome_1)
			(PROGRAM = extproc)
			(ENVS = "EXTPROC_DLLS=ONLY:F:\app\windows\product\11.2.0\dbhome_1\bin\oraclr11.dll")
			)
	
	
			//添加部分，ORCL为实例名
			(SID_DESC =
			(GLOBAL_DBNAME = ORCL)
			(ORACLE_HOME = F:\app\windows\product\11.2.0\dbhome_1)
			(SID_NAME = ORCL)
			)
	
	
		)
```
4. 重启服务
		
5. sys:系统管理员，不能以normal身份登录
6. system：普通用户    
sqlplus sys/Lm20191103@orcl as sysdba
### 命令行用户登录：
- system登录:    
    `sqlplus system/system@orcl`
- sys登录:    
    `sqlplus sys/sys@orcl as sysdba`
- scott登录:    
    ` sqlplus scott/scott@orcl  `  
    ` sQL>select * from scott.emp;  `  
	`SQL >select name from V$database;`
	

```
1. SYS:当创建个数据库时，SYS用户 将被默认创建并授予DBA角色。(常用 )
4. SYSTEM:与SYS一样，在创建Oracle数据库时,SYSTEM用户被默认创建并被授予DBA角色，用于创建显示管理信息的表或视图，以及被各种Oracle数据库应用和工具使用的内容表或视图。(常用)
5. SCOTT, 一个测试用户，基本没有什么权限。
```
  

#### 1. 最重要的区别，存储的数据的重要性不同
##### sys:     
1. sys是超级用户。
2. 所有Oracle的数据字典的基表和视图都存放在sys用户空间中，这些基表和视图对于Oracle的运行是至关重要的，由数据库自已维护，任何用户都不能手动更改。
3. sys用户自动创建，拥有dba,sysdba,sysoper等角色或权限，是Oracle权限最高的用户。

##### system:
1. system:数据库内置的普通管理员。
1. system用户空间用于存放次一级的内部数据，如Oracle的 一些特性或工具的管理信息。
2. system用户自动创建，拥有dba, sysdba 等角色或系统权限。

#### 2. 其次的区别，权限的不同。
1. 最直接的区别： sys 可以创建数据库，而system不可以。
1. sys用户必须以as sysdba形式登录。
2. sysdba属于system privilege,也称为administrative   privilege,拥有例如数据库开启关闭之类些系统管理级别的权限。

---

## 在SQL Plus中可以执行SQL语句和SQL Plus命令:
```
1. SQL语句不区分大小写。
2. SQL语句可输入在一行或多行中。
3. 关键字不能缩写，也不能跨行分开写。
4. 子句通常放在单独的行中。应使用缩进来提高可读性。
5. 在SQL*Plus 中，必须使用分号(;) 结束每条SQL语句。
```


1. 连接命令：   
    1. conn[ect]
    - 用法: conn 用户名/密码@网络服务标识[as sysdbal/sysoper]
    - 当用特权用户身份连接时，必须带上as sysdba或as
    2. sysoper
    - 如:
       
    ```
        SQL >connect sys/sys@orcl as sysdba;    

        SQL >conn system/system;    
        
        SQL >conn scott/tiger;
    ```

        
2. 编辑命令: 

    1. List: 显示缓冲区内容(显示上一条命令)    
    - 语法格式: L[ist]
    ```
        SQL>L     
        
        SQL>L 2     
        
        SQL>L 1 2  
    ```
    2. Append语句:向缓冲区中的当前行尾部添加指定的文本    
    语法格式: append text
	3.  Change：修改缓冲区文本    
	语法格式：change /old/new
	4.  run(/) : 执行缓冲区中的SQL语句
	
3. 举例:

    + System登录显示scott.emp表内容。
   
    ```
     SQL>conn system/system
     SQL>select * from emp
	 SQL>List
	 sQL>run
	 SQL >change lemp/scott.emp
	 SQL >list 
	 SQL >run
    ```

    因为行编辑麻烦，所以使用Edit命令编辑缓冲区。Edit命令打
	开Notepad对缓冲区的内容进行操作编辑。    
    SQL>Edit

4. save命令   
把当前SQL缓冲区的内容保存到指定的文件当中。  
save的语法是:   
SAV[E] [FILE] file_ name [Create] | [REPLACE]|[APPEND]
    - Append表示把当前的内容添加到已经存在的文件中。
    - Replace表示覆盖当前已有的文件。默认的扩展名是.sql
    - 例子： `SQL>select * from scott.emp; SQL>save e:\SQL_employee.sql `   
    把当前缓冲区中的内容存到文件SQL_employee.sql中

5.  get命令  
把文件内容调入缓冲区。
    -  get的语法是：`get filename [LIST] [NOLIST]   ` 
    ` SQL>get e:\SQL_employee.sql`

    -  Edit命令也可以打开编辑指定的sq|脚本:
	`  SQL >edit e:\SQL_employee.sql;`

6. Start和@命令   
调用执行脚本文件。 
    - start命令语法:     
    `start filename [arg1 arg2..]`
    `SQL>start e:\sql_employee.sql`    
    - @命令语法:
    `@filename [arg1 arg2..]` 
    `SQL>@e:\sql_employee.sql`
	- 两个命令的差别在于:
	    - start的只能在Sqlplus会话内部使用
	    - @命令既可以在会话内部运行，也可以在启动sqlplus时的命令行级别运行。
	    - 
1.  spool 命令
	- 格式: spool filename
	- 说明:该命令可以把sq|*plus屏幕上的内容输出到指定文件，包括你输入的sq|语句及其执行结果。
	``` 
        SQL>spool d:\screen.txt
	    SQL >select * from scott.emp;
	    SQL>spool off; 
    ```
	(只有关闭spool输出，才会在输出文件中看到输出的内容)
	> 如在d:\b.sql,建立文本文件，内容有select * from emp;以及输出结果。
	
5. describe命令返回存储对象的描述。
	- sQL>desc 表名; 显示表结构

5. 环境变量的显示与设置命令  
- 显示:  

  `SQL >show al`

  `SQL >show linesize pagesize`
- 设置:
    
    ` SQL>set linesize 100 pagesize 20`

	` SQL >show linesize pagesize`
	
  - pagesize :设置一页显示的行数。在默认情况下，该值为14
  - linesize:设置一行的字符数量， 默认值为80。
  - pause:查询的结果超过-次屏幕，设置Pause值 使其暂停显示，直到用户按Enter键继续，默认值为OFF，
    - 用SET PAUSE ON[OFF]命令设置。
	- 例子：`SQL> SET PAUSE ON[OFF]`
	
 
6. 用户加锁：alter user scott account lock;<br/>用户解锁：alter user scott account unlock;
7. 修改密码为tiger：alter user scott identified by tiger;<br/>

--- 

# 第二章 ： 系统结构：
1. 数据库服务器的主要组成以及这些组成部分之间的联系和操作方式。
    1. 服务器：磁盘上的数据库（DB）和对磁盘上的数据进行管理的数据库管理系统（）
    2. DB：对应数据库的的存储结构

2. Oracle数据库存储结构分为：
    1. 物理存储：数据库在操作系统中的数据组织与管理方式。文件、数据块
    2. 逻辑存储：数据库在数据库系统内部的数据组织与管理方式。数据表等
关系：一般物理储存结构变现为一系列文件形式，是可见的；
逻辑存储结构是对物理存储结构的组织一管理，一般是不可见的。



3. 数据库：是用于保存数据的一系列物理结构和逻辑结构
4. 软件结构（实例）：DBMS的运行方式。包括：内存结构和后台运行。在服务器运行过程中内存结构 和一些列 进程 组成的。每个运行的Oracle数据库都对应一个Oracle例程，成为实例

总结:
Oracle数据库服务器由数据库和实例组成
数据库和实例的关系：
数据库是Oracle用于保存数据的一系列物理结构和逻辑结构
用户直接与实例交互，由实例访问数据库。每个数据库至少有
一个与之对应的实例
![Oracle图片1](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片1.png)
## 物理结构

1. 数据文件(.DBF)：用于储存数据库中所有数据；
2. 控制文件(.CTL)：用于记录和描述数据库的物理存储结构信息
3. （重做）日志文件(.log)：用于记录外部程序（用户）对数据库的修改操作
4. 初始化参数(.ORA):用于设置数据库启动时参数初始值;
5. 跟踪文件:用于记录用户进程、数据库后台进程的运行情况;
6. 归档文件(.ARC) :用于保存已经写满的重做日志文件
7. 口令文件:用于保存具有SYSDBA, SYSOPER权限的用户名和SYS用户口令


## 逻辑结构
数据库的逻辑结构是面向用户的，描述了数据库在逻辑上是如何组织和存储数据,数据库的逻辑结构支配一一个数据库如何使用其物理空间。
### 数据块(多个)-->区(多个)-->段(多个)-->表空间(多个)-->数据库
1. 数据块：最小的逻辑存储单元（即最小的I/o读写单元）。分为标准块和非标准块两种。  由数据库初始化参数 DB_BLOCK_SIZE 设置，大小不变。
![Oracle图片2](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片2.png)
2. ==区==：由一系列连续的数据块构成的逻辑存储单元，是存储空间分配的最小单元。（例：空表也会被分配一个区）
3. ==段==：由一个或多个连续或不联系的区组成的逻辑存储单元。分类：表段、索引段、临时段、回退段
4. ==表空间==：Oracle数据库最大的逻辑存储单元。由多个段组成。一个表空间可以对应若干 数据文件（属于物理逻辑存储结构）。
    1. 表空间分类：
        1. 系统表空间
        2. 非系统表空间：撤销表空间、临时表空间、用户表空间
![Oracle图片3](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片3.png)

#### 物理和逻辑存储结构的对应关系:
Oracle数据库的物理存储结构与逻辑存储结构之间的基本关系:
1. 一个数据库在物理上包含多个数据文件，在逻辑上包含多个表空间。
2. 一个表空间包含一个或多个数据文件，一个数据文件只能从属于某个表空间。
3. 数据库的逻辑块由一个或多个操作系统块构成。一个逻辑区只能从属于
一个数据文件，而一个数据文件可包含一个或多个逻辑区
![Oracle图片4](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片4.png)

## 内存结构
1. 系统全局区SGA：是一组共享内存结构
    1. 数据高速缓冲区（Data Buffer Cache）：
        - 在数据缓冲区中被修改后的数据由数据写入进程(DBWR)写到硬盘的数据文件中永久保存。
        - 提高获取和更新数据的性能
        1. 大小：DB_CACHE_SIZE
        3. 数据高速缓冲区越大，用户需要的数据在内存中的可能性越大，即缓存命中率高，从而减少了Oracle访问硬盘数据的次数，提高数据库系统执行的效率。需要确定一个合理的数据数据缓冲区的大小。
        ![Oracle图片5](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片5.png)
        4. 缓冲块的类型:
            - 脏缓存块(Dirty Buffers) :脏缓存块中保存的是已经被修改过的数据。
            - 空闲缓存块(FreeBuffers):空闲缓存块中不包含任何数据，它们等待后台进程或服务器进程向其中写入数据。
            -  命中缓存块(PinnedBuffers):命中缓存块是那些正被使用的数据块，同时还有很多会话等待修改或访问的数据块。
            - 干净缓存块(CleanBuffers):干净缓存块是指那些当前没有被使用，即将被换出内存的缓存块。
    ![Oracle图片6](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片6.png)
    2. 重做日志缓冲区：
        3. 存放数据库事务提交的操作信息，这些信息对数据库的恢复有着重要作用。当重做日志缓冲区被添满时，由日志写入进程把重做日志缓冲区的内容写到磁盘的重做日志文件中保存。
        3. 大小：log_buffer
        4. log_ buffer值越 大，重做日志缓冲区就可以存放更多的事务提交的记录，减少了数据被频繁写入到重做日志文件中的次数。
    3. 共享池：库高速缓存、数据字典高速缓存
        1. 功能：用于缓存与sql和pl/sql语句、数据字典、资源锁以及其他控制结构相关数据
        2. 组成：
            1. 库缓存：用于缓存已经解释并执行过的sql语句和pl/sql程序代码，以提高sql或pl/sql程序的执行效率；包括sql工作区和pl/sql工作区
            3. 数据字典缓存区：保存最常用的而数据字典信息。如：数据库对象信息、账户信息、数据库结构信息等。
        3. 大小：SHARED_POOL_ SIZE
        4. 合适的共享池大小，可使编译过的程序代码长驻内存，大大降低重复执行相同的SQl语句、PL/SQL程序的系统开销，从而提高数据库的性能。
    3. 大型池(Large Pool)：用于缓冲大型 I/O 请求的可选区域，以便支持并行查询、共享服务器、Oracle XA 以及某些类型的备份操作
    4. 流池(stream Pool)：由 Oracle Streams 使用
    5. java池(Java Pool)：用于存放 Java 虚拟机 (JVM) 中特定于会话的 Java 代码和数据
2. 程序全局区PGA（大小和SGA相比小很多，通常不讲）：保存用户私有的
    1. 排序区：
    2. 游标信息区：
    3. 会话信息区
    4. 堆栈区
    

## 进程结构
### Oracle进程结构

1. 进程的概念:进程是操作系统中一个独立的可以调度的活动，用于完成指定的任务。
2. 进程与程序的区别在于:
    - 进程是动态的概念，即动态创建，完成任务后立即消亡而程序是一个静态实体。    
    - 进程强调执行过程，而程序仅仅是指令的有序集合。
    
    1. 用户进程
    1.  服务器进程(Server process):服务器进程是接收用户进程信息，并根据请求与数据库进行通信。这些通信实现数据操作，完成用户对数据库数据的处理要求。
    1. 服务器进程主要完成以下任务:
        - 解析并执行用户提交的sql语句及PL/sql程序;
        - SGA的数据高速缓冲区中搜索用户进程所要访问的在数据，如果数据不在缓冲区中，则需要从硬盘数据文件中读取所需的数据，再讲它们复制到缓冲区中;
        - 将用户改变数据库的操作信息写入日志缓冲区中;
        - 将查询或执行后的结果数据返回给用户进程;
    2. 后台进程:
        1. 为了保证Oracle数据库在任意一个时刻可以处理多用户的并发请求，进行复杂的数据操作，而且还要优化系统性能，Oracle数据库起用了一些相互独立的附加进程，称为后台进程。服务器进程在执行用户进程请求时，调用后台进程来实现对数据库的操作。
    1. 后台进程主要完成以下任务:
        - 在内存与磁盘之间进行I/0操作;
        - 监视各个服务器进程状态;
        - 协调各个服务器进程的任务;
        - 维护系统性能和可靠性等。
### Oracle例程后台进程
1. 数据库写入进程(DBWR )
2. 日志写入进程(LGWR)
3. 日志归档进程(ARCH)
4. 查点进程(CKPT)
5. 系统监控进程(SMON)
6. 进程监控进程(PMON)等
![Oracle图片7](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片7.png)

1.==数据库写入进程(DBWR)==
- 数据库写入进程(databasewriter,DBwr)将缓冲区里的数据写入到数据文件。数据库写入进程的作用是将已更改的数据块从内存写入数据文件。使缓冲区有更多的空闲缓冲块，保证服务进程将所需要的数据从数据文件中读取到数据高速缓冲区，提高缓存命中率。
- 默认情况下，启动例程时只启动了一个数据库写入进程，即为DBW0
- 初始化参数DB_ WRITER PROCESSES最多定义20个数据库写入进程执行写入操作
- 每个数据库写入进程都分配了0~9或a~j编号
2.日志写入进程(LGWR)
- 日志写入进程负责把重做日志缓冲区的数据写入重做日志文
件中永久保存。
- 数据库写入进程在工作之前，需要了解日志写入进程是否已
经把相关的日志缓冲区中记载的数据写入重做日志文件中，
如果相关的日志缓冲区中的记录还没有被写入，DBWR会通
知LGWR完成相应的工作，然后DBWR才开始写入。
3. 检查点进程（CKPT）
- 检查点是一个事件，当该事件发生时（每隔一段时间发生），==DBWR==进程把数据高速缓冲区中==脏缓存块==写入数据文件中，同时Oracle将对数据库控制文件和数据文件的头部的==同步序号==进行更新，以记录下当前的数据库结构和形态，以确保数据文件、控制文件和重做日志文件的一致性
![Oracle图片8](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片8.png)
4. SMON
- 功能：在  启动时负责对数据库进行恢复
- 回收不再使用的临时空间
- 将各个表控件的空闲碎片合并
- 
5. PMON（进程监控进程）
 ![Oracle图片9](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片9.png)
6. 日志归档进程（ARCH）
- 归档进程负责在日志切换后将已经写满的重做日志文件复制到归档目标中，防止写满的重做日志文件被覆盖
- 最多可启动十个归档进程
- 该后台进程只有在ARCHIVELOG(归档日志)模式下才有效
- 默认情况下只有两个归档日志进程(ARC0和ARC1)
### 7. 数据字典
- 在数据库创建按过程中创建的，保存了数据库系统信息以及数据库中所有的对象信息，是数据库运行的技术
-  一系列的表和视图，这些表和视图对于所有的用户(包括DBA)，都是只读的
- 只有Oracle系统可以对数据字典进行管理与维护
- 在Oracle数据库中，所有数据字典表和视图都属于sys模式，存储于system表空间中
- Oracle数据字典保存数据库本身的系统信息及所有数据库对
象信息，包括:
    1. 各种数据库对象的定义信息，包括表、视图、索引、同义词、序列、存储过程、函数、包、触发器及其他各种对象;
    1. 数据库存储空间分配信息，如为某个数据库对象分配了多少空间，已经使用子多少空间等;
    1. 数据库安全信息，包括用户、权限、角色、完整性等;
    1. 数据库运行时的性能和统计信息;
    1. 其他数据库本身的基本信息。
- 数据字典的主要用途：
    1. 通过访问数据字典获取用户、模式对象、数据库对象定义与存储等信息，以判断端用户权限额合法性、模式对象的存在性以及存储空间的可用性
    2. 使用ddl语句修饰修改数据库队形后，将在数据字典中记录所做的修改
    3. 在任何数据库用户都可以从数据字典只读视图中任何数据库用户都可以从数据字典只读视图中获取各种数据库对象信息;
    4. DBA可以从数据字典动态性能视图中获取数据库的运行状态，作为进行性能调整的依据。
- 维护与管理：
    1. DDL：如增加或者减少表空间，增加或减少用户。
    2. DCL：如授予用户权限，回收用户权限。
    3. DML：某些DML语句，如引起表空间扩展的插入、修改语句

#### 数据字典的分类：
- 数据字典结构分为: 数据字典表和数据字典视图
    1. 静态数据字典视图
        通过对静态数据字典表进行解密和处理，创建了一系列用户可读的静态数据字典视图。在数据库创建过程中，通过自动运行catalog.sql脚本创建静态数据字典视图及其公共同义词，并进行授权
    2. 动态数据字典视图
        在动态性能表上创建的视图称为动态数据字典视图，又称为动态性能视图。所有动态性能视图命名都以V$开头，Oracle自动为这些视图创建了以V$开头命名的公共同义词，因此动态性能视图又称为“V$视图”
- 根据数据字典对象的虚实性可分为: 静态数据字典 和 动态数据字典 。
    1. 静态数据字典表：
        静态数据字典表是在数据库创建过程中自动运行sql.bsq脚本创建的，由SYS用户所拥有，表中信息都是经过加密处理的。静态数据字典表的命名中通常包含$符号。只有Oracle才能读/写这些静态数据字典表。
    2. 动态数据字典表：
动态数据字典表是在数据库实例运行过程中由Oracle动态创建和维护的一系列“虚表”，在实例关闭时被释放。动态数据字典表中记录与数据库运行的性能相关的统计信息，因此又称为动态性能表。通常，动态性能表的命名以X$开头。动态性能表由SYS用户所拥有

- 查询表dictionary,可以获得全部可以访问的数据字典表或数据字典视图的名称和解释;
- 查询表dict columns，，可以获得全部可以访问的数据字典表或数据字典视图中的字段名称和解释。
    - 示例
    ```
    SQL>SELECT * FROM dictionary;
    SQL> SELECT * FROM dict columns WHERE
    TABLE NAME='USER TABLES';
    ```
- 静态数据字典表的使用
    - 静态数据字典表只能由Oracle进行维护，用户不能对这些表进行直接操作。
- 静态数据字典视图的使用
    - 通常，用户通过对静态数据字典视图的查询可以获取所需要的所有数据库信息。Oracle静 态数据字典视图可以分为3类，各类视图具有独特的前缀。
    
    名称前缀 |  含义
    :-: | :-:
    USER_ |  包含了当前数据库用户所拥有的所有模式对象的信息
    ALL_  |   包含了当前数据库用户可以访问的所有模式对象的信息
    DBA_  | 包含了所有数据库对象信息，只有具有DBA角色的用户才能够访问这些视图
- 动态性能表的使用
    - 动态性能表都属于SYS用户，Oracle使用这些表生成动态性能视图。
- 动态性能视图的使用
    - 动态性能视图是SYS用户所拥有的，在默认情况下，只有SYS用户和拥有DBA角色的用户可以访问。与静态数据字典表和视图不同，在数据库启动的不同阶段只能访问不同的动态性能视图。 

![Oracle图片10](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片10.png)

--- 


# 第五章  用户管理
### 物理存储结构的规划
1. 规划数据文件。创建数据文件数量，设置文件的大小，扩展方式及文件在磁盘上的分配。规划数据文件的目的是数据库存储应满足业务数据变化的需要。
2. 规划控制文件和重做日志文件。文件的数量，存放位置等。目的是既能形成冗余，避免数据丢失，又能提高系统I/O的性能。
3. 将数据库设置成归档模式，及归档路径等相关信息的设置。目的保证当系统出现介质故障时能够完全的进行数据库的恢复。

### 逻辑存储结构的规划
1. 规划创建多个永久表空间，以便于实现数据的分区管理。
2. 规划创建索引空间，以便于实现索引数据和业务数据的分离。
3. 规划创建临时表空间，以便于对临时信息的管理。
4. 规划创建撤销表空间，以便实现对回退信息的管理。



## 表空间
![Oracle图片11](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片11.png)
1.表空间时Oracle数据库中最大的逻辑容器
2.一个表空间包含多个数据文件
3.数据库容量在物理上有数据文件的大小和数量决定，逻辑上由表空间大小和数量决定
![Oracle图片12](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片12.png)
### 1.创建表空间
1. 在创建本地管理方式下的表空间时，首先应该确定表空间的名称类型、对应的数据文件的名称和位置以及表空间的管理方式、区的分配方式、段的管理方式。
2. 表空间名称不能超过30个字符，必须以字母开头，可以包含字母数字以及一些特殊字符等;
3. 表空间属性:
    - 类型:永久性表空间(PERMANENT TABLESPACE)、临时表空间(TEMP TABLESPACE)、撤销表空间(UNDOTABLESPACE )、大文件表空间( BIGFILE TABLESPACE)
    - 表空间管理方式:字典管理方式(DICTIONARY)和本地管理方式(LOCAL)，默认是本地管理方式
    - 区分配方式:自动分配( AUTOALLOCATE)和定制分配(UNIFORM)，默认是自动分配
    - 段的管理方式:自动管理(AUTO) 和手动管理(MANUAL)，默认是自动管理
    ```
    例4：创建一个永久性的表空间hrtbs4，区定制分配，段采用手动管理方式。
    SQL> CREATE TABLESPACE hrtbs4 DATAFILE
        'd:\app\administrator\oradata\hrtbs4_1.dbf' 
        SIZE 50M EXTENT MANAGEMENT LOCAL UNIFORM SIZE 512K 
        SEGMENT SPACE MANAGEMENT MANUAL;
    ```
注： EXTENT MANAGEMENT LOCAL :表空间定置管理（可以不写，目前大多默认）
```
   例5：创建一个大文件表空间，文件大小为1G，区的分配采用定制方式。
    SQL> CREATE BIGFILE TABLESPACE big_tbs DATAFILE
    'd:\app\administrator\oradata\orcl\big01.dbf' 
    SIZE 1G UNIFORM SIZE 512K;
```  
注意：大文件表空间中段的管理只能采用自动管理方式，而不能采用手动管理方式。
```
        例6：创建一个临时表空间hrtemp1
    
         SQL>CREATE TEMPORARY TABLESPACE hrtemp1 TEMPFILE
         'd:\app\administrator\oradata\hrtemp1_1.dbf ' 
         SIZE 20M EXTENT MANAGEMENT LOCAL 
         UNIFORM SIZE 15M;
    
```



```
    例7：创建一个临时表空间hrtemp2，并放入临时表空间组temp_group 。
         同时，将临时表空间hrtemp1也放入该temp_group中。
    
        SQL>CREATE TEMPORARY TABLESPACE hrtemp2 TEMPFILE
        'd:\app\administrator\oradata\hrtemp2_1.dbf'  
        SIZE 20M  EXTENT MANAGEMENT LOCAL UNIFORM SIZE 15M
        TABLESPACE GROUP temp_group;
        ALTER TABLESPACE HRTEMP1 TABLESPACE GROUP temp_group;
```

### 修改表空间
表空间创建之后，都可以对表空间进行修改，包括： 
1. 表空间的扩展
    - 添加数据文件
    - 改变已有数据文件的大小
        - 改变数据文件的可扩展性
        - 重新设置数据文件的大小。
- 扩展表空间方法一：为表空间添加数据文件
- 可以通过ALTER TABLESPACE…ADD DATAFILE语句为永久表空间添加数据文件。
- 通过ALTER TABLESPACE…ADD TEMPFILE语句为临时表空间添加数据文件。

```
    例9：向USERS表空间中添加一个大小为10MB的数据文件。
       
         SQL>ALTER TABLESPACE users ADD DATAFILE 
        'd:\app\administrator\oradata\ users02.dbf' 
        SIZE 10M;  
```
    
- 说明：
    1. 如果添加的文件不存在，正常增加
    2. reuse语句使用是针对已经删除的表空间中的文件（但在操作系统层面没有删除的文件）

- 扩展表空间之方法二：改变数据文件的大小1
- 可以通过改变表空间已有数据文件的大小，达到扩展表空间的目的。

- 如果在创建表空间或为表空间增加数据文件时没有指定AUTOEXTEND ON选项，则该文件的大小是固定的。如果为数据文件指定了AUTOEXTEND ON选项，当数据文件被填满时，数据文件会自动扩展，即表空间被扩展了。
```
    例11：修改USERS表空间数据文件users02.dbf为自动增长方式。
       
        SQL> ALTER DATABASE DATAFILE
        'd:\app\administrator\oradata\users02.dbf'               
        AUTOEXTEND ON NEXT 1M MAXSIZE UNLIMITED;
```
```
     例12：取消USERS表空间数据文件USERS02.DBF的自动增长方式。
        SQL>ALTER DATABASE DATAFILE
        'd:\app\administrator\oradata\users02.dbf'
        AUTOEXTEND OFF;
```    
- 扩展表空间之方法三：改变数据文件的大小2
- 可以使用`ALTER DATABASE DATAFILE…RESIZE` 改变表空间已有数据文件的大小。
    ```
        例13：将USERS表空间的数据文件users02.dbf大小设置为8MB。
        
            SQL>ALTER DATABASE DATAFILE 
            'd:\app\administrator\oradata\users02.dbf'RESIZE 8M;
    ```   
2. 可用性
3. 读/写状态的修改

### 表空间删除
1. 语法: `DROP TABLESPACE tablespace_name`
    1. 如果表空间非空，应带有子句：
    ` INCLUDING CONTENTS`
    2. 若要删除操作系统下的数据文件，应带有子句：`AND DATAFILES`
- 示例
    （删除表空间，同时删除其所对应的数据文件）
    ```
    SQL>DROP TABLESPACE userdata(//tableSpaceName) 
    INCLUDING CONTENTS AND DATAFILES;
    ```
2. 如果其他表空间中的约束（外键）引用了要删除表空间中的主键或者唯一性约束，需要使用`CASCADE CONSTRAINTS`子句删除参照完整性约束，否则删除表空间时会报错。
    - 删除hrundo1表空间，同时删除其所对应的数据文件，以及其他表空间中与hrundo1表空间相关的参照完整性约束。
        ``` 
         DROP TABLESPACE hrundo1 INCLUDING CONTENTS   
          AND DATAFILES CASCADE CONSTRAINTS;
        ```

### 查询表空间信息
1. 表空间信息:
    - DBA_TABLESPACES
    - V$TABLESPACE
2. 数据文件信息:
    - DBA_DATA_FILES
    - V$DATAFILE
3. 临时文件信息:
    - DBA_TEMP_FILES
    - V$TEMPFILE
    
>表空间描述:<br/>
    `SQL> SELECT tablespace_name,block_size,initial_extent,max_extents FROM dba_tablespaces;`
> ![Oracle图片13](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片13.png)

> 查询表空间的名称,区管理方式,存储分配方式,类型等基本信息:<br/>
    `SELECT tablespace_name,extent_management,allocaton_type,contents FROM dba_tablespaces;`
> 查询表空间的数据文件信息:<br/>
    `SELECT file_name,blocks,tablespace_name 
    FROM dba_data_files;`
> 查询数据文件的基本信息:<br/>
    `SELECT name ,file#,rfile#,status,bytes
    FROM v$datafile;`
> **注： #，表示什么的号，一般为数字列。**

### 数据文件设置与管理
- 数据文件(Data files)用于存储数据和相关脚本的文件，包括系统数据(数据字典)、用户数据(表、索引、簇等)、撤销(Undo)数据、临时数据等。
    - 数据文件存储两种类型的数据：用户数据和系统数据。
        1. 用户数据.用户数据是指用于应用系统的数据，包括与应用系统的所有相关信息。如本书的人力资源管理系统中的员工信息表、职位信息、部门信息等。
        2. 系统数据.系统数据是指用于管理用户数据和Oracle数据库本身的数据。如表的结构、空间、用户、数据文件的位置（存放路径、访问时间等），数据字典。
> Oracle数据库中有一种特殊的数据文件，称为临时数据文件，属于数据库的临时表空间。临时数据文件中的内容是临时性的，在一定条件下自动释放。

### 数据文件
- 数据文件的存储策略
    - 由于对数据库的操作最终转换为对数据文件的操作，因此在数据库运行过程中对数据文件进行频繁的读写操作。为了提供I/O效率，应该合理的分配数据文件的存储位置。
    - 把不同存储内容的数据文件放置在不同的硬盘上，可以并行访问数据，提高系统读写的效率。
    - 初始化参数文件、控制文件、重做日志文件最好不要与数据文件存放在同一个磁盘上，以免数据库发生介质故障时，无法恢复数据库。 

> 数据文件与表空间的关系： <BR/>
    1. 一个表空间可以包含几个数据文件<BR/>
    2. 一个数据文件只能对应一个表空间
    
![Oracle图片14](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片14.png)
---

- 数据文件的管理
    - 创建数据文件
    - 修改数据文件的大小
    - 改变数据文件的可用性
    - 改变数据文件的名称和位置
    - 查询数据文件的信息

### 创建数据文件
1. 数据文件依附于表空间而存在，创建数据文件就是向表空间添加文件
2. 在创建数据文件时应该根据文件数据量的大小确定文件的大小以及文件的增长方式。 
3. 语法：
    ```
        - ALTER TABLESPACE…ADD DATAFILE…
        - ALTER TABLESPACE…ADD TEMPFILE…
    ```
4. 例子：
    ```
    例: 向TEMP表空间添加一个大小为5MB的临时数据文件。
       SQL>ALTER TABLESPACE temp ADD TEMPFILE
       'd:\app\administrator\oradata\temp03.dbf'
       SIZE  5M;
    ```
### 修改数据文件大小
1. 方法
    - 设置数据文件为自动增长方式。
    - 手工改变数据文件的大小。
2. 设置数据文件为自动增长方式
    - 创建时设置数据文件为自动增长
    - 创建后修改数据文件为自动增长
    
    - `AUTOEXTEND ON NEXT…MAXSIZE…| UNLIMITED`
3. 手工改变数据文件的大小
    -`ALTER DATABASE  DATAFILE…RESIZE…`
    ```
        例：为ORCL数据库的USERS表空间添加一个自动增长的数据文件。
    SQL>ALTER TABLESPACE users ADD DATAFILE
        ‘d:\app\administrator\oradata\orcl\userdata03.dbf' SIZE 10M 
         AUTOEXTEND ON NEXT 512K  MAXSIZE 250M;
    
    例：修改ORCL数据库USERS表空间的数据文件userdata02.dbf为自动增长。
    SQL>ALTER DATABASE DATAFILE
        'd:\app\administrator\oradata\orcl\userdata02.dbf ' 
        AUTOEXTEND ON NEXT  512K MAXSIZE UNLIMITED;
    ```
### 改变数据文件的可用性
1. 概念:可以通过将数据文件联机或脱机来改变数据文件的可用性。
2. 在下面几种情况下需要改变数据文件的可用性：
    - 要进行数据文件的脱机备份时，需要先将数据文件脱机；
    - 需要重命名数据文件或改变数据文件的位置时，需要先将数据文件脱机；
    - 如果Oracle在写入某个数据文件时发生错误，会自动将该数据文件设置为脱机状态，并且记录在警告文件中。排除故障后，需要以手动方式重新将该数据文件恢复为联机状态。
    - 数据文件丢失或损坏，需要在启动数据库之前将数据文件脱机。
3. 归档模式下数据文件可用性的改变
    - 数据文件可用性的改变
    `ALTER DATABASE DATAFILE… ONLINE|OFFLINE`

    - 临时数据文件可用性的改变
    `ALTER DATABASE TEMPFILE… ONLINE|OFFLINE`
4. 例子：（数据文件脱机后需要进行恢复再联机才可以完成联机）
    ```
    1. 将USERS表空间的数据文件USERS02.DBF脱机
       SQL>ALTER DATABASE DATAFILE 
       'd:\app\administrator\oradata\ users02.dbf' OFFLINE;
    2. 在归档模式下，将数据文件联机之前需要使用RECOVER DATAFILE语句对数据文件进行恢复
       SQL> RECOVER DATAFILE    'd:\app\administrator\oradata\users02.dbf‘
    3. 将USERS表空间的数据文件USERS02.DBF联机
        SQL>ALTER DATABASE DATAFILE
        'd:\app\administrator\oradata\users02.dbf' ONLINE;
    ```
###  改变数据文件的名称和位置
1. 改变数据文件的名称和位置分为两种情况
    - 如果数据文件属于同一个表空间，使用:
    
    `ALTER ==TABLESPACE== … RENAME DATAFILE … TO`
    - 如果数据文件属于多个表空间，使用:
    
    `ALTER ==DATABASE==  … RENAME FILE … TO`

2. 注意：
    > 改变数据文件的名称或位置时，Oracle只是改变记录在控制文件和数据字典中的数据文件信息，并没有改变操作系统中数据文件的名称和位置，因此需要DBA手动更改操作系统中数据文件的名称和位置
3. 改变同一个表空间的数据文件步骤：
    - (1)表空间脱机 
    
     `ALTER TABLESPACE tablespace_name… OFFLINE`
    - (2)修改操作系统中文件名称或位置
    - (3)执行ALTER 重命名语句
    
    `ALTER TABLESPACE tablespace_name…RENAME DATAFILE…TO`
    - (4)表空间联机
    `ALTER TABLESPACE tablespace…ONLINE`
```
将ORCL数据库中USERS表空间的数据文件users01.dbf移动到d:\app\administrator\oradata目录中。
    (1) ALTER TABLESPACE users OFFLINE;
    (2) HOST COPY        
          d:\app\administrator\oradata\orcl\users01.dbf (原位置)
          d:\app\administrator\oradata\users01.dbf （新位置）
    (3) ALTER TABLESPACE users RENAME DATAFILE
         'd:\app\administrator\oradata\orcl\users01.dbf' TO
         'd:\app\administrator\oradata\users01.dbf'
    (4) ALTER TABLESPACE users ONLINE;

```
### 查询数据文件信息
1. 数据文件信息
    - dba_data_files:包含数据库中所有数据文件的信息，包括数据文件所属的表空间、数据文件编号等。
    - v$datafile:包含从控制文件中获取的数据文件信息。
2. 临时文件信息
    - dba_temp_files:包含数据库中所有临时数据文件的信息。
    - $tempfile:包含所有临时文件的基本信息。 
3. 例子：
    ```
    1. 查询数据文件动态信息
    SELECT name,file#,status,checkpoint_change# FROM v$datafile 
    2. 查询数据文件的增长方式
    SELECT tablespace_name,bytes,autoextensible,
       file_name FROM   dba_data_files 
    3. 查询临时数据文件信息
    SELECT tablespace_name,file_name, autoextensible FROM dba_temp_files;

    ```
---
---
---
### 控制文件设置与管理
- 控制文件是一个很小的二进制文件。用于记录和维护数据库的物理结构, 包括:数据库名称、数据文件和重做日志文件的名称和位置等。
- 一个实例只能访问一个数据库，通过控制文件在实例和数据库之间建立关联。
- Oracle启动时通过控制文件查找数据文件位置和联机重做日志。
- 数据库运行时，控制文件被不断更新。
- 数据库至少要包含一个控制文件,一个数据库也可以同时拥有多个控制文件，Oracle建议使用多个控制文件避免因单个控制文件损坏而导致的数据库无法启动。
- 控制文件对数据库至关重要，应联机保存多个备份，存储在不同的磁盘上。


1. 数据库控制文件名通过init.ora文件的CONTROL_FILES  参数规定。
    - 主要包含信息类型：
    - (1) 数据名 
    - (2) 数据库创建时间 
    - (3) 数据文件和重做日志文件的存放位置 
    - (4) 表空间名 
    - (5) 当前日志序列号 
    - (6) 检查点信息 
    - (7) 关于重做日志和归档的当前状态信息 
![Oracle图片15](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片15.png)

    - v$controlfile中保存着控制文件信息的最基本信息。
    - (1) DESC  v$controlfile;
    - (2) SELECT * FROM v$controlfile;
    ```
        SQL>SELECT name FROM v$controlfile;
        运行结果：
        NAME
        -- -- --
        D:\app\administrator\oradata\orcl\control01.ctl
        D:\app\administrator\recvery_area\orcl\control02.ctl
    ```

2. 创建控制文件
    - 实现多路复用控制文件
    - 备份控制文件
    - 删除控制文件
    - 查看控制文件的信息

3. 创建控制文件的情形
    - 创建数据库时，需要创建控制文件；
    - 控制文件全部丢失或损坏；
    - 需要修改某个永久性数据库结构参数；
    ```
        如：
            数据库名称、
            MAXLOGFILES（ 最大重做日子文件数量）、
            MAXLOGMEMBERS（重做日志文件组中最大成员数量）、
            MAXDATAFILES（最大数据文件数量）、
            MAXINSTANCES（同时可访问数据库最大实例个数）等。
    ```

4. 创建控制文件的基本步骤 
    - 列出数据库中所有的数据文件和重做日志文件的名称和路径
    ```
        select member from v$logfile;
        select name from v$datafile;
        select value from v$parameter where name='control_files';   
    ```
    - 如果数据库仍然处于运行状态，则关闭数据库
   ` SHUTDOWN IMMEDIATE`
    - 在操作系统级别备份所有的数据文件和联机重做日志文件
    - 启动实例，STARTUP NOMOUNT

4. 创建控制文件的基本步骤 (续)
    - 利用前面得到的文件列表，执行CREATE CONTROLFILE创建一个新控制文件。
    - 在操作系统级别对新建的控制文件进行备份
  `ALTER DATABASE BACKUP CONTROLFILE TO 'FILE'` 
    - 如果数据库需要恢复，则进行恢复数据库操作
    - 如果创建控制文件时指定了NORESTLOGS，可以完全恢复数据库。
    ` RECOVER DATABASE ;`
    - 如果创建控制文件时指定了RESETLOGS，则必须在恢复时指定USING BACKUP CONTROLFILE。        
     `RECOVER DATABASE USING BACKUP CONTROLFILE;`
4. 创建控制文件的基本步骤 (续)
    - 重新打开数据库
        - 如果数据库不需要恢复或已经对数据库进行了完全恢复，则可以正常打开数据库。
    `ALTER DATABASE OPEN;`
        - 如果在创建控制文件时使用了RESETLOGS参数，则必须指定以RESETLOGS方式打开数据库
    `ALTER DATABASE OPEN  RESETLOGS;`


### 补充：oracle的启动状态

Oracle数据库启动的基本步骤 
1. nomount：读初始化参数文件，启动实例。
   ==startup nomount== //读取初始化参数init.ora文件，启动instance，即启动SGA和后台进程，这种启动只需要init.ora文件。这种方式启动下可执行：重建控制文件、重建数据库
2. mount ： 执行“nomount”，然后打开控制文件，确认数据文件和联机日志文件的位置，但此时不对数据文件和日志文件进行校验检查(是否存在)。这种方式启动下可执行：数据库日志归档、数据库介质恢复、使数据文件联机或脱机。
3. open ：先执行“nomount”，然后执行“mount”，再打开包括Redo log文件在内的所有数据库文件，即所有数据文件，日志文件。这种方式下可访问数据库中的数据。可以对全体用户提供服务了。
![Oracle图片16](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片16.png)


### 实现多路复用控制文件
1.控制文件多路复用的特点
1. 在数据库服务器上将控制文件存放在多个磁盘分区或者多块硬盘上。
2. 数据库系统在需要更新控制文件的时候，就会自动同时更新多个控制文件。如此的话，当其中一个控制文件出现损坏时，系统会自动启用另外的控制文件。所以采用多路复用控制文件可以在很大程度上提高控制文件的安全性。
3. 最重要的是，在控制文件转换的过程之中，不会有停机现象的产生。
4. 多个镜像文件通过参数文件的 control_files设置。
<br/>

2.多路复用控制文件创建步骤
1. 编辑初始化参数CONTROL_FILES 
用来(从逻辑上（dbms角度）增加控制文件。)
2. 关闭数据库 
`SHUTDOWN IMMEDIATE; `
3. 拷贝一个原有的控制文件到新的位置，并重新命名(从物理上（操作系统角度）增加控制文件，与第一步中增加的文件相对应。)
4. 重新启动数据库 
STARTUP 
- 例子：
    ```
    例：当前数据库的控制文件为control01.ctl和control02.ctl，再添加一个名为control03.ctl的控制文件。
        （1）alter system set control_files=
                   'd:\app\administrator\oradata\control01.ctl',
                   'd:\app\administrator\oradata\control02.ctl',
                   'd:\app\administrator\control03.ctl'    scope=spfile;
        （2）shutdown immediate
        （3）host copy   
        d:\app\administrator\oradata\control01.ctl
        d:\app\administrator\control03.ctl
        （4）startup
    ```

### 删除控制文件
删除控制文件的步骤：
- 编辑CONTROL_FILES初始化参数，使其不包含要删除的控制文件（dbms级上逻辑删除）
- 关闭数据库
- 在操作系统中删除控制文件（实际删除文件）
- 重新启动数据库 


---
操作：
- ![Oracle图片17](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片17.png)
- 退出数据库，
- 在操作系统下建好要添加的控制文件
![Oracle图片18](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片18.png)
- ORA-01507错误处理：
    - ![Oracle图片19](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片19.png)

# 第六章
![Oracle图片20](https://gitee.com/RoleHalo/blog-image/raw/master/image/Oracle图片20.png)
####    5. DQL:数据检索，包括select
### 模式
1. 创建数据库，并完成数据库的存储设置后，就可以根据应用的需求设计来创建所需要的数据库对象，并使用这些数据库对象。
2. Oracle数据库常用对象主要包括表、索引、视图、序列、分区表与分区索引等等。

3. 模式(Schema)概念
     - 是指一系列逻辑数据结构或对象的集合。在创建用户的时候，会同时生成一个与用户同名的方案(模式)，此方案归同名用户所有。
    - 通常情况下，用户所创建数据库对象都保存在与自己同名的模式中。
    - 同一模式中数据库对象的名称必须惟一，而在不同模式中的数据库对象可以同名。
    - 默认情况下，用户引用的对象是与自己同名模式中的对象，如果要引用其他模式中的对象，需要在该对象名之前指明对象所属模式。  
---
## 约束与参数

约束（constraint）
约束作用
    是在表中定义的用于维护数据库完整性的一些规则。通过对表中列定义约束，可以防止在执行DML（Data Manipulation Language）操作时，将不符合要求的数据插入到表中。
约束类型
PRIMARY KEY：主键约束
UNIQUE：惟一性约束
CHECK：检查约束
FOREIGN KEY：外键约束
NULL/NOT NULL：空/非空约束

约束的两种形式：列级约束和表级约束。
列级约束：从形式上看，在每列定义完后马上定义的约束，在逗号之前就定义好了。
定义列级约束的语法为：
     [CONSTRAINT constraint_name] constraint_type 
     [conditioin]; 
举例：
create   table     parent(c1 number  primary key );    
create   table    child  (c  number primary key ,   c2 number  references parent(c1));。
注意
Oracle约束通过名称进行标识。在定义时可以通过CONSTRAINT关键字定义约束命名。如果用户没有为约束命名，Oracle将自动为约束命名。 

表级约束：在表中列都定义完后在单独定义约束
语法： [CONSTRAINT constraint_name]
           constraint_type([column1_name,
           column2_name,…]|[condition]);
举例：
Create   table  child( c number ,  c2  number  , primary key (c2),  foreign key(c2)  references  parent(c1));
有些时候，列级约束无法实现某种约束的定义，比如联合主键的定义，就要用到表级约束:
create table test(id1 number , id2 number, primary key(id1, id2));

### PRIMARY KEY：主键约束

PRIMARY KEY特点
定义主键，起惟一标识作用，其值不能为NULL，也不能重复；
一个表中只能定义一个主键约束；
建立主键约束的同时，在该列上建立一个惟一性索引，可以为它指定存储位置和存储参数；
主键约束可以是列级约束，也可以是表级约束。

### 惟一性约束：UNIQUE
惟一性约束特点
定义为惟一性约束的某一列或多个列的组合的取值必须惟一；
如果某一列或多个列仅定义惟一性约束，而没有定义非空约束，则该约束列可以包含多个空值；
Oracle自动在惟一性约束列上建立一个惟一性索引，可以为它指定存储位置和存储参数；
惟一性约束可以是列级约束，也可以是表级约束。 

#### PRIMARY KEY与UNIQUE比较
在一个基本表中只能定义一个PRIMARY KEY约束，但可定义多个UNIQUE约束；
对于指定为PRIMARY KEY的一个列或多个列的组合，其中任何一个列都不能出现空值，而对于UNIQUE所约束的唯一键，则允许为空。
不能为同一个列或一组列既定义UNIQUE约束，又定义PRIMARY KEY约束。

### 检查约束
特点
检查约束用来限制列值所允许的取值范围，其表达式中必须引用相应列，并且表达式的计算结果必须是一个布尔值；
一个列可以定义多个检查约束；
检查约束可以是列级约束，也可以是表级约束。 

### 外键约束：FOREIGN KEY
概念
FOREIGN KEY约束指定某一个列或一组列作为外部键，其中，包含外部键的表称为从表，包含外部键所引用的主键或唯一键的表称主表。
系统保证从表在外部键上的取值要么是主表中某一个主键值或唯一键值，要么取空值。以此保证两个表之间的连接，确保了实体的参照完整性。

特点
定义外键约束的列的取值要么是主表参照列的值，要么为空；
外键列只能参照于主表中的主键约束列或惟一性约束列；
可以在一列或多列组合上定义外键约束；
外键约束可以是列级约束，也可以是表级约束。

### 空/非空约束：NULL/NOT NULL 
特点
在同一个表中可以定义多个NOT NULL约束；
只能是列级约束。

## 参数
### 参数（parameter_list）
1. 在定义表时，可以通过参数设置表存储在哪一个表空间中，和存储空间分配等。
    - TABLESPACE：TABLESPACE子句用于指定表存储的表空间。 
    - STORAGE：STORAGE子句用于设置表的存储参数。若不指定，则继承表空间的存储参数设置。 
NITIAL 
NEXT
PCTINCREASE
MINEXTENTS
MAXEXTENTS
BUFFER_POOL (KEEP、RECYCLE、DEFAULT) 

STORAGE参数设置需注意
如果表空间管理方式为EXTENT MANAGEMENT LOCAL AUTOALLOCATE，则在STORAGE中只能指定INITIAL，NEXT和MINEXTENTS这3个参数；
如果表空间管理方式为EXTENT MANAGEMENT LOCAL UNIFORM，则不能指定任何STORAGE子句；
如果表空间管理方式为EXTENT MANAGEMENT DICTIONARY，则在STORAG中可以设置任何参数。

数据块管理参数 
PCTFREE：用于指定数据块中必须保留的最小空闲空间。
PCTUSED：用于指定当数据块空闲空间达到PCTFREE参数的限制后，数据块能够被再次使用前，已占用的存储空间必须低于的比例。
INITRANS：用于指定能够并发访问同一个数据块的事务的数量。
MAXTRANS：用于指定能够并发访问同一个数据块的事务的最大数量。


LOGGING与NOLOGGING子句
默认为NOLOGGING，即表的创建操作不会记录到重做日志文件中，尤其适合通过查询创建表的情况。使用LOGGING子句，表的创建操作（包括通过查询创建表时的插入记录操作）都将记录到重做日志文件中。
PARALLEL、NOPARALLEL   并行建表
CACHE、NOCACHE  表中数据是否缓存

### 索引
1. 索引概念及作用：索引是为了加速对表中元组的检索而创建的一种分散存储结构；
2. 是对表而建立的，由除存放表的数据页面以外的索引页面组成，独立于被索引的表；
3. 通过使用索引加速行的检索，但减慢更新的速度；
4. 快速定位数据，减少磁盘 I/O；
5. Oracle自动使用、维护索引



### 序列



### 分区
#### 1. 分区概念
- 所谓的分区是指将一个巨型表或巨型索引分成若干独立的组成部分进行存储和管理，每一个相对小的、可以独立管理的部分，称为原来表或索引的分区。
- 每个分区都具有相同的逻辑属性，但物理属性可以不同。如具有相同列、数据类型、约束等，但可以具有不同的存储参数、位于不同的表空间等。
分区后，表中每个记录或索引条目根据分区条件分散存储到不同分区中 。

#### 2. 对巨型表进行分区具有下列优点：
- 提高数据的安全性，一个分区的损坏不影响其他分区中数据的正常使用。
- 将表的各个分区存储在不同磁盘上，提高数据的并行操作能力。
- 简化数据的管理，可以将某些分区设置为不可用状态，某些分区设置为可用状态，某些分区设置为只读状态，某些分区设置为读写状态。
- 操作的透明性，对表进行分区并不影响对数据进行操作的SQL语句。


- 分区条件：
    - 表的大小超过2GB
    - 要对一个表进行并行DML操作，必须分区;
    - 为了平衡硬盘的I/O操作，将一个表分散存储在不同的表空间中，必须对它进行分区;
    - 如果需要将表一部分设置为只读，另一部分为可更新的，必须对表进行分区;

#### 范围分区：
- 语法:
```
        create table table(…)
        partition by range (column1[,column2,…])
        ( partition partition1 values less than(literal|maxvalue) [tablespace tablespace]
        [,partition partition2 values less than(literal|maxvalue) [tablespace tablespace],…]
        )…
```
    
- 其中：
    - PARTITION BY RANGE：指明采用范围分区方法。
    - column：分区列，可以是单列分区，也可以是多列分区。
    - PARTITION partition1 ：设置分区名称。
    - VALUES LESS THAN：设置分区列值的上界。
    - TABLESPACE：设置分区对应的表空间。

- 例子：
    ```
    创建一个分区表，将学生信息根据其出生日期不同进行分区，将1980年1月1日前出生的学生信息保存在TBS1表空间，
    1980年1月1日到1990年1月1日出生的学生信息保存在TBS2表空间中，其他学生信息保存在TBS3表空间中。 
    create table student_range(
        sno number(6) primary key, 
        sname   varchar2(10),   
        sage int,  
        birthday date
    )
     partition by range(birthday)
    (   partition p1 values less than(to_date('1980-1-1', 'yyyy-mm-dd'))   tablespace  tbs1,
     	partition p2 values less than(to_date('1990-1-1', 'yyyy-mm-dd'))  tablespace  tbs2,
     	partition p3 values less than(maxvalue) tablespace tbs3 
    );
    
    ```
####  列表分区
- 语法:
```
    create table table(…)
    partition by list(column)
    ( partition  partition1   values([literal|null]|[default])  [tablespace  tablespace]
     [,partition partition2  values([literal|null]|[default]) [tablespace tablespace],…]
    )…
```
- 例子：创建一个分区表，将学生信息按性别不同进行分区，男学生信息保存在表空间TBS1中，而女学生信息保存在TBS2中 。
> 注意需要先创建TBS1， TBS2
CREATE TABLESPACE TBS1 DATAFILE  'E:\oracle\product\10.2.0\oradata\orcl\ORCLTBS1_1.DBF' SIZE 50M;

```
        create table student_list(
    	sno number(6) primary key,
        sname varchar2(10),
        sex   char(2) check(sex in ('m', 'f'))
        )
        partition by list(sex)
        (  
            partition  stu_male  values('m') tablespace tbs1,
            partition stu_female values('f') tablespace  tbs2
        );
    
```
#### 散列分区
- 语法：
    ```
       create table table(…)
        partition by hash (column1[,column2,…]) 
        [(partition partition [tablespace tablespace][,…])]|
        [partitions hash_partition_quantity  store in (tablespace1[,…])]…
        参数：
        paritition by hash（col1,…)
        使用partition指定分区数量及store in指定分区存储空间；或使用partiton指定每个分区名称以及其存储空间。
    ```
- 例子：
    ```
        例：创建一个分区表，根据学号将学生信息均匀分布到TBS1 
               和TBS2两个表空间中 。
             create table student_hash (
               sno number(6) primary key,
               sname varchar2(10)
              )
             partition by hash(sno)
            ( partition p1 tablespace  tbs1,
               partition p2 tablespace  tbs2 
            ); 
    ```
#### 复合分区
1.  复合分区包括:
    - 范围-列表复合分区:
        - 范围-列表复合分区先对表进行范围分区，然后再对每个分区进行列表分区，即在一个范围分区中创建多个列表子分区。
        - 创建一个范围-列表复合分区表，将1980年1月1日前出生的男、女学生信息分别保存在TBS1和TBS2表空间中，1980年1月1日到1990年1月1日出生的男、女学生信息分别保存在TBS3和TBS4表空间中，其他学生信息保存在TBS5表空间间中。  创建一个范围-列表复合分区表，将1980年1月1日前出生的男、女学生信息分别保存在TBS1和TBS2表空间中，1980年1月1日到1990年1月1日出生的男、女学生信息分别保存在TBS3和TBS4表空间中，其他学生信息保存在TBS5表空间中。
    
    ```
        create table student_range_list(
             sno number(6) primary key,
             sname varchar2(10), 
             sex   char(2) check(sex in ('m','f')),
             sage  number(4), birthday date
         )
        partition by range(birthday)
        subpartition by list(sex)
        (partition p1 values less than(to_date('1980-1-1', 'yyyy-mm-dd'))
            (subpartition p1_sub1 values('m') tablespace tbs1,
             subpartition p1_sub2 values('f') tablespace tbs2),
        partition p2 values less than(to_date('1990-1-1', 'yyyy-mm-dd'))
            (subpartition p2_sub1 values('m') tablespace tbs3,
             subpartition p2_sub2 values('f') tablespace tbs4),
        partition p3 values less than(maxvalue) tablespace tbs5
        ); 
    
    ```
    - 范围-散列复合分区:
        - 范围-散列复合分区先对表进行范围分区，然后再对每个分区进行散列分区，即在一个范围分区中创建多个散列子分区。
        - 示例:
        - 创建一个范围-散列复合分区表，将1980年1月1日前出生的学生信息均匀地保存在ORCLTBS1和ORCLTBS2表空间中，1980年1月1日到1990年1月1日出生的学生信息保存在ORCLTBS3和ORCLTBS4表空间中，其他学生信息保存在ORCLTBS5表空间中。  
    ```
        create table student_range_hash(
       	sno number(6) primary key,
     	sname varchar2(10),
      	sage  number(4),
      	birthday date
      	)
      	partition by range(birthday)
      	subpartition by hash(sage)
    	(partition p1 values less than(to_date('1980-1-1', 'yyyy-mm-dd'))
			(subpartition p1_sub1 tablespace orcltbs1,
			 subpartition p1_sub2 tablespace orcltbs2),
        partition p2 values less than(to_date("1990-1-1", "yyyy-mm-dd"))
            (subpartition p2_sub1 tablespace orcltbs3,
             subpartition p2_sub2 tablespace orcltbs4),
        partition p3 values less than(maxvalue) tablespace orcltbs5
      	);
    
    ```
- 创建复合分区时需要指定:
    - 分区方法`（partition by range）`
    - 分区列
    - 子分区方法`（subpartition by hash， subpartition by list）`
    - 子分区列
    - 每个分区中子分区数量或子分区的描述。
