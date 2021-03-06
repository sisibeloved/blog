---
title: RocketMQ安装及部署
date: 2018-03-20 10:04:41
categories: DevOps
tags:
- RocketMQ
---

## 版本信息

### **Windows**

- OS : Windows 10 x64
- RocketMQ : 4.2.0
- JDK : 1.8.0_131
- Maven : 3.5.0

### **Linux**

- OS : Ubuntu 16.04LTS x64
- RocketMQ : 4.2.0
- JDK : 1.8.0_151
- Maven : 3.5.2

<!-- more -->

## 安装步骤

### **Windows**

1. 若下载源码，使用Maven进行编译
2. 配置环境变量(值为RocketMQ的安装路径)
  `ROCKETMQ_HOME=D:\RocketMQ`
3. 修改runbroker.cmd第40行，添加双引号
  将`set "JAVA_OPT=%JAVA_OPT% -cp %CLASSPATH%"`
  改为`set "JAVA_OPT=%JAVA_OPT% -cp "%CLASSPATH%""`

### **Linux**

1. 若下载源码，使用Maven进行编译

2. 配置环境变量(值为RocketMQ的安装路径)
  `export ROCKETMQ_HOME=/usr/local/rocketmq/`
  `export PATH=$ROCKETMQ_HOME/bin:$PATH`

3. 修改默认配置。由于RocketMQ默认配置要求很高，比如内存至少就要4个G，开发调试环境根本吃不消，所以我们开始启动前需要先修改这些参数。否则的话，我们很有会遇到内存分配或者不够的问题。
  (1). 修改target/apache-rocketmq-all/bin/runserver.sh
  `JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=320m"`
  (2). 修改target/apache-rocketmq-all/bin/runbroker.sh
  `JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m`
  (3). 修改target/apache-rocketmq-all/bin/tools.sh
  `JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"`

## 启动步骤

1. 启动Name Server。在bin目录下运行：
  `nohup sh mqnamesrv & tail -f ~/logs/rocketmqlogs/namesrv.log`
2. 启动Broker。在bin目录下运行：
  `nohup sh mqbroker -n localhost:9876 & tail -f ~/logs/rocketmqlogs/broker.log`
  这里-n后面跟的是Name Server的地址。
3. 若要对Broker进行配置。在bin目录下运行：
  `sh mqbroker -m > broker1.properties`
  然后可以对生成的.properties文件进行修改。
4. 根据配置启动：
  `sh mqbroker -c broker1.properties`
5. 关闭Broker：
  `sh mqshutdown broker`（通用）
  `sh mqadmin wipeWritePerm -b brokerName -n namerverAddr`（可选）

## 常见错误

1. 启动Name Server的时候报：
  `Please set the ROCKETMQ_HOME variable in your environment!`
  配置一下ROCKETMQ_HOME的环境变量。
2. 启动Broker的时候报找不到主类JDK，修改runbroker.cmd第40行
  `set "JAVA_OPT=%JAVA_OPT% -cp "%CLASSPATH%""`
  原来的%CLASSPATH%少了双引号。
3. 启动Producer报错
  `org.apache.rocketmq.remoting.exception.RemotingConnectException: connect to <172.17.0.1:10909> failed`
  多种原因：
- 检查IP地址，经检查对应服务器安装有其他网桥，将其卸载即可
- 在Producer中将VIP通道关闭：`producer.setVipChannelEnabled(false);`

4. 启动Producer报错
  `org.apache.rocketmq.client.exception.MQClientException: No route info of this topic, TopicTest2`
  - Broker没有正确连接到Name Server
  或

  - Producer没有正确连接到Name Server
  或

  - Broker禁止自动创建Topic，且用户没有通过手工方式创建Topic

  默认配置启动broker时`autoCreateTopicEnable=true`已设置，所以检查连接即可