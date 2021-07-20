---
title: 如何在目标内网中发现更多存活主机 [arp,icmp,tcp/udp,smb,snmp ...]
date: 2016-06-26 08:25:20
tags: discovery
categories: "pentset"
author: klion
---

<br>
0x01 基于不同平台下的各种arp扫描方法 
&nbsp;&nbsp;&nbsp;&nbsp;首先,尝试基于arp的各种内网主机发现方式,它可以轻易bypass掉各类应用层防火墙,这是大家都知道的,如果是专业的arp防火墙,呵呵……

0x02 在win下进行各种arp扫描:
```
# start /b arpscan.exe -t 192.168.3.0/24 >> result.txt
```
![''](/img/common_ethnet_arp.png)
	
利用powershell脚本进行arp扫描,这也是个人比较推荐的方式,轻量且免杀效果较好,系统自带,灵活方便:
```
# powershell.exe -exec bypass -Command "Import-Module C:\Invoke-ARPScan.ps1;Invoke-ARPScan -CIDR 192.168.3.0/24"  >> result.txt
```
![''](/img/common_ethnet_arp_powershell2.png)
	
说到powershell 这里就不得不顺带提下`empire`,它里面同样提供了用于arp扫描的模块,而且比msf的arp更好用[一款优秀的域内网渗透框架,经常进行win内网渗透的朋友,应该用的非常多,这里就不多说了]:	
![''](/img/common_arp_empire.jpg)
	
使用老旧的nmap,另外在win下使用,可能需要你先装好所需的win运行库和npcap,在安装该库的时候,默认选项即可,注意,如果是在中文系统中安装还有点儿问题,推荐用英文系统,所需的所有依赖库在nmap程序包中都已自带,说实话,个人并不建议直接把nmap丢到目标机器上用,没有图形界面的情况下安装依赖库还是比较麻烦的,另外,如果仅仅是存活扫描,它肯定也不会是首选,起码不是自己的首选:
```
# nmap -sn -PR  192.168.3.0/24  以arp的方式扫描
```
![''](/img/common_ethnet_arp_nmap.png)
	
在cain中也带了arp扫描功能,虽然,工具已经n年没有更新过了,但依然经典,建议在03以下的系统使用,另外,它需要免杀:
![''](/img/common_ethnet_arp_cain.png)<br>
		
0x03 在 linux 下进行 arp 扫描 :
&nbsp;&nbsp;&nbsp;&nbsp;其实,在一些主流的linux发型版软件包中已经默认自带了nmap,不过是5.0版本的,一般的运维可能也不会装,所以还是自己动手编译装下吧,这里顺便多一嘴,在目标机器上安装工具,尽量源码编译安装,走的时候,相对容易处理干净:
```
# wget https://nmap.org/dist/nmap-7.40.tar.bz2
# bzip2 -cd nmap-7.40.tar.bz2 | tar xvf -
# cd nmap-7.40 
# ./configure  这里可以用--prefix指定安装路径
# echo $?
# make
# make install
# echo $?
# make install
# echo $?	
# nmap -sn -PR 192.168.3.0/24  尝试arp扫描
```
![''](/img/common_arp_nmap_win.jpg)
	
编译安装arpscan:
```
# unzip arpscan.zip
# cd arpscan
# autoreconf --install
# ./configure  工具依赖libpcap库,这里需要先把库装好
     # wget http://www.tcpdump.org/release/libpcap-1.1.1.tar.gz
     # tar -zxvf libpcap-1.1.1.tar.gz
     # cd libpcap-1.1.1
     # ./configure
     # echo $?
     # make && make install
     # echo $?
     # cd ..
# ./configure 再次执行检测
# echo $?
# make
# echo $?
# chmod -R 755 ./*  先给下权限,不然执行检测时无法通过
# make check
# make install
# echo $?
# arp-scan --interface=eth3 --localnet 扫描指定网卡的网段
```
![''](/img/common_arp_scan_linux.jpg)
	
0x04 其它的一些arp扫描方法:
&nbsp;&nbsp;&nbsp;&nbsp;如果你直接就处在别人的vpn内网中,也可以选择用netdiscover(kali自带),指定那个vpn内网的网卡接口进行arp扫描即可,速度还行
```
# netdiscover -r 192.168.3.0/24 -i eth0
```
![''](/img/common_ethnet_arp_netdis.png)
	
使用msf内置的各种arp扫描模块,还是那句话,如果你目前已经直接处在对方的内网环境中,可以直接用下面的模块
```
msf > use  auxiliary/scanner/discovery/arp_sweep
msf > show options
msf > set  interface eth0
msf > set  smac 00:0c:29:92:fd:85
msf > set  rhosts 192.168.3.0/24
msf > set  threads 20
msf > set  shost 192.168.3.28
msf > run
```
![''](/img/common_ethnet_arp_msf.png)
		
如果你只是拿到对方内网中的一个meterpreter,也可以用meterpreter中内置的arp扫描模块,不过在此之前,你可能还需要在中间先添加一条路由:
``` bash
meterpreter > getsystem 	另外,在目标机器上扫描时,务必先提权,纯属个人建议,会方便很多,不然扫描过程中可能会有些问题
meterpreter > run autoroute -s 192.168.244.0/24
meterpreter > run post/windows/gather/arp_scanner RHOSTS=192.168.3.0/24
```
![''](/img/common_arp_scan_msf_linux.jpg)
<br><br>
0x05 基于icmp的各种内网主机发现方式,如果防火墙过滤的icmp请求,这种方式基本就废了,不过如果是在域还是挺好使的:

0x06 在win下进行各种icmp扫描:

cmd中执行如下命令,对整个C段进行ping扫描
```
# for /L %I in (1,1,254) DO @ping -w 1 -n 1 192.168.3.%I | findstr "TTL=" >> result.txt   扫描从1到254这么多台机器
```
![''](/img/common_ethnet_dos_icmp.png)
	
nmap中同样也提供了基于icmp的扫描方式,PE就是最普通的icmp echo request,另外,还有基于timestamp和netmask request discovery的icmp扫描方式
```
# nmap -sn -PE 192.168.3.0/24
```
![''](/img/common_win_nmap_connect.jpg)
	
使用nping,在nmap程序包中一般也会自带,win下使用暂时还有些儿问题,工具也比较老了,不多说
```
# nping --icmp --icmp-type time 192.168.3.0/24 | findstr "reply"
```
![''](/img/common_ethnet_dos_nping.png)

利用powershell对目标内网进行icmp扫描
```
# powershell.exe -exec bypass -Command "Import-Module C:\Invoke-TSPingSweep.ps1;Invoke-TSPingSweep -StartAddress 192.168.3.1 -EndAddress 192.168.3.254 -ResolveHost -ScanPort -Port 21,22,23,25,53,80,81,82,83,84,85,86,87,88,89,110,111,143,389,443,445,873,1025,1433,1521,2601,3306,3389,3690,5432,5900,7001,8000,8080,8081,8082,8083,8084,8085,8086,8087,8089,9090,10000"    目标网段,并非仅限C段,比如你也可以写成这种方式192.168.3.1 - 192.168.31.254
```
![''](/img/common_ethnet_arp_powershell1.png)
	
0x07 在linux下使用各类icmp扫描:
最简单的方式,将下面的代码保存至shell中,赋予其执行权限,执行该脚本即可
```
#!/bin/bash
for ip in 192.168.3.{1..254} 
do 
   ping $ip -c 1 &> /dev/null 
   if [ $? -eq 0 ];then 
	echo $ip is alive .... 
   fi 
done
```
![''](/img/linux_ping_scan.jpg)
	
同上,依然可以使用nmap的icmp扫描
```
# nmap -sn -PE 192.168.3.0/24
```
![''](/img/common_linux_nmap_connect.jpg)
	
使用nping,用法依然是跟上面一致
```
# nping --icmp --icmp-type time 192.168.3.0/24 | grep "reply"
```
![''](/img/linux_nping.jpg)
<br>

0x07 基于smb和netbios的内网主机发现方式,这种方式通常在win内网中非常实用:
win下:
```
# nbtscan.exe -m  192.168.3.0/24   非常经典的小工具
```
![''](/img/common_nbtscan.png)
	
linux下:
```
# wget http://www.unixwiz.net/tools/nbtscan-source-1.0.35.tgz
# tar -zxvf nbtscan-source-1.0.35.tgz
# make
# echo $?
# ./nbtscan -h
# ./nbtscan -m 192.168.3.0/24
```
![''](/img/common_nbtscan_linux.jpg)
	
其它的一些smb发现方式:

通过已经弹回的meterpreter,在目标机器上添加路由之后,使用smb_version模块亦可实现同样的目的:
```
/auxiliary/scanner/smb/smb_version
```
![''](/img/msf_smb_version_s.jpg)
<br>

0x08 基于常规tcp/udp端口扫描的内网主机发现方式,还是那句话,如果防火墙或者其他防护系统阻隔了对某些端口方法,依然是个废:

scanline tcp/udp端口扫描,非常经典的小工具,单文件,实际渗透中比较方便:
```
# sl -htz 192.168.3.1-160   默认不指定端口的情况下,会按它自己的高危来扫,至于扫哪些端口,请自行查看mcafee官网
# sl -hz -t 21,22,23,25,53,80-89,110,111,143,389,443,445,873,1025,1433,1521,2601,3306,3389,3690,5432,5900,7001,8000,8080-8089,9090,10000 -u 161 192.168.3.1-160 >> result.txt
```
![''](/img/common_sl.png)
	
nmap tcp/udp端口扫描:
```
# nmap --script smb-enum-shares.nse -p445 192.168.3.0/24  扫描可读写共享,能力非常有限,已经有很多更好的替代品
```
![''](/img/common_nmap_smbshares.png)

superscan tcp/udp端口扫描:
纯图形化界面,使用非常简单,另外,它也可以专门用来枚举特定机器的信息,只是用于枚举的时候,不能直接指定网段,简直可惜
![''](/img/common_superscan.jpg)
	
msf中也内置了各种各样的服务端口扫描模块,不过,你可能需要先提权然后添加路由才可以正常使用,当然,如果你直接处在对方内网中就不用了:
```
msf > use auxiliary/scanner/portscan/*
msf > use auxiliary/scanner/smb/*
msf > use auxiliary/scanner/smtp/*
msf > use auxiliary/scanner/snmp/*
msf > use auxiliary/scanner/telnet/*
……
```
![''](/img/common_msf_portscan.jpg)
	
另外,还有个非常不错的py小脚本F-NAScan,速度很快,在linux内网机器上会非常好用[前提是要对应版本的py环境才行]
```
# python F-NAScan.py -h 192.168.3.1-192.168.3.250 -p 21,22,23,25,53,80,81,82,83,84,85,86,87,88,89,110,111,143,389,443,445,873,1025,1433,1521,2601,3306,3389,3690,5432,5900,7001,8000,8080,8081,8082,8083,8084 -m 30 -t 5
```
![''](/img/common_f_nascan.jpg)
<br>

0x09 如果你当前shell权限确实很有限或由于其它各种各样的原因导致我们暂时没法代理进内网,借助web脚本实现对内网进行窥探无疑是个非常好的方式:

基于 aspx 的内网存活探测脚本:
![''](/img/web_commom_aspx.jpg)
	
基于 php 的内网存活探测脚本:
![''](/img/web_commom_php.jpg)
	
基于 jsp 的内网存活探测脚本:
![''](/img/web_commom_jsp.jpg)

0x10 在域内环境下的主机发现方式,一般在域内,各种条件相对来说还是比较宽松的,因为大多数都可能是办公网:
```
# net view
# dsquery computer  其实,域内最好用的外部也就是nbtscan了
```
![''](/img/domain.jpg)
	
0x11 基于snmp的内网信息搜集方式:
```
待续……
```

0x12 最后,再介绍个好玩的ip流量监控小工具
```
iptraf
```
	
一个查端口对应的服务的小工具,有兴趣可自行尝试
```
whatportis
```
<br>
小结:
&nbsp;&nbsp;&nbsp;&nbsp;所有的扫描,有条件的情况下务必都在管理员权限下运行,对于内网主机发现,个人平时大概用到的,基本也就这些了,只不过在实际扫的时候,记得线程给的不要太高,一次扫的端口不要过多,如果工具里面自带的有随机扫ip的选项,最好也把它加上,另外,在内网中,理论上来讲,你应该首先瞄准找一些能快速getshell的内网机器,比如,sa,smb,ftp`针对linux`,能读写的匿名共享等……严禁大规模漫无目的的扫描,当你拿到一台机器以后,马上上去把能抓的密码hash都抓一下,比如,浏览器中的各种密码,本机的hash,常用软件中的各种密码hash等等……,实在不行再上键盘记录,拿到这些密码以后再慢慢拓展其它机器,切记,尽量不要进行长时间大流量的扫描动作`基本上我们现在所用的工具报文标志早已被写进各类ids的识别规则里了,尤其是针对nmap的`,会在对方系统中留下大批的扫描日志不说,稍微严谨点儿的内网可能会触发报警,甚至直接锁定ip,内网不比web,在web中你也许可以相对放开点去搞,但内网中,务必小心谨慎,权限来之不易,绝对不能让它轻易的掉了,尽你所能的稳住当前机器,另外,在平时的内网渗透中,能用系统自身工具搞定的,尽量都用系统自带的工具来搞`其实,系统自己就已经有非常多的渗透工具,可能只是暂时还没有很好的被发掘出来`,这一点非常重要,大家从现在开始,也应该尽量养成自己这样的习惯,尽可能的减少使用一些外部工具,越少越好,安全性没法保障的同时,可能实际使用中也并不是很方便,灵活性就更不用说了,尤其在一些比较畸形的内网中,由于种种限制,从远程下工具还是比较困难的,另外,别人的工具很可能有相当一部分工具都是需要自己免杀的`单单基于汇编层免杀还是很有限的`,如果你不会免杀,这事儿就很麻烦了,毕竟不是源自自己的手,用别人的始终不太放心,所以,有些东西还是大家自己考量吧……这里只是一点个人建议而已,当然,如果你有喜欢逆别人工具的习惯,这里所说的一切,您都可以直接忽略,如果自己不会逆向,还是谨慎点儿好,如果你想更仔细的去观察基于不同协议更底层的扫描细节,用wireshark吧,它绝对是我见过的最牛最实用的渗透工具,是的,没有之一,祝,愉快
