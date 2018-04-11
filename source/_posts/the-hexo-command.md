---
title: Hexo 的简单命令
date: 2016-03-14 11:47:10
tags: Hexo
---

## hexo
``` Shell
npm install hexo -g #安装  
npm update hexo -g #升级  
hexo init #初始化
```
## 简写
``` Shell
hexo n "我的博客" == hexo new "我的博客" #新建文章s
hexo p == hexo publish
hexo g == hexo generate#生成
hexo s == hexo server #启动服务预览
hexo d == hexo deploy#部署
```

## 服务器
``` Shell
hexo server #Hexo 会监视文件变动并自动更新，您无须重启服务器。
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP


hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
```
## 监视文件变动
``` Shell
hexo generate #使用 Hexo 生成静态文件快速而且简单
hexo generate --watch #监视文件变动
```

## 完成后部署
``` Shell
两个命令的作用是相同的
hexo generate --deploy
hexo deploy --generate
hexo deploy -g

hexo server -g
```
## 草稿
``` Shell
hexo publish [layout] <title>
```

## 模版
``` Shell
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub

hexo new [layout] <title>
hexo new photo "My Gallery"
hexo new "Hello World" --lang tw
```

|变量	|描述
|:----  |:---
|layout	|布局
|title	|标题
|date	|文件建立日期


``` Shell
title: 使用Hexo搭建个人博客
layout: post
date: 2014-03-03 19:07:43
comments: true
categories: Blog
tags: [Hexo]
keywords: Hexo, Blog
description: 生命在于折腾，又把博客折腾到Hexo了。给Hexo点赞。
```

## 模版（Scaffold）
``` Shell
hexo new photo "My Gallery"
```

|变量	|描述
|:----- |:----
|layout	|布局
|title	|标题
|date	|文件建立日期


## 写作

``` Shell
hexo new page <title>
hexo new post <title>
```

|变量	|描述
|:------|:------
|:title	|标题
|:year	|建立的年份（4 位数）
|:month	|建立的月份（2 位数）
|:i_month	|建立的月份（去掉开头的零）
|:day	|建立的日期（2 位数）
|:i_day	|建立的日期（去掉开头的零）


## 推送到服务器上

``` Shell
hexo n #写文章
hexo g #生成
hexo d #部署 #可与hexo g合并为 hexo d -g
```

## 报错
1.找不到git部署

``` Shell
ERROR Deployer not found: git
```

解决方法
``` Shell
npm install hexo-deployer-git --save
```

## 3.部署类型设置git
hexo 3.0 部署类型不再是**github**，**_config.yml** 中修改

# Deployment
``` Shell
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@***.github.com:***/***.github.io.git
  branch: master
```

## 4. xcodebuild

xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
``` Shell
npm install bcrypt
```

## 5. RSS不显示
安装RSS插件

``` Shell
npm install hexo-generator-feed --save
```

开启RSS功能

编辑**hexo/_config.yml**，添加如下代码：

``` Shell
rss: /atom.xml #rss地址  默认即可
```

## 开启评论
1.我使用多说代替自带的评论，在多说 网站注册 > 后台管理 > 添加新站点 > 工具 === 复制通用代码 里面有 short_name

1、在根目录 _config.yml 添加一行  disqus_shortname: jslite 是在多说注册时产生的
2、复制到 themes\landscape\layout\_partial\article.ejs
把

``` JavaScript
<% if (!index && post.comments && config.disqus_shortname){ %>
<section id="comments">
<div id="disqus_thread">
  <noscript>Please enable JavaScript to view the <a href="//disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
</div>
</section>
<% } %>
```
改为
``` JavaScript
<% if (!index && post.comments && config.disqus_shortname){ %>
  <section id="comments">
    <!-- 多说评论框 start -->
    <div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
    <!-- 多说评论框 end -->
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
    var duoshuoQuery = {short_name:'<%= config.disqus_shortname %>'};
      (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
      })();
      </script>
    <!-- 多说公共JS代码 end -->
  </section>
<% } %>
```


