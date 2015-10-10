---
layout: page
title: "Redmine"
description: ""
---
{% include JB/setup %}

> 环境：Linode, Ubuntu 12.04

# 1. 安装

由于Ubuntu 12.04中的Redmine版本偏低（1.x），因此采用从源代码安装的方法。

## 1.1 下载

下载包见：[下载地址](http://www.redmine.org/projects/redmine/wiki/Download)。


下载redmine-2.5.1.tar.gz，并将其解压缩至：/srv/redmine-2.5.1中。


## 1.2 运行环境

需要Apache，Passenger，Ruby。以下是安装过程：

```bash
$ sudo apt-get install apache2 mysql-server mysql-client ruby1.9.3
```

如果ruby 1.8.7已经安装，可以使用以下命令将其卸载：

```bash
$ sudo apt-get remove ruby
$ update-alternatives ruby
$ sudo rm -rf /var/lib/ruby/1.8/
```

### 1.2.1 安装Passenger

> 参考文档：[Phusion Passenger users guide, Apache version](http://www.modrails.com/documentation/Users%20guide%20Apache.html).




## 1.2 数据库配置

## 1.3 运行

## 1.4 配置Passenger


# 2. 配置


# 3. 插件

## 3.1 redmine dashboard

## 3.2 redmine jenkins


