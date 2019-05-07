---
title: PHP+MySql+Apache+WordPress Win10 配置
date: 2016-05-04 22:48:32
tags: 
- "WordPress"
- "PHP"
- "Apache"
- "MySql"
categories: "搭建环境"
---

闲话少说，首先是安装，但是版本的选择很重要。

## PHP 5.6
 
当然是[官网]("http://php.net/downloads.php")啦！

![“php1”](/images/amps1.png)

这里选择5.6的版本，PHP 7太新了，怕兼容性出现问题。点击[Windows downloads]("http://windows.php.net/download#php-5.6")

版本有Thread Safe/Non Thread Safe 和 X86 X64的两种组合，随自己的喜欢了。Non Thread Safe不会检查线程是否安全。这里下载的是Zip格式的压缩包

![“php2”](/images/amps2.png)

解压缩，我的路径是 F:\php5.6 复制份php.ini-development，并改名为PHP.ini，用记事本打开

查找 extension_dir 将

    ;extension=php_mysql.dll
    ;extension=php_mysqli.dll

前面的注释符";"去掉，使PHP支持mysql

## Apache 2.4

点[这里]("http://httpd.apache.org/download.cgi")下载！这里选择2.4版本，因为和PHP 5.6一样都是在vc11下编译的

![“apache1”](/images/amps3.png)

点击 Files for Microsoft Windows，这里有五个版本，第一项ApacheHaus是个第三方下载平台，在它的网站下载独立的Apache压缩包。另外四个中，第二个也是独立的Apache下载地址，另外三个是集成开发环境。这里直接选择了第一个

![“apache2”](/images/amps4.png)

解压缩，我的路径是 F:\Apache24 接下来的工作就是比较主要的了，配置 httpd.conf

- 修改监听端口

如果原有80端口被其他服务占用，可以修改 `Listen 80` 为 `Listen 8081` (例如)

- 修改权限

apache 2.4 不再支持deny/allow 等指令，改用 denied/granted，将denied改为granted，或者注释掉重写。
否则会出现的一个问题是 `You don’t have permission to access / on this server`
如果根目录下没有index文件，也会出现这个问题

```
<Directory />
    AllowOverride none
    # Require all denied
	Require all granted
</Directory>
```

- 修改站点根目录

`DocumentRoot "${SRVROOT}/htdocs"` 改为自己的目录
例如我的 `DocumentRoot "D:/PHPWeb"`

- 支持PHP

在配置文件最后添加以下代码，配置php的module 和 php.ini 的路径

```
# php5 support
LoadModule php5_module "F:/php5.6/php5apache2_4.dll"
AddHandler application/x-httpd-php .php
# configure the path to php.ini
PHPIniDir "F:/php5.6"
```

- 启动Apache

在bin目录下执行httpd.exe，正常情况就是没有信息返回

![“apache3”](/images/amps5.png)

在 `D:/PHPWeb` 内新建 `index.php`, 内容填入以下PHP代码

    <?php Echo "Hello, CJW!";?>

此时可以在浏览器输入localhost:8081/index.php，可以看到打印的结果，此时PHP+Apache已经搭建好

![“apache4”](/images/amps6.png)

## WordPress

点[这里]("https://wordpress.org/download/")下载，我选择zip版本，然后解压缩

[官网教程]("http://codex.wordpress.org.cn/")写的也比较清楚，主要有几个步骤需要注意

- 上传WordPress到一个远程主机

教程里说的这一步，可以把zip解压到根目录里，例如我的 `"D:\PHPWeb\wordpress"`

- 设置 wp-config.php

将 wp-config-sample.php 改名为 wp-config.php，然后就是配置数据库，教程里写的很清楚。

- 测试

在浏览器里输入 `http://localhost:8081/wordpress/` 进行测试，第一次访问是注册界面

如果出现 `You don’t have permission to access / on this server` 检查Apache配置，或者看是否缺少index文件

![“wp”](/images/amps7.png)

## MySql 5.6

安装过程基本一路next，注意填入的用户名和密码要记住，在wp-config.php配置时需要填入