---
title: Install Graphite
description: >
  Install Graphite on ubuntu.
layout: post
categories: distribute
---

在 Ubuntu 12.04 上安装 Graphite 监控工具
=========

2013年11月23日

--------------

Graphite 是一个（可运行在廉价硬件上的）企业级开源监控工具，用于采集服务器实时信息并进行统计，可采集 n 个服务器实时状态，如：用户请求消息，Memcached 命中率，RabbitMQ 消息服务器状态、操作系统负载、等等。Graphite 使用 Python 编写，采用 Django 框架，使用自己的简单文本协议通讯，服务平均每分钟有4800次更新操作，简单的文本协议和强大的绘图功能可以方便地扩展到任何需要监控的系统上。

和其他监控工具不同的是，Graphite 自己本身并不收集具体的数据，这些数据收集的具体工作通常由第三方工具或插件完成（如 Ganglia, collectd, statsd, Collectl 等，参考：使用 collectd 和 Graphite 监控服务器）。所以上面说 “Graphite 是一个系统监控工具” 的说法不完全正确，更准确的说法应该是 “Graphite 是一个数据绘图工具”，得到数据后绘图，它并不关心具体数据，你甚至可以把每天收到的邮件数当作参数传给 Graphite 制成图，然后一段时间后就生产一个漂亮的邮件繁忙展示图，并可以看出趋势。

简单的说，Graphite 做两件事：1、存储数据；2、按需绘图。

安装必要软件包：
--------------

```sh
$ sudo apt-get install apache2 libapache2-mod-wsgi python-django \
python-twisted python-cairo python-pip python-django-tagging
```

用 pip 安装 whisper (简单的存放和操作数据的库), carbon (监控数据的 Twisted 守护进程) 和 graphite-web (Django webapp)：

```sh
$ sudo pip install whisper
$ sudo pip install carbon
$ sudo pip install graphite-web
```
初始化配置，直接用 example 文件里的默认配置就可以：

```sh
$ cd /opt/graphite/conf/
$ sudo cp carbon.conf.example carbon.conf
$ sudo cp storage-schemas.conf.example storage-schemas.conf
$ sudo cp graphite.wsgi.example graphite.wsgi
```
修改 apache 配置，增加一个 vhost 或者偷懒下载一个配置文件覆盖 default，覆盖后需要重新 reload 配置：

```sh
$ wget http://launchpad.net/graphite/0.9/0.9.9/+download/graphite-web-0.9.9.tar.gz
$ tar -zxvf graphite-web-0.9.9.tar.gz
$ cd graphite-web-0.9.9
$ sudo cp examples/example-graphite-vhost.conf /etc/apache2/sites-available/default
```
sockets 最好不要放在 /etc/httpd/ 下面（不同 Linux 发行版本对不同目录的权限问题很混淆人），ubuntu 版本可以放在 /var/run/apache2 下，所以修改 default 文件里的 WSGISocketPrefix 部分：

```sh
$ sudo vi /etc/apache2/sites-available/default
...
WSGISocketPrefix /var/run/apache2/wsgi
...

$ sudo /etc/init.d/apache2 reload
```
初始化 graphite 需要的数据库，修改 storage 的权限，用拷贝的方式创建 local_settings.py 文件：

```sh
$ cd /opt/graphite/webapp/graphite/
$ sudo python manage.py syncdb
$ sudo chown -R www-data:www-data /opt/graphite/storage/
$ sudo cp local_settings.py.example local_settings.py
$ sudo /etc/init.d/apache2 restart
```
启动 carbon：

```sh
$ cd /opt/graphite/
$ sudo ./bin/carbon-cache.py start
```
这个时候，我访问http://localhost的时候，遇到了500 Internal Server Error的错误，于是修改了/opt/graphite/webapp/graphite/local_settings.py，启用了DEBUG=True。然后再查看/opt/graphite/storage/log/webapp/error.log，发现是因为webapp没有权限访问/storage这个目录，可能是因为我们上面使用了chown命令，改变了改目录的所有权，所以我就执行了一下：

```sh
$ sudo chmod 777 -R /opt/graphite/storage
```
再次访问 http://localhost，就可以看到图表信息了！
注意：Graphite只负责显示图形，并不负责收集数据，这与unix的理念相符合，专心做好一件事。

License
----
