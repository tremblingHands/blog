---
layout: post
title: "简单的构建一个自己的博客"
author: "omnihorizon"
subtitle: "git page"
header-img: "img/in-post/post-establish-a-blog/jekyll.png"
tags:
- git
- jekyll

---
1. 在git上创建账号
2. 创建一个repo(名字例如blog)
3. 进入该repo中的setting界面，找到GitHub Pages一栏，选择Source:master branch![]({{site.baseurl}}/img/in-post/post-establish-a-blog/2017-12-26-establish-a-blog-1.png)
4. 生成的地址便是你之后的博客地址![]({{site.baseurl}}/img/in-post/post-establish-a-blog/2017-12-26-establish-a-blog-2.png)
5. 从[jekyllthemes](http://jekyllthemes.org) 中选择一个theme并git到本地
6. 修改’_config.yml’文件
    将baseurl替换成你的repo名字，例如baseurl:"/blog"
    将url替换成你的git的名字，例如url:"https://github.com/tremblingHands”

6. 将本地push到你的git上之前创建的repo上
7. 现在你就可以在”_post”文件下创建你的博文了，通过你的博客地址就可以访问你的博文
