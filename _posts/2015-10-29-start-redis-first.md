---
layout: post
title:  redis入门01：centos下安装redis
date:   2015/10/29 20:02:21 
category: "java"
---


## 安装 ##
- yum install wget
- wget http://download.redis.io/releases/redis-3.0.1.tar.gz
- tar xzf redis-3.0.1.tar.gz
- cd redis-3.0.1
- make
- yum install tcl
- make test
- make install

## 修改配置文件redis.conf ##
- 内容修改 daemonize no --> daemonize yes 
- 创建目录 mkdir /usr/local/redis
- cp redis.conf /usr/local/redis


## 编写启动shell: /etc/init.d/redis ##
    # chkconfig: 2345 10 90
    # description: Start and Stop redis
      
    PATH=/usr/local/bin:/sbin:/usr/bin:/bin
      
    REDISPORT=6379 #实际环境而定
    EXEC=/usr/local/bin/redis-server #实际环境而定
    REDIS_CLI=/usr/local/bin/redis-cli #实际环境而定
      
    PIDFILE=/var/run/redis.pid
    CONF="/usr/local/redis/redis.conf" #实际环境而定
      
    case "$1" in
    start)
    if [ -f $PIDFILE ]
    then
    echo "$PIDFILE exists, process is already running or crashed."
    else
    echo "Starting Redis server..."
    $EXEC $CONF
    fi
    if [ "$?"="0" ]
    then
    echo "Redis is running..."
    fi
    ;;
    stop)
    if [ ! -f $PIDFILE ]
    then
    echo "$PIDFILE exists, process is not running."
    else
    PID=$(cat $PIDFILE)
    echo "Stopping..."
    $REDIS_CLI -p $REDISPORT SHUTDOWN
    while [ -x $PIDFILE ]
    do
    echo "Waiting for Redis to shutdown..."
    sleep 1
    done
    echo "Redis stopped"
    fi
    ;;
    restart|force-reload)
    ${0} stop
    ${0} start
    ;;
    *)
    echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2
     exitxit 1
    esac

## 赋予运行权限，命令如下： ##
- chmod +x /etc/init.d/redis

## 启动或停止redis ##
- service redis start
- service redis stop
      
## 开启服务自启动 ##
- chkconfig redis on

## shell运行 ##
- redis-cli

The End.
    