---
layout: post
title:  "Github Pages + Jekyll = blog"
date:   2015-12-10 20:52:00
---

使用 [Github] 的 [Github Pages] 特性，可以快速建站。使用方法为：

* 个人网站：在 [Github] 上建 username.github.io 库，提交网站内容到 master 分支；
* 项目网站：提交网站内容到项目的 gh-pages 分支。

## 开始

{% highlight bash %}
# 安装时如遇“有关法规”问题，可使用 https://ruby.taobao.org 镜像
gem install github-pages

# 生成新 jekyll 站
jekyll new lotreal.github.io

# 提交内容到 Github
cd lotreal.github.io
git init .
git add .
git commit -m "jekyll new lotreal.github.io"
git remote add origin git@github.com:lotreal/lotreal.github.io.git
git push -u origin master
{% endhighlight %}

## 完成
访问 http://lotreal.github.io/ 即可

> 另：在 lotreal.github.io 目录下使用命令：jekyll serve --watch 可在本地预览网站效果。

### 相关链接

* [Jekyll Documnetation](http://jekyllrb.com/docs/home/)
* [Using Jekyll with Pages](https://help.github.com/articles/using-jekyll-with-pages/)
* [Markdown Basic](http://daringfireball.net/projects/markdown/basics)
* [可用代码高亮语言列表](http://pygments.org/docs/lexers/)

[Github]:       https://github.com
[Github Pages]: https://pages.github.com
