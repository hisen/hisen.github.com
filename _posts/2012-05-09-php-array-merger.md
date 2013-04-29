---
layout: post
title: php合并数组两种方式
category: php
keywords: php,php数组合并
---

<pre>
&lt;?php
$a = array(1,2,3,4,5,6);
$b = array(7,8,9);
$c = array_merge ($a,$b);
print_r($c);
$c = $a+$b;
print_r($c);
$c = $b+$a;
print_r($c);
</pre>

<div>
首先看下php手册对array_merge的说明：
<strong>array_merge()</strong> 将一个或多个数组的单元合并起来，一个数组中的值附加在前一个数组的后面。返回作为结果的数组。
如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。
如果只给了一个数组并且该数组是数字索引的，则键名会以连续方式重新索引。
 

再看手册中<strong>数组运算符</strong>对两个数组$a+$b的说明：
+ 运算符把右边的数组元素（除去键值与左边的数组元素相同的那些元素）附加到左边的数组后面，但是重复的键值不会被覆盖。
</div>

<pre>
所以上面的结果将是：
Array ([0] =&gt; 1  [1] =&gt; 2  [2] =&gt; 3  [3] =&gt; 4  [4] =&gt; 5  [5] =&gt; 6  [6] =&gt; 7  [7] =&gt; 8  [8] =&gt; 9 )
Array ([0] =&gt; 1  [1] =&gt; 2  [2] =&gt; 3  [3] =&gt; 4  [4] =&gt; 5  [5] =&gt; 6 )
Array ([0] =&gt; 7  [1] =&gt; 8  [2] =&gt; 9  [3] =&gt; 4  [4] =&gt; 5  [5] =&gt; 6 )
</pre>
