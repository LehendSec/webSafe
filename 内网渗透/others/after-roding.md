---
title: 关于接下来的路
date: 2018-01-1 04:09:17
tags: 接下来的路
categories: "接下来的路"
author: klion
---

```
首先,在这里先跟兄弟们道个歉,可能从今天开始,我就要独自完成一整套稍微系统点的内网渗透教程
并准备整理出书,所以,博客的更新不得不先暂停一段时间,说实话,这也是思考很久之后的结果
```

主要是考虑到以下原因
```
一、博客中的内容过于松散杂乱,并不适合系统学习
```

```
二、随着文章量越来越大,加上自己的不断成长,总是会不知不觉发现之前文章中还存在着各种不完美
   但苦于个人精力有限并不是每个地方都能及时看到,所以现在才想全部重新系统的开始,对大家来讲,这无疑是件好事,当然,质量肯定会比之前那些要好很多很多
```

```
三、另外,也是想通过这种方式来时刻督促着自己学习和巩固,因为这对自己本身就是一种比较大的挑战
   `你自己会` 和 `你能把别人清清楚楚的讲会` 是完全两个不同的理解级别,有些朋友可能深有体会
```
<!-- more -->
```
四、一直以来,实在没太多精力去维护自己的圈子
    说心里话,挺对不起里面的朋友的,待这次教程完成后,以后会把主要精力都放在那里,博客以后只顺带维护,等圈子人数够了,博客也会适时关闭
```

```
五、确实更因为生活所迫,所以才不得不更多的考虑到现实,毕竟还要养家糊口,希望大家能谅解 ^_^
```

```
关于教程的整体定位,可能会更偏向实用,如果你只是想随便玩玩,很显然,这些并不太适合您,您完全可以另谋高就
至于课程质量,我只能说,会比您在我博客中看到的那些起码要好很多很多
另外,它跟市面上的所有渗透教程,可能都有着质的区别,其实看目录就明白了,哼哼...
对此我并不能过多的保证什么,唯一能保证的就是认认真真把我该干的好好坚持干下去,仅此而已
最后,需要你稍微有一点基础,不然有些地方理解起来,可能稍微有些费劲
关于具体价格,后期待教程全部完成后,会详细公布在个人博客上
另外,其中还有很大一部分是博客中并没有涉及到的,敬请后期留意,非常感谢这段时间来大家的持续关注,谢谢 ^_^ 具体的课程列表如下
最后,不用问我课程大概什么时候才能完成,因为我自己也不知道,话说回来,其实,真的有能力自学就好了...^_^
```

0x01 apt攻防指南 [ 内网穿透 ]
```
-> 如何正确使用 Tor										
-> 初步了解各类常规内网穿透手段										
-> 深度理解端口转发的底层实现细节									
-> 熟知要用到各类端口转发及隧道的典型渗透场景								
-> 关于 windows 平台下各类常规tcp端口转发及重定向工具的熟练使用,如,netsh,lcx,htran...	
-> 关于 linux 平台下各类常规tcp/udp端口转发工具的熟练使用,socat,iptables,lcx,Ngrok...	
-> 真正理解meterpreter的 `portfwd` 双内网通信实现原理							
-> 什么是 socks协议 及 socks5										
-> socks5代理又是什么东西										
-> 针对各类全平台原生socks代理工具的灵活应用,如,ew,termite,ssocks...				
-> socks 代理客户端入门使用之 proxifier & sockscap 篇							
-> 深入理解http隧道基本工作流程										
-> 如何利用 http隧道来bypass 各类防火墙,如,abptts,reGeorg,tunna,reduh...			
-> socks 代理客户端入门使用之 proxychains 篇								
-> 为什么socks代理不能进行arp嗅探或欺骗,即理解socks代理与vpn的本质区别				
-> 基于ssh隧道的常规本地及远程端口加密转发详解								
-> 配合putty灵活使用ssh的动态端口转发功能								
-> 如何将meterpreter封装在ssh隧道中进行加密传递								
-> 理解icmp隧道												
-> 特殊环境下的icmp隧道穿透技巧						
-> 理解dns隧道											
-> 利用dns隧道进行各种常规tcp端口转发								
-> 什么是GRE隧道									
-> 如何利用GRE隧道直达目标内网										
-> 利用msf sock4a模块弹回多级内网下的meterpreter							
-> 熟练使用CobaltStrike中的各种socks代理功能								
-> 关于目标内网中某些机器无法正常上网的问题								
-> 更多其它内容,持续更新中...
```

0x02 apt攻防指南 [ 常规内网渗透 ]
```
-> 先从一个边界shell开始
-> 如何利用当前机器尽可能摸清目标内网的基本拓扑结构
   -> 如,判断目标是什么类型的内网,存在几个vlan,vlan间是否能互通....
-> 熟练基于各种协议的内网存活主机扫描与深度挖掘方法,如,powershell,smb,arp...
-> 从wireshark中仔细观察理解基于各种协议的内网存活扫描动作
-> 主动搜集目标内网中的各类敏感信息,如,各类账号,密码,邮箱,以及各类配置文件...
-> 密码搜集之 使用tcpdump实时被动抓取本机的各类明文账号密码
-> 密码搜集之 `net-creds 小脚本`简要分析及利用
-> 精心准备各类常规弱口令字典以及制作专门针对目标内网的高质量爆破字典
-> 扫描目标内网中的各类可快速getshell的高危服务端口
   -> 如,sa,各类内网web,内网中的各类数据库,smb,redis,nfs,ftp,mail,snmp...
-> 熟练使用各类系统及web漏洞扫描工具
   -> 如,openvas,nessus,awvs,Netsparker...
-> 使用nmap快速对指定内网进行各种常规服务漏洞扫描
-> 尝试利用snmp搜集目标内网中的路由,交换信息
-> 熟练使用msf中的各类基础服务漏洞利用及信息搜集模块...
-> 灵活使用各类服务爆破工具,如,powershell,medusa,hydra...
-> 从wireshark中深度观察理解内网中的各种服务爆破行为
-> 熟练使用win & linux平台下的各类嗅探欺骗工具
   -> 如,Responder,Dsniff,bettercap,ettercap,cain...
-> 从 wireshark 中观察内网中的各类arp嗅探及欺骗动作
-> 使用 wireshark 来捕捉内网中的各类msf信息搜集模块的数据特征
-> 针对 windows 各个常用溢出提权exp集合汇总
-> 针对 linux 各个常见内核版本溢出提权exp集合汇总
-> win UAC bypass 技巧集汇总
-> 如何利用powershell在win目标机器上建立隐藏账户
-> 针对 https 明文密码的嗅探尝试
-> 如何在win下免杀导出指定进程数据
-> xss平台部署及beef内网应用
-> 主动劫持内网特定机器的浏览器
-> 密码搜集之 截获目标机器中指定浏览器中的https明文账号密码
-> win平台下的各类远程执行命令方式
   -> 如,msxsl,DCOM,schtasks,at,psexec,wmic,wmiexec,powershell,smbexec...
-> 理解不同版本 win及linux 系统用户密码的加密算法
-> 关于不同win版本下的 hash 抓取方式...
-> hashcat 从入门到进阶...
-> john 从入门到进阶...
-> rainbowcrack 从入门到进阶...
-> 关于密码设置复杂性要求的一些个人建议
-> 理解dll劫持
-> 基于常规exe,dll的win平台后门植入技巧...
-> 基于powershell的各类win平台后门植入技巧...
-> 针对win 各个系统内置工具的劫持利用...
-> 针对linux平台下的各类常规后门植入技巧...
-> 了解 win & linux 平台下一些比较经典的开源RAT...
-> 熟练使用各类免杀框架及脚本...
-> 如何利用目标机器上现有的环境快速弹回一个可交互shell...
-> 常规win环境下实现快速批量挂马的几种办法...
-> 使用 wireshark 主动侦测内网特定机器上的reverse_http meterpreter
-> 初步认识cisco后门植入
-> 针对win系统自身的日志处理...
-> 针对linux系统自身的日志处理...
-> 针对第三方基础服务的痕迹处理,如,web,数据库...
-> linux 系统中快速打包下载重要数据的一些方法
-> win平台快速打包下载数据的一些方法
-> 利用 http隧道对内网指定数据库服务器进行脱裤
-> 关于同内网中win和linux机器间相互交叉渗透的方式,如,powershell操作ssh,shell中操作telnet...
-> 关于NSA工具包中部分0day使用
-> CIA Hive 使用指南
-> 跨vlan的问题
-> 更多其它内容,持续更新中...
```

0x03 apt攻防指南 [ 域内网渗透 ]
```
-> 理解windows工作组,域以及域控制器
-> 深入理解域内基本通信过程
-> 深入理解windows的安全认证机制
-> 深入理解域控制器基本组成架构
-> 理解各个构件的主要作用
-> 搜集当前机器以及域内的各类敏感信息
   -> 如何确定当前为域内网
   -> 确定当前域内所有在线的机器列表
   -> 确定所有域管用户及组成员
   -> 确定主域控所在位置
   -> ...
-> 利用powershell快速扫描指定域
-> 尝试各类可快速getshell的漏洞机器,搜集一批管理员hash撞域管
-> 从其它的点搜集各类明文密码及hash,如,各类配置文件,sysvol,spn...
-> 使用查找指定域管进程的方式来尝试抓取指定机器中的域管hash
-> 关于域内超hash传递,以及黄金及白银票据的具体利用方法
-> 登陆域控,完整导出域内所有用户hash 
-> 域内批量挂马的一些方式
-> 维持域管权限的各种办法
-> 关于MS14-068的利用
-> 更多其它内容,持续更新中...
```

0x04 apt攻防指南 [ powershell 攻防系列 ]
```
-> powershell 必要基础
-> powerview
-> Sherlock
-> PowerShell-Suite
-> PowerSploit
-> nishang
-> empire
-> CrackMapExec
-> PowerShell-AD-Recon
-> PowerCat
-> 更多其它内容,持续更新中...
```

0x05 apt攻防指南 [ Cobalt strike 从入门到进阶 ]
```
-> 关于 Cobalt strike
-> 什么是团队服务器以及如何使用团队服务器
-> 初步认识监听器
-> 创建基于不同协议的监听器
-> 创建并初步使用各种payload
-> 发送钓鱼信
-> 生成各类钓鱼页面
-> 实用的派生功能
-> 和msf联动灵活渗透目标内网
-> 如何使用dns隧道进行通信
-> 关于beacon shell内置的各类功能使用
-> 如何配合其它0day一起使用Cobalt strike
-> 使用强大的报告汇总功能
-> 更多其它内容,持续更新中...
```

0x06 apt攻防指南 [ Metasploit ] 
```
web
-> 常规web漏洞exp模块集合汇总
-> 开源web程序exp模块集合汇总
-> 针对各类中间件的exp模块集合汇总

内网
-> 各类内网信息搜集扫描模块集合汇总
-> 针对各类基础服务漏洞的利用模块集合汇总

payload生成
-> msfvenom 从入门到进阶

后渗透
-> meterpreter 从入门到进阶
-> 更多其它内容,持续更新中...
```

0x07 apt攻防指南 [ 发信 ]
```
-> 针对各类较新office 0day的利用,如,CVE-2017-11882,CVE-2017-8570,CVE 2017-0199...
-> 利用PPSX钓鱼
-> 利用快捷方式,如,CVE-2017-8464...
-> 制作无痕chm backdoor
-> 利用winrar的高级扩展属性
-> 图片伪装捆绑
-> 如何将msf的payload嵌到现有的Android app中
-> 制作各类钓鱼页面,并编写账号密码接收脚本
-> 手动触发hta
-> 更多其它内容,持续更新中...
```

0x08 apt攻防指南 [ 实地 ]
```
-> aircrack & hashcat 非字典高速爆破目标wifi密码
-> wifite 全自动获取目标wifi握手包
-> 利用reaver尝试爆破目标pin码
-> 伪造目标ap尝试嗅探
-> NetHunter 从入门到进阶
-> 更多其它内容,持续更新中...
```

<font size="5" color="#00FF7F" style="font-weight:bold;">
以上所有内容仅供网络安全研究及企业入侵防护参考
严禁用于任何非法用途,由此所产生的一切后果与本博客及博客作者无任何关系,请知晓,谢谢 !
</font>
