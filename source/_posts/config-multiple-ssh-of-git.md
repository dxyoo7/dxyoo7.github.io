---
title: 管理git生成的多个ssh key
date: 2016-05-06 14:33:16
tags: git ssh
---

## 问题阐述
当有多个git账号的时候，比如一个github，用于自己进行一些开发活动，再来一个gitlab，一般是公司内部的git。这两者你的邮箱如果不同的话，就会涉及到一个问题，生成第二个git的key的时候会覆盖第一个的key，导致必然有一个用不了。

## 问题解决
我们可以在~/.ssh目录下新建一个config文件配置一下，就可以解决问题

具体步骤
> * 生成第一个ssh key(这里我用于github，用的gmail邮箱)
```Bash
ssh-keygen -t rsa -C "yourmail@gmail.com"
```

这里不要一路回传，让你选择在哪里选择存放key的时候写个名字，比如 id_rsa_github，之后的两个可以回车。
完成之后我们可以看到~/.ssh目录下多了两个文件

> * 生成第二个ssh key（这里我用于gitlab，用的是公司邮箱）
 ```Bash
 ssh-keygen -t rsa -C "yourmail@gmail.com"
 ```

 还是一样不要一路回车，在第一个对话的时候继续写个名字，比如 id_rsa_gitlab,之后的两个可以回车。
完成之后我们可以看到如2中图所标记，一样出现两个文件。（一个公钥一个私钥）

> * 打开ssh-agent
这里如果你用的github官方的bash，'ssh-agent -s',如果是其他的，比如'msysgit,eval $(ssh-agent -s)`

> * 添加私钥

```Bash
 ssh-add ~/.ssh/id_rsa_github
 ssh-add ~/.ssh/id_rsa_gitlab
```

> * 创建并修改config文件

在~/.ssh/目录下新建文件，然后将名字后缀一起改成config即可
在bash下的话直接'touch config' 即可。
添加一下内容

```Bash
# gitlab
Host git.mycompany.com
    HostName git.mycompany.com  //这里填你们公司的git网址即可
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitlab
    User zhangjun

# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
    User ZJsnowman
```

> * 在github和gitlab上添加公钥即可，这里不再多说。

> * 测试

github
'ssh -vT git@github.com'

gitlab
'ssh -vT git@git.mycompany.com'

-v 是输出编译信息，然后根据编译信息自己去解决问题吧。就我自己来说一般是config里的host那块写错了。

## 补充一下
如果之前有设置全局用户名和邮箱的话，需要unset一下

```Bash 
git config --global --unset user.name
git config --global --unset user.email
```

然后在不同的仓库下设置局部的用户名和邮箱
比如在公司的repository下
```Bash
git config user.name "yourname"
git config user.email "youremail"
```
在自己的github的仓库在执行刚刚的命令一遍即可。

这样就可以在不同的仓库，已不同的账号登录。
