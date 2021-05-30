---
layout: post
title: Rails 服务器配置预估
date: 2021-05-30 16:04
---

工作中有新的开发项目开始时，经常会被问及的问题就是服务器需要购买怎样的配置。

## 最低硬件配置

一般来说部署一个正式环境的 Rails app，需要的最低硬件配置如下：

- 1 CPU
- 1 GB 内存（需同时配置 2 GB 虚拟内存）
- 10 GB 硬盘空间
- 64 位 Linux 操作系统

## 一些性能测试结论

根据写于 2017 年的 [Rails Speed with Ruby 2.4.0 and Current Discourse](https://engineering.appfolio.com/appfolio-engineering/2017/5/22/rails-speed-with-ruby-240-and-discourse-180) 和 2021 年的 [How Fast is Ruby 3 on Rails?](https://www.fastruby.io/blog/rails/ruby/performance/how-fast-is-ruby-3-on-rails.html) 这两篇文章，跑在 Ruby 2.6、2.7、3.0 这几个版本下的 [Discourse](https://www.discourse.org/)，在单个 Amazon EC2 m4.2xlarge 实例（4 vCPU / 16 GB Mem）配置下，RPS 为 170 左右。

至于带宽，我之前在某个网页上看到以下经验之谈，现在找不到出处了。

> 对于普通的资讯类网站，最低带宽购买 5 Mbps 才能满足基本需求。

## 响应时间

普通业务的接口响应时间在 200 ms 左右是可以接受的。

## 小型网站架构

一般开始一个严肃的商业网联网项目时，服务构架与配置如下：

- 云服务器，配置为 2 CPU、4 GB 内存、20 GB 硬盘空间、4 Mbps 带宽，用于运行 NGINX、Rails 等应用（一年费用约 6000）
- 云数据库，配置 1 CPU、2 GB 内存，用于数据用户业务存储（一年费用约 2500）
- 云缓存（Redis），配置为 2 GB 单机版，用于数据缓存（一年费用约 1200）
- 对象存储，用于存储文件、图片、视频、音频等数据（流量不大的情况下也就两三百元）

基本一年的服务器相关支出为 1 万元。根据上述两篇文章的性能测试结论，推测这个小型网站配置处理的能力约为 60 RPS。如果业务量上去了，那就升级服务器配置好了。

## 评估思路

不同种类应用，用户的访问行为会有不同。假设一个 30 万用户的电商网站，80% 的用户集中在每天的 20 ~ 22 时访问，那平均 RPS 为 33，那购买上述小型网站的服务器配置就足够日常运营需求了。

**-EOF-**
