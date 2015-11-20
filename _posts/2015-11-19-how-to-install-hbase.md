---
layout: post
title:  Hbase学习笔记之环境搭建（伪分布模式）
date:   2015-11-19 12:55:11
category: "大数据"
---

## 三种运行模式 ##
- 本地模式：相关进程运行在同一个jvm中，如果其中一个任务出现问题，jvm就会崩溃
- 伪分布式模式：相关进程各自运行在不同的jvm中，适合学习都适用
- 集群模式：企业中使用的

## 一、Linux环境搭建 ##
### 关闭防火墙 ###
- 临时关闭 service iptables stop
- 永久关闭 chkconfig iptables off
### 关闭SELinux ###
- vim /etc/sysconfig/selinux
- SELINUX=enforcing -->SELINUX=disabled
### 配置IP、DNS ###
- 右上角，编辑Network Connections 
- -->Edit... -->IPv4 Settings -->Method(Manual) 
- --Add -->192.168.1.11 255.255.255.0 192.168.1.1 DNS servers:(8.8.8.8)
- -->OK
### 重启网卡  ###
- service network restart
### 配置主机名 ###
- vim /etc/sysconfig/network
- HOSTNAME=localhost --> HOSTNAME=hbase01.cd.com
### 配置IP映射关系 ###
- vim /etc/hosts
- 添加 10.237.154.69 hbase01.cd.com
### 设置SSH免密码登录 ###
- ssh-keygen -t rsa
- 四个回车
- ssh-copy-id 10.237.154.69
- yes
- 输入密码
- 测试 ssh 10.237.154.69,如果不需要密码，登录成功，即ssh无密码登录是成功的
- 重启linux
### 安装JDK ###
- 将所有需要的程序放到一个目录
- mkdir bigdata
- cd bigdata
- mkdir tools
- mkdir softwares
- cd tools
### 下载jdk-7u67-linux-x64.tar.gz ###
#### wget下载百度云文件 ####
- wget -c -O 文件名.后序名 "云盘下载地址"
- tar -xzf jdk-7u67-linux-x64.gz -C ../softwares/
- cd ../softwares/
- cd jdk-7u67-linux-x64
- pwd
### 将目录配置到环境变量中 ###
- vim /etc/profile
- 添加
- export JAVA_HOME=/root/bigdata/softwares/jdk1.7.0_67
- export PATH=$PATH:$JAVA_HOME/bin
- 保存
### 刷新配置，让配置生效 ###
- source /etc/profile
### 验证是否已经安装成功 ###
- java -version
## 二、Hadoop环境搭建 ##
### 下载编译 ###
- http://archive.cloudera.com/cdh5/cdh/5/
- 进入hadoop.apache.org 下载2.6.0版本
- 下载 wget http://apache.fayea.com/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
- 解压 tar -xzf hadoop-2.6.0.tar.gz -C ../softwares/
- cd ../softwares/
### 跟随官方文档进行配置 ###
> http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/SingleCluster.html
### 修改hadoop-env.sh  ###
> 设置jdk变量
> export JAVA_HOME=/usr/lib/jvm/jdk7
### 修改core-site.xml ###
    <configuration>
    	<property>
    <name>fs.defaultFS</name>
    <value>hdfs://192.168.2.11:8020</value>
    </property>
    	<property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/bigdata/softwares/hadoop-2.6.0/data/tmp</value>
    </property>
    </configuration>
### 配置HDFS、YARN ###
#### 修改hdfs-site.xml ####
    <configuration>
    <property>
    <name>dfs.replication</name>
    <value>1</value>
    </property>
    </configuration>
#### 修改yarn-site.xml ####
    <configuration>
    <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    </property>
    </configuration>
#### 修改mapred-site.xml ####
将mapred-site.xml.template重命名为mapred-site.xml，内容是

    <configuration>
    <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    </property>
    </configuration>

### 格式化 ###
bin/hdfs namenode -format
### 启动并测试 ###
#### 启动 ####
- sbin/start-dfs.sh
- sbin/start-yarn.sh 

#### 使用wordcount程序测试 ####
- bin/hadoop fs -mkdir -p /user/root/mr/wc/in
- bin/hadoop fs -put /etc/profile /user/root/mr/wc/in
- bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar wordcount /user/root/mr/wc/in/profile /user/root/mr/wc/out
- bin/hadoop fs -ls /user/root/mr/wc/out
- bin/hadoop fs -cat /user/root/mr/wc/out/part-r-00000

## 三、Hbase环境搭建 ##
### 下载解压 ###
### 修改hbase-env.sh ###
 - export JAVA_HOME=/usr/lib/jvm/jdk7
### 修改hbase-site.xml ###
    <configuration>
    <property>
    	<name>hbase.rootdir</name>
    	<value>hdfs://192.168.2.11:8020/hbase</value>
    </property>
    <property>
    	<name>hbase.zookeeper.property.dataDir</name>
    	<value>/usr/local/bigdata/softwares/hbase-0.98.16-hadoop2/data/zkData</value>
    </property>
    <property>
    	<name>hbase.cluster.distributed</name>
    	<value>true</value>
    </property>
    </configuration>
### 修改regionservers ###
	localhost-->192.168.2.11
### 启动 ###
    bin/hbase-daemon.sh start zookeeper
    bin/hbase-daemon.sh start master
    bin/hbase-daemon.sh start regionserver

### 通过jps查看进行 ###
	3149 NameNode
	3407 SecondaryNameNode
	27532 Jps
	27191 HQuorumPeer
	3259 DataNode
	27383 HRegionServer
	3549 ResourceManager
	3634 NodeManager
	27258 HMaster

### 查看hbase的webui ###
> http://192.168.2.11:60010/

安装完成