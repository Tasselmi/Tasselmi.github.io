---
layout:     post
title:      "深入理解ClickHouse之2-部署单机版ClickHouse"
subtitle:   "————Linux用户和Mac OSX用户看过来"
date:       2018-12-24 12:00:00
author:     "LiangFan"
header-img: "img/my-post/2018-clickhouse.jpeg"
tags:
    - ClickHouse 
    - OLAP
---

由于ClickHouse是不支持Windows系统的，所以只能安装在Linux或者Mac OSX系统上。如果Windows用户需要尝试的话，需要通过虚拟机或者其它方式安装。本文将一步步带领Linux用户和Mac OSX用户从零开始安装部署单机版ClickHouse。

根据官网的描述，ClickHouse可以运行在任何Linux系统上，前提是支持SSE 4.2；ClickHouse可以运行在64位Mac OSX上；暂不支持Windows；以下为官网原话：

> ClickHouse can run on any Linux, FreeBSD or Mac OS X with x86_64 CPU architecture.Though pre-built binaries are typically compiled to leverage SSE 4.2 instruction set, so unless otherwise stated usage of CPU that supports it becomes an additional system requirement.

ubuntu系统安装ClickHouse需如下几步：

#### 1. 查看当前CPU是否支持SSE 4.2
```shell
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
```
#### 2. 将ClickHouse repo添加到/etc/apt/sources.list文件
```shell
sudo vi /etc/apt/sources.list
paste deb http://repo.yandex.ru/clickhouse/deb/stable/ main/ into the file.
```
#### 3. 下载和安装ClickHouse
```shell
sudo apt-get update
sudo apt-get install clickhouse-client clickhouse-server
```
#### 4. 启动和停止ClickHouse
1）后台守护模式运行和停止：
```shell
sudo service clickhouse-server start
sudo service clickhouse-server stop
```
关于这里的命令比如start或者stop，自己可以看下文件里面的.sh文件就大概能推断一二了。

2）Terminal打印日志模式运行：
```shell
clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```
这个模式下停止直接用`ctrl+C`就行了，与其它软件类似。

3）运行clickhouse-client：
./clickhouse-client

server和client一个是服务端，一个是客户端，这个应该是很好理解的，与单机版的 mysql 类似。

如果觉得每次需要cd到文件目录来启动很麻烦，还可以把`clickhouse-client` 和 `clickhouse-server`添加到系统路径中，这样就比较方便了。













