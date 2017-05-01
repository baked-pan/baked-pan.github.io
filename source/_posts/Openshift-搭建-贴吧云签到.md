---
title: Openshift 搭建 - 贴吧云签到
date: 2017-04-30 17:04:09
updated:
tags:
- PHP
categories:
 - PHP
keywords:
- 签到
- PHP
description:
---

# 所需工具
 - WinSCP [官网下载][3]
 - PuTTYGen [官网下载][4], [softpedia][5]
 - Tieba-Cloud-Sign [Github][6]

# 步骤

 1. 注册 [Openshift][7] 账号 
 2. 进入控制台选择`Creat your first application`
 3. 下拉列表选择 PHP 5.4
 4. 填写应用名字跟自定义域名，执行 `create application`创建
 5. 导航菜单进入 Application 页面，进入生成的应用
 6. 依次添加 MySQL5.5，phpmyadmin4.0，记住创建过程中绿框显示数据库管理信息
 7. 点击 Application 页面下方 `Or, see the entire list of cartridges you can add`，添加 Cron1.4
 8. 在 Cartridges 列表中， MySQL 5.5 显示`Database: xx User: xx Password: show`,点击 `show` 显示密码，并使用这个账户登录 phpmyadmin( phpMyAdmin 4.0 右方的箭头 )
 9. 记住右上方 `数据库服务器 `的详细信息
 10. 用puttygen 生成 `公钥/私钥` 对,保存私钥到文件中，复制框中的公钥，进入导航菜单进入 Settings  页面,在 `Public Keys`列表下点击 `Add a new key...`，将复制框中的公钥粘贴进去
 11. 回到 Application 页面，进入生成的应用，点击右方链接 `Want to log in to your application?`并复制框中的字串
 12. 在 winscp 中见字串粘贴到主机名中，删去下方的用户名前的`ssh`
 13. 点击高级-SSH-密钥交换，载入第10步保存的私钥文件
 14. zip 下载整个代码，解压
 15. 然后进入 winscp，左边打开你解压的文件夹，右边依次打开 app-root/runtime/repo
 16. 勾选解压的内容物，上传
 17. 进入 Application 页面，点击右上角刷新图标重启应用
 18. 进入应用域名(https方式)
 19. 安装应用的过程中提示页面显示`你是在应用引擎或者不可写的主机上使用本程序吗？`时候**选择**`不，我不是`
 20. 填写数据库配置
 21. 设置 cron 计划任务：winscp 进入`进入 app-root-runtime-repo-.openshift-cron-minutely
在 winscp 中路径会显示为 / var/lib/openshift/*********/app-root/runtime/repo/.openshift/cron/minutely`, 新建一个 `xx.sh` 文件 ，填入 `php $OPENSHIFT_REPO_DIR/do.php`
 22. 进应用域名(https方式)，按照引导进行设置


  [1]: http://www.chiark.greenend.org.uk/~sgtatham/putty/
  [2]: https://github.com/larryli/PuTTY/releases
  [3]: https://winscp.net/eng/docs/lang:chs
  [4]: http://puttygen.software.informer.com/0.6/
  [5]: http://www.softpedia.com/get/Network-Tools/Misc-Networking-Tools/PuTTY-Key-Generator.shtml
  [6]: https://github.com/MoeNetwork/Tieba-Cloud-Sign
  [7]: https://www.openshift.com/