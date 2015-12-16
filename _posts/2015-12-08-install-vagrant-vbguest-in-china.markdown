---
layout: post
title:  "解决中国局域网中 vagrant plugin install vagrant-vbguest 出错的问题"
date:   2015-12-08 21:00:00
---

{% gist 1e16a3fe24191ec86137 %}

升级 VirtualBox 或 Vagrant 虚拟机时，常遇到 VirtualBox guest additions **版本不一致**，导致 vagrant 启动失败的问题。
解决方式就是安装合适的 vbguest additions 版本。使用 vagrant 插件 **vagrant-vbguest** 可以**自动处理**这个问题，非常方便。
安装方法是：
```
vagrant plugin install vagrant-vbguest
```

但是，根据中国的相关法律法规，gem 不予下载，连接会被重置。就算在 $HOME/.gemrc 里设置了国内镜像，都无法解决这个问题。

这是因为 vagrant 的插件使用了单独的 ruby 环境。而使用[上面的脚本](https://gist.github.com/lotreal/1e16a3fe24191ec86137)，把所依赖的 gem 下载到 vagrant 的 gem 缓存里，就可以成功安装这个插件了。
