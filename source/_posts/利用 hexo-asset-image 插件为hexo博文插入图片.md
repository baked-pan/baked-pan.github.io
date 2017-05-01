---
title: 利用 hexo-asset-image 插件为hexo博文插入图片
date: 2017-04-21 21:31:43
tags: 
- hexo
- 插件
categories:
- hexo
- 插件


---
# 前言
hexo博文上插入图片一直的惯用方法是使用七牛云或者贴图库又或者新浪，但考虑到七牛存储的不直观，贴图库的不稳定跟新浪不知道什么时候会关闭的接口，不如将图片存放在github上边，下边是一个可行的方法

# 开始

 1. 把主页配置文件`_config.yml` 里边的`post_asset_folder:`设置为 `true`
 2. 在 hexo 目录下执行`npm install hexo-asset-image --save`,这里下载安装一个可以上传本地图片的插件，来自[hexo-asset-image][1]
 3. 完成后当运行`hexo n "XX.md"`生成博文模板时，会在`/source/_posts`文件夹中额外生成名字为XX的文件夹

# 正式使用
在XX博文中引入图片时，按照 markdown 的格式引入图片，并将图片放入XX文件夹内，格式为
``` markdown
![你想输入的替代文字](xx/图片名.jpg)
```



  [1]: https://github.com/CodeFalling/hexo-asset-image