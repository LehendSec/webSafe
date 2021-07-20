---
title: 如何将你的 apache 把控的'密不透风'
date: 2017-11-27 04:09:17
tags: apachesec
categories: "apachesec"
author: klion
---


0x01 为防止配置或端口冲突,在装之前,你需要先仔细检查当前系统有没有装apache,如果有`先把apache服务停掉,然后卸载apache`,等会儿用源码重新编译安装
```
# rpm -qa httpd
# rpm -e --nodeps * 强制卸载apache
```

演示环境
```
CentOS6.8 x86_64    最小化,带基础库安装 eth0 : 192.168.3.45 eth1 : 192.168.4.16 eth2 : 192.168.5.16
httpd-2.2.34.tar.gz apache官方提供的源码包
```

0x02 下载apache源码包,这里暂时选择2.2.x系列的最新版,不建议再用比这个还老的版本了,漏洞比较多
```
# wget http://apache.website-solution.net/httpd/httpd-2.2.34.tar.gz
# tar xf httpd-2.2.34.tar.gz && cd httpd-2.2.34
```

0x03 直接到源码中去`改掉apache的详细版本信息`,跟部署nginx一样,尽可能地扰乱入侵者的判断,这里就把它模拟成IIS 7.5,实际系统应为win server 2008r2
```
# vi include/ap_release.h	
```
<!-- more -->
```
#define AP_SERVER_BASEVENDOR "IIS Software Foundation"
#define AP_SERVER_BASEPROJECT "IIS HTTP Server"
#define AP_SERVER_BASEPRODUCT "Microsoft-IIS/7.5"

#define AP_SERVER_MAJORVERSION_NUMBER 7
#define AP_SERVER_MINORVERSION_NUMBER 5
#define AP_SERVER_PATCHLEVEL_NUMBER   0
#define AP_SERVER_DEVBUILD_BOOLEAN    0
```

别忘了,把系统平台也一并改了,很显然,`unix平台是不可能装IIS的`,不然你就把自己卖了
```
# vi os/unix/os.h
  #define PLATFORM "Win32"
```

0x04 开始编译,安装并启动apache,此处暂时选择激活大多数模块,让apache以worker模式进行工作,默认会用prefork模式,效率不高,所以我们把它改成worker
```
# yum install zlib zlib-devel gcc-c++ -y
# ./configure --prefix=/usr/local/httpd-2.2.34 \
--enable-deflate \
--enable-expires \
--enable-headers \
--enable-modules=most \
--enable-so \
--with-mpm=worker \
--enable-rewrite

# make && make install
# ln -s /usr/local/httpd-2.2.34/ /usr/local/httpd
```

关于 `apachectl 工具` 自身的一些选项用途
```
# /usr/local/httpd/bin/apachectl -t	   检查配置文件语法,重启服务前会用
# /usr/local/httpd/bin/apachectl -V	   查看编译参数,如果这个apache当初不是你编译的可能会用到
# /usr/local/httpd/bin/apachectl -v	   查看apache详细版本号
# /usr/local/httpd/bin/apachectl -S	   查看apache所有的虚拟主机配置
# /usr/local/httpd/bin/apachectl -l	   查看所有已编译的模块
# /usr/local/httpd/bin/apachectl -M	   查看所有已加载的模块
# /usr/local/httpd/bin/apachectl start	   启动apache
# lsof -i :80				   检查默认端口有没有起来
# /usr/local/httpd/bin/apachectl graceful  平滑重启apache
# /usr/local/httpd/bin/apachectl stop	   关闭apache
```

0x05 理解apache安装目录下各个一级目录的作用
```
# tree -L 1 /usr/local/httpd
├── bin	    # apache自身管理工具存放目录,如,ab,apachectl[本质还是调用httpd],httpd,apxs,htpasswd...
├── build
├── cgi-bin
├── conf    # 各类配置文件存放目录,如,httpd.conf,extra [apache 的扩展配置文件存放目录]
├── error
├── htdocs  # 默认的站点根目录,实际部署中,一般不用
├── icons
├── include
├── lib
├── logs    # 各类日志文件存放目录,如,访问日志,错误日志
├── man	    # 自带的帮助手册
├── manual
└── modules # 模块存放目录
```


0x06  编辑apache主配置文件`httpd.conf`
```
# mkdir /var/html/{bwapp,dvws,drupal7} -p   为后面配置虚拟主机先创建好网站目录
# echo "welcome to my website! by bwapp" > /var/html/bwapp/index.html
# echo "Hello dvws,come on" > /var/html/dvws/index.html
# useradd -s /sbin/nologin -M httpd	    以系统伪用户身份启动apache服务,防止入侵者利用apache提权
# cd /usr/local/httpd/conf/ && mv httpd.conf httpd.conf.bak
# egrep -v "^$|#" /usr/local/httpd/conf/httpd.conf.bak > httpd.conf   这里先简化下apache主配置文件,方便等会儿配置
```

除了下面提到的这些配置项,其它的配置都可以不用要,如,`cgi,默认网站根目录`,因为等会儿这些全部会自定义,所以,在这里可以全部先删掉
```
# vi /usr/local/httpd/conf/httpd.conf

ServerRoot "/usr/local/httpd-2.2.34" # 定义apache的安装目录
Listen 80       # apache默认监听的web端口,没有指定ip的情况下,默认是监听在0.0.0.0
Listen 81       # 在后面配置基于端口的虚拟主机时需要先在此定义好监听端口
Listen 82
<IfModule !mpm_netware_module>
<IfModule !mpm_winnt_module>
User httpd   	# apache的服务用户,再次强调,不要傻到用root身份来运行apache服务
Group httpd  	# apache的服务用户组
</IfModule>
</IfModule>
ServerAdmin klion@sec.org    # 管理员邮箱地址
ServerName 127.0.0.1:80	     # 解决FQDN的问题
<Directory />		     # 对系统根目录的权限设置
    Options FollowSymLinks
    AllowOverride None
    Order deny,allow	     # 禁止用户直接访问系统根目录
    Deny from all			
</Directory>
<IfModule dir_module>
    DirectoryIndex index.html index.php  # 设置主页索引文件
</IfModule>
<FilesMatch "^\.ht">	# 当匹配到以.ht开头的文件apache要做什么样的反应
    Order allow,deny
    Deny from all
    Satisfy All
</FilesMatch>
ErrorLog "logs/error_log" 	# apache自身的错误日志存放位置 
LogLevel warn			# 定义apache日志的报告级别,默认是警告
<IfModule log_config_module>	# 定义访问日志格式
    # 在下面combined格式日志中还要多记录一个字段`X-Forwarded-For`,用于记录客户端真实ip
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" \"%{X-Forwarded-For}i\"" combined 
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" common
</IfModule>
DefaultType text/plain
<IfModule headers_module>
    RequestHeader unset Proxy early
</IfModule>
<IfModule mime_module>	# mime类型设定,也就是说,遇到什么类型的文件就做什么样的操作
    TypesConfig conf/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz	
    # 意思就是说,当apache遇到.php这样的后缀就丢给libphp5模块去处理,也就是说,如果这里是.jpg,它依然会被当做php去解析,如,图片马...
    AddType application/x-httpd-php .php 
</IfModule>
<IfModule ssl_module>
SSLRandomSeed startup builtin
SSLRandomSeed connect builtin
</IfModule>
Include conf/extra/httpd-mpm.conf  # 包含apache扩展配置文件
Include conf/extra/httpd-vhosts.conf	  
Include conf/extra/httpd-default.conf
<Directory "/var/html/bwapp">
    # 我们在Indexes前面加上了'-',意思就是禁止目录遍历,防止敏感文件泄露,此项非常重要,另外,关闭CGI,SSI,以及Follow Symbolic Links
    Options -Indexes -Includes -ExecCGI –FollowSymLinks 
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>
<Directory "/var/html/dvws">
    Options -Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>
<Directory "/var/html/drupal7">
    Options -Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>
#LoadModule php5_module  /usr/local/httpd-2.2.34/modules/libphp5.so
LoadModule php5_module   modules/libphp5.so	# 该模块在你编译安装完php以后会自动生成并自动加入该配置文件
#LoadModule rewrite_module modules/mod_rewrite.so
TraceEnable off 	# 禁用trace方法,防止xss
```

`让apache支持mod_rewrite模块`,因为经常要用到该模块,添加重写功能以后只需要在对应的虚拟主机中添加`AllowOverride All`项,即可读取`.htaccess`中的正则,此项为`Nnoe`时则表示不读取,除此之外,对于模块,用什么就加什么即可,因为有些漏洞只发生在特定的模块上,所以为了防止别人利用,不用的就不要加了
```
# cd httpd-2.2.34/modules/mappers/
# /usr/local/httpd/bin/apxs -c mod_rewrite.c
# /usr/local/httpd/bin/apxs -i -a -n mod_rewrite mod_rewrite.la
  #LoadModule mod_rewrite_module modules/mod_rewrite.so # 中间可能会报两个错,改名,注释掉即可
# /usr/local/httpd/bin/apachectl configtest
# /usr/local/httpd/bin/apachectl -t
# /usr/local/httpd/bin/apachectl graceful
```

0x07 编辑扩展配置文件,存在`/usr/local/httpd/conf/extra/`目录下

配置`基于域名的虚拟主机` `httpd-vhosts.conf`
```
# vi /usr/local/httpd/conf/extra/httpd-vhosts.conf

NameVirtualHost *:80
<VirtualHost *:80>
    ServerAdmin bwapp@sec.org		# 网站管理员邮箱
    DocumentRoot "/var/html/bwapp"	# 站点根目录
    ServerName bwapp.cc		 	# 主域名
    ServerAlias www.bwapp.cc 		# 别名,不用可以删掉
    ErrorLog "logs/bwapp-error_log"	# 错误日志
    CustomLog "logs/bwapp-access_log" combined # 选择日志格式,combined会记录的更详细,方便日后审查
</VirtualHost>
```

配置`基于端口的虚拟主机`,需要先在`httpd.conf`中定义好要监听的端口,然后再在`httpd-vhosts.conf`中定义即可
```
# vi /usr/local/httpd/conf/httpd.conf
  Listen 81
```
```
# vi /usr/local/httpd/conf/extra/httpd-vhosts.conf

NameVirtualHost *:81
<VirtualHost *:81>
    ServerAdmin dvws@sec.org
    DocumentRoot "/var/html/dvws"
    ServerName dvws.cc
    ServerAlias www.dvws.cc
    ErrorLog "logs/dvws-error_log"
    CustomLog "logs/dvws-access_log" combined
</VirtualHost>
```

`基于ip的虚拟主机`,需要当前机器有多个可用ip地址
```
# vi /usr/local/httpd/conf/httpd.conf
  Listen 82
```
```
# vi /usr/local/httpd/conf/extra/httpd-vhosts.conf

NameVirtualHost 192.168.5.16:82
<VirtualHost 192.168.5.16:82>
    ServerAdmin drupal7@sec.org
    DocumentRoot "/var/html/drupal7"
    ServerName 192.168.5.16
    ErrorLog "logs/drupal7-error_log"
    CustomLog "logs/drupal7-access_log" combined
</VirtualHost>
```

```
# egrep -v "#|^$" httpd-vhosts.conf > httpd-vhosts.conf.bak	备份所有虚拟主机配置
```

在 `httpd-mpm.conf`文件中可以调节worker,prefork模式下的详细参数,如,并发之类...因为之前在编译时已经指定使用worker模式,所以这里你只需配置worker里面的参数即可,并非此处重点就不细说了
```
# vi /usr/local/httpd/conf/extra/httpd-mpm.conf
```

`httpd-default.conf` 文件主要用来设置一些和http响应头有关的内容,如,.htaccess文件,隐藏apache版本什么的,因为之前在源码中已经彻底修改过apache默认版本信息,所以这里就不用再改了
```
# vi /usr/local/httpd/conf/extra/httpd-default.conf
  ServerTokens Prod
  ServerSignature Off
```

0x08 解决FQDN`完整域名`的问题,不然每次重启都要等半天,添加如下内容即可
```
# vi /usr/local/httpd/conf/httpd.conf
  ServerName 127.0.0.1:80
```

0x09 利用`cronlog工具`来帮我们实现自动轮询apache访问日志,也可用apache自带的rotatelogs工具配合系统定时任务实现,根据个人喜好而定

编译安装cronlog
```
# tar xf cronolog-1.6.2.tar.gz
# cd cronolog-1.6.2
# ./configure && make && make install
```

接下来只需要到指定的虚拟主机中修改日志格式即可,之后重启apache即可生效
```
# vi /usr/local/httpd/conf/extra/httpd-vhosts.conf
<VirtualHost *:80>
    ServerAdmin bwapp@sec.org
    DocumentRoot "/var/html/bwapp"
    ServerName bwapp.cc
    ServerAlias www.bwapp.cc
    ErrorLog "logs/bwapp-error_log"
    CustomLog "|/usr/local/sbin/cronolog /usr/local/httpd/logs/bwapp-access_%Y%m%d.log" combined
</VirtualHost>

# /usr/local/httpd/bin/apachectl -t
# /usr/local/httpd/bin/apachectl graceful
```

0x10 有时我们需要除一些无用的日志信息,便于我们后续能尽快从日志中分析各种攻击行为,如,一些静态数据,css,js,各类图片文件,你就可以用下面的方式,让apache不要记录这类数据的访问日志

先到`httpd.conf`中去定义好 `img` 变量
```
# vi /usr/local/httpd/conf/httpd.conf

<Directory "/var/html/bwapp">
    SetEnvIf Request_URI ".*\.gif$" img
    SetEnvIf Request_URI ".*\.jpg$" img
    SetEnvIf Request_URI ".*\.png$" img
    SetEnvIf Request_URI ".*\.bmp$" img
    SetEnvIf Request_URI ".*\.swf$" img
    SetEnvIf Request_URI ".*\.js$" img
    SetEnvIf Request_URI ".*\.css$" img
    Options -Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>
```

然后再到`httpd-vhosts.conf`文件中,去找到你想引用到的虚拟主机上去引用即可,如下
```
# vi /usr/local/httpd/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    ServerAdmin bwapp@sec.org
    DocumentRoot "/var/html/bwapp/bWAPP/"
    ServerName bwapp.cc
    ServerAlias www.bwapp.cc
    ErrorLog "logs/bwapp-error_log"
    CustomLog "|/usr/local/sbin/cronolog /usr/local/httpd/logs/bwapp-access_%Y%m%d.log" combined env=!img
</VirtualHost>
```

0x11 尽可能控死网站目录权限,只让用户在上传目录中能写,其它的目录一律不让写,具体授权过程如下`/var/html/bwapp/bWAPP/`为网站根目录
```
# chown -R root.root /var/html/bwapp/bWAPP/
# find /var/html/bwapp/bWAPP -type f | xargs chmod 644
# find /var/html/bwapp/bWAPP -type d | xargs chmod 755
# mkdir /var/html/bwapp/bWAPP/{upload,admin_login} -p
# chown -R httpd.httpd /var/html/bwapp/bWAPP/upload
```

0x12 接着,禁止用户在上传目录中执行后端脚本,如`php`,实现方式有两种,个人更推荐后者

第一种,直接往客户端丢403,注意linux下大小写敏感,不然容易被bypass掉,所以这里要加上i,让其不区分大小写
```
<Directory "/var/html/bwapp/bWAPP/upload">
<FilesMatch "\.(?i:php|php3|php4|php5)$">
    Order allow,deny
    Deny from all
</FilesMatch>
</Directory>
```

第二种,在上传目录中,让php直接以普通文本来解析,此时右键源代码,你会发现php压根就没被解析
```
<Directory "/var/html/bwapp/bWAPP/upload">
    AddType text/html .php
</Directory>
```

0x13 禁止用户直接从公网访问网站后台,只允许特定的内网ip段才能访问,对于其它的一些敏感目录,你都可以这么干
```
<Directory "/var/html/bwapp/bWAPP/admin_login">
    Order deny,allow
    allow from 192.168.3.0/24
    deny from all
</Directory>
```

0x14 防止发生解析漏洞,注意linux下大小写的问题,以防逃逸,只需要加到对应的网站目录配置下即可
```
<Directory "/var/html/bwapp">
...
<Files ~ "\.(php.|php3.|php4.|php5.)">
    Order Allow,Deny
    Deny from all
</Files>
...
</Directory>
```

0x15 跟进apache官方发布的各类高危漏洞补丁,适时进行修补,另外,请把你用于测试的各类探针都收好,如,`phpinfo`之流,千万不要因为自己的粗心露点了...

0x16 更多,待续...

<br>
小结:
&nbsp;&nbsp;&nbsp;&nbsp;关于apache,我想,到这里就不用再多说了吧,一个古董级的web服务,大家应该早都轻车熟路,我也就不多废话了,至于`LAMP架构`的加固,现在想必你也应该明白了,把`LNMP架构`的加固套过来即可,除了用的web服务不一样,PHP和Mysql的安全部署方式基本都是一模一样的,不一样的地方可能就在于对php的解析,不过都大同小异,换汤不换药,单单利用apache能做的防御毕竟很有限,但对付一般的`脚本小子`,这种防御早已绰绰有余,如果真的还有更高的安全要求,还是更推荐大家直接去深度定制各种开源或者商用WAF,另外,可以再针对性的写一些实时动态入侵预警脚本相互配合着使用,最近准备把针对各类基础服务的防入侵做成一个完整的系列,留作备忘,其实,也真的非常希望,有些厂商,能更有针对性的防,不要只为了赚钱而赚钱,废话到此为止吧,还没完,咱们待续...最后,也期待能与大家一起多交流 ^_^

