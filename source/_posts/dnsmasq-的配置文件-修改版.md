---
title: dnsmasq 的配置文件(修改版)
date: 2017-04-28 14:56:26
updated:
tags:
- dnsmasq
- DNS
categories:
- DNS
- dnsmasq
keywords:
- dnsmasq

---


# 前言 
 - 在代码格式中我使用了诸如“// 1”的注释表示，1表示取消当前注释则开启说明所表示的功能，0则相反
 - 在这篇文章中你会见到一些专业术语，这里尽可能地将它们列出来，并尽可能地在文末提供了注释
---
| 中文 | 英文 |  缩写   |     |  
| ---- | ---- | --- | --- | 
|  按需拨号    |  dial-on-demand    |     |     |  
|  选项    |  option    |     |     |  
|域|domain||
|主机|host||
|非路由|non-routed||
|空间|space||
| 服务定位资源记录||SRV||
|会话初始协议|Session Initiation Protocol|SIP||
|域名服务器|nameserver||
|域名|domain name||
|会话|talk||
|用户身份证明|User Identification|UID||
|组身份证明|group Identification|GID||
|接口|Interface||
|监听|listen||
|租借|lease||
|中继代理|relay agent||
|电子掩码|netmask||
|网络 ID|network-id||
|参数|parameter||
|客户机标识符|client identifier||
|操作系统|Operating System|OS||
|启动|boot||
| 标志|flag||
| 供应商类| vendorclass||
|用户类|userclass||
|字串|substring||
|模式|pattern||
| 映射|mappings||
|预设路径|default route||
|网络时间协议|Network Time Protocol|NTP||
|存活时间|time-to-live|ttl||
|子网|subnet||
|静态|static||
|特性|specific||
|封装|encapsulate||
|前缀|prefix||
|超时|timeout||
|装载|Load||
|多路广播|multicast||
|广播|broadcast||
|简单文件传输协议|Trivial File Transfer Protocol|TFTP||
|电子签名|Verisign||
|邮件交换|Mail Exchanger|MX||
|指针记录|Pointer Record|PTR||
|别名|alias||


---
# DNS
## 基础转发规则
格式是一个选项一行，合法的多个选项跟在命令行中的多个合格的选项一样。见“/usr/sbin/dnsmasq –help” 或者“man 8 dnsmasq” 里的更多细节。下面的两个选项让你更好地使用网络，因为它们告诉 dnsmasq 过滤那些公共 DNS 不能处理的不需要装在服务器( 特别是根服务器中 )的请求。如果你有个按需拨号（dial-on-demand）的连接，这些服务器会阻止这些不需要的连接的建立( bring up )。
### 从不转发那些不存在的名( 没有点或者域部分 )

``` 
#domain-needed //1
```
### 从不为存在于那些非路由（non-routed ）地址的空间转发地址

``` vala
#bogus-priv   //1
```

### 过滤无用的来源为Windows的DNS请求
取消这个注释以过滤无用的来源为Windows的DNS请求，这些请求会引起不必要的按需拨号的连接。注意 ( 在其他地方 )，这阻止所有<span id = "return_from_SRV_record"></span> [服务定位资源记录 (SRV )](#SRV_record)，所以除非你使用诸如<span id = "return_from_Kerberos">[Kerberos](#Kerberos), <span id = "return_from_SIP"></span>[SIP](#SIP), <span id = "return_from_XMPP"></span>[XMMP](#XMPP) 或者<span id ="return_from_Google-talk"></span> [Google-talk](#Google-talk) ，不要使用它。这个选项只影响转发,为 dnsmasq 生成的SRV 记录 ( 通过 srv-host= lines ) 不受其限制

``` gcode
#filterwin2k //1
```
## 上游服务器相关
### 从上游服务器中获取 dns
改变这一行如果你想要从上游服务器中获取dns，上游的设置文件在`/etc/resolv.conf`

``` vala
#resolv-file=   //填入上游文件名
```

### 强制上游 DNS 寻找顺序
默认情况下，dnsmasq 会发送请求给任意一个知道回复的上游服务器并设置最优服务器。取消这里的注释以强制dnsmasq 按设置文件/`etc/resolv.conf`中顺序对上游服务器发出尝试请求

``` vala
#strict-order //1
```
### 不读取conf 或其他文件
如果你不希望dnsmasq 读取`/etc/resolv.conf`或其他文件而是在这个文件中获得上游域名服务器<span id = "return_from_other_servers"></span>([如下](#other_servers))，取消这一条注释

### 不查找conf，resolv文件
如果你不需要dnsmasq 查找`/etc/resolv.conf`或其他`resolv文件`和重读，注释掉这里

``` vala
#no-poll  //1
```
<span id = "other_servers"></span>
### 其他域名服务器
在这里添加其他域名服务器，如果是私有域名，添加域名规格

``` 
#server=/localnet/192.168.0.1 “localnet ”是域名规格
or
#server=114.114.114.114

```
[返回](#return_from_other_servers)
## 请求转发类型
### 路由器选择
一个路由器选择PTR请求到域名服务器( nameserver  )：这会发送所有对`192.168.3/24`的请求`address->name` 到域名服务器`10.1.2.3`

``` 
#server=/3.168.192.in-addr.arpa/10.1.2.3
```

### 仅本地域名
添加仅本地( local-only )域名,对这些域名的请求将仅由 `/etc/hosts` 或者 `DHCP` 回应

``` shell
#local=/localnet/
```

### 强制跳转
 - 添加强制到特定IP的域名。下边的例子发送任何在 `doubleclick-net`到本地服务器

```
#address=/doubleclick.net/127.0.0.1
```
 - address或者server与IPv6也是支持的

``` 
#address=/www.thekelleys.org.uk/fe80::20d:60ff:fe36:f83
```

## 控制会话方式
你可以控制dnsmasq 与一个服务器的会话方式：
- 这个强制对10.1.2.3的请求通过eth1：`server=10.1.2.3@eth1`
- 这个设置与10.1.2.3的会话的源(ie local  )地址到192.168.1.1，端口（port）55(显然这个机器必须要有与这个IP绑定的接口)：`server=10.1.2.3@192.168.1.1#55`


## 改变uid和gid
如果你想要dnsmasq改变uid和gid到其他设置，而不使用预设值，编辑下边的行

``` 
#user=

#group=
```
## 接口
### 监听
 - 如果你想要dnsmasq仅在特定接口（和回路）监听 DHCP 和 DNS 请求，给接口命名（如：`eth0`）,重复行如果你有多个接口

``` vala
#interface=
```

 - 或者你设置特定接口不监听

``` stylus
#except-interface=
```

- 或者哪个通过特定地址（address）监听（包括127.0.0.1如果有使用的话）

``` stylus
#listen-address=
```

### 一个接口上提供 DNS
如果你只想在一个接口上提供 DNS 服务，如下设置它，然后使用如下行为它关闭DHCP

``` vala
#no-dhcp-interface=
```
### 接口接受通配符地址
在支持的系统上,dnsmasq绑定通配符地址，即使在它只在一些接口上监听，然后它抛弃那些不应该回复的请求。即使在接口不断变化和改变地址的情况下，这仍然有好处.如果你确实要dnsmasq去绑定它正在监听的接口，注释掉这个选择.是在你在于一个机器上运行另一个域名服务器的情况下，你可能不止一次需要它

``` shell
#bind-interfaces
```
## hosts
### 读取/etc/hosts文件
如果你不想要dnsmasq去读取`/etc/hosts`, 注释掉这一行

``` vala
#no-hosts
```
### 添加其他hosts文件
或者你需要其他文件，和` /etc/hosts`，用这个

``` vala
#addn-hosts=/etc/banner_add_hosts
```
### 域名自动添加
设置这个（和域名：见下边），如果你想要一个域名自动被添加到hosts-file 的简单名中

``` vala
#expand-hosts
```
## dnsmasq 域名
### 为dnsmasq设置一个域名，这是可选的，但一旦设置，它执行了如下行为

 1. 允许 DHCP 主机有一个完整合法的域名，只要域部分符合这个设定
 2. 设置“domain” DHCP 选项从而隐藏地所有系统上的域由 DHCP 设定
 3. 为 “expand-hosts”提供域部分

``` 
#domain=thekelleys.org.uk
```
- 为不同的子网设置不同域名

``` 
#domain=wireless.thekelleys.org.uk,192.168.2.0/24
```
- 更多选项，比子网更加扩展

``` stylus
#domain=reserved.thekelleys.org.uk,192.68.3.100,192.168.3.200
```
---
# DHCP
## 启用完整的 DHCP
注释掉这一行以启用完整的 DHCP 服务，你需要提供地址（addresses）范围（range）以供租借（租借时间是可选的）。
如果你有多个网络，在每个应用DHCP 服务的网络重复这个

``` 
#dhcp-range=192.168.0.50,192.168.0.150,12h
```
## DHCP 范围类型
### 电子掩码
这是一个拥有电子掩码DHCP 范围的例子。在我们通过一个中继代理（relay agent）到达dnsmasq DHCP 服务的情况下，电子掩码是必须的。如果你并不知道什么是DHCP 中继代理，你大概不需要注意这个

``` stylus
#dhcp-range=192.168.0.50,192.168.0.150,255.255.255.0,12h
```
### 网络ID
这是一个拥有网络ID（network-id）的DHCP范围的例子，所以一些DHCP选项可以被设置仅对这个网络生效

``` 
#dhcp-range=red,192.168.0.50,192.168.0.150
```
## 对主机的处理
使用DHCP 为特定主机应用参数。这里是一些有效的选择，所以我们为每个提供了例子.注意IP 地址**不应该**出现在上述的范围中，它们只需要在同一个网络。参数的顺序不做要求，name,adddress 和 MAC 的任意排序是允许的。
 1.总是为主机分配以太网地址。
 
- 11:22:33:44:55:66
- The IP address 192.168.0.60 

``` 

#dhcp-host=11:22:33:44:55:66,192.168.0.60
```
 2.总是用硬件地址为主机设置名

 - 11:22:33:44:55:66 to be “fred”

``` stylus
#dhcp-host=11:22:33:44:55:66,fred
```
 3.总是给主机以太网地址，名字“fred”，ip地址，租借时间45分


``` stylus
#dhcp-host=11:22:33:44:55:66,fred,192.168.0.60,45m
```

 4.给一个主机两个以太网地址（用","分隔)和IP地址.dnsmasq会假定这两个以太网接口没有被在同一时间使用，然后把IP地址分配给第二个，尽管这个IP地址已经被第一个接口使用。在使用无线和有线地址的笔记本上十分有用

 
``` 
#dhcp-host=11:22:33:44:55:66,12:34:56:78:90:12,192.168.0.60
```

 5.给一个声称它的名是“bert"的主机 IP 地址和无限（infinite）租借时间



``` 
#dhcp-host=bert,192.168.0.70,infinite
```
 6.总是给主机客户机标识符
 - 标识符 01:02:02:04
 - IP 地址 192.168.0.60

``` 
#dhcp-host=id:01:02:02:04,192.168.0.60
```
 7.总是给主机客户标识符


 - 标识符 marjorie
 - IP地址 192.168.0.60  

```
#dhcp-host=id:marjorie,192.168.0.60
```
 8.启用在` /etc/hosts`中赋予名“judge”的地址，当一个机器声称自己名是”judge“的主机请求 DHCP 租借时，将地址赋予它

``` vala
#dhcp-host=judge
```
 9.永远不给以太网地址是11:22:33:44:55:66的机器提供DHCP服务

``` 
#dhcp-host=11:22:33:44:55:66,ignore
```

 10.忽略以太网地址为11：22：33：44:55的提供用户标识符的机器。在阻止那些运行不同OS或PXE boot和 PS BOOT之间切换的机器受到不同的处理时很有用

``` stylus
#dhcp-host=11:22:33:44:55:66,id:*
```
 11.发送标志为“red”的额外选项给以太网地址为11:22:33:44:55的机器

``` vala
#dhcp-host=11:22:33:44:55:66,net:red
```

 12.发送标志为“red”的额外选项给以太网地址以11:22:33开始的机器

``` stylus
#dhcp-host=11:22:33:::*,net:red
```

 13.忽略任何在`dhcp-host` 行或 `etc/ethers`文件 中被指定的客户( clients ). 等同于<span id = "return_from_ISC"></span>[ISC](#ISC)“ deny unkown-clients” .这依赖于主机匹配时建立的特殊“known”标志

 14.发送标志为“red”的额外选择给DHCP 供应商类（ vendorclass）串( string )包含字串( substring  )“Linux”的机器
 

``` shell
 #dhcp-vendorclass=red,Linux
```

 15.发送标志为“red”的额外选择给用户类( userclass )串 string )包含字串字串( substring )“accounts”的机器
 

``` shell
 #dhcp-userclass=red,accounts
```

 16.发送标志为“red”的额外选择给 MAC 地址匹配模式( pattern )的机器
 

``` clean
 #dhcp-mac=red,00:60:8C:::*
```
## 对发现的`物理地址/IP`对执行动作
如果取消这段注释，dsnamasq将会读取`/etc/ethers`文件并对发现的`物理地址/IP`对( `ethernet-address/IP` )做出执行动作，正像它们被赋予的“–dhcp-host options”。在你出于其他目的保留 `物理地址/主机`( MAC-address/host ) 映射(  mappings  )时十分有用

``` shell
 #read-ethers
```
## 发送的选项
发送一些选项给请求 DHCP 租借的主机，详见`RFC 2132`，dnsmasq按照以名的方法给出常用的选择：执行`dnsmasq –help dhcp`可以查看选择表。注意，dnsmasq提供完整的预设值( defaults )包括所有的常用设定，比如网络掩码，广播地址，DNS 服务器和预设路径（route）.你很大可能不用用到`dhcp-options`。如果你使用window 客户端，Samba，请参照这一部分尾部的推荐值
### 路径
 - 推翻dnsmasq提供的预设路径（default route），这假定路由器与运行dnsmasq为同一个机器

``` lsl
#dhcp-option=3,1.2.3.4
```
 - 做同样的事情，单使用选择名（option name）

``` x86asm
#dhcp-option=option:router,1.2.3.4
```
 - 推翻dnsmasq提供的预设路径（default route），但不发送任何的预设路径。注意这只适用于在由预设`(1, 3, 6, 12, 28)`提供选择，同一行 选择`0长度`( zero-length  )选择以替换其他选择字段(numbers)

``` vala
#dhcp-option=3
```
## 网络时间协议 NTP
 - 设置网络时间协议( NTP )时间服务器地址为 `192.168.0.4`和`10.10.0.5`

``` x86asm
#dhcp-option=option:ntp-server,192.168.0.4,10.10.0.5
```
 - 设置网络时间协议( NTP )时间服务器到同一个机器，正如运行dnsmasq

``` lsl
#dhcp-option=42,0.0.0.0
```
 - 设置网络时间协议( NTP )到域名“welly”

``` vala
#dhcp-option=40,welly

```
## 存活时间
 - 设置预设存活时间( time-to-live )50

``` lsl
#dhcp-option=23,50
```
## 标志类
 - 设置“all subnets are local”(所有子网为本地)标志

``` lsl
#dhcp-option=27,1
```
## 标志+选项类
 - 发送  etherboot magic 标志( flag )和etherboot 选项( 串 )

``` vala
#dhcp-option=128,e4:45:74:68:00:00

#dhcp-option=129,NIC=eepro100
```

 - 指定一个仅被发送给“red”网络的选项( 见 dhcp-range for the declaration of the “red” network )。注意`net: `部分在 `option: `部分前

``` x86asm
#dhcp-option = net:red, option:ntp-server, 192.168.1.1
```
## 建立 dnsmasq
下边的建立 dnsmasq 的DHCP 选项与`http://www.samba.org/samba/ftp/docs/textdocs/DHCP-Server-Configuration.txt`中关于适用于一台的运行 dnsmasq 和 samba 服务的主机安装典型 dnsmasq 的`ISC dhcpcd`一样。当你使用Windows客户端和Samba，你可能需要取消一部分或者全部的注释

``` lsl
#dhcp-option=19,0 # option ip-forwarding off

#dhcp-option=44,0.0.0.0 # set netbios-over-TCP/IP nameserver(s) aka WINS server(s)

#dhcp-option=45,0.0.0.0 # netbios datagram distribution server

#dhcp-option=46,8 # netbios node type
```
## 域名查找
### DNS 域名查找 DHCP
发送 RFC-3397 DNS 域名查找 DHCP 选项。注意：你的 DHCP 客户端可能不支持这个

``` stylus
#dhcp-option=option:domain-search,eng.apple.com,marketing.apple.com
```
##  静态路径
##  发送 RFC-3442 无分级( classless  )静态路径( 注意网络掩码的编码 )

``` lsl
#dhcp-option=121,192.168.1.0/24,1.2.3.4,10.0.0.0/8,5.6.7.8
```
## 提供商类特性选项
 - 发送提供商类特性选项，这些特性封装在 DHCP option 43 。特性的含义由提供商类定义，所以这些选项只在客户端提供的提供商类匹配这里给出的类( 字串匹配是允许的，所以 “MSFT” 跟 “MSFT” 和 “MSFT 5.0” 匹配)。这个例子为 PXEClients 设置了 mtftp address 到0.0.0.0

``` x86asm
#dhcp-option=vendor:PXEClient,1,0.0.0.0
```
 - 当它关闭时，发送 microsoft-specific 选项告诉 Windows 释放 DHCP 租借。注意
" i "标志，它告诉dnsmasq 发送 4-byte 整值，这正是 microsoft 想要的，见`http://technet2.microsoft.com/WindowsServer/en/library/a70f1bb7-d2d4-49f0-96d6-4b7414ecfaae1033.mspx?mfr=true`

``` vala
#dhcp-option=vendor:MSFT,2,1i
```
 - 发送 Encapsulated-vendor-class ID，这个在 Etherboot  辨识 DHCP 伺服器时候被需求

``` vala
#dhcp-option=vendor:Etherboot,60,”Etherboot”
```
## PXELinux
### 发送选项给 PXELinux 
注意我们需要发送它们，尽管它们不出现在参数请求表 ( parameter request list )中,所以我们需要 `dhcp-option-force` ,见` http://syslinux.zytor.com/pxe.php#special` .Magic number --- 在完成任何辨识之前是需要的

``` vala
#dhcp-option-force=208,f1:00:74:7e
```
### 配置文件名

``` x86asm
#dhcp-option-force=209,configs/common
```
### 路径前缀( prefix )

``` gams
#dhcp-option-force=210,/tftpboot/pxelinux/files/
```

### 重启时间( 注意“ i ”发送 32-bit 值 )

``` lsl
#dhcp-option-force=211,30i
```
### 为 netboot/PXE 设置 boot 文件名
你仅在通过网络启动机器时候用到它，并且你需要一个 TFTP 伺服器，无论  dnsmasq 建立在 TFTP 或者其他外部位置.( 见下：如何启用 TFTP 伺服器)

``` vala
#dhcp-boot=pxelinux.0
```
### Boot for Etherboot gPXE. 
想法是发送两个不同的文件名，第一个载入 gPXE , 第二个告诉 gPXE 需要读取的内容,  dhcp-match 为来自 gPXE 的请求设置 gpxe 标志

``` shell
#dhcp-match=gpxe,175 # gPXE sends a 175 option.

#dhcp-boot=net:#gpxe,undionly.kpxe

#dhcp-boot=mybootimage
```
### Encapsulated options for Etherboot gPXE. 
所有选项包装在 option 175

``` shell
#dhcp-option=encap:175, 1, 5b # priority code

#dhcp-option=encap:175, 176, 1b # no-proxydhcp

#dhcp-option=encap:175, 177, string # bus-id

#dhcp-option=encap:175, 189, 1b # BIOS drive code

#dhcp-option=encap:175, 190, user # iSCSI username

#dhcp-option=encap:175, 191, pass # iSCSI password
```
### Test for the architecture of a netboot client. PXE clients 如 option 93 发送它们的结构

``` shell
#dhcp-match=peecees, option:client-arch, 0 #x86-32

#dhcp-match=itanics, option:client-arch, 2 #IA64

#dhcp-match=hammers, option:client-arch, 6 #x86-64

#dhcp-match=mactels, option:client-arch, 7 #EFI x86-64
```
### 执行
#### 执行真正的 PXE，不要仅启动单个文件, 
 - 这是 dhcp-boot 的一个选择

``` maxima
#pxe-prompt=”What system shall I netboot?”
```
 - 或者在第一个可执行动作前加入超时( timeout )

``` stylus
#pxe-prompt=”Press F8 for menu.”, 60
```
#### Available boot services. for PXE.

``` shell
#pxe-service=x86PC, “Boot from local disk”, 0
```
### 装载
 - 从 dnsmasq TFTP server 装载  /pxelinux.

``` vala
#pxe-service=x86PC, “Install Linux”, pxelinux
```
 - 从 dnsmasq TFTP server (at 1.2.3.4)装载  /pxelinux.注意这在老式 PXE ROMS 无效

``` lsl
#pxe-service=x86PC, “Install Linux”, pxelinux, 1.2.3.4
```
## bootserver
 - 在网络上使用 bootserver 发现我的多路广播( multicast )和广播(  broadcast )

``` vala
#pxe-service=x86PC, “Install windows from RIS server”, 1
```
 - 在一个已知 IP 地址使用 bootserver

``` lsl
#pxe-service=x86PC, “Install windows from RIS server”, 1, 1.2.3.4
```
## TFTP
如果你有一个可用的 multicast-FTP, 对它的信息可以以相似的方式上传送---使用 options 1 到 5，见 `http://download.intel.com/design/archives/wfm/downloads/pxespec.pdf` 第19页. 
### 启用 dnsmasq’s built-in TFTP server

``` shell
#enable-tftp  //1
```
### 设置 FTP  文件 root directory 

``` lasso
#tftp-root=/var/ftpd //1
```
### 使 TFTP 伺服器更加安全
使用这个：只有 user dnsmasq 拥有的文件可以通过网络运行和发送到

``` vala
#tftp-secure //1
```
### 只有“red”标志被设定的时候设置 boot 文件名

``` stata
#dhcp-boot=net:red,pxelinux.red-net  
```
### 例子
包含  dhcp-boot ，外部 TFTP 伺服器( 名和 IP 地址在文件名字段后给出 ). 可能在老旧的  PXE ROMS 上失效. 被–pxe-service 覆盖( Overridden )

``` lsl
#dhcp-boot=/var/ftpd/pxelinux.0,boothost,192.168.0.3
```
## 服务器
### 租借

#### 租借限制
设置 DHCP 租借限制, 预设是150

``` vala
#dhcp-lease-max=150
```
### 磁盘
#### 数据库占用
DHCP 服务器需要一定的磁盘空间保存数据库. 默认全路径, 改变它使用如下一行

``` lasso
#dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
```
### authoritative 模式
设置DHCP 伺服器到 authoritative 模式.
在这个模式它将会干涉和接管在网络上广播的客户端的租约, 无论这个客户端有没有租约记录. 当一个机器在一个新的网络上被唤醒时，这个模式避免了长时间的超时. 不要启用这个如果你几乎不会遇到当你为校园/公司设定 DHCP 服务器时发生意外中止的情况.  ISC server 使用同样的 option, 见这个链接:`http://www.isc.org/index.pl?/sw/dhcp/authoritative.php`

``` vala
#dhcp-authoritative
```
### 创建或者销毁
当一个DHCP 租借创建或者销毁时运行一个 executable . 这个发送到脚本( script )的声明是"add" 或者"del" .当只有一个时,之后加上 mac 地址，IP 地址，主机名

``` shell
#dhcp-script=/bin/echo
```
### 缓存
#### 设置缓存大小

``` vala
#cache-size=150
```
#### 关闭 negative caching
取消这个注释

``` vala
#no-negcache
```
#### 个存活时间
一般地, 来源于 `/etc/hosts` 和 `DHCP lease file` 有一个0的存活时间设定，习惯上约定不要进行进一步缓存( cache further ),如果你想要在服务器上低负载地为可能过期的数据进行交换, 设置一个存活时间( Time-To-Live )

``` shell
#local-ttl=
```
## 如果你想要 dnsmasq 去检测依据电子签名( Verisign )的尝试从而发送未注册的 .com 和 .net 主机到它的网站发现服务然后让 dnsmasq 替代性地返回正确的 NXDOMAIN 回答( respond ), 取消这一行的注释. 你可以添加类似的(多)行为其他的注册( 使用通配符 A 记录 )执行同一效果

``` lsl
#bogus-nxdomain=64.94.110.11
```
## 修改DNS结果
如果你想修改从上游 DNS 服务器传递过来的 DNS 结果，使用别名( alias ) 选项. 这只在 IPv4 上工作. 

 - 这个别名让结果为 1.2.3.4 显示为 5.6.7.8

``` lsl
#alias=1.2.3.4,5.6.7.8
```
 - 这个映射 1.2.3.x 到 5.6.7.x

``` lsl
#alias=1.2.3.0,5.6.7.0,255.255.255.0
```

 - 这个映射范围 192.168.0.10->192.168.0.40 到 10.0.0.10->10.0.0.40

``` lsl
#alias=192.168.0.10-192.168.0.40,10.0.0.0,255.255.255.0
```
## MX 记录
改变这一行，如果你想要 dmsmasq 服务 MX 记录. 返回一个 MX 记录:名`maildomain.com ` 目标( target )`servermachine.com`权重( preference )`50`.

``` stylus
#mx-host=maildomain.com,servermachine.com,50
```
### 使用 localmx option 为创建的 MX 记录设置默认目标

``` vala
#mx-target=servermachine.com
```


### 为所有本地机器返回一个指向 mx-target 的 MX 记录

``` vala
#localmx  //1
```

### 为所有本地机器返回一个指向它们自己的 MX 记录

``` vala
#selfmx  //1
```
## SRV 记录
改变下边的(多)行如果你想 dnsmasq 去服务 SRV 记录. 这在你想要服务请求活动目录的 idap 请求和其他 windows-originated DNS 请求时是十分有用的.见`RFC 2782`. 你可能会添加多条 `srv-host` 行, 域(field )是,,,,. 如果一个域没有名( 仅有服务和协议部分)，`domain=` ---config option给出, 将被使用. 注意这个服务不需要 expand-hosts 的设定; 
 - 一个 SRV 记录发送 目的 example.com 域的 LDAP 到ldapserver.example.com 端口389 (port) 

``` stylus
#srv-host=_ldap._tcp.example.com,ldapserver.example.com,389
```

 - 一个 SRV 记录发送 目的 example.com 域的 LDAP 到ldapserver.example.com 端口289 (port)(使用 domain= )

``` stylus
#domain=example.com
#srv-host=_ldap._tcp,ldapserver.example.com,389
```
 - 两个 LDAP 的 SRV 记录, 每个有不同的优先级


``` stylus
srv-host=_ldap._tcp.example.com,ldapserver.example.com,389,1

#srv-host=_ldap._tcp.example.com,ldapserver.example.com,389,2
```
 - 一个 SRV 记录 表明没有域 example.com 的 LDAP 服务器

``` stylus
#srv-host=_ldap._tcp.example.com
```
## PTR 记录
下边的多行表示如何让 dnsmasq 服务一个任意的( arbitrary  ) PTR 记录. 这对 DNS-SD 十分有用. 注意: 为 SRV 记录执行的 domain-name 扩展( expasion ) 不发生在 PTR 记录

``` ada
#ptr-record=_http._tcp.dns-sd-services,”New Employee Page._http._tcp.dns-sd-services”
```

## TXT 记录
修改下边的多行以使 dnsmasq 服务 TXT 记录. 这在诸如 SPF 和 zeroconf 时使用. 注意： 为 SRV 记录执行的 domain-name 扩展( expasion ) 不发生在 TXT 记录

``` stylus
#Example SPF.

#txt-record=example.com,”v=spf1 a -all”

#Example zeroconf

#txt-record=_http._tcp.example.com,name=value,paper=A4
```
## 别名 alias
为一个 “local” DNS 名提供别名( alias ). 注意：这只为那些名来源于`DHCP` 和 `/etc/hosts` 的目标工作. 给主机 “ bert ” 其他的名,bertrand

``` vala
#cname=bertand,bert
```
---
## 日志 log
### 想要 debugging, 通过 dnsmasq 为每个 DNS 查询保存日志( log )

``` stylus
#log-queries  //1
```
### 为 DHCP 交换的额外信息保存日志

``` shell
#log-dhcp  //1
```
# 包括其他多项的设置选项( configuration options)

``` stata
#conf-file=/etc/dnsmasq.more.conf

#conf-dir=/etc/dnsmasq.d
```


---
# 其他注释 (  ) ,
<span id = "SRV_record" ></span>
## SRV记录:
### 定义 ：
DNS 服务器的数据库中支持的一种资源记录的类型，它记录了哪台计算机提供了哪个服务这么一个简单的信息。一般是为 Microsoft 的活动目录设置时的应用。

### 简介：
DNS 可以独立于活动目录，但是活动目录必须有 DNS 的帮助才能工作。为了活动目录能够正常的工作，DNS 服务器必须支持服务定位（SRV）资源记录，资源记录把服务名字映射为提供服务的服务器名字。活动目录客户和域控制器使用 SRV 资源记录决定域控制器的 IP 地址。


### SRV 记录功能包括（基于它们在 DNS 控制台的分组）

 1.‘　_MSDCS。这个分组中，SRV 记录是根据它们的状态来收集的。各种状态包括 DC、域调用、GC 以及 PDC。DC 和 GC 按站点来划分，这样一来，AD 客户端就能快速的知道去哪里寻找本地服务。“域调用” 用于支持复制。每个 DC 都获得了一个 GUID，它会在调用复制时用到。PDC 条目包含了被设定为 PDC 模拟器的 DC 的 SRV 记录。
 2.‘ _SITES。站点代表的是一个高速连接区域，根据 DC 的站点从属关系来建立了 DC 索引之后，客户端就可以检查_SITES 来寻找本地服务，而不必通过 WAN 来发送它们的 LDAP 查询请求。标准 LDAP 查询端口是 389，全局编录查询则使用 3268。
 3.‘　_TCP。在这个分组中，收集了 DNS 区域中的所有 DC。如果客户端找不到它们特定的站点，或者具有本地 SRV 记录的任何 DC 都没有响应，需要寻找网络中其他地方的 DC，就应该将这些客户端放到这个分组中。
 4.‘　_UDP。Keberos v5 允许客户端使用 “无连接” 服务来获取票证并更改密码。这是通过与相同服务的 TCP 端口对应的 UDP 端口来完成的。具体说，票证交换使用 UDP 的 88 端口，而密码更改使用 464。
[返回](#return_from_SRV_record)

---

<span id = "Kerberos"><span>
## Kerberos：
### 简介
一种网络认证协议，其设计目标是通过密钥系统为客户机 / 服务器应用程序提供强大的认证服务。该认证过程的实现不依赖于主机操作系统的认证，无需基于主机地址的信任，不要求网络上所有主机的物理安全，并假定网络上传送的数据包可以被任意地读取、修改和插入数据。在以上情况下， Kerberos 作为一种可信任的第三方认证服务，是通过传统的密码技术（如：共享密钥）执行认证服务的。

### 认证过程
具体如下：客户机向认证服务器（AS）发送请求，要求得到某服务器的证书，然后 AS 的响应包含这些用客户端密钥加密的证书。证书的构成为： 1) 服务器 “ticket” ； 2) 一个临时加密密钥（又称为会话密钥 “session key”） 。客户机将 ticket （包括用服务器密钥加密的客户机身份和一份会话密钥的拷贝）传送到服务器上。会话密钥可以（现已经由客户机和服务器共享）用来认证客户机或认证服务器，也可用来为通信双方以后的通讯提供加密服务，或通过交换独立子会话密钥为通信双方提供进一步的通信加密服务。

上述认证交换过程需要只读方式访问 Kerberos 数据库。但有时，数据库中的记录必须进行修改，如添加新的规则或改变规则密钥时。修改过程通过客户机和第三方 Kerberos 服务器（Kerberos 管理器 KADM）间的协议完成。有关管理协议在此不作介绍。另外也有一种协议用于维护多份 Kerberos 数据库的拷贝，这可以认为是执行过程中的细节问题，并且会不断改变以适应各种不同数据库技术。

Kerberos 又指麻省理工学院为这个协议开发的一套计算机网络安全系统。系统设计上采用客户端 / 服务器结构与 DES 加密技术，并且能够进行相互认证，即客户端和服务器端均可对对方进行身份认证。可以用于防止窃听、防止 replay 攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。Kerberos 的扩展产品也使用公开密钥加密方法进行认证。
[返回](#return_from_Kerberos)

--- 


<span id = "SIP" ></span>
## SIP
### SIP （会话发起协议）
SIP(Session Initiation Protocol) 是一个应用层的信令控制协议。用于创建、修改和释放一个或多个参与者的会话。这些会话可以是 Internet 多媒体会议 、IP 电话或多媒体分发。会话的参与者可以通过组播（multicast）、网状单播（unicast）或两者的混合体进行通信。

SIP 与负责语音质量的资源预留协议 (RSVP) 互操作。它还与若干个其他协议进行协作，包括负责定位的轻型目录访问协议 (LDAP)、负责身份验证的远程身份验证拨入用户服务 (RADIUS) 以及负责实时传输的 RTP 等多个协议。

### 压缩机制
SIP 压缩机制主要是通过改变 SIP 消息的长度来降低时延。典型的 SIP 消息的大小由几百到几千字节，为了适合在窄带无线信道上传输，IMS 对 SIP 进行了扩展，支持 SIP 消息的压缩。当无线信道一定时， 一条 SIP 消息所含帧数 k 仅取决于消息大小。从时延模型可以看出，不仅影响 SIP 消息传输时延， 还影响 SIP 重传的概率， 对自适应的定时器来说，k 还成了影响定时器初值的关键因素。

### 应用
google 发布世界上首个开源的 Html5 sip 客户端
HTML5 SIP 客户端是一款开源的，完全利用 JavaScript 编写的集社交 (FaceBook，Twitter，Google+)，在线游戏，电子商务等应用于一体。无扩展，无插件或是必备的网关，视频堆栈技术依赖于 WebRTC。如同主页里的 Demo 视频演示，你可以轻松实现 Chrome 和 IOS/Android 移动设备之间的实时视频 / 音频通话。
该客户端是一项在浏览器中可被用来连接任意 SIP 或者 IMS 网络进行拨打和接收音频 / 视频通话及即时信息技术。该协议解析器 (SIP，SDP...) 通过使用 Ragel 查找表进行了高度优化，很适合硬件（内存和运算能力）受限的嵌入式系统使用。

[返回](#return_from_SIP)

---
<span id ="XMPP"></span>
## XMPP( 可扩展通讯和表示协议 )
简介: 
可扩展通讯和表示协议 (XMPP) 可用于服务类实时通讯、表示和需求响应服务中的 XML 数据元流式传输。XMPP 以 Jabber 协议为基础，而 Jabber 是即时通讯中常用的开放式协议。XMPP is the IETF's formalization of the base XML streaming protocols for instant messaging and presence developed within the Jabber open-source community in 1999
XMPP（可扩展消息处理现场协议）是基于可扩展标记语言（XML）的协议，它用于即时消息（IM）以及在线现场探测。它在促进服务器之间的准即时操作。这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息，即使其操作系统和浏览器不同。
XMPP 的前身是 Jabber，一个开源形式组织产生的网络即时通信协议。XMPP 目前被 IETF 国际标准组织完成了标准化工作。标准化的核心结果分为两部分；

 - 核心的 XML 流传输协议

 - 基于 XMLFreeEIM 流传输的即时通讯扩展应用

XMPP 的核心 XML 流传输协议的定义使得 XMPP 能够在一个比以往网络通信协议更规范的平台上。借助于 XML 易于解析和阅读的特性，使得 XMPP 的协议能够非常漂亮。
XMPP 的即时通讯扩展应用部分是根据 IETF 在这之前对即时通讯的一个抽象定义的，与其他业已得到广泛使用的即时通讯协议，诸如 AIM，QQ 等有功能完整，完善等先进性。
XMPP 的扩展协议 Jingle 使得其支持语音和视频。
XMPP 的官方文档是 RFC 3920.
[返回](#return_from_XMPP)

---
<span id = "Google-talk"></span>
## Google-talk
### 简介
Google Talk 是 Google 的 IM 工具，除了具有 IM 功能外，另外还加上了 Voip 功能，界面清新大方，可直接链接 Gmail，接受查看邮件。

### 功能
Google Talk 的一个优势是，它能够与其它即时通讯软件服务进行连接。由于 Google Talk 是基于 Jabber 开源标准，这种标准允许用户和其它的即时讯息系统相连，比如苹果电脑的 iChat，GAIM，Trillian Pro 以及 Psi。Google Talk 只能够在 Windows 平台上运行。如果要进行语音通话，用户需要配备麦克风与音箱。

Google Talk 的用户无法使用这种软件与 AIM，MSN Messenger 或者雅虎 Messenger 的用户进行互通。

在使用了 Google Talk 之后，Sullivan 对这种软件的声音质量给了高分，但是，Google Talk 缺乏视频聊天功能，另外， Google Talk 还缺乏目录索引以及聊天文本的搜索功能。

[返回](#return_from_Google-talk)

---
<span id = "ISC"></span>
## ISC
### 简介
ISC 服务器控制。是 Intel 的服务器管理软件。只适用于使用 Intel 架构的带有集成管理功能主板的服务器。采用这种技术后，用户在一台普通的客户机上，就可以监测网络上所有使用 Intel 主板的服务器，监控和判断服务器的工作状态是否正常。一旦服务器内部硬件传感器进行实时监控或第三方硬件中的任何一项出现错误，就会报警提示管理人员。并且，监测端和服务器端之间的网络可以是局域网也可以是广域网，可直接通过网络对服务器进行启动、关闭或重新置位，极大地方便了管理和维护工作。

[返回](#return_from_ISC)

---

