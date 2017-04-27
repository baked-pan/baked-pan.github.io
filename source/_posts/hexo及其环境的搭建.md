---
title: hexo及其环境的搭建(Windows+HTTP版)
date: 2017-04-20 21:59:07
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
 3.  访问`用户名.github.io`进入主页

## Tips 
hexo现在支持更加简单的命令格式

``` stata
hexo g == hexo generate

hexo d == hexo deploy

hexo s == hexo server

hexo n == hexo new

hexo g -d == hexo generate&&hexo deploy
```
更新内容需要执行

``` stata
hexo clean
hexo g -d
```



  [1]: https://git-for-windows.github.io/
  [2]: https://pan.baidu.com/s/1kU5OCOB#list/path=/pub/git
  [3]: hexo及其环境的搭建/Git安装.jpg
  [4]: https://nodejs.org/en/download/
  [5]: hexo及其环境的搭建/Hexo初始.jpg
