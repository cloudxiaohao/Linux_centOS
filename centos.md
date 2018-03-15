# 环境搭建

@(操作系统-Linux)


## 1、进行操作系统的安装(操作系统目前为CentOS-7)
 >1、设置对应的操作系统语言
 
 >2、设置操作系统的时区和对应的时间（DATE&TIME）
 
 >3、修改SYSTEM 对应的INSTALLATION DESTINATION  
>>1、在 Other Storge Options 选项中选择 I will configuere partitioning 
>>2、选中指定的磁盘 ，点击Done。

>4、界面(MANUAL PART|TIONING) 
>>1、选项partitioning schema（选择Stendard Partition）		
>>2、选择Click here to create them automatically 自动创建
  
>5、 进行磁盘分配处理
>>1、只留 /boot 和 / 根磁盘。将其他的磁盘的内容进行删除， 系统盘预留500M
>>2、选择/ 根磁盘，清空右侧选项Desired Capacity 中的内容。切换到/boot 磁盘

>6、修改SYSTEM-KDUMP 
>>1、 去掉Enable-kdump 选项

>7、开始安装，设置系统的初始化密码，安装继续，完成后点击reboot 进行系统的重启操作

## 2、centos操作系统进行网络的配置操作
>1、查看当前网卡的状态信息
```
# ip addr
```

>2、如果网卡的信息不存在，进行网卡信息对应的配置操作
>> 修改网卡配置文件：/etc/sysconfig/network-scripts/ifcfg-ens33
```
    TYPE=Ethernet
    BOOTPROTO=static  # ip地址 改为静态配置
    NAME=ens33
    UUID=8d71a8dc-4cc3-4b4d-b2d7-9c5727c57ed1
    DEVICE=ens33
    ONBOOT=yes # 系统将在启动时开启该接口
    IPADDR=192.168.233.139 # 配置静态ip地址
    GATEWAY=192.168.233.2 # 局域网配置网管
    NETMASK=255.255.255.0 # 配置子网掩码
    NM_CONTROLLED=no # 表示该接口将通过该配置文件进行设置，而不是通过网络管理器进行管理
```
>> 修改操作系统网卡对应的DNS服务配置文件

```
	#  修改网络配置文件：/etc/sysconfig/network
    nameserver 8.8.8.8
    search localdomain
```

>3、重启网卡服务
>> 方法一
```
	# systemctl restart network.service
	# ip add
```

>> 方法二
```
	# ifdown em2 # 关闭指定端口网卡
	# ifup em2 # 开启指定端口网卡
	# ip add
```

##3、创建一个名称为python的用户，并赋予其管理员权限
>1、创建一个指定的用户python， 并未其添加密码
```
	[root@localhost /]# adduser python
	[root@localhost /]# passwd python
	Changing password for user python.
	New password: 
	Retype new password: 
```


> 2、进入指定的文件中，修改指定的行数
>> 方法一、修改 /etc/sudoers 文件，找到下面一行，把前面的注释（#）去掉
```
	## Allows people in group wheel to run all commands
	%wheel    ALL=(ALL)    ALL

	然后修改用户，使其属于root组（wheel），命令如下：
	#usermod -g root tommy
```

>> 方法二、修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行
```
	## Allow root to run any commands anywhere
	root    ALL=(ALL)     ALL
	tommy   ALL=(ALL)     ALL
```

##4、yum源的配置
>1、概念
>>yum 的宗旨是自动化地升级，安装/移除rpm 包，收集rpm 包的相关信息，检查依赖性并自动提示用户解决
>>yum 的理念是使用一个中心仓库(repository)管理一部分甚至一个distribution 的应用程序相互关系，根据计算出来的软件依赖关系进行相关的升级、安装、删除等等操作，减少了Linux 用户一直头痛的dependencies 的问题。这一点上，yum 和apt 相同。apt 原为debian 的deb 类型软件管理所使用，但是现在也能用到RedHat 门下的rpm 了
>>yum 可以同时配置多个资源库(Repository)，简洁的配置文件（/etc/yum.conf），自动解决增加或删除rpm 包时遇到的依赖性问题，保持与RPM 数据库的一致性。

>2、yum的安装、卸载、重新安装
>>1、查看系统默认安装的yum
```
	# rpm -qa|grep yum
```
>>2、卸载yum
```
	# rpm -e yum-fastestmirror-1.1.16-14.el5.centos.1 yum-metadata-parser-1.1.2-3.el5.centos yum-3.2.22-33.el5.centos
```
>>3、重新安装yum
>>这里可以通过wget 从网上下载相关包安装，也可以挂载系统安装光盘进行安装，这里选择挂载系统安装光盘进行安装。
```
	# mount /dev/cdrom /mnt/cdrom/
	# rpm -ivh yum-3.2.22-33.el5.centos.noarch.rpm yum-fastestmirror-1.1.16-14.el5.centos.1.noarch.rpm yum-metadata-parser-1.1.2-3.el5.centos.i386.rpm

	# yum -v
```

>3、yum的配置
>>yum 的配置文件分为两部分：main 和repository
>>- main 部分定义了全局配置选项，整个yum 配置文件应该只有一个main。常位于/etc/yum.conf 中。
>>- repository 部分定义了每个源/服务器的具体配置，可以有一到多个。常位于/etc/yum.repo.d 目录下的各文件中。

>>yum.conf 文件一般位于/etc目录下，一般其中只包含main部分的配置选项。
```
	# cat /etc/yum.conf
	
	[main]
	cachedir=/var/cache/yum
	　　//yum 缓存的目录，yum 在此存储下载的rpm 包和数据库，默认设置为/var/cache/yum
	keepcache=0
	　　//安装完成后是否保留软件包，0为不保留（默认为0），1为保留
	debuglevel=2
	　　//Debug 信息输出等级，范围为0-10，缺省为2
	logfile=/var/log/yum.log
	　　//yum 日志文件位置。用户可以到/var/log/yum.log 文件去查询过去所做的更新。
	pkgpolicy=newest
	　　//包的策略。一共有两个选项，newest 和last，这个作用是如果你设置了多个repository，而同一软件在不同的repository 中同时存在，yum 应该安装哪一个，如果是newest，则yum 会安装最新的那个版本。如果是last，则yum 会将服务器id 以字母表排序，并选择最后的那个服务器上的软件安装。一般都是选newest。
	distroverpkg=redhat-release
	　　//指定一个软件包，yum 会根据这个包判断你的发行版本，默认是redhat-release，也可以是安装的任何针对自己发行版的rpm 包。
	tolerant=1
	　　//有1和0两个选项，表示yum 是否容忍命令行发生与软件包有关的错误，比如你要安装1,2,3三个包，而其中3此前已经安装了，如果你设为1,则yum 不会出现错误信息。默认是0。
	exactarch=1
	　　//有1和0两个选项，设置为1，则yum 只会安装和系统架构匹配的软件包，例如，yum 不会将i686的软件包安装在适合i386的系统中。默认为1。
	retries=6
	　　//网络连接发生错误后的重试次数，如果设为0，则会无限重试。默认值为6.
	obsoletes=1
	　　//这是一个update 的参数，具体请参阅yum(8)，简单的说就是相当于upgrade，允许更新陈旧的RPM包。
	plugins=1
	　　//是否启用插件，默认1为允许，0表示不允许。我们一般会用yum-fastestmirror这个插件。
	bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum
	
	# Note: yum-RHN-plugin doesn't honor this.
	metadata_expire=1h
	
	installonly_limit = 5
	
	# PUT YOUR REPOS HERE OR IN separate files named file.repo
	# in /etc/yum.repos.d	
```
```
除了上述之外，还有一些可以添加的选项，如：
　　exclude=selinux*　　// 排除某些软件在升级名单之外，可以用通配符，列表中各个项目要用空格隔开，这个对于安装了诸如美化包，中文补丁的朋友特别有用。
　　gpgcheck=1　　// 有1和0两个选择，分别代表是否是否进行gpg(GNU Private Guard) 校验，以确定rpm 包的来源是有效和安全的。这个选项如果设置在[main]部分，则对每个repository 都有效。默认值为0。
```


## 5、linux操作系统，进行对应的虚拟化添加
>1、安装virtualenv
```
	# yum install python-virtualenv
```
>2、安装virtualenvwrapper（是virtualenv的扩展工具，可以方便的创建、删除、复制、切换不同的虚拟环境。）
>>1、安装virtualenvwrapper
```
	# easy_install virtualenvwrapper
```
>>2、创建一个文件夹，用于存放所有的虚拟环境：
```
	# mkdir .virtualenvs
```
>>3、设置环境变量，把下面两行添加到~/.bashrc里。
```
	export WORKON_HOME=/home/python/.virtualenvs
	source /usr/bin/virtualenvwrapper.sh
```

>>4、虚拟环境的相关操作
```
	1、mkvirtualenv [虚拟环境名称]：创建
    2、rmvirtualenv [虚拟环境名称]：删除
    3、workon：查看当前虚拟环境list
    4、workon [虚拟环境名称]：进入对应的虚拟环境
    5、deactivate：退出
    6、所有的虚拟环境都位于/home/.virtualenvs目录下
    7、pip list/pip freeze：查看当前虚拟环境中所安装的包
```

## 6、centos 安装pip
>1、首先安装epel扩展源：
```
	# yum -y install epel-release
```

>2、更新完成之后，就可安装pip:
```
	# yum -y install python-pip
```

>3、安装完成之后清除cache：
```
	# yum clean all
```

## 7、centos 安装wget
>1、安装指令
```
	# yum install wget
```
>2、wget部分使用技巧
>>下载 http://example.com 网站上 packages 目录中的所有文件。其中，-np 的作用是不遍历父目录，-nd 表示不在本机重新创建目录结构。
```
	# wget -r -np -nd http://example.com/packages/
```


## 8、centos 安装git
>1、安装git
```
	# yum install git
```
>2、在用户下创建一个文件夹用于存放git的全部仓库
```
    # cd home/python/github/
    # 新建一个目录，用于存放git的全部仓库
    # mkdir repositories
    # 设置该目录的所有权
    # chown python:python -R ./repositories
    # 修改该目录的操作权限
    # chmod 700 ./repositories
```
>3、使用git在github上克隆项目，还要设置好密钥对
```
	# yum install git
```
