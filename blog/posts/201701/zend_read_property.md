<!--
author: checkking
date: 2017-01-06
title: Zend读取对象属性
tags: php
category:php
status: publish
summary: 
-->
<html>
	<head>
		<link href="/_core/style.css" rel="stylesheet" type="text/css" />
		<meta charset="UTF-8" />
        <title>11.2 PHP中的面向对象（二）</title>
        
        <script src="/_core/jquery-1.8.3.min.js" type="text/javascript"></script>
        <script src="/_core/syntaxhighlighter_3.0.83/scripts/shCore.js" type="text/javascript"></script>
        <link href="/_core/syntaxhighlighter_3.0.83/styles/shCore.css" rel="stylesheet" type="text/css" />
		<link href="/_core/syntaxhighlighter_3.0.83/styles/shThemeDefault.css" rel="stylesheet" type="text/css" />
		<script src="/_core/syntaxhighlighter_3.0.83/scripts/shAutoloader.js" type="text/javascript"></script>
		
	</head>
	<body>
	<div id='layout-main'>
			<dl>
				<dd></dd>
				<dd></dd>
								<dd style='padding-left:10px'>
					<a href='#读取对象的属性'>读取对象的属性</a>
				</dd>
								<dd style='padding-left:20px'>
					<a href='#更新对象的属性'>更新对象的属性</a>
				</dd>
								<dd style='padding-left:20px'>
					<a href='#读写对象与类属性的实例'>读写对象与类属性的实例</a>
				</dd>
								<dd style='padding-left:20px'>
					<a href='#一些其它的快捷函数'>一些其它的快捷函数</a>
				</dd>
								<dd style='padding-left:30px'>
					<a href='#更新对象与类的属性'>更新对象与类的属性</a>
				</dd>
								<dd style='padding-left:10px'>
					<a href='#links'>links</a>
				</dd>
								<dd></dd>
				<dt>
					<a href='javascript:void(0);' onclick="$(this).parents('dl:first').toggleClass('hide');">展开/折起</a>					-
					<a href='javascript:void(0);' onclick="$(document).scrollTop(0);">头部</a>
				</dt>
			</dl>
		</div>
				
		<div id="article">
		<h1>11.2 PHP中的面向对象（二）</h1>

<p>在上一节里我们已经看了下如何操作一个对象的方法，这一节主要描述与对象属性有关的东西。有关如何对它进行定义的操作我们已经在上一章中描述过了，这里不再叙述，只讲对其的操作。</p>

<h2 name='读取对象的属性' id='读取对象的属性'>读取对象的属性</h2>

<pre class='brush:c'>
ZEND_API zval *zend_read_property(zend_class_entry *scope, zval *object, char *name, int name_length, zend_bool silent TSRMLS_DC);

ZEND_API zval *zend_read_static_property(zend_class_entry *scope, char *name, int name_length, zend_bool silent TSRMLS_DC);

</pre>

<p>zend_read_property函数用于读取对象的属性，而zend_read_static_property则用于读取静态属性。可以看出，静态属性是直接保存在类上的，与具体的对象无关。
silent参数：</p>

<ul>
    <li>0: 如果属性不存在，则抛出一个notice错误。</li>
    <li>1: 如果属性不存在，不报错。</li>
</ul>

<p>如果所查的属性不存在，那么此函数将返回IS_NULL类型的zval。</p>

<h3 name='更新对象的属性' id='更新对象的属性'>更新对象的属性</h3>

<pre class='brush:c'>
ZEND_API void zend_update_property(zend_class_entry *scope, zval *object, char *name, int name_length, zval *value TSRMLS_DC);
ZEND_API int zend_update_static_property(zend_class_entry *scope, char *name, int name_length, zval *value TSRMLS_DC);

</pre>

<p>zend_update_property用来更新对象的属性，zend_update_static_property用来更新类的静态属性。如果对象或者类中没有相关的属性，函数将自动的添加上。</p>

<h3 name='读写对象与类属性的实例' id='读写对象与类属性的实例'>读写对象与类属性的实例</h3>

<p>假设我们已经在扩展中定义好下面的类：</p>

<pre class='brush:php'>
class baby
{
    public $age;
    public static $area;
    
    public function __construct($age, $area)
    {
        $this-&gt;age = $age;
        self::$area = $area;
        
        var_dump($this-&gt;age, self::$area);
    }
}

ZEND_METHOD(baby, __construct)
{
    zval *age, *area;
    zend_class_entry *ce;
    ce = Z_OBJCE_P(getThis());
    if( zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, &quot;zz&quot;, &amp;age, &amp;area) == FAILURE )
    {
        printf(&quot;Error\n&quot;);
        RETURN_NULL();
    }
    zend_update_property(ce, getThis(), &quot;age&quot;, sizeof(&quot;age&quot;)-1, age TSRMLS_CC);
    zend_update_static_property(ce, &quot;area&quot;, sizeof(&quot;area&quot;)-1, area TSRMLS_CC);
    
    age = NULL;
    area = NULL;
    
    age = zend_read_property(ce, getThis(), &quot;age&quot;, sizeof(&quot;age&quot;)-1, 0 TSRMLS_DC);
    php_var_dump(&amp;age, 1 TSRMLS_CC);
    
    area = zend_read_static_property(ce, &quot;area&quot;, sizeof(&quot;area&quot;)-1, 0 TSRMLS_DC);
    php_var_dump(&amp;area, 1 TSRMLS_CC);
    
}

</pre>

<div class="tip-common">
感谢  [@看你取的破名](http://weibo.com/taokuizu)  发现的错误。
</div>

<h3 name='一些其它的快捷函数' id='一些其它的快捷函数'>一些其它的快捷函数</h3>

<h4 name='更新对象与类的属性' id='更新对象与类的属性'>更新对象与类的属性</h4>

<pre class='brush:c'>
ZEND_API void zend_update_property_null(zend_class_entry *scope, zval *object, char *name, int name_length TSRMLS_DC);
ZEND_API void zend_update_property_bool(zend_class_entry *scope, zval *object, char *name, int name_length, long value TSRMLS_DC);
ZEND_API void zend_update_property_long(zend_class_entry *scope, zval *object, char *name, int name_length, long value TSRMLS_DC);
ZEND_API void zend_update_property_double(zend_class_entry *scope, zval *object, char *name, int name_length, double value TSRMLS_DC);
ZEND_API void zend_update_property_string(zend_class_entry *scope, zval *object, char *name, int name_length, const char *value TSRMLS_DC);
ZEND_API void zend_update_property_stringl(zend_class_entry *scope, zval *object, char *name, int name_length, const char *value, int value_length TSRMLS_DC);


ZEND_API int zend_update_static_property_null(zend_class_entry *scope, char *name, int name_length TSRMLS_DC);
ZEND_API int zend_update_static_property_bool(zend_class_entry *scope, char *name, int name_length, long value TSRMLS_DC);
ZEND_API int zend_update_static_property_long(zend_class_entry *scope, char *name, int name_length, long value TSRMLS_DC);
ZEND_API int zend_update_static_property_double(zend_class_entry *scope, char *name, int name_length, double value TSRMLS_DC);
ZEND_API int zend_update_static_property_string(zend_class_entry *scope, char *name, int name_length, const char *value TSRMLS_DC);
ZEND_API int zend_update_static_property_stringl(zend_class_entry *scope, char *name, int name_length, const char *value, int value_length TSRMLS_DC);

</pre>
</body>
</html>
<script language="javascript">
$(document).ready(function(){
    SyntaxHighlighter.autoloader(
      'js jscript javascript    /_core/syntaxhighlighter_3.0.83/scripts/shBrushJScript.js',
      'applescript              /js/shBrushAppleScript.js',
      'c cpp                    /_core/syntaxhighlighter_3.0.83/scripts/shBrushCpp.js',
      'php                      /_core/syntaxhighlighter_3.0.83/scripts/shBrushPhp.js',
      'html xml /_core/syntaxhighlighter_3.0.83/scripts/shBrushXml.js'            
    );
    SyntaxHighlighter.all();
});
</script>
  <!--[if gt IE 8]>
  <div style='position:absolute;top:0px;width:100%;border: 1px solid #F7941D; background: #FEEFDA; text-align: center; clear: both; height: 75px;'>
    <div style='position: absolute; right: 3px; top: 3px; font-family: courier new; font-weight: bold;'><a href='#' onclick='javascript:this.parentNode.parentNode.style.display="none"; return false;'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-cornerx.jpg' style='border: none;' alt='Close this notice'/></a></div>
    <div style='width: 640px; margin: 0 auto; text-align: left; padding: 0; overflow: hidden; color: black;'>
      <div style='width: 75px; float: left;'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-warning.jpg' alt='Warning!'/></div>
      <div style='width: 275px; float: left; font-family: Arial, sans-serif;'>
        <div style='font-size: 14px; font-weight: bold; margin-top: 12px;'>You are using an outdated browser</div>
        <div style='font-size: 12px; margin-top: 6px; line-height: 12px;'>For a better experience using this site, please upgrade to a modern web browser.</div>
      </div>
      <div style='width: 75px; float: left;'><a href='http://www.firefox.com' target='_blank'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-firefox.jpg' style='border: none;' alt='Get Firefox 3.5'/></a></div>
      <div style='width: 75px; float: left;'><a href='http://www.browserforthebetter.com/download.html' target='_blank'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-ie8.jpg' style='border: none;' alt='Get Internet Explorer 8'/></a></div>
      <div style='width: 73px; float: left;'><a href='http://www.apple.com/safari/download/' target='_blank'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-safari.jpg' style='border: none;' alt='Get Safari 4'/></a></div>
      <div style='float: left;'><a href='http://www.google.com/chrome' target='_blank'><img src='http://www.ie6nomore.com/files/theme/ie6nomore-chrome.jpg' style='border: none;' alt='Get Google Chrome'/></a></div>
    </div>
  </div>
  <![endif]-->
