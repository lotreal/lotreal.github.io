---
layout: post
title:  "自制 Vagrant Box"
date:   2015-12-18 21:00:00
---

第一次 [Vagrant] Up 时, 会以 Vagrantfile 里配置的 [Vagrant Box] 为蓝本克隆一个新的虚拟机。

{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "leblanc"
end
{% endhighlight %}

本文介绍如何自制一个名为 **leblanc** 的 Centos 7 Vagrant Box。

## 0x01 准备
1. 安装 Vagrant, VirtualBox：
```
brew cask install vagrant virtualbox
```

2. 打开 VirtualBox, 新建 **leblanc** 虚拟机：

		VirtualBox settings changed from default
			NAME: leblanc
			Type: Linux
			Version: Red Hat 64 bit
			Motherboard->Base Memory: 1024
			Disabled Audio
			Network->Adapter1->NAT
			Hard drive size: 40 GB dynamic

3. 启动 leblanc 并安装 Centos 7。

## 0x02 打包

{% highlight bash %}
# 打包 ./package.box
vagrant package --base leblanc
# 将 ./package.box 导入 vagrant 并命令为 leblanc
vagrant box add leblanc ./package.box

# 列出现有的 vagrant box
vagrant box list
{% endhighlight %}

现在就可以在 Vagrantfile 里使用 leblanc 了。

## 0x03 Q&A

Q: 如何更新 leblanc(vagrant box) ，比如加入一个软件？

A: 只需启动 leblanc 虚拟机，登入所需操作后，重复 0x02 的打包操作即可。

Q: vagrant up 时，报 SSH 无法登陆错误。

A: 参考 [Vagrant SSH Settings] 设置 ssh 相关 username, private_key_path 或 password。

[Vagrant]: https://www.vagrantup.com/
[Vagrant Box]: https://docs.vagrantup.com/v2/getting-started/boxes.html
[Vagrant SSH Settings]: https://docs.vagrantup.com/v2/vagrantfile/ssh_settings.html
