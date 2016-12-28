<!--
author: checkking
date: 2016-12-28
title: 一步一步编写一个php动态拓展
tags: php
category: php
status: publish
summary: 一个php动态拓展的实例
-->
首先到php的源码的ext目录下(**必须要到这个目录下，否则有问题**)
```
cd path-to/php-5.6.2/ext
```
新建一个ck_extension.def的文件,内容如下：
```c
string self_concat(string str, int n)
```
执行：
```bash
./ext_skel --extname=ck_extension --proto=ck_extension.def
```
就会创建一个ck_extension目录，里面生成了一些源码
打开config.m4，去掉相应行的dnl,去掉后的内容变成：
```
PHP_ARG_WITH(ck_extension, for ck_extension support,
dnl Make sure that the comment is aligned:
[  --with-ck_extension             Include ck_extension support])
```
####编写c代码
在ck_extension.c中对应的self_concat方法修改如下：
```c
PHP_FUNCTION(self_concat)
{
	char *str = NULL;
	int argc = ZEND_NUM_ARGS();
	int str_len;
	long n;

    char *result;
    char *ptr;
    int result_len;

	if (zend_parse_parameters(argc TSRMLS_CC, "sl", &str, &str_len, &n) == FAILURE) 
		return;

    result_len = (str_len * n);
    result = (char *) emalloc(result_len + 1);
    ptr = result;
    while (n--) {
        memcpy(ptr, str, str_len);
        ptr += str_len;
    }
    *ptr = '\0';
    RETURN_STRINGL(result, result_len, 0);
}
```
软后执行
```bash
/usr/local/php/bin/phpize 
```
就会生成configure文件，然后执行:
```bash
./configure --with-php-config=/path/to/php-config
make
make install
```
如果中间没报错就安装成功了，到/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/目录下可以看到对应的so文件
#### 配置php.ini
```
[ck_extension]
extension=ck_extension.so
```
重启php-fpm,查看是否安装成功:
```
php -m | grep ck_extension
```
#### 使用ck_extension
编写php文件如下：
```
<?php
$str = 'helloword';
print self_concat($str, 3);
echo "\n";
?>
```
执行结果符合预期!
