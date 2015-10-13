


更新和升级 ：
------
yum update # 全部更新

比如：
yum update package 更新指定程序包package
比如：
yum update dhcp
 
 
yum check-update 检查可更新的程序
 
 
查找和显示 
----
yum info <package> 显示安装包信息
比如：
yum info dhcp
 
 
yum list 显示所有已经安装和可以安装的程序包
 
 
yum list <package> 显示指定程序包安装情况
 
 
yum search <keyword>查找软件包
 
删除程序 
----
yum remove or erase package 删除程序包
 
 
注意：
 
yum 会把下载的软件包和header存储在cache中，而不会自动删除。如果我们觉得它们占用了磁盘空间，可以使用yum clean指令进行清除. 
 
yum clean headers  清除header
 
yum clean packages  清除下载的rpm包
 
yum clean all  清除header与rpm包
 
.yum高级管理应用技巧
 
技巧1:加快你的yum的速度.使用yum的扩展插件yum-fastestmirror，个人认为这个插件非常有效，速度真的是明显提高
 
 
  技巧2:软件组安装有时我们安装完系统，管理有一类软件没有安装，比如用于开发的开发包,我们可以用软件包来安装。
 
     #yum grouplist 这样可以列出所有的软件包
 
     比如我们要安装开发有关的包
 
     #yum groupinstall "Development Libraries"
 
     #yum groupinstall "Development Tools"
 
     比如我们要安装中方支持
 
     #yum groupinstall "Chinese Support"
 
     #yum deplist package1  #查看程序package1依赖情况
 
 
以上所有命令参数的使用都可以用man来查看
 
 
 
 
升级内核：#yum install kernel-headers kernel-devel



