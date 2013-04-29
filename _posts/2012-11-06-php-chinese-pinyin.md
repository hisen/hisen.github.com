---
layout: post
title: php中文转拼音的一种方法与思路
category: php
keywords: 中文转拼音
---

中文转拼音这个需求在很多项目系统中都可能只用到，不仅仅只是web项目。
其中转换的程序也有很多种写法，但是原理都是相通的。自己目前接触的项目中也有这种功能需求，需要将传入的客户信息转换成拼音，以便在jquery-autocomplete这插件中使用。但是在了解了原理之后，发现这个转换程序存在bug。

<strong>首先来讲中文转拼音的条件：</strong>

1，本文讲的转换是基于gb2312编码方式的,并且不考虑多音字。
如果程序中使用了其他编码方式，可以在调用到转换程序之前进行编码转换，使需要转换的值为gb2312的

2，字符对应gb2312编码表的方式（什么是区位码）

3，拼音的键值对应关系表（这个需要自己整理，本文不涉及如何去完成这个表）

首先需要了解ASCII编码表http://www.asciitable.com，十进制的0-127代码了ASCII码的相关字符。

gb2312编码http://ash.jp/code/cn/gb2312tbl.htm，gb2312标准共收录6763个汉字，其中一级汉字3755个，二级汉字3008个；同时收录了包括拉丁字母、希腊字母、日文平假名及片假名字母、俄语西里尔字母在内的682个字符。
gb2312中对所收汉字进行了“分区”处理，每区含有94个汉字／符号。这种表示方式也称为区位码。
<ul>
	<li>01-09区为特殊符号。</li>
	<li>16-55区为一级汉字，按拼音排序。</li>
	<li>56-87区为二级汉字，按部首／笔画排序。</li>
	<li>10-15区及88-94区则未有编码。</li>
</ul>

举例来说，“啊”字是gb2312之中的第一个汉字，它的区位码就是1601，参照gb2312的编码表即对应的是16行（区）01别（位）。
一个GB2312的字符 是用两个字节来表示的，算法是： 一个GB2312的字符  ==  (0xA0 + 区号，0xA0 + 位号)。0xA0  是一个16进制数字 换算成 十进制其实就等于160 。高位字节 等于  160 加上 区号，你可以理解为，其实就是 GB2312编码字符  是从160 起步的。(理解这点很重要下面会在转换中使用到)

理解了区位码之后，然后程序是如何对输入的中文进行处理的呢？
步骤大致如下：
获取gb2312的中文ASCII码=》获取区位码=》根据定义好的拼音键值对应关系表进行匹配=》返回结果（汉字对应的拼音）

大致原理代码如下：
<pre>
&lt;?php
    $str = '杭'; // 如果非gb2312，先转换成gb2312的
    $strLength = strlen($str);
    $ord_arr = array();
    for($i=0;$i&lt;$strLength;$i++)
    {
        $ord = ord(substr($str,$i,1));
        if($ord&lt;=127)
        {
            echo $ord; // 小于等于127的都是ASCII码
        }
        else
        {
            $ord_arr[] = $ord;
        }
    }
    // 对返回的值进行十六进制转换
    $result_y = dechex($ord_arr[0])-160; //区
    $result_x = dechex($ord_arr[1])-160;//位
    $zoneBit = $result_y.$result_x; //区位码
    //这里可能会问这个160哪来的，那会到上面看汉字“啊”的区位码介绍，和算法
</pre>
到此计算转行的就结束了，剩下的就是自己再整理一份“拼音的键值对应关系表”，百度文库有一份《汉字区位简明对照》可以自行下载，截图如下
<a href="{{ site:url }}/uploads/2012/11/chinese.png"><img src="{{ site:url }}/uploads/2012/11/chinese.png" alt="汉字区位码" title="汉字区位码" width="502" height="294" class="alignnone size-full wp-image-582" /></a>
最后再说明下。网上很多php版本的汉字转拼音都是有问题的，部分汉字转换的有出入。也包括crm中使用的（遵义=》zheyi，泸州=》zhe等等），看了时间居然是2010-01-19 12:57:52,还好功能很小估计销售也不怎么使用。