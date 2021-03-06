# 《Oracle数据库精讲与疑难解析》

标签（空格分隔）： DataBase

---

>另參考：
><http://blog.csdn.net/robertkun/article/details/17031281>

默認：
SYSTEM    MANAGER
sys       change_on_install

#第4章 Oracle 网络管理

《Oracle数据库精讲与疑难解析》


包括：服务器端，客户端，防火墙等。Oracle网络是以操作系统为基础的。

两个角度；操作系统的网络配置；Oracle自身的网络配置。

## Oracle网络体系结构


### Oracle Net
是Oracle的网络组件，用于建立客户端或中间件到服务器的连接和维护。


在客户端，Oracle Net是后台组件，SQL plus 等会用到它。在服务器端，Oracle Net包括一个活动的进程叫监听器
（它负责外部程序和服务器直接的连接）。Oracle Net必须同时位于客户端和服务器。

### 监听器  （服务器端）

一旦客户端通过监听器与服务器连接之后，就不需要监听器的参与。


监听器配置文件 listener.ora ，在监听器启动时读取。该文件可以通过 lsnrctl status来查看，它会列出其路径。


#### 连接描述符(connect identifier)   在客户端配置
在Oracle Net中， 。。 。 由连接描述符来识别。

由于连接描述符涉及的信息多，我们把它起一个别名，这样的别名叫做"网络服务名"，这样连接时就只需输入网络服务名即可， 它存储在 tnsnames.ora文件中。
在服务器上该文件的位置由环境变量 TNS_ADMIN指定；通常存放在 $ORACLE_HOME/network/admin

两种连接形式
使用连接字符串：  >sqlplus system/wy@xxxx:1521/nbo                #其中主机名xxxx，端口1521，实例名 nbo
使用网络服务名： >sqlplus system/wy@tonbo						   #tonbo是网络服务名， 


### 配置Oracle网络的工具

Oracle网络助手：
Oracle Net Manage网络管理工具：命令是： netmgr
Listener Control Utility是一个命令行工具：
...


## Oracle 网络管理

### 服务器端配置

#### 配置监听器
1. 通过Net Manager。（最终目的也是修改listener.ora文件）命令是： netmgr    （不知为什么界面选项不全）
2. 或通过网络配置助手，使用命令： netca
3. 直接通过编辑 listener.ora文件

配置好监听器后使用命令：lsnrctl start LISTENER 启动LISTENER这个监听器。由于LISTENER是默认监听器，所以可以直接lsnrctl start。

也可以先进入监听器控件，直接使用命令 lsnrctl 然后再使用 start 或 stop；或使用 help 查看帮助；使用 services查看详细信息。
监听器状态： ready表示可以接受连接；blocked表示不能接受连接;... 。



	/ora01/app/oracle/product/11.2.0/db_1/network/admin/listener.ora
​	


我出现的错误:
```
#该错误的解决方法是编辑文件 dbstart 和 dbshut文件，将 ORACLE_HOME_LISTNER=$1 该为 ORACLE_HOME_LISTNER=$ORACLE_HOME
dbstart       #提示信息
ORACLE_HOME_LISTNER is not SET, unable to auto-start Oracle Net Listener
```


```
#命令 lsnrctl status
TNS-12541: TNS:no listener TNS-12560: TNS:protocol adapter error TNS-00511: No listener Linux Error: 111: Connection refused
```

更改了 /etc/hosts 文件后又出现： 
```
#该消息位于/ora01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
<msg time='2016-09-24T18:32:42.792+08:00' org_id='oracle' comp_id='tnslsnr'
 type='UNKNOWN' level='16' host_id='centos7.localdomain
#centos7.localdomain
# localhost.localdomain'
 host_addr='::1%1'>
 <txt>TNS-12533: TNS:illegal ADDRESS parameters
 TNS-12560: TNS:protocol adapter error
  TNS-00503: Illegal ADDRESS parameters

```


原因就是我不会配置 /etc/hosts文件，造成各种错误，
我在locallhost行添加了centos7，并在.bashrc中添加了 ORACLE_HOSTNAME=centos7。
解决方法：编辑该监听器在listener.ora中的HOST = 127.0.0.1；这样启动时没有问题。不知道为什么必须输入ip地址才行。


由于经常需要进入这几个目录，所以为其设置了别名，在.bashrc文件中。


### 客户端配置

首先确认已安装客户端软件。


#### 配置步骤：
1. 启动Oracle Net Configuration Assistant。（也可以使用Net Manager来配置）
    在$ORACLE_HOME/bin下运行 netca 命令
2. 命名方法配置：一般选择本地命名
3. 设置服务名：就是实例名
4. 设置主机名和端口号
5. 设置网络服务名（就是之前一系列连接字符串的别名），自己起一个好记忆的。比如这里是 NNC


#### 测试客户端到服务器的连接：
（直接在命令行）使用 tnsping命令

测试示例： tnsping NNC


#### 登录远程数据库
要对远程数据库进行访问，必须先登录远程数据库。可以使用sqlplus或 OEM 进行登录。

使用命令 sqlplus 
SQL> system@NNC
请输入口令：



# 第5章 系统管家婆 `sql*plus`
由于markdown中的 `*`不好处理，就直接使用 sqlplus代替 `sql*plus`。

`sql*plus`有命令行接口、图形界面接口(GUI)、以及基于Web界面的接口。图形界面接口(GUI)启动方式是使用命令: sqlplusw

`sql*plus`也有自己的命令(只能在`sql*plus`中执行的专有命令)和环境。

**在sqlplus中执行操作系统命令：**
使用HOST关键字，后跟操作系统命令。

## 5.1 SQL缓冲区（SQL Buffer）
该缓冲区用于存放用户输入的SQL语句或PL/SQL块。但是它不存储sqlplus的专有命令。
该内容一直被保持，直到被下一条sql语句覆盖。我们可以通过list命令查看sql缓冲区内容。


## 5.3 sqlplus的环境配置
sqlplus环境变量，控制sqlplus的行为。如：列宽，是否自动提交，是否显示记录编号等。

可以通过使用set命令设置sqlplus环境变量，也可通过show命令显示环境变量。


### sqlplus站点配置文件
site profile ： glogin.sql文件
该文件位于： $ORALE_HOME/sqlplus/admin/ 

### sqlplus用户配置文件

## 。。。

## sqlplus专有命令








### 登录注销命令
命令： connect （连接数据库）
用法：登录数据库



命令：disconncect （退出登录）
语法： disc[onnect]
用法：退出数据库，但并不退出sqlplus



### 编辑命令

命令： INPUT  （添加行）
格式： I[NPUT] [text]
语法：在当前缓冲区中的当前行在添加一行

```
命令：list  (显示行的内容)
语法：L[IST] [n | n m | n * | * | * LAST]
用法：显示指定行的内容
```

命令：del （删除）
语法：与list类似
用法：删除缓冲区中指定的行。 
示例：del		(删除当前行）


##sqlplus实务与疑难解析
- sqlplus中执行sql语句：sql语句以分号`;`或斜杠`/`结束，其中斜杠必须独占一行。
- 设置sqlplus自动提交：sqlplus提交的命令为 commit；设置自动提交的方法： set autocommit on
- 终止语句执行： ctrl + c
- 在sqlplus中执行操作系统命令： 使用 HOST 关键字，加操作系统命令。 host ls
- 在sqlplus中执行存储过程： exec + 存储过程名字
- 如何重复执行一条sql语句： 直接使用斜杠`/` 
- 如何在启动sqlplus时不出现登录界面： sqlplus /nolog
- 如何运行一个sql脚本文件：使用start或`@`命令都可以。后跟脚本的路径。
- 把sql缓冲区的内容保存到操作系统文件中：使用save命令保存，使用get命令从文件中取出。例如; save 'd:\mysql.txt';另还有 save 'd:\mysql.txt' append; 进行追加。save 'd:\mysql.txt' replace; 替换
- 如何编辑sql缓冲区中的语句：使用 edit 调用操作系统编辑器对缓冲区中内容进行修改(使用哪个编辑机由glogin.sql中的DEFINE_EDITOR来决定)
- 如何显示和更改sqlplus使用的系统默认编辑器： DEFINE_EDITOR  。更改： DEFINE_EDITOR = vi
- 如何将查询结果保存到文件中：先使用spool命令后接文件路径；之后执行的命令结果会输出到文件中；在使用 spool off 关闭输出。
- 查看sqlplus的系统变量：查看所有系统变量 show all； 查看单个系统变量 show 当个系统变量名。
- 设置sqlplus系统变量： 使用set命令。


#### sqlplus的输出比较凌乱：
因为sqlplus不能对sql语句返回的结果进行自动格式化。可以使用column命令和format命令对其进行格式化，但比较麻烦，所以当结果凌乱时建议使用其它工具。

示例： 
```
#将recv_time列显示的宽度限制为40个字符；其中A表示字符。
column recv_time format A40;

```





# 第6章 数据库的启动与关闭

## 6.1 数据库启动
### 6.1.1 数据库启动原理
启动时要经历3个阶段： 

1. 启动实例（start An Instance)
2. 装载数据库（Mount The Database)
3. 打开数据库 (Open The Database)


这一章真是太棒了，见书。


#### 启动实例（start An Instance)

根据参数文件(PFILE或SPFILE) --- 分配系统全局区(SGA)内存 ---- 启动一系列后台进程

这些内存和进程合起来就组成实例（Oracle Instance）

实例是用来驱动数据库的。在集群环境中多个实例可以同时驱动一个数据库。

数据库处于NOMOUNT状态；还为和实例关联，该阶段主要用于对数据库进行维护（如重建控制文件）

参数文件还指定了控制文件的位置。

#### 装载数据库（Mount The Database)


根据参数文件  ----  控制文件  == 再由控制文件分别得到 数据文件 和 联机日志文件(Redo log file)的名称和位置。

实例与数据库进行了关联，但还是不能访问。

处于MOUNT阶段的数据库，主要进行数据库的维护（如：恢复数据库等）

#### 打开数据库 (Open The Database)

打开数据文件和联机日志文件。



### 6.1.2 数据库启动实务

#### 如何使数据库自动启动和关闭 Linux篇

windows篇见书。

目的：在操作系统启动和关闭时自动启动和关闭数据库。

使用dbstart 与 dbshut，它们会读取 /etc/oratab 文件。


1.所以先配置 oratab 文件： 

将要启用自动启动功能的实例，最后的参数N改为Y，如： 
ORA11G:/u01/app/oracle/product/11.2.0/db_1:Y

2.现在手动使用 dbstart 命令，看能否成功启动。


3.编写脚本文件。

脚本内容见书

#### 将数据库启动到NOMOUNT状态
在sqlplus中：  

	SQL> startup nomount

如果要从此状态进入mount状态使用： 

	SQL> alter database mount

#### 将数据库启动到MOUNT状态

	SQL> startup mount


如果要从此状态进入打开状态使用： 

	SQL> alter database open

​	
#### 打开数据库

	SQL> startup

​	
#### 如何判断数据库已经启动

如果出现下面的一条语句就表示数据库**没有**启动；
```
connecte to an idle instance
ORA-01034:  ...
ORA-27101: ...
```

#### 如何知道数据库处于何种状态
可以查询数据字典视图：根据 `v$database.open_mode` 指（open_mode列）和 `v$instance.status` 一起来判断

    select status from v$instance;  

## 6.2 数据库关闭

经历与数据库打开相反的三个状态。

#### 关闭数据库示例
sqlplus中

	SQL> shutdown immediate 

这种方式最安全，过程有点慢。


	SQL> shutdown transactional

这种方式不会使客户端的数据丢失... 较慢。

另还有最快最不安全的 abort 方式。






# 第8章 掌握两个管理问题--表空间和数据文件的管理

表空间： 逻辑结构。由数据文件组成。（表空间不足是常见问题）
数据文件：物理结构


## 8.1 数据存储结构
存储结构分为物理存储结构和逻辑结构。

### 8.1.1 物理结构
物理结构由数据文件(Datafiles)，联机日志文件(Online Redo logs),控制文件(control files)组成

除此之外还有一些其它文件，如参数文件。


#### 8.1.1.1 数据文件
一个数据库有多个数据文件。数据库中的表(tables)索引(Indexes)的数据，物理上放在数据文件中。

一个数据文件由多个操作系统块(OS Block)组成。（并非Oracle的数据块）

数据文件可以设置为自动扩展。

一个或多个数据文件形成一个表空间。


如果用户查询的表的数据不在内存中，则会读取该表所在的数据文件，并把数据放到内存。


#### 8.1.1.2 联机日志文件
包含重做记录（redo records）。当某次意外导致数据没有从内存写入到数据文件，则可以从联机日志文件中获得这些改变，并将其写回到数据文件中。



#### 8.1.1.3 控制文件
存放数据库的物理结构信息



### 8.1.2 逻辑结构

逻辑结构由： 数据块（Data Block)、区（Extent）、段（Segment）和表空间（Tablespace）组成。


```
                         数据库
                 
                 表空间            表空间
            
            段       段        段         段
         
        区    区     区    区  ...
        
    块    块    块      块    块   ...

```

#### 8.1.2.1 表空间

逻辑单元，表空间：用于存储数据库对象（比如：表，索引等）【而这些对象实际存放在数据文件中】


