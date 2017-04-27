---
title: hexo及其环境的搭建及进阶(Windows+HTTP版)
date: 2017-04-20 21:59:07
update: 2017-04-28 02:00:01
tags: 
- hexo
- 环境
categories:
- hexo
- 开发

---
# 工具 / 原料 

- Windows（Mac 也是差不多，可参照）  
- Git  
- Node.js

## Git安装
下载并安装[Git][1]，[度娘备份][2]，msysgit 是 Windows 版的 Git，按照默认安装。

安装完成后，在开始菜单里找到 “Git”->“Git Bash”，如图表示安装成功
![Git安装][3]

## Node.js安装
下载并安装[Node.js][4],安装完成后在CMD输入  `node -v`显示版本表示安装node.js成功,输入`npm -v` 显示版本表示自带的npm环境安装成功

npm 的作用就是对 Node.js 依赖的包进行管理，也可以理解为用来安装 / 卸载 Node.js 需要装的东西

注：当不显示版本时请添加对应的环境变量并重启系统

## 安装 Hexo

 1. 利用 npm 命令即可安装。在任意位置点击鼠标右键，选择 Git Bash。输入命令`npm install -g hexo`注意：-g是指全局安装hexo。
# 部署hexo
## 创建Hexo文件夹
1、新建文件夹 Hexo 
2、

``` scilab
cd Hexo
hexo init //Hexo 即会自动在目标文件夹建立网站所需要的所有文件。
npm install //安装依赖包
```
## 本地查看 
执行以下命令(在Hexo文件夹下)，然后到浏览器输入localhost:4000看看。

``` vbscript
hexo generate
hexo server
```
![hexo初始][5]

## 注册Github账号 
## 创建Repository仓库
新建`用户名.github.io`的仓库
## 修改配置文件 
1.到刚创建的Repository下，获得SSH地址

2.编辑Hexo根目录下的`_config.yml`

3.修改文件里面的deploy。其中的repository 添加为SSH，如

``` less
deploy:
  type: git
  repo: https://github.com/jasmine-na/jasmine-na.github.io.git
  branch: master
```
## 完成部署

 1. 下载hexo-deployer-git

``` sql
npm install hexo-deployer-git –save
```


 2. 最后一步，Git Bash下，依次键入以下指令

 

``` stata
 hexo clean //清除缓存文件 db.json 和已生成的静态文件 public

hexo generate //生成网站静态文件到默认设置的 public 文件夹

hexo deploy //自动生成网站静态文件，并部署到设定的仓库。
```
hexo deploy过程中会提示输入账号名和密码，Username是Github账号名称，而不是邮箱；Password就是Github的密码
访问`用户名.github.io`进入主页
# 进阶
## Tips 
hexo现在支持更加简单的命令格式

``` stata
hexo g == hexo generate

hexo d == hexo deploy

hexo s == hexo server

hexo n == hexo new

hexo g -d == hexo generate&&hexo deploy
hexo generate --watch  //监视文件变动并立即重新生成静态文件

```
更新内容需要执行:

``` stata
hexo clean
hexo g -d
```
### 如果您想要更改端口，或是在执行时遇到了 `EADDRINUSE` 错误，可以在执行时使用 `-p `选项指定其他端口，如下：
```
hexo server -p 5000

```
### 静态模式
在静态模式下，服务器只处理 `public` 文件夹内的文件，而不会处理文件变动，在执行时，您应该先自行执行 `hexo generate`，此模式通常用于生产环境（production mode）下。
```
hexo server -s
```

### 自定义 IP
服务器默认运行在 0.0.0.0，您可以覆盖默认的 IP 设置，如下：
```
hexo server -i 192.168.1.1

```
定这个参数后，您就只能通过该 IP 才能访问站点。例如，对于一台使用无线网络的笔记本电脑，除了指向本机的`127.0.0.1`外，通常还有一个`192.168.*.*`的局域网 IP，如果像上面那样使用`-i`参数，就不能用`127.0.0.1`来访问站点了。对于有公网 IP 的主机，如果您指定一个局域网 IP 作为`-i`参数的值，那么就无法通过公网来访问站点。
### 相对路径引用的标签插件
通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确。在 Hexo 2 时代，社区创建了很多插件来解决这个问题。但是，随着 Hexo 3 的发布，许多新的标签插件被加入到了核心代码中。这使得你可以更简单地在文章中引用你的资源。
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```
比如说：当你打开文章资源文件夹功能后，你把一个 `example.jpg` 图片放在了你的资源文件夹中，如果通过使用相对路径的常规 markdown 语法` ![](/example.jpg) `，它将 不会 出现在首页上。（但是它会在文章中按你期待的方式工作）
正确的引用图片方式是使用下列的标签插件而不是 markdown ：
```
{% asset_img example.jpg This is an example image %}

```
通过这种方式，图片将会同时出现在文章和主页以及归档页中。比如下边的图片

{% asset_img example.png [hexo及其环境的搭建及进阶(Windows+HTTP版)] %}
这里使用的语句是
```
{% asset_img example.png [hexo及其环境的搭建及进阶(Windows+HTTP版)] %}
```


  [1]: https://git-for-windows.github.io/
  [2]: https://pan.baidu.com/s/1kU5OCOB#list/path=/pub/git
  [3]: hexo及其环境的搭建及进阶(Windows+HTTP版)/Git安装.jpg
  [4]: https://nodejs.org/en/download/
  [5]: hexo及其环境的搭建及进阶(Windows+HTTP版)/Hexo初始.jpg
