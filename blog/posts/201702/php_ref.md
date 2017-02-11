<!--
author: checkking
date: 2017-02-08
title: PHP对象和引用
tags: php
category: php
status: publish
summary: 解释php引用的疑惑
-->
There seems to be some confusion here. The distinction between pointers and references is not particularly helpful.
The behavior in some of the "comprehensive" examples already posted can be explained in simpler unifying terms. Hayley's code, for example, is doing EXACTLY what you should expect it should. (Using >= 5.3)

First principle:

 A pointer stores a memory address to access an object. Any time an object is assigned, a pointer is generated. (I haven't delved TOO deeply into the Zend engine yet, but as far as I can see, this applies)

2nd principle, and source of the most confusion:

Passing a variable to a function is done by default as a value pass, ie, you are working with a copy. "But objects are passed by reference!" A common misconception both here and in the Java world. I never said a copy OF WHAT. The default passing is done by value. Always. WHAT is being copied and passed, however, is the pointer. When using the "->", you will of course be accessing the same internals as the original variable in the caller function. Just using "=" will only play with
copies.

3rd principle:
"&" automatically and permanently sets another variable name/pointer to the same memory address as something else until you decouple them. It is correct to use the term "alias" here. Think of it as joining two pointers at the hip until forcibly separated with "unset()". This functionality exists both in the same scope and when an argument is passed to a function. Often the passed argument is called a "reference," due to certain distinctions between "passing by value" and "passing
by reference" that were clearer in C and C++.

Just remember: pointers to objects, not objects themselves, are passed to functions. These pointers are COPIES of the original unless you use "&" in your parameter list to actually pass the originals. Only when you dig into the internals of an object will the originals change.

Example:

```php
<?php
//The two are meant to be the same
$a = "Clark Kent"; //a==Clark Kent
$b = &$a; //The two will now share the same fate.

$b="Superman"; // $a=="Superman" too.
echo $a; 
echo $a="Clark Kent"; // $b=="Clark Kent" too.
unset($b); // $b divorced from $a
$b="Bizarro";
echo $a; // $a=="Clark Kent" still, since $b is a free agent pointer now.

//The two are NOT meant to be the same.
$c="King";
$d="Pretender to the Throne";
echo $c."\n"; // $c=="King"
echo $d."\n"; // $d=="Pretender to the Throne"
swapByValue($c, $d);
echo $c."\n"; // $c=="King"
echo $d."\n"; // $d=="Pretender to the Throne"
swapByRef($c, $d);
echo $c."\n"; // $c=="Pretender to the Throne"
echo $d."\n"; // $d=="King"

function swapByValue($x, $y){
    $temp=$x;
    $x=$y;
    $y=$temp;
    //All this beautiful work will disappear
    //because it was done on COPIES of pointers.
    //The originals pointers still point as they did.
}

function swapByRef(&$x, &$y){
    $temp=$x;
    $x=$y;
    $y=$temp;
    //Note the parameter list: now we switched 'em REAL good.
}

?>
```
