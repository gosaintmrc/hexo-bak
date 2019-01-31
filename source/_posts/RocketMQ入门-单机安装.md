---
title: RocketMQ入门---单机安装
date: 2019-01-31 14:17:15
categories: 
- 消息中间件
tags: RocketMQ
---

​             本次是在windows下的安装。下载最新的4.3的版本。下载地址：**http://mirror.bit.edu.cn/apache/rocketmq/4.3.2/rocketmq-all-4.3.2-bin-release.zip 。下载完成解压后的目录如下图所示：**

![img](RocketMQ入门-单机安装\1.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

配置环境变量到bin目录。

![img](RocketMQ入门-单机安装\2.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​               1 启动nameServer

​                进入bin目录，使用如下的命令启动nameServer:

```bash
 start mqnamesrv.cmd
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```
出现如上图显示的：The Name Server boot Success.此时表示启动成功。上述的警告是我的JVM使用的CMS垃圾收集器。RocketMQ推荐使用G1收集器。
```

​                2 启动broker

​                进入bin目录，使用如下的命令启动broker:

```python
 start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```
RocketMQ插件部署
```

1 下载

​        地址：<https://github.com/apache/rocketmq-externals>

> 下载后编译：
>
> 下载完成之后，进入‘rocketmq-externals\rocketmq-console\src\main\resources’文件夹，打
>
> 开‘application.properties’进行配置。

​        9876端口是默认的RocketMQ的端口

![img](RocketMQ入门-单机安装\3.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2. 编译启动

​            进入‘\rocketmq-externals\rocketmq-console’文件夹，执行‘mvn clean package -Dmaven.test.skip=true’，编译生成。

​            编译成功之后，Cmd进入‘target’文件夹，执行‘java -jar rocketmq-console-ng-1.0.0.jar’，启动‘rocketmq-console-ng-1.0.0.jar’。

3.测试

​        浏览器中输入‘127.0.0.1：8081’，成功后即可查看。

![img](RocketMQ入门-单机安装\4.png)

