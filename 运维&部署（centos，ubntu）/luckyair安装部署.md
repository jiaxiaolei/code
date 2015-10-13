

centos 安装实录
==============

基础配置
----
CentOS Linux release 7.1.1503 (Core)
# cat /etc/redhat-release


操作系统安装
--------
1. 虚拟机中安装时注意选择可用的网卡. 一般修改网络连接方式为桥接（bridge）后，会自动分配网卡。
2. 安装的时候最好默认安装一些基础工具包。没找到
- 账号密码：
  root:Hkhl2015  
  cic:Hkhl2015

3. 网络不可用的时候：
$  ifup
$ 
# 修改网络配置：
$ cd /etc/sysconfig/network-scripts

$ vim ifcfg-enp0s3

最后一行： 
ONBOOT=no # 改为yes.


## yum 更新和升级

$ sudo yum –y update yum
$ sudo yum upgrade 
# 很多依赖包都会被升级

# 首先为yum添加epel源：
$ sudo yum install –y epel-release
$ sudo yum clean all
$ sudo yum update


## 安装基础工具

# pip 
$ sudo easy_install pip

# unzip 
$ sudo yum install unzip




# 安装 vim
$ sudo yum install -y vim

# 安装mysql
$ yum -y install mariadb*  
$ systemctl start mariadb.service 
$ systemctl enable mariadb.service  


# 关闭防火墙

$ service firewalld stop

# 启动一个http server, 测试是否可用：
$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...


## 安装 mysql

$ yum -y install mariadb*  
$ systemctl start mariadb.service 
$ systemctl enable mariadb.service  


## 安装redis

$ yum -y install redis 
$ systemctl start redis.service 
$ systemctl enable redis.service  



# 安装 git（选择）
$ sudo yum install -y git


## 基础配置：
$ git config --global user.email "281304051@qq.com"
$ git config --global user.name "Jia Xiaolei"

## 配置 commit 
# 得到 commit_template.commit_template 
$ git config commit.template .commit_template.commit_template

## 配置 push-for-review:
$ git config alias.push-for-review "push origin HEAD:refs/for/dev"


## 建立相关的文件：

# 代码管理
mkdir -p /home/cic/project


# 临时目录管理
mkdir -p /home/cic/software


# nginx log
mkdir -p /var/log/nginx/{update}

# supervisor log
mkdir -p /var/log/supervisor/{celeryd,update}


## 从代码仓库同步代码：

1. 生成 ssh key
$ ssh-keygen
## 一直回车。 

添加到ssh
 vim ~/.ssh/id_rsa.pub

2. 添加到 gerrit 上。

git clone ssh://jiaxiaolei@review.cicaero.com:29418/luckyair_update_server


cd luckyair_update_server
git remote set-url origin ssh://jiaxiaolei@review.cicaero.com:29418/luckyair_update_server

$ git remote -v
```
origin	ssh://jiaxiaolei@review.cicaero.com:29418/luckyair_update_server (fetch)
origin	ssh://jiaxiaolei@review.cicaero.com:29418/luckyair_update_server (push)
```

$ git remote update

$ scp -p -P 29418 jiaxiaolei@review.cicaero.com:hooks/commit-msg .git/hooks/

git checkout -b dev origin/dev


nginx 
=====

$ sudo yum install -y nginx

# 启动 nginx 
$ sudo nginx 

supervisor 
======

$ sudo yum install -y supervisor
# 启动 supervisor
$ sudo supervisord


virtualenv/virtualenv-wrapper
=============================

virtualenv
----------
$ sudo easy_install virtualenv	
# 创建env的虚拟环境：
$ virtualenv env


virtulaenv-wrapper
-----------------

$ sudo pip install virtualenvwrapper



#创建目录用来存放虚拟环境
$ mkdir $HOME/.virtualenvs
#在~/.bashrc中添加行
$ echo 'export WORKON_HOME=$HOME/.virtualenvs' >> ~/.bashrc
$ echo 'source /bin/virtualenvwrapper.sh' >> ~/.bashrc
#NOTE: 
这里注意一下 virtualenvwrapper.sh 的路径：可以which virtualenvwrapper.sh 查找一下在当前服务器上的路径：
/bin/virtualenvwrapper.sh

# 执行 .bashrc 中的指令
$ source ~/.bashrc	
或者
$ . ~/.bashrc
# 此时 virtualenvwrapper 就可以使用了
$ workon
  
# 建立虚拟环境： env
$ mkvirtualenv env

安装 python 包(在virtualenv 中进行安装)
=============

# 安装 gcc, 否则 mysql-python 会安装不成功
$ yum install gcc libffi-devel python-devel openssl-devel	

$ cd /home/cic/project/luckyair_update_server/deploy
$ pip install -r requirements/ground.txt


初始化数据库：
======
$ scp db_luckyair_2015-10-11.sql cic@192.168.1.110:/home/cic/software

$ mysql -uroot -e "use mysql;update user set password=password('Hkhl2015') where user='root'"


# 通过root用户建立cic:Hkhl2015操作账号, 并配置权限： 
# Step 1:  所有host 的使用
$ mysql -uroot  -pHkhl2015 --default-character-set=utf8 -e 'GRANT ALL PRIVILEGES ON *.* TO cic@"%" IDENTIFIED BY "Hkhl2015"'
# Step 2: localhost 的使用
$ mysql -uroot  -pHkhl2015 --default-character-set=utf8 -e 'GRANT ALL PRIVILEGES ON *.* TO cic@"localhost" IDENTIFIED BY "Hkhl2015"'

# Step 3: 更新一下权限
$ mysql -uroot -pHkhl2015 --default-character-set=utf8 -e 'flush privileges';

# 建立数据库：
$ mysql -ucic -pHkhl2015 --default-character=utf8;

mysql> CREATE DATABASE `db_luckyair` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> use db_luckyair;
mysql> source db_luckyair.sql;


初始化静态文件：
=========


$ cd /home/cic/project/luckyair_update_server
$ rm conf; ln -s conf.d/dev conf


整理nginx 配置文件：
=================

$ sudo mkdir -p /etc/nginx/sites-enabled

cd /etc/nginx/sites-enabled
 sudo ln -s /home/cic/project/luckyair_update_server/conf/nginx.out/sites-enabled/* .


 cd /etc/nginx
$ sudo mv nginx.conf nginx.conf.bak

$ sudo ln -s /home/cic/project/luckyair_update_server/conf/nginx.out/nginx.conf .


$ sudo mkdir -p /var/log/nginx/update/

# 重启服务。
$ sudo service nginx restart


整理supervisor 配置文件：
======================

sudo mv /etc/supervisord.conf /etc/supervisord.conf.bak

sudo ln -s /home/cic/project/luckyair_update_server/conf/supervisord/supervisord.conf /etc/



$ sudo mkdir -p /etc/services-enabled


$ cd /etc/services-enabled
$ sudo ln -s  /home/cic/project/luckyair_update_server/conf/supervisord/services-enabled/* .

$ mkdir -p /var/log/supervisor/{celeryd,update}


sudo mkdir -p /var/log/nginx

sudo mkdir -p /var/log/supervisor


=====================
=========备忘录====
##
1. 只拷贝 部分 news, joke
2. static 移除中间目录 比如 news..
3. 增加缓存
4. 删除历史数据：...

$ ll|grep -v 'zip'|xargs sudo rm -rf


[CentOS 7] 安装nginx
http://jingyan.baidu.com/article/aa6a2c14dc36640d4d19c47e.html

下载对应当前系统版本的nginx包(package)
# wget  http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
2
建立nginx的yum仓库
# rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm
3
下载并安装nginx
# yum install nginx
4
启动nginx服务
systemctl start nginx
5



CentOS7.0安装Nginx 1.7.4
http://www.centoscn.com/image-text/install/2014/0812/3480.html


CentOS 7下的软件安装方法及策略
http://seisman.info/how-to-install-softwares-under-centos-7.html
简介：

官方源
CentOS自带的四个官方源中，默认打开的有base、updates、extras，这三个源中包含了约9000个软件包，是最稳定、也是最值得信赖的源。

大型第三方源，已确认不会替换官方源的包，且相互之间无冲突
EPEL：包含6500多个软件，科研必备
ELRepo：包含几十个各种硬件的驱动程序
Nux Dextop：多媒体相关的软件包（与EPEL的个别软件相冲突，可忽略）
有些小型第三方源，仅包含了几个软件，确认与官方源和EPEL源不会冲突，也可以添加
Google Chrome：包含了Google Chrome，不会与官方源和EPEL源冲突；
Adobe：仅包含flash插件，已确认不会冲突；
dropbox：仅包含dropbox一个软件，已确认不会冲突；


解压即用
有些软件，官方提供了压缩包，解压之后即可直接运行其中的二进制文件，比如很多Java写的软件。这类软件没有给源代码，而是给了可以在当前平台下直接执行的二进制文件。大多数非开源的商业软件都采取这种办法。

比如sublime_text、pycharm、mendeley、TauP、sac等，直接解压，然后将解压后的文件夹复制到/opt目录下，然后将该软件的bin目录加入到PATH中即可。
比如Mathematics、Matlab、intel studio，软件包中提供安装脚本，执行该脚本即可安装；
Linux下的习惯是，商业软件或第三方软件都安装到/opt目录下，这也是大多数商业软件包的默认安装路径，尽量遵循该习惯。


Linuxbrew
Linuxbrew是由OS X平台下非常流行的Homebrew移植到Linux下的。Linuxbrew可以作为系统自带的包管理器的一个补充。其特色在于：

所有软件都安装在${HOME}/.linuxbrew目录下；
软件的版本相对很新；
install、uninstall、info、list、update、upgrade等功能
若库中没有需要的软件包，可以很简单地自己创建formulae



Centos 7 系统安装完毕修改网卡名为eth0
http://jingyan.baidu.com/article/7f41ecec1b022e593d095c1e.html
简介：
从CentOS/RHEL7起，可预见的命名规则变成了默认...



centos ifconfig
===============

centos7没有安装ifconfig命令的解决方法
http://www.centoscn.com/CentosBug/osbug/2014/0916/3750.html
简介：

yum search ifconfig 查找 ifconfig 包含在哪个安装包中。 



CentOS 7最小化安装后找不到‘ifconfig’命令——修复小提示
-------------------------------------------------
http://linux.cn/article-3631-1.html
简介：

ifconfig 已经过时了...

$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.110  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::a00:27ff:fe68:c0aa  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:68:c0:aa  txqueuelen 1000  (Ethernet)
        RX packets 89802522  bytes 75598528411 (70.4 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 87404457  bytes 114307646121 (106.4 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 523532  bytes 324577030 (309.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 523532  bytes 324577030 (309.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0



使用“ip addr”和“ip link”命令来查找网卡详情。要知道统计数据，可以使用“ip -s link”。


要查看网络接口统计数据，输入命令：

ip link

或者：

ip -s link



找出哪个包提供了ifconfig命令。

yum provides ifconfig


yum whatprovides ifconfig






