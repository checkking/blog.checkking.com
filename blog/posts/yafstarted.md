<!--
author: checkking
date: 2016-12-26
title: 开始使用Yaf
tags: yaf,php
category:php
status: publish
summary: 开始使用Yaf
-->
### 安装
```bash
git clone https://github.com/laruence/yaf.git
cd yaf && /usr/local/php/bin/phpize && ./configure --with-php-config=/path/to/php-config
```
> **注意：**/usr/local/php/bin/phpize 这一步可能报错：Cannot find autoconf. Please check your autoconf installation and the $PHP_AUTOCONF ，安装: apt-get install autoconf
> 如果make出错，很可能是版本不对，php-5.6.x对应的版本是yaf-2.3.4
然后在php.ini中载入yaf.so, 重启PHP
```bash
[yaf]
yaf.environ=product
yaf.library=NULL
yaf.cache_config=0
yaf.name_suffix=1
yaf.name_separator=""
yaf.forward_limit=5
yaf.use_namespace=0
yaf.use_spl_autoload=0
extension=yaf.so
```

在echo phpinfo()中找到yaf,说明安装成功.
