# 3-RocketMQ控制台

[TOC]

## 3.1 搭建Tomcat环境


## 3.2 RocketMQ-console

1. 使用XFTP上传war文件
2. 解压war文件
```
[root@VM_1_209_centos webapps]# unzip rocketmq-console-3.2.6.war -d rocketmq-console

```
3. 编辑properties文件
```
[root@VM_1_209_centos webapps]# rm -rf rocketmq-console-3.2.6.war 
[root@VM_1_209_centos webapps]# ls
ROOT  docs  examples  host-manager  manager  rocketmq-console
[root@VM_1_209_centos webapps]# cd rocketmq-console/
[root@VM_1_209_centos rocketmq-console]# ls
META-INF  WEB-INF  css  index.html  js
[root@VM_1_209_centos rocketmq-console]# cd WEB-INF/
[root@VM_1_209_centos WEB-INF]# ll
total 28
drwxr-xr-x  4 root root 4096 Jun 14  2016 classes
-rw-r--r--  1 root root   59 Jun 14  2016 index.html
drwxr-xr-x  2 root root 4096 Jun 14  2016 lib
-rw-r--r--  1 root root 2382 Jun 14  2016 springmvc-servlet.xml
-rw-r--r--  1 root root  676 Jun 14  2016 velocity-toolbox.xml
drwxr-xr-x 10 root root 4096 Jun 14  2016 vm
-rw-r--r--  1 root root 1073 Jun 14  2016 web.xml
[root@VM_1_209_centos WEB-INF]# cd classes/
[root@VM_1_209_centos classes]# ls
com  config.properties  logback.xml  spring
[root@VM_1_209_centos classes]# vim config.properties 

```
4. 启动tomcat

```
[root@VM_1_209_centos bin]# ./startup.sh
Using CATALINA_BASE:   /usr/local/rocketmq-console-tomcat
Using CATALINA_HOME:   /usr/local/rocketmq-console-tomcat
Using CATALINA_TMPDIR: /usr/local/rocketmq-console-tomcat/temp
Using JRE_HOME:        /usr/local/java/jdk1.8.0_121
Using CLASSPATH:       /usr/local/rocketmq-console-tomcat/bin/bootstrap.jar:/usr/local/rocketmq-console-tomcat/bin/tomcat-juli.jar
Tomcat started.

```

## 坑与问题

- 启动后发现控制台brokerAddr和实际不一致问题
```
在broker配置文件中添加属性

brokerIP1=实际ip地址

注意属性名为brokerIP1  确实有个1
```