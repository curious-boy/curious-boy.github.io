---
layout: post
title:  centos 6.5 配置thrift 0.92
date:   2015/11/24 11:26:41 
category: "C++"
---

## 更新系统库 ##
- sudo yum -y update

## 安装平台开发工具 ##
sudo yum -y groupinstall "Development Tools"

## 升级构建工具集 autoconf/automake/bison ##
- sudo yum install -y wget

### 升级 autoconf ###
- wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
- tar xvf autoconf-2.69.tar.gz
- cd autoconf-2.69
- ./configure --prefix=/usr
- make
- sudo make install
- cd ..

### 升级 automake ###
- wget http://ftp.gnu.org/gnu/automake/automake-1.14.tar.gz
- tar xvf automake-1.14.tar.gz
- cd automake-1.14
- ./configure --prefix=/usr
- make
- sudo make install
- cd ..

### 升级 bison ###
- wget http://ftp.gnu.org/gnu/bison/bison-2.5.1.tar.gz
- tar xvf bison-2.5.1.tar.gz
- cd bison-2.5.1
- ./configure --prefix=/usr
- make
- sudo make install
- cd ..

## 安装可选C++语言依赖库 ##

### 安装C++依赖库 ###
- sudo yum -y install zlib-devel openssl-devel

### 安装库libevent ###
- wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
- tar -xvf libevent-2.0.21-stable.tar.gz
- cd libevent-2.0.21-stable
- ./configure --prefix=/usr
- make 
- make install

### 升级boost库，版本>=1.53 ###
- wget http://sourceforge.net/projects/boost/files/boost/1.53.0/boost_1_53_0.tar.gz
- tar xvf boost_1_53_0.tar.gz
- cd boost_1_53_0
- ./bootstrap.sh
- sudo ./b2 install

## 编译安装apahce thrift idl 编译器 ##
- git clone https://git-wip-us.apache.org/repos/asf/thrift.git
- cd thrift
- ./bootstrap.sh
- ./configure --with-lua=no
- make
- sudo make install

完成！