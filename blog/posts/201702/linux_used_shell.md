<!--
author: checkking
date: 2017-02-07
title: 工作中常用的Linux命令
tags: linux
category:linux
status: publish
summary: 工作中常用的Linux命令
-->
#### 统计

1. 统计当前目录下的所有文件行数：
```bash
wc -l *
```
2. 当前目录以及子目录的所有文件行数：
```bash
find  . * | xargs wc -l
```
3. 统计目录以及子目录所有c文件行数：
```bash
find  . -name "*.c" | xargs wc -l
```

#### 查找

1. 查找当前目录下所有的文件包含某字符串
```bash
grep str *
```
2. 查找包含某字符串的所有c++文件
```bash
find . -type f -name "*.h" -o -name "*.cpp" | xargs grep 'hello'
```

#### svn

1. 删除代码路径下的svn
```bash
find . -name ".svn" | xargs rm -rf
```

#### 系统

1. 打开core dump
```bash
ulimit -c unlimited
```
查看coredump是否打开：
```bash
ulimit -c
```
通过修改 /proc/sys/kernel/core_uses_pid 文件可以让生成 core 文件名是否自动加上 pid 号
```bash
echo 1 > /proc/sys/kernel/core_uses_pid
```

#### 进程

1. 删除某个进程
```bash
ps -ef | grep twisted | awk '{print $2}' | xargs kill 
```
