﻿# configure、make、make install、/usr、/usr/bin、/opt
标签: Linux

---

> 本文纯属抄袭，请不要发布本文。

## 1.configure、make、make install
### 参考1：[例解 autoconf 和 automake 生成 Makefile 文件](http://www.ibm.com/developerworks/cn/linux/l-makefile/index.html#icomments) 
无论是在Linux还是在Unix环境中，make都是一个非常重要的编译命令。不管是自己进行项目开发还是安装应用软件，我们都经常要用到make或 make install。利用make工具， 我们可以将大型的开发项目分解成为多个更易于管理的模块，对于一个包括几百个源文件的应用程序，使用make和makefile工具就可以轻而易举的理顺各个源文件之间纷繁复杂的相互关系。

但是如果通过查阅make的帮助文档来手工编写Makefile,对任何程序员都是一场挑战。幸而有GNU 提供的Autoconf及Automake这两套工具使得编写makefile不再是一个难题。

本文将介绍如何利用 GNU Autoconf 及 Automake 这两套工具来协助我们自动产生 Makefile 文件， 并且让开发出来的软件可以像大多数源码包那样，只需"./configure", "make","make install"  就可以把程序安装到系统中。

#### 生成 Makefile 的来龙去脉 
首先进入 project 目录，在该目录下运行一系列命令，创建和修改几个文件，就可以生成符合该平台的Makefile文件，操作过程如下：

1. 运行autoscan命令
2. 将configure.scan 文件重命名为configure.in，并修改configure.in文件
2. 在project目录下新建Makefile.am文件，并在core和shell目录下也新建makefile.am文件
2. 在project目录下新建NEWS、 README、 ChangeLog 、AUTHORS文件
2. 将/usr/share/automake-1.X/目录下的depcomp和complie文件拷贝到本目录下
2. 运行aclocal命令
2. 运行autoconf命令
2. 运行automake -a命令
2. 运行./confiugre脚本

可以通过图1 看出产生Makefile的流程，如图所示：
![生成Makefile流程图][1]
<center>图1. 生成Makefile流程图</center>

#### 安装路径
automake设置了默认的安装路径：

1)	标准安装路径
默认安装路径为：`$(prefix) = /usr/local`，可以通过`./configure --prefix=<new_path>`的方法来覆盖。
其它的预定义目录还包括：`bindir = $(prefix)/bin`, `libdir = $(prefix)/lib`, `datadir = $(prefix)/share`, `sysconfdir = $(prefix)/etc`等等。

`

对于安装的情况，库将会安装到`$(prefix)/lib`目录下，可执行文件将会安装到`${prefix}/bin`。


## 2. Linux 的源码安装工具 CheckInstall
参考：[Linux 的源码安装工具 CheckInstall](http://www.ibm.com/developerworks/cn/linux/l-cn-checkinstall/)

### 2.1 用 GNU Autoconf 安装程序
一般说来，我们编译安装一个由 GNU Autoconf 配置的程序是采用如下的步骤：

    ./configure && make && make install
        
这个 configure 脚本文件是用来“猜”出一系列系统相关的变量，这些变量是在后面的编译过程要用到的。它将检查系统变量值是否满足编译要求，然后使用这些变量在程序包内每个文件夹下生成 Makefile。此外，configure 脚本还会生成其它文件：

- 每个文件夹／子文件夹下的一个或多个 Makefile(s)
- 一个名叫 config.status 的脚本
- 一个文本文件 config.log

Configure 脚本文件成功运行之后, 你会输入 make 来编译程序，得到你需要的可执行文件。如果 make 成功的完成，你可以使用 make install 来安装这个程序。

**然而，当你想删除它的时候呢？**
Makefile 文件只包括了很少情况下的卸载例程。当然，你可以把程序安装到临时文件夹，然后记下所有由程序生成或修改的文件，最后删除他们。但是如果这个程序要经常重新编译，这样做是非常痛苦的，工作量也相当大。Felipe Eduardo 所写的 **CheckInstall** 就是用来解决这个问题的。

### 2.2 用Checkinstall安装程序

它采用自己的指令 checkinstall 来代替 make install。其他两个指令保留下来跟以前一样，因此，现在这个指令序列使用 CheckInstall 变成：

    ./configure && make && checkinstall

指令 checkinstall 不仅默认运行了 make install，而且还监测所有安装过程中的写操作。为此，CheckInstall 使用了 Pancrazio de Mauro 所写的程序 Installwatch。在 make install 成功完成之后，CheckInstall 会产生一个 Slackware-，Debian- 或RPM- 安装包，然后按照软件包的默认配置来安装程序，并在当前目录（或标准安装包存储目录）留下一个生成的安装包。你可以通过修改变量 PAK_DIR 来修改这个保存目录。

**运行命令“checkinstall –h”显示所有可用的子参数**
这些子参数大致分为：
- 安装选项（Install options）
- 脚本处理选项（Scripting options）
- 信息显示选项（Info display options）
- 安装包选项（Package tuning options）
- 清除选项（Cleanup options）
- 关于 CheckInstall （About CheckInstall）。

如果 CheckInstall 带着这些参数运行，它会使用这些参数值来代替配置文件 checkinstallrc 中相应的值。


CheckInstall 也有自己的局限之处。它不能处理静态连接的程序，因为这样 Installwatch 就不能监测到安装过程中修改过文件了。总体说来，有两类连接库：动态的和静态的。这些连接库通过 include 指令整合到程序中。静态连接过的程序已经包含了所有需要的库文件，运行时也就不需要再将这些库载入内存中。这种程序与安装在系统中的连接库无关，因为所谓的连接器（Linker）已经在编译时把这些库内置到可执行程序里了。

### 2.3 Checkinstall的配置
CheckInstall 的配置
你可以通过修改配置文件 /usr/local/lib/checkinstall/checkinstallrc 来改变 CheckInstall 的默认配置。
文件值得注意的变量有 INSTYPE，INSTALL 和 PAK_DIR。

- INSTYPE 变量决定生成何种类型安装包
- PAK_DIR 变量决定安装包的存储目录。
- INSTALL 变量决定是只生成安装包还是一起将这个包马上安装。
    0－只生成安装包
    1－不仅生成安装包，还将包立即安装


>**运行checkinstall制作安装包，生成 rpm（或dep) 包期间会出现一些选项，选择默认的即可。**


## 3./usr、/usr/bin、/opt
在一次编译安装git查找资料的过程，不同教程发现指定了不同的安装目录 ... 
1. [Git的INSTALL文档](https://github.com/git/git/blob/master/INSTALL "GitHub")
2. [Ubuntu 编译安装最新版 git][2]
编译安装时使用：  
```
make prefix=/usr/local all

sudo make prefix=/usr/local install
```


**安装目录的参考：** 

### 3.1 [关于 /usr 和 /usr/local 的讨论](http://blog.csdn.net/chaehom/article/details/7778256)

在传统的unix系统中，/usr通常只包含系统发行时自带的程序，而/usr/local则是本地系统管理员用来自由添加程序的目录。
对于Linux发行版，如 RedHat， Debian 等等，一个可能的规定是：/usr目录只能由发行版的软件包管理工具负责管理，而对/usr/local却没有这样做。正是因为采用这种方式，软件包管理工具的数据库才能知道在/usr目录内的每一个文件。


### 3.2 [Linux中/usr与/var目录详解](http://www.cnblogs.com/Jamesliang/articles/1486690.html) 

/usr 文件系统经常很大，因为所有程序安装在这里. /usr 里的所有文件一般来自Linux distribution；本地安装的程序和其他东西在/usr/local 下.这样可能在升级新版系统或新distribution时无须重新安装全部程序. 


### 3.3 [/bin,/sbin,/usr/sbin,/usr/bin 目录](http://blog.csdn.net/kkdelta/article/details/7708250)

**首先区别下/sbin和/bin：**
*从命令功能来看*， /sbin 下的命令属于基本的系统命令，如shutdown，reboot，用于启动系统，修复系统，/bin下存放一些普通的基本命令，如ls,chmod等，这些命令在Linux系统里的配置文件脚本里经常用到。

*从用户权限的角度看*， /sbin目录下的命令通常只有管理员才可以运行，/bin下的命令管理员和一般的用户都可以使用。

*从可运行时间角度看*， /sbin,/bin能够在挂载其他文件系统前就可以使用。

 /bin,/sbin目录是在系统启动后挂载到根文件系统中的，所以/sbin,/bin目录必须和根文件系统在同一分区；
/usr/bin,usr/sbin可以和根文件系统不在一个分区。
/usr/sbin存放的一些非必须的系统命令；/usr/bin存放一些用户命令，如led(控制LED灯的)。


### 3.4 [linux下面/usr/local和opt目录有何区别](http://agen2008.blog.163.com/blog/static/337032612012499525725/)
/usr/local下一般是你安装软件的目录，这个目录就相当于在windows下的programefiles这个目录  /opt这个目录是一些大型软件的安装目录，或者是一些服务程序的安装目录 

/usr/local
    This is where most manually installed(ie. outside of your package manager) software goes.It has the same structure as /usr. It is a good idea to leave /usr to your package manager and put any custom scripts and things into /usr/local, since nothing important normally lives in /usr/local.

/usr/local
    这里主要存放那些手动安装的软件，即不是通过“新立得”或apt-get安装的软件。它和/usr目录具有相类似的目录结构。让软件包管理器来管理/usr目录，而把自定义的脚本(scripts)放到/usr/local目录下面，我想这应该是个不错的主意。


---

**/bin** 是系统的一些指令。bin为binary的简写主要放置一些系统的必备执行档例如:cat、cp、chmod df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等。

**/sbin** 一般是指超级用户指令。主要放置一些系统管理的必备程式例如:cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、 ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown等。

**/usr/bin**　是你在后期安装的一些软件的运行脚本。主要放置一些应用软体工具的必备执行档例如c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome*、 gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb*、wget等。

**/usr/sbin**  放置一些用户安装的系统管理的必备程式例如:dhcpd、httpd、imap、in.*d、inetd、lpd、named、netconfig、nmbd、samba、sendmail、squid、swap、tcpd、tcpdump等。



  [1]: http://www.ibm.com/developerworks/cn/linux/l-makefile/images/image002.gif
  [2]: https://www.ldsink.com/archives/ubuntu-compile-and-install-git.html "他人的安装方法"