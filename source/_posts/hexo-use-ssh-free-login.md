---
title: hexo 使用 SSH 部署
date: 2016-03-14 15:07:19
tags:
---

使用https提交git往往都需要你重复键入用户名和密码，实在太烦，与其这样还不如给配置SSH一劳永逸，下面动手；



## 配置ssh key

先确认你机器上是否配置了SSH
前往 ``~/.ssh`` 目录查看是否有文件，有的话跳过这节

没有的话先成SSH


``` Shell
$ cd ~
$ mkdir .ssh
$ cd .ssh
$ ssh-keygen -t rsa -C "youremail"
# Creates a new ssh key, using the provided email as a label
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```

指定保存的目录，直接回车，保存当前目录


## 配置Github

打开 ``~/.ssh/id_rsa.pub`` 复制 至 **github** [New SSH Key](https://github.com/settings/ssh)

测试

``` Shell
ssh -T git@github.com
```

再配置git提交信息
``` Shell
$ git config --global user.name "yourname"
$ git config --global user.email "youremail"
```

检查一下，是否进行了相应的更改

``` Shell
$ git remote -v
origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
origin  https://github.com/USERNAME/REPOSITORY.git (push)
```

更改https为SSH
``` Shell
$ git remote set-url origin git@github.com:USERNAME/REPOSITORY2.git
```

记得修改 ``hexo _config.yml`` 文件，仓库地址

``` Xml
deploy:
  type: git
  repository: git@github.com:dxyoo7/dxyoo7.github.io.git       #修改为SSH协议
  branch: master
```


