---
title: aircrack & hashcat 非字典高速破解目标无线密码
date: 2015-04-15 10:09:10
tags: wpa2
categories: "wpa/wpa2 crack"
author: klion
---
0x01 挂载好外置无线网卡
&nbsp;&nbsp;&nbsp;&nbsp;把用于抓包的无线网卡挂到kali中,正常情况下,网卡在成功载入系统后,应该是如下的效果,在虚拟机设置中选择'可移动设备'把你的网卡连到虚拟机里
!["将网卡载入kali"](/img/1.png)
如下图可以看到系统已自动识别该网卡,并且已自动搜到附近的无线信号
![查看系统是否已经识别该网卡](/img/2.png)
![看看能不能收到附近的无线信号](/img/3.png)

0x02 检查无线网卡驱动是否能正常工作
&nbsp;&nbsp;&nbsp;&nbsp;紧接着,来看下当前系统中是否已经识别出该无线网卡的驱动,假设当前系统内核中已经有了该种无线网卡的驱动,它就会直接显示出当前无线网卡所使用的芯片组,可以看到,我这里用的是RT2800的芯片组(推荐大家直接去某宝买),如果没有,则可能需要你自己去手工编译安装相应的无线网卡驱动,然后再更新下内核模块即可(在linux中编译安装驱动着实是一件比较麻烦的事情,建议买的时候就直接去买aircrack所支持的网卡型号,起码用的时候不至于这么费劲,要相信万能的某宝),但这并不代表你只要把无线网卡驱动安装好就可以用了,需要搞清楚的是,你所使用的无线网卡芯片组必须要被aircrack所支持才可以,因为用无线网卡的最终目的还是用来捕获握手包
```
# airmon-ng  查看当前系统中的所有可用的无线网卡接口
```
![看看aircrack是否已经识别芯片组](/img/4.png)<br>

0x03 关于aircrack所支持的网卡芯片组列表,请自行参考aircrack官方说明:
```
https://www.aircrack-ng.org/doku.php?id=compatibility_drivers 
```

0x04 为了保证aircrack套件在运行的时候不被其他进程所干扰,我们需要先执行以下命令
```
# airmon-ng check kill
```
![aircrack启动检查](/img/5.png)

0x05 可以看到,当前网卡默认还处在管理模式,这时我们需要手动将其变成监听模式,这样才能进行正常的抓包
```
# iwconfig  使用此命令可查看无线网卡的当前工作模式
```
![查看网卡工作模式](/img/6.png)
```
# airmon-ng start wlan0   把网卡改为监听模式
```
![将网卡切换为监听模式](/img/7.png)
```
# iwconfig  再次查看无线网卡工作模式是否真的已经改过来了
```
![看看网卡工作模式是否已经切换成功](/img/45.png)

0x06
&nbsp;&nbsp;&nbsp;&nbsp;另外,你的无线网卡在启动监听模式以后,网卡接口名称就变成了wlan0mon,以后只要是在aircrack套件中需要指定网卡接口名称的,都要用这个名字,在老版本的aircrack中默认名称是mon0,而新版本则统一变成了wlan0mon,恩,一切准备就绪之后,我们开始尝试扫描附近的无线接入点,找个有客户端在线的再单独监听,一定要注意,"目标无线必须要有客户端在线",否则是抓不到包的,这也是整个无线破解最核心的地方,因为我们要把对方的某个在线客户端蹬掉线,才能截获他的握手包
```
# airodump-ng wlan0mon 开始尝试扫描附近的无线信号,这里就用我们自己的wifi来进行测试,用于测试的wifi的mac和热点名称如下(klionsec)
```
![扫描附近的无线信号](/img/8.png)

0x07
&nbsp;&nbsp;&nbsp;&nbsp;通过上面的扫描,我们选定了名称为"klionsec"的wpa2无线热点作为我们的攻击目标,这里我们需要先记录下目标无线的工作信道以及对应的mac,(后面单独监听时需要用到这些信息),而后,单独监听目标无线热点,注意这里在监听目标无线的过程中不要断开,直到整个抓包过程完成为止,接下来要做的事情就是等待客户端上线,然后进行抓包,例如,下面就表示有一个客户端在线,其实,抓握手包的原理就是先把这个在线的用户给蹬掉线,然后再截获它的握手包,而这个包里就有我们想要的无线密码
```
# airodump-ng --bssid DC:EE:06:96:B7:B8 -c 6 -w sec wlan0mon 监听目标无线,并把截获到的数据写到指定文件中
```
![单独监听目标无线](/img/9.png)

0x08
&nbsp;&nbsp;&nbsp;&nbsp;发现客户端在线稳定后,就可以向目标发射'ddos'流量了,直到我们在监听的终端下看到有握手包出现为止,如果第一轮包发完成后,并没看到握手包,别着急,先等个几十秒,或者隔个五六秒再发一次即可,正常情况下,基本一次就能搞定
```
# aireplay-ng --deauth 15 -a DC:EE:06:96:B7:B8 wlan0mon
```
![开始发动ddos攻击](/img/10.png)

可以看到,这时握手包已被正常抓获,此时监听也就可以断开了,注意观察终端的右上角,那个带有`handshake`标志的就是握手包的意思
![成功获取握手包](/img/11.png)

0x09
&nbsp;&nbsp;&nbsp;&nbsp;在我们抓获握手包以后,接下来的事情就非常简单了,你可以直接用aircrack加载弱口令字典进行爆破,当然,个人是十分不建议用字典(效率,实用性,太低,过于浪费时间),推荐大家直接把包处理一下丢给hashcat或者某宝去跑就行了,两种方法具体操作如下:

(1)第一种,利用aircrack加载字典进行爆破,反正我自己很少用,基本没用过,先不说速度如何,关键还是看你的字典是否靠谱,实际测试中,个人并不建议用,因为根本没有靠谱性可言,因为这里仅仅是测试,实际渗透中,哪有太多的时间让你去跑[比如,实地渗透]
```
# aircrack-ng --bssid DC:EE:06:96:B7:B8 -w pass.txt sec-01.cap   会有三个文件,但最终的密码在cap文件
```
![破解握手包中的无线密码](/img/12.png)

(2)第二种,直接利用hashcat跑hash,不过需要你事先稍微整理下数据包
```
# wpaclean wpapass.cap sec-01.cap  	  可以看到这里已经成功识别出了目标无线id
```
![提取数据包中的hash](/img/13.png)
```
# aircrack-ng wpapass.cap -J wpahash  把数据包转换成hashcat能认识的hash类型
```
![转换成hashcat认识的hash格式](/img/15.png)
```
# hashcat -m 2500 -a 3 wpahash.hccap ?u?l?l?l?l?d?d?d  因为我事先已经知道密码,所以直接这样给掩码会跑的更快一些
```

下面是hashcat的破解结果,可以看到,像wpa这种加密算法,对于hashcat来讲,几乎是瞬间就被秒出来了,因为事先忘记了把wifi改的简单点(`汗……`),所以这里不得不拿之前的一个测试案例来说明了,因为这次用于测试的wifi密码比较复杂,要是硬破,估计明年也破不出来,所以没必要在这里浪费时间,重要的是把方法告诉大家就好,测试结果并不是最重要的,能把要说明的问题说清楚即可
![破解结果](/img/hash.png)<br>

0x10
&nbsp;&nbsp;&nbsp;&nbsp;至此,整个无线密码破解的经典步骤就算完成了,纵观全文,其实,并没多少技术含量在里面,跟着我的文档一步步的来,抓个包,跑个密码基本还是没什么问题的,其实,如果真的特别想关注底层的细节,不妨自己通过wireshark手工完成这一过程,可以明确告诉大家的是,这样是绝对可行的,请自行尝试
<br>
一点小结:
&nbsp;&nbsp;&nbsp;&nbsp;在破解无线密码的问题上,大家大可不用太过纠结,破密码的最终目的,也是希望能通过这种方式来在目标上开个口子,仅此而已,如果真的是哪天运气爆棚,直接捅到目标的办公网,自然是求之不得,因为毕竟是在实地,可能留给我们的时间也不会太多,还会有其它诸多的不便,想短时间内把整个内网摸透,可能也来不及,但想办法先在目标内网留个shell稳住入口,回去接着慢慢搞,还是可行的,一般对于从外部打进去非常难的情况下,这也许也是一种切实可行的渗透手段,祝大家好运吧,不过,最后,还是有句话不得不提醒大家,访问未授权的系统本身就是违法的,望大家洁身自好……!

