---
layout: post
title:  "fpcdm/dockerfiles"
date:   2015-11-06 21:58:00
---

+ centos:centos7
  - lotreal/base
    - lotreal/consul
      - lotreal/apache-php
        - lotreal/codeigniter-app


* lotreal/base
- 使用重庆时区
- 安装 epel-release, remi
- 安装 supervisor, bind-utils, yum-cron 等
- 添加 azir 私钥
- 添加 supervisord.conf
- 添加 start 脚本
- CMD ["/usr/local/bin/start"]
