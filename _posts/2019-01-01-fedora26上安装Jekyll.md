---
layout: post
title:  "在fedora26上安装Jekyll"
date:   2019-01-01 00:00:00
categories: common
permalink: /archivers/jekyll
---

经历一番曲折，总算在fedora26上装上了Jeklly

<!--more-->

```
gem sources -r https://rubygems.org/
// 注意这个网址，几经更改过了。
gem sources -a https://gems.ruby-china.com
// 查看是否更改成功
gem sources -l
// 先安装缺少的组件
yum install ruby-devel
yum install gcc-c++
gem install jekyll-paginate
gem install bundler
gem install minima
// 新建一个myblog项目
jekyll new myblog
cd myblog
// 加上host，可以在虚拟机外部访问
jekyll serve --watch --host=0.0.0.0
```

