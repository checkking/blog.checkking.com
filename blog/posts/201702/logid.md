<!--
author: checkking
date: 2017-02-07
title: 业务系统logid生成
tags: php
category: php
status: publish
summary: Logid是日常分析线上问题重要手段
-->
Logid是日常分析线上问题重要手段, 通常一个在线系统需要对每个请求带上一个唯一的标识，这个标识就是logid。并且贯穿于整个HTTP交互过程中接触的模块。
logid生成算法采用的是时间+随机数的方式拼接成的字符串，其中时间是从CUT（Coordinated Universal Time）时间1970年1月1日00:00:00（称为UNIX系统的Epoch时间）到当前时刻的秒数，随机数是1-1000之间的随机数，不足四位以0填充。
如果nginx的流量较大，上述算法会有较大的概率产生重复的log_id。因此将时间改为毫秒级，并且只取1小时内的毫秒数，随机数仍采用三位000-999的随机数。公式如下：

logid=当前秒数（不超过3600秒，4位）+当前时间（ms，3位）+随机数（3位）       （1）

由本算法产生的最大logid是3599999999，小于uint的最大值2^32=4294967296，确保了logid在不同模块之间传递的正确性。

获得logId的函数如下：
```php
public static function genLogID() {
    if(defined('LOG_ID')){
        return LOG_ID;
    }
    if(!empty($_SERVER['HTTP_X_BD_LOGID']) && intval(trim($_SERVER['HTTP_X_BD_LOGID'])) !== 0){
        define('LOG_ID', trim($_SERVER['HTTP_X_BD_LOGID']));
    } elseif (isset($_REQUEST['logid']) && intval($_REQUEST['logid']) !== 0) {
        define('LOG_ID', intval($_REQUEST['logid']));    
    } else {
        $arr = gettimeofday();
        $logId = sprintf('%4d%3d%3d', $arr['sec'], $arr['usec'], mt_rand(100,999));
        define('LOG_ID', $logId);
    }
}
```
