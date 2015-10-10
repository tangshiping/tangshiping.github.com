---
layout: page
title: "Jenkins With Grails"
description: ""
---
{% include JB/setup %}

> 环境：Linode，Ubuntu 12.04

# 1. 安装JDK

> 参考：[How To Install Java on Ubuntu with Apt-Get | DigitalOcean](https://www.digitalocean.com/community/articles/how-to-install-java-on-ubuntu-with-apt-get)

以安装Oracle JDK 7为例。

```bash
$ sudo apt-get install python-software-properties
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java7-installer
```

## 1.1 选择JDK版本

```
sudo update-alternatives --config java
```

如果安装多个JDK版本，将显示以下内容：

```bash
There are 2 choices for the alternative java (providing /usr/bin/java).

Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      auto mode
  1            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      manual mode
  2            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      manual mode

Press enter to keep the current choice[*], or type selection number:
```

对javac也可以同样选择默认版本：

```bash
$ sudo update-alternatives --config javac
```

## 1.2 设置`JAVA_HOME`变量

可以使用`sudo update-alternatives --config java`来得到安装的JDK版本及所在路径。

对于Oracle JDK 7的路径是：`/usr/lib/jvm/java-7-oracle`。

修改`/etc/environment`文件，加入以下内容：

```
JAVA_HOME="/usr/lib/jvm/java-7-oracle"
```

执行`source /etc/environment`使其生效，并用`echo $JAVA_HOME`来验证设置是否生效。

# 2. 安装GVM

> 参考：[GVM by gvmtool](http://gvmtool.net/)

在Jenkins中不能使用GVM安装的Grails，但是作为开发环境管理，还是很有用的。

安装命令：

```bash
$ curl -s get.gvmtool.net | bash
```

之后按照提示信息操作即可。

## 2.1 安装Groovy

安装命令：

```bash
$ gvm install groovy
```

即可安装最新版本。

## 2.2 安装Grails

安装命令：

```bash
$ gvm install grails
```

# 3. 安装Jenkins

> 参考：[Installing Jenkins on Ubuntu - Jenkins - Jenkins Wiki](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)

## 3.1 安装

```bash
$ wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt-get update
$ sudo apt-get install jenkins
```

## 3.2 升级

```bash
$ sudo apt-get update
$ sudo apt-get install jenkins
```

## 3.3 相关信息

- Jenkins被daemon启动，启动脚本见`/etc/init.d/jenkins`
- 日志文件：`/var/log/jenkins/jenkins.log`
- 配置参数：`/etc/default/jenkins`
- 监听端口：8080

# 4. 安装Tomcat7

作为应用服务器使用。

```bash
$ sudo apt-get install tomcat7
```

- 相关的配置文件：`/etc/default/tomcat7`
- 应用位置：`/var/lib/tomcat7/webapps`
- 实例相关的配置文件：`/etc/tomcat7/server.xml`

修改server.xml，将端口改为`8081`。

## 配置Apache Proxy

操作步骤：

```bash
$ sudo a2enmod proxy
$ sudo a2enmod proxy_http
$ sudo a2dissite default
```

在`/etc/apache2/sites-available`目录中创建文件`tomcat7`，内容如下：

```
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	ServerName ci.company.com
	ServerAlias ci
	ProxyRequests Off
	<Proxy *>
		Order deny,allow
		Allow from all
	</Proxy>
	ProxyPreserveHost on
	ProxyPass / http://localhost:8080/
</VirtualHost>
```

继续:

```bash
$ sudo a2ensite tomcat7
$ sudo service apache2 reload
```

如需要禁用某个站点：

```bash
$ sudo a2dissite tomcat7
$ sudo service apache2 reload
```

**问题记录：**

使用`sudo service tomcat7 start`时，有时会提示未设置`JAVA_HOME`，可以直接修改`/etc/default/tomcat7`，将`JAVA_HOME`的正确值写入。

# 5. mysql和phpmyadmin

## 5.1 mysql安装与配置

命令：

```bash
$ sudo apt-get install mysql-server
```

配置文件：`/etc/mysql/my.cnf`

可运行`mysql_secure_installation`或`dpkg-reconfigure mysql-server-5.5`来修改root密码。

调优：MySQL Tuner

```bash
$ sudo apt-get install mysqltuner
$ mysqltuner
```

### 设置用户和数据库

以`root`登录后，执行以下命令：

```bash
$ mysql -uroot -p
mysql>create database zhiji_dashboard_test;
mysql>create database zhiji_dashboard_prod;
mysql>grant all privileges on zhiji_dashboard_test.* to zhiji@localhost identified by 'zhiji';
mysql>grant all privileges on zhiji_dashboard_prod.* to zhiji@localhost identified by 'zhiji';
```

建立两个数据库，分别用于测试和生产环境，同时创建名为`zhiji`，密码为`zhiji`的用户对这两个数据库进行维护。

## 5.2 phpmyadmin

安装：

```bash
$ sudo apt-get install phpmyadmin
```

相关的配置文件：

- /etc/phpmyadmin/config.inc.php
- /etc/dbconfig-common/phpmyadmin.conf

执行`sudo dpkg-reconfigure phpmyadmin`即可重新生成后一个配置文件。

## 5.3 访问

[默认的访问地址](http://106.187.100.138/phpmyadmin)

但是由于在tocmat7的配置文件中已经设置了转发，因此需要修改该文件，在`ProxyPreserveHost on`后加上：

```
ProxyPass /phpmyadmin !
```

表示对该路径的访问将不被apache转发。

# 6. Jenkins与Redmine的集成

可以从两个方向进行，即在Jenkins中集成Redmine，也可以在Redmine中集成Jenkins，下面分别说明：

## 6.1 在Jenkins中集成Redmine

首先安装插件`Redmine-plugin`, 并在全局设置中配置redmine信息。

然后在Job中配置Redmine的项目。

在git提交的信息中插入以下内容：


```
refs nnn
references nnn
IssueID nnn
fixes nnn
closes nnn
#nnn
refs nnn,nnn,nnn
```

可以在“变更记录”中链接到Redmine中。






