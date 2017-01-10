---
title: 使用hexo，如果换了电脑怎么更新博客
date: 2016-09-22 11:37:54
tags: hexo
---

#使用hexo，如果换了电脑怎么更新博客

当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：

1. 使用git clone git@github.com:{username}/{username}.github.io.git拷贝仓库；
2. 切换源文件分支，（因为hexo使用master管理存放生成的page文件，我使用的hexo分支管理源文件）所以我要切换回hexo分支。
3. npm install hexo、npm install、npm install hexo-deployer-git（记得，不需要hexo init这条指令）。
