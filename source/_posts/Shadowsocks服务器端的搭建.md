---
title: Shadowsocks服务器端的搭建(请善用返回功能)
date: 2017-04-26 21:59:00
tags: 
- SS
- centos
- 搭建
- 服务器
categories: 
- 服务器
- SS

---
# 一.概要
一个帮你绕过防火墙的迅速的隧道代理.
特性:
 - TCP & UDP support
 - User management API
 - TCP fast open
 - Workers and graceful restart
 - Destination IP blacklist

# 二.服务端
## I安装
### Debian / Ubuntu:

``` 
apt-get install python-pip
pip install shadowsocks
```
### CentOS:

``` 
yum install python-setuptools && easy_install pip
pip install shadowsocks
enter code here
```
### Windows:
不推荐部署在Windows上，因为在Windows上的`select`API非常有限。如果你想要为多个用户提供服务，请选择部署在LINUX上

 1.下载并安装l [Python for Windows][1], 在64位Windows操作系统中可以选择 x86-64 MSI 安装包 .
 2.在这个过程中你需要安装 `pip`
![Shadowsocks_Server_on_Windows.png][2]
 3. 安装 [OpenSSL for Windows][3]. 如果你安装了 **64**位 Python, 你应该选择**64**位 OpenSSL.
 4. 如在 Linux上安装,在命令提示符下,输入命令行
``` 
 pip install shadowsocks 
```
 5.如果你想用 `salsa20` or `chacha20` 加密, 下载 [libsodium][4] 并将所有dll文件(不需要路径)放进  `C:\Windows\System32` or `C:\Windows\SysWOW64 (32bit Python on 64bit Windows)`.


## II使用
``` 
ssserver -p 443 -k password -m aes-256-cfb
```
### 后台运行:
``` 
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
```
### 停止:
``` 
sudo ssserver -d stop
```
### 检查日志:
``` 
sudo less /var/log/shadowsocks.log
```
### 用 `-h` 查看所有参数
## II以配置文件使用(建议)
### 创造配置文件并运行
<span id = "config.json"></span>
#### 创造配置文件 /etc/shadowsocks.json
 - 单用户模板
``` 
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
 - 多个用户模板
``` 
{
"server":"my_server_ip",
"port_password":{
     "8381":"password1",
     "8382":"password2",
     "8383":"password3",
     "8384":"password4"
     },
"timeout":300
"method":"aes-256-cfb",
"fast_open":false,
"workers":1
}
```
<span id = "return_from_Encryption"></span>
<span id = "return_from_fast_open"></span>
各个字段的解释:

| Name          | Explanation                                       |
| ------------- | ------------------------------------------------- |
| server        | 监听的主机地址                                    |
| server_port   | 服务端口                                          |
| local_address | 监听的本地地址                                    |
| local_port    | 本地端口                                          |
| password      | 密码(提供给加密)                                  |
| timeout       | 超时                                              |
| method        | 加密方式，预置: "aes-256-cfb"，见[III加密格式](#Encryption) |
| fast_open     | 使用[TCP_FASTOPEN](#fast_open), true /false                     |
| workers       |          number of workers, available on Unix/Linux                                         |

#### 前台运行
``` 
ssserver -c /etc/shadowsocks.json
```
#### 后台运行
``` 
ssserver -c /etc/shadowsocks.json -d start //启动服务
ssserver -c /etc/shadowsocks.json -d stop  //关闭服务
```


### 开始服务:
``` 
ssserver -c /etc/shadowsocks.json //前台运行
```
<span id = "Encryption"></span>
## III加密格式
支持的密码类

|                        | Python | libev | Go  | Qt  |
| ---------------------- | ------ | ----- | --- | --- |
| SSL library (AES, etc) | Y      | Y     | Y   | Y   |
| RC4-MD5                | Y      | Y     | Y   | Y   |
| Salsa20, Chacha20      | Y      | Y     | Y   | Y   |
### M2Crypto
使用`M2Crypto`会稍微加快加密过程
Debian:
``` 
apt-get install python-m2crypto
```
CentOS:
``` 
yum install m2crypto
```
### rc4-md5
`rc4-md5` 是安全，快速的使用不同的值链接加密方式. 推荐在 OpenWRT路由器上使用.
### salsa20 and chacha20
`salsa20` and `chacha20` 同是快速的序列密码. 在x86_64系统上最优部署的`salsa20` 甚至能获得比`r4`快两倍的速度 (尽管ARM运行较慢).
安装 [libsodium][5] >= 1.0.0 如果你想要使用它们.
``` 
sudo apt-get install build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.8/libsodium-1.0.8.tar.gz
tar xf libsodium-1.0.8.tar.gz && cd libsodium-1.0.8
./configure && make -j2
sudo make install
sudo ldconfig
```
[返回字段的解释处](#return_from_Encryption)

### 弃用的密码
这些旧的密码是慢的或者不安全的，不要使用它们:
 - rc4
 - des-cfb
 - table
 - salsa20-ctr


## IV其他
<span id = "fast_open"></span>
<span id ="return_from_Optimizing_Shadowsocks" ></span>
<span id ="return_from_Feature_Comparison_across_Different_Versions" ></span>
### fast_open 
如果你的伺服器和客户程序都部署在 Linux 3.7.1 或者更高版本,你可以打开 fast_open 以获得更低延迟.

首先你在的配置文件 [config.json](#config.json)将设置`fast_open`为 `true`

然后在你的OS上暂时打开`fast open`：
``` 
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```
要永久打开`fast open`, 见 [Optimizing Shadowsocks](#Optimizing_Shadowsocks).
注意: 只有一些[版本](#Feature_Comparison_across_Different_Versions)支持这个特性.

[返回字段解释处](#return_from_fast_open)

<span id = "Feature_Comparison_across_Different_Versions" ></span>
### Here's the page answering questions: does A support B?
#### Servers
|                  | [Python][6] | [libev][7] | [Go][8]  |
| ---------------- | ------ | ----- | --- |
| Fast Open        | Y      | Y     | N   |
| Multiple Users   | Y      | Y     | Y   |
| Management API   | Y      | Y     | N   |
| Workers          | Y      | N     | N   |
| Graceful Restart | Y      | N     | N   |
| ss-redir         | N      | Y     | N   |
| ss-tunnel        | N      | Y     | N   |
| UDP Relay        | Y      | Y     | N   |
| OTA              |  Y      | Y     | Y   |
#### Clients

|                    | [Windows][9] | [ShadowsocksX][10] | [Qt5][11] | [Android][12] | [iOS App Store][13] | [iOS Cydia][14] |
| ------------------ | ------- | ------------ | --- | ------- | ------------- | --------- |
| System Proxy       | Y       | Y            | N   | Y       | N             | Y         |
| CHNRoutes          | Y       | Y            | N   | Y       | Y             | Y         |
| PAC Configuration  | Y       | Y            | N   | N       | N             | N         |
| Profile Switching  | Y       | Y            | Y   | Y       | N             | Y         |
| QR Code Scan       | Y       | Y            | Y   | Y       | Y             | Y         |
| QR Code Generation | Y       | Y            | Y   | Y       | N             | Y         |

[返回fast_open处 ](#return_from_Feature_Comparison_across_Different_Versions)

<span id = "Optimizing_Shadowsocks"></span>
### Optimizing Shadowsocks
如果你在日志中看到大量`error: too many open files`, 你应该优化你的系统. 这个引导应用到所有shadowsocks 服务器 (Python, libev, etc).

#### 在 Debian 7
新建 `/etc/sysctl.d/local.conf` 并包含以下内容:
``` 
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla

# for low-latency network, use cubic instead
# net.ipv4.tcp_congestion_control = cubic
```
然后
``` 
sysctl --system
```
对于旧的操作系统：
``` 
sysctl -p /etc/sysctl.d/local.conf
```
<span id = "return_from_Configure_Shadowsocks_with_Supervisor"><span>
注意: **不要启用** `net.ipv4.tcp_tw_recycle`!!! 见 [这个文章][15].
如果你使用[Supervisor](#Configure_Shadowsocks_with_Supervisor),确保你在`/etc/default/supervisor`有如下行  . 一旦你添加了这一行, 重启 Supervisor (`service stop supervisor &&  service start supervisor`).

``` 
ulimit -n 51200
```
如果你用其他方式在后台运行 shadowsocks , 确保你在init 脚本添加了 `ulimit -n 51200` .
优化后，一个执行千计连接的繁忙的Shadowsocks服务器 占用大概30MB 内存 和 10% CPU. 请注意在同样的情况下**Linux kernel 经常使用 >100MB RAM** 去处理 buffer 和 cache. 在使用了上述的 sysct 设置, 你用RAM换取了速度. 如果你想要使用更少的RAM，减少 rmem 和 wmem 的大小.

[返回 fast_open 处](#return_from_Optimizing_Shadowsocks)

<span id = "Configure_Shadowsocks_with_Supervisor"></span>

### Supervisor 运行 Shadowsocks
说明： 从 Shadowsocks 2.6 开始，你可以直接在后台运行 Shadowsocks，无需 Supervisor 。 这样省掉了 Supervisor 进程占用的内存。

``` stylus
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```
对于老版本：

编辑 `/etc/shadowsocks.json`

``` json
{
    "server":"0.0.0.0",
    "server_port":7325,
    "local_port":1080,
    "password":"my password",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
记得改密码和服务端端口，不要用默认的。

执行

``` vim
apt-get update
apt-get install python-pip python-m2crypto supervisor
pip install shadowsocks
```
编辑 `/etc/supervisor/conf.d/shadowsocks.conf`

``` ini
[program:shadowsocks]
command=ssserver -c /etc/shadowsocks.json
autorestart=true
user=nobody
```
如果端口 < 1024，把上面的 user=nobody 改成 user=root。

在 `/etc/default/supervisor` 最后加一行：

``` 
ulimit -n 51200
```

执行

``` 
service supervisor start
supervisorctl reload
```
如果遇到问题，可以检查日志：

``` 
supervisorctl tail -f shadowsocks stderr
```
如果修改了 shadowsocks 配置 `/etc/shadowsocks.json`， 可以重启 shadowsocks：

``` 
supervisorctl restart shadowsocks
```
如果修改了 Supervisor 的配置文件 `/etc/supervisor/`*， 可以更新 supervisor 配置：

``` 
supervisorctl update
```

[返回Optimizing Shadowsocks处](#return_from_Configure_Shadowsocks_with_Supervisor)










## V文档
你可以在 [Wiki][17].上找到所有原版文档
## VI开源许可证
Copyright 2015 clowwindy
Licensed under the [Apache License][18]; you may not use this file except in compliance with the License. You may obtain a copy of the License at
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.



## VII后记
在腾讯云上搭建的时候遇到了端口初始化失败的问题，后来发现`"server"`这一行填写的必须是内网 IP. 



  [1]: https://www.python.org/downloads/windows/
  [2]: Shadowsocks服务器端的搭建/Shadowsocks_Server_on_Windows.png
  [3]: https://slproweb.com/products/Win32OpenSSL.html
  [4]: http://download.libsodium.org/libsodium/releases/
  [5]: https://github.com/jedisct1/libsodium
  [6]: https://github.com/shadowsocks/shadowsocks
  [7]: https://github.com/shadowsocks/shadowsocks-libev
  [8]: https://github.com/shadowsocks/shadowsocks-go
  [9]: https://github.com/shadowsocks/shadowsocks-csharp
  [10]: https://github.com/shadowsocks/shadowsocks-iOS
  [11]: https://github.com/shadowsocks/shadowsocks-qt5
  [12]: https://github.com/shadowsocks/shadowsocks-android
  [13]: https://github.com/shadowsocks/shadowsocks-iOS
  [14]: https://github.com/linusyang/MobileShadowSocks
  [15]: http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html
  [17]: https://github.com/shadowsocks/shadowsocks/wiki
  [18]: https://choosealicense.com/licenses/apache-2.0/