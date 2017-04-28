---
title: dnsmasq 的配置文件(本地化配置中)
date: 2017-04-28 14:56:26
updated:
tags:
- dnsmasq
- DNS
categories:
- DNS

---

# dnsmasq 的配置文件(本地化配置中)

## 前言，在命令行格式中我使用了1 / 0 表示，1表示取消当前注释则开启说明所表示的功能，0则相反

## 格式是一个选项一行，合法的多个选项跟在命令行中的多个合格的选项一样。见“/usr/sbin/dnsmasq –help” 或者“man 8 dnsmasq” 里的更多细节。下面的两个选项让你更好地使用网络，因为它们

 - 告诉 dnsmasq 过滤那些公共 DNS 不能处理的不需要装在服务器( 特别是根服务器中 )的请求。如果你有个按需拨号（dial-on-demand）的连接，这些服务器会阻止这些不需要的连接的建立( bring up )。
 - 不转发那些不存在的名( 没有点或者域部分 )

``` 
#domain-needed //1
```
- 不为存在于那些没有根路由（non-routed ）地址的空间转发地址

``` vala
#bogus-priv   //1
```

## 取消这个注释以过滤无用的来源为Windows的DNS请求，这些请求会引起不必要的按需拨号的连接。注意 ( 在其他地方 )，这阻止所有<span id = "return_from_SRV_record"></span> [服务定位资源记录 (SRV )](#SRV_record)，所以除非你使用诸如<span id = "return_from_Kerberos">[Kerberos](#Kerberos), <span id = "return_from_SIP"></span>[SIP](#SIP), <span id = "return_from_XMPP"></span>[XMMP](#XMPP) 或者<span id ="return_from_Google-talk"></span> [Google-talk](#Google-talk) ，不要使用它。这个选项只影响转发,为 dnsmasq 生成的SRV 记录 ( 通过 srv-host= lines ) 不受其限制

``` gcode
#filterwin2k //1
```
## 改变这一行如果你想要从上游服务器中获取dns，上游的设置文件在`/etc/resolv.conf`

``` vala
#resolv-file=   //填入上游文件名
```

## 默认情况下，dnsmasq 会发送请求给任意一个知道回复的上游服务器并设置最优服务器。取消这里的注释以强制dnsmasq 按设置文件/`etc/resolv.conf`中顺序尝试对上游服务器尝试请求

``` vala
#strict-order //1
```
## 如果你不需要dnsmasq 读取`/etc/resolv.conf`或其他`resolv文件`和重读，注释掉这里

``` vala
#no-poll  //1
```
## 在这里添加其他域名服务器，如果是私有域名，添加域名规格

``` 
#server=/localnet/192.168.0.1
```
## 一个路由器选择PTR请求到域名服务器( nameserver  )：这会发送所有对`192.168.3/24`的请求`address->name` 到域名服务器`10.1.2.3`

``` 
#server=/3.168.192.in-addr.arpa/10.1.2.3
```

## 添加仅本地( local-only )域名,对这些域名的请求将仅由 `/etc/hosts` 或者 `DHCP` 回应

``` shell
#local=/localnet/
```

## 添加强制到特定IP的域名。下边的例子发送任何在 `doubleclick-net`到本地服务器

```
#address=/doubleclick.net/127.0.0.1
```
### address或者server与IPv6也是支持的

``` 
#address=/www.thekelleys.org.uk/fe80::20d:60ff:fe36:f83
```

## 你可以控制dnsmasq 与一个服务器的会话方式：
- 这个强制对10.1.2.3的请求通过eth1：`server=10.1.2.3@eth1`
- 这个设置与10.1.2.3的会话的源(ie local  )地址到192.168.1.1，端口（port）55(显然这个机器必须要有与这个IP绑定的接口)：`server=10.1.2.3@192.168.1.1#55`


## 如果你想要dnsmasq改变uid和gid到其他设置，而不使用预设值，编辑下边的行

``` 
#user=

#group=
```
## 如果你想要dnsmasq仅在特定接口（和回路）监听 DHCP 和 DNS 请求，给接口命名（如：`eth0`）,重复行如果你有多个接口

``` vala
#interface=
```

### 或者你设置特定接口不监听

``` stylus
#except-interface=
```
### 或者哪个通过特定地址（address）监听（包括127.0.0.1如果有使用的话）

``` stylus
#listen-address=
```

## 如果你只想在一个接口上提供 DNS 服务，如下设置它，然后使用如下行为它关闭DHCP

``` vala
#no-dhcp-interface=
```
## 在支持的系统上,dnsmasq绑定通配符地址，即使在它只在一些接口上监听，然后它抛弃那些不应该回复的请求。即使在接口不断变化和改变地址的情况下，这仍然有好处.如果你确实要dnsmasq去绑定它正在监听的接口，注释掉这个选择.是在你在于一个机器上运行另一个域名服务器的情况下，你可能不止一次需要它

``` shell
#bind-interfaces
```
## 如果你不想要dnsmasq去读取`/etc/hosts`, 注释掉这一行

``` vala
#no-hosts
```
## 或者你需要其他文件，和` /etc/hosts`，用这个

``` vala
#addn-hosts=/etc/banner_add_hosts
```
## 设置这个（或者下边的域名），如果你想要一个域名自动被添加到hosts-file 的简单名中

``` vala
#expand-hosts
```
## 为dnsmasq设置一个域名，这是可选的，但一旦设置，它执行了如下行为

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

## 注释掉这一行以启用完整的 DHCP 服务，你需要提供地址（addresses）范围（range）以供租借（租借时间是可选的）。
如果你有多个网络，在每个应用DHCP 服务的网络重复这个

``` 
#dhcp-range=192.168.0.50,192.168.0.150,12h
```

## 这是一个拥有电子掩码DHCP 范围的例子。在我们通过一个中继代理（relay agent）到达dnsmasq DHCP 服务的情况下，电子掩码是必须的。如果你并不知道什么是DHCP 中继代理，你大概不需要注意这个

``` stylus
#dhcp-range=192.168.0.50,192.168.0.150,255.255.255.0,12h
```
## 这是一个拥有网络ID（network-id）的DHCP范围的例子，所以一些DHCP选项可以被设置仅对这个网络生效

``` 
#dhcp-range=red,192.168.0.50,192.168.0.150
```
## 使用DHCP 为特定主机应用参数。这里是一些有效的选择，所以我们为每个提供了例子.注意IP 地址**不应该**出现在上述的范围中，它们只需要在同一个网络。参数的顺序不做要求，name,adddress 和 MAC 的任意排序是允许的。
 1. 总是为主机分配以太网地址。
 
- 11:22:33:44:55:66
- The IP address 192.168.0.60 

``` 

#dhcp-host=11:22:33:44:55:66,192.168.0.60
```
 2. 总是用硬件地址为主机设置名

 - 11:22:33:44:55:66 to be “fred”

``` stylus
#dhcp-host=11:22:33:44:55:66,fred
```
 3. 总是给书记以太网地址，名字“fred”，ip地址，租借时间45分


``` stylus
#dhcp-host=11:22:33:44:55:66,fred,192.168.0.60,45m
```

 4. 给一个主机两个以太网地址（用","分隔)和IP地址.dnsmasq会假定这两个以太网接口没有被在同一时间使用，然后把IP地址分配给第二个，尽管这个IP地址已经被第一个接口使用。在使用无线和有线地址的笔记本上十分有用

 
``` 
#dhcp-host=11:22:33:44:55:66,12:34:56:78:90:12,192.168.0.60
```

 5. 给一个声称它的名是“bert"的主机 IP 地址和无限（infinite）租借时间



``` 
#dhcp-host=bert,192.168.0.70,infinite
```
 6. 总是给主机客户机标识符
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



 10.55

 

---
# 其他注释 (  ) ,
<span id = "SRV_record" ></span>
## SRV记录:
### 定义 ：
DNS 服务器的数据库中支持的一种资源记录的类型，它记录了哪台计算机提供了哪个服务这么一个简单的信息。一般是为 Microsoft 的活动目录设置时的应用。

### 简介：
DNS 可以独立于活动目录，但是活动目录必须有 DNS 的帮助才能工作。为了活动目录能够正常的工作，DNS 服务器必须支持服务定位（SRV）资源记录，资源记录把服务名字映射为提供服务的服务器名字。活动目录客户和域控制器使用 SRV 资源记录决定域控制器的 IP 地址。
[返回](#return_from_SRV_record)

### SRV 记录功能包括（基于它们在 DNS 控制台的分组）

 1.‘　_MSDCS。这个分组中，SRV 记录是根据它们的状态来收集的。各种状态包括 DC、域调用、GC 以及 PDC。DC 和 GC 按站点来划分，这样一来，AD 客户端就能快速的知道去哪里寻找本地服务。“域调用” 用于支持复制。每个 DC 都获得了一个 GUID，它会在调用复制时用到。PDC 条目包含了被设定为 PDC 模拟器的 DC 的 SRV 记录。
 2.‘ _SITES。站点代表的是一个高速连接区域，根据 DC 的站点从属关系来建立了 DC 索引之后，客户端就可以检查_SITES 来寻找本地服务，而不必通过 WAN 来发送它们的 LDAP 查询请求。标准 LDAP 查询端口是 389，全局编录查询则使用 3268。
 3.‘　_TCP。在这个分组中，收集了 DNS 区域中的所有 DC。如果客户端找不到它们特定的站点，或者具有本地 SRV 记录的任何 DC 都没有响应，需要寻找网络中其他地方的 DC，就应该将这些客户端放到这个分组中。
 4.‘　_UDP。Keberos v5 允许客户端使用 “无连接” 服务来获取票证并更改密码。这是通过与相同服务的 TCP 端口对应的 UDP 端口来完成的。具体说，票证交换使用 UDP 的 88 端口，而密码更改使用 464。


---

<span id = "Kerberos"><span>
## Kerberos：
### 简介
一种网络认证协议，其设计目标是通过密钥系统为客户机 / 服务器应用程序提供强大的认证服务。该认证过程的实现不依赖于主机操作系统的认证，无需基于主机地址的信任，不要求网络上所有主机的物理安全，并假定网络上传送的数据包可以被任意地读取、修改和插入数据。在以上情况下， Kerberos 作为一种可信任的第三方认证服务，是通过传统的密码技术（如：共享密钥）执行认证服务的。

### 认证过程
具体如下：客户机向认证服务器（AS）发送请求，要求得到某服务器的证书，然后 AS 的响应包含这些用客户端密钥加密的证书。证书的构成为： 1) 服务器 “ticket” ； 2) 一个临时加密密钥（又称为会话密钥 “session key”） 。客户机将 ticket （包括用服务器密钥加密的客户机身份和一份会话密钥的拷贝）传送到服务器上。会话密钥可以（现已经由客户机和服务器共享）用来认证客户机或认证服务器，也可用来为通信双方以后的通讯提供加密服务，或通过交换独立子会话密钥为通信双方提供进一步的通信加密服务。

上述认证交换过程需要只读方式访问 Kerberos 数据库。但有时，数据库中的记录必须进行修改，如添加新的规则或改变规则密钥时。修改过程通过客户机和第三方 Kerberos 服务器（Kerberos 管理器 KADM）间的协议完成。有关管理协议在此不作介绍。另外也有一种协议用于维护多份 Kerberos 数据库的拷贝，这可以认为是执行过程中的细节问题，并且会不断改变以适应各种不同数据库技术。

Kerberos 又指麻省理工学院为这个协议开发的一套计算机网络安全系统。系统设计上采用客户端 / 服务器结构与 DES 加密技术，并且能够进行相互认证，即客户端和服务器端均可对对方进行身份认证。可以用于防止窃听、防止 replay 攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。Kerberos 的扩展产品也使用公开密钥加密方法进行认证。

--- 

[返回](#return_from_Kerberos)
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

<span id = "Google-talk"></span>
## Google-talk
### 简介
Google Talk 是 Google 的 IM 工具，除了具有 IM 功能外，另外还加上了 Voip 功能，界面清新大方，可直接链接 Gmail，接受查看邮件。

### 功能
Google Talk 的一个优势是，它能够与其它即时通讯软件服务进行连接。由于 Google Talk 是基于 Jabber 开源标准，这种标准允许用户和其它的即时讯息系统相连，比如苹果电脑的 iChat，GAIM，Trillian Pro 以及 Psi。Google Talk 只能够在 Windows 平台上运行。如果要进行语音通话，用户需要配备麦克风与音箱。

Google Talk 的用户无法使用这种软件与 AIM，MSN Messenger 或者雅虎 Messenger 的用户进行互通。

在使用了 Google Talk 之后，Sullivan 对这种软件的声音质量给了高分，但是，Google Talk 缺乏视频聊天功能，另外， Google Talk 还缺乏目录索引以及聊天文本的搜索功能。

[返回](#return_from_Google-talk)
