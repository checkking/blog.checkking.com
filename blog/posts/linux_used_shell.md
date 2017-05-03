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

4. 统计某小时中各分钟的请求个数:
```bash
cat access.log | grep 3cf0266e   | grep "16/Mar/2017:15" | awk '{print $4}' | awk -F':' '{sum[$3]++}END{for (i in sum) print i, sum[i]}' | sort -k1
```

5. 正则统计某个字符串出现的次数:
```bash
cat log | awk '{match($0, /(BASE_[_A-Z]+)/, a); cnt[a[1]]++} END{for (i in cnt) {print i"\t"cnt[i]}}' 
```

#### 查找与替换

1. 查找当前目录下所有的文件包含某字符串
```bash
grep str *
```
2. 查找包含某字符串的所有c++文件
```bash
find . -type f -name "*.h" -o -name "*.cpp" | xargs grep 'hello'
```

3. 替换掉某个文件夹下所有字符串
```bash
find . -type f |xargs sed -i 's/odp_0331/odp/g' 
```

#### 文本处理

1. 将多行归并成一行文本
```bash
cat txt | awk '{a=(a","$0)}END{print a}'

2. 对一个文件按行shuffle
```bash
awk 'BEGIN{srand()}{a[rand()NR]=$0}END{for (l in a) prin a[l]}' file
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
ps -ef | grep {execute file name} | awk '{print $2}' | xargs kill 
```

2. 查看某个进程的启动路径
```bash
ll /proc/{pid}/
```

3. 根据pid查看进程是否存活
```bash
now=`date +%Y%m%d%H%M%S`
pid=$(`cat ${base_path}/var/run.pid`)
ps ax | awk '{print $1}' | grep "^${pid}$"
if [ $? == 0 ]; then
    echo "[WARNNING]""$now"" run.sh is already running, stop this script." >&2
    exit 1
else
    echo "[WARNNING]""$now"" previous pid[""$pid"" failed!" >&2
fi
```
