# Hadoop 安装笔记 #
## 系统环境 ubuntu ##
## 1、安装jdk ##
apt-get install openjdk-7-jdk
## 2、设置环境变量 ##
### vim /etc/profile  （ubuntu）###
### vim ~/.bashrc (centos) ###
    export JAVA_HOME=/usr/lib/jvm/jdk7
    export JRE_HOME=${JAVA_HOME}/jre
    export HADOOP_HOME=/opt/hadoop-1.2.1
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$HADOOP_HOME/lib
    export PATH=${JAVA_HOME}/bin:$HADOOP_HOME/bin:$PATH
## 3、下载hadoop，设置环境变量 ##
- wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz
- cd /opt
- cp */hadoop-1.2.1.tar.gz ./
- tar -xzvf hadoop-1.2.1.tar.gz
- cd hadoop-1.2.1
## 4、修改四个配置文件,均在conf目录下 ##
### a) 修改hadoop-env.sh，设置 JAVA_HOME ###
    # The java implementation to use.  Required.
    export JAVA_HOME=/usr/lib/jvm/jdk7
### b) 修改core-site.xml,设置hadoop.tmp.dir,dfs.name.dir,fs.default.name ###
    <configuration>
    <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop</value>
    </property>
    <property>
    <name>dfs.name.dir</name>
    <value>/hadoop/name</value>
    </property>
    <property>
    <name>fs.default.name</name>
    <value>hdfs://127.0.0.1:9000</value>
    </property>
    </configuration>
### c) 修改mapred-site.xml,设置mapred.job.tracker ###
    <configuration>
    <property>
    <name>mapred.job.tracker</name>
    <value>imooc:9001</value>
    </property>
    </configuration>
### d) 修改hdfs-site.xml,设置dfs.data.dir ###
    <configuration>
    <property>
    <name>dfs.data.dir</name>
    <value>/hadoop/data</value>
    </property>
    </configuration>
## 5、设置hadoop ##
### 命令 ###
    hadoop namenode -format //节点格式化
    start-all.sh //启动
    jps //查看java进程
### 结果 ###
    25392 Jps
    24152 DataNode
    24281 SecondaryNameNode
    25283 JobTracker
    24015 NameNode
    24493 TaskTracker
完成安装！
## 6、安装完成后，hdfs文件系统的基本命令操作 ##
    hadoop fs -put hadoop-env.sh 
    hadoop fs -put hadoop-env.sh input/
    hadoop fs -ls /
    hadoop fs -ls /user
    hadoop fs -ls /user/root
    hadoop fs -rm input
    hadoop fs -mkdir input
    hadoop fs -ls /usr/root
    hadoop fs -ls /user/root
    hadoop fs -put hadoop-env.sh input/
    hadoop fs -ls /user/root/input/
    hadoop fs -cat /user/root/input/hadoop-env.sh
    hadoop fs -get input/hadoop-env.sh hadoop-env2.sh


