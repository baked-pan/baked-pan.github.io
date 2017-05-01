---
title: 利用 git-backup 插件备份hexo数据
date: 2017-04-25 13:14:36
tags: 
- hexo
- 备份
- 插件
categories:
- hexo
- 插件
---

# 前言
hexo文件夹中重要的是`source`跟`scaffolds`，`themes`目录`.gitignore`，`_config.yml`文件，插件目录可以通过命令行重新下载，只要备份了这些便可以恢复原先的内容

# 插件下载

[项目地址.][1]

## 如果hexo版本 2.x.x, 执行:

``` sql
$ npm install hexo-git-backup@0.0.91 --save
```
如果版本 3.x.x, 执行:

``` sql
$ npm install hexo-git-backup --save
```
# 升级插件
如果你使用 --save安装了插件, 执行：

``` sql
$ npm remove hexo-git-backup
$ npm install hexo-git-backup --save
```
# 设置
你应该在 `hexo/_config.yml.`文件中为插件添加设置

``` less
backup:
    type: git
	message: update xxx //可选，diy信息
	theme: coney,landscape  //可选，主题备份
    repository:
       github: git@github.com:xxx/xxx.git,branchName //备份的目的路径，必填branch分支
       gitcafe: git@github.com:xxx/xxx.git,branchName //可选，备份到gitcafe
	   
```
Now you can backup all the blog!

# 用法

``` sql
hexo backup 
```
或者

``` nginx
hexo b
```



# 问题
## 你可能会遇到系统权限问题。

### Error: EISDIR, open it is caused by permission. just do 'sudo hexo b'

``` nginx
sudo hexo b
```
## 恢复
重装系统或者在其他电脑上修改博客时，执行以下不走

 1. 安装 hexo 环境
 2. `git clone `执行克隆
 3. 将 backup（取决于备份时候的设定） 分支复制到 hexo 目录
 4. 执行`hexo -d `, 重新配置账户

  [1]: https://github.com/coneycode/hexo-git-backup.git