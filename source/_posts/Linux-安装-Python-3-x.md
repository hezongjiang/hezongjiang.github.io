---
title: Linux 安装 Python 3.x
catalog: true
date: 2019-01-23 19:54:23
subtitle:
header-img: python.jpg
tags:
---
一般来说，Linux 都自带了 Python 2.x
```
# python --version
Python 2.7.10
```

如果本机安装了 Python 2.x，尽量不要管它，使用 Python3 运行Python 3.x 脚本就好，因为可能有程序依赖目前的 Python 2 环境，

比如yum！不要动现有的 Python 2 环境！下面我们直接安装 Python 3。

#### 一、安装依赖环境
Python 还依赖于许多其他环境，通过下面这一行，全部安装。
```
# yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```
#### 二、下载Python3
可以在[这里](https://www.python.org/ftp/python/)找到你想要的 Python 版本，例如想下载 Python3.6.1：
```
# wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz
```

#### 三、安装python3
1、个人习惯安装在 `/usr/local/python3` 创建目录：
```
# mkdir -p /usr/local/python3
```
2、解压下载好的 `Python-3.x.x.tgz` 包，具体包名根据下载的 Python 版本的不同⽽不同，如：我下载的是 `Python3.6.1` 那我这里就是 `Python-3.6.1.tgz`
```
# tar -zxvf Python-3.6.1.tgz
```
3、进入解压后的目录。
```
# cd Python-3.6.1
```
4、然后配置安装目录，安装到我们之前创建的目录 `/usr/local/python3` 里，这样做的好处是下次想卸载软件直接卸载该目录下的就可以了：
```
# ./configure --prefix=/usr/local/python3
```
5、接着编译一下：
```
# make
```
6、最后就是安装了：
```
# make install
```
7、建立 python3 的软链接，其实就相当于快捷方式，注意：这里软连接的建立与上面提到的安装 python3 的路径相关
```
# ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```
到这里，Python3 已经安装好了，测试一下是否安装成功：
```
# python3 -V
```
如果安装成功，会直接输出 Python3 的版本号。
并且 Python3 已经自带了 pip3，只需要建立一个软连接就可以使用了：
```
# ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```
最后测试pip3是否安装成功：
```
# pip3 -V
```