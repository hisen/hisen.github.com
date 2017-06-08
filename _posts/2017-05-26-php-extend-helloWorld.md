---
layout: post
title: php7扩展开发-helloWorld
category: php
keywords: 扩展
---

1  ./ext_skel --extname=helloWorld

2 编辑config.m4 去除下面三行前面的注释符dnl

```
PHP_ARG_ENABLE(helloWorld, whether to enable helloWorld support,
Make sure that the comment is aligned:
[  --enable-helloWorld           Enable helloWorld support])
```

3 创建自定义函数  
搜索static int le_helloWorld，另起一行在下面添加

```
PHP_FUNCTION(helloWorld)
{
        char *arg = NULL;
        size_t arg_len, len;
        zend_string *strg;

        if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
                return;
        }
        strg = strpprintf(0, "hello word, %.78s", arg);
        RETURN_STR(strg);
}
```

搜索
const zend_function_entry helloWorld_functions[]添加自定义函数helloWorld，使Zend引擎在载入扩展时，自动把则个数组中的所有函数引入到函数表

```
const zend_function_entry helloWorld_functions[] = {
        PHP_FE(helloWorld,     NULL)
        PHP_FE(confirm_helloWorld_compiled,     NULL)
        PHP_FE_END     
};
```

4 保存退出
执行

```
phpize  
./configure --with-php-config=/usr/local/php/bin/php-config  
make && make install  
```
并在php.ini文件中配置改扩展文件  



5 命令行下执行如下命令即可查看效果

```
php -r "echo helloWorld('hisenking');"
```


####API说明：  

```
入参解析
zend_parse_parameters() 类型说明
修饰符	附加参数的类型	描述
b	zend_bool	Boolean 值
l	long	integer (long) 值
d	double	float (double) 值
s	char*, int	二进制的安全串
h	HashTable*	数组的哈希表
```
使用 zend_parse_parameters 来解析， al|zb 表示第一个参数为 array，第二个参数为 long，第三个参数为 zval，第四个参数为 bool。 并且后面两个参数为可选  


php7还可以使用 Fast Parameter Parsing API 方式来解析参数，如array_slice的入参解析源码

```
ZEND_PARSE_PARAMETERS_START(2, 4)
		Z_PARAM_ARRAY(input)
		Z_PARAM_LONG(offset)
		Z_PARAM_OPTIONAL
		Z_PARAM_ZVAL(z_length)
		Z_PARAM_BOOL(preserve_keys)
ZEND_PARSE_PARAMETERS_END();


说明
specifier	Fast ZPP API macro	args
|	Z_PARAM_OPTIONAL
a	Z_PARAM_ARRAY(dest)	dest - zval*
A	Z_PARAM_ARRAY_OR_OBJECT(dest)	dest - zval*
b	Z_PARAM_BOOL(dest)	dest - zend_bool
C	Z_PARAM_CLASS(dest)	dest - zend_class_entry*
d	Z_PARAM_DOUBLE(dest)	dest - double
f	Z_PARAM_FUNC(fci, fcc)	fci - zend_fcall_info, fcc - zend_fcall_info_cache
h	Z_PARAM_ARRAY_HT(dest)	dest - HashTable*
H	Z_PARAM_ARRAY_OR_OBJECT_HT(dest)	dest - HashTable*
l	Z_PARAM_LONG(dest)	dest - long
L	Z_PARAM_STRICT_LONG(dest)	dest - long
o	Z_PARAM_OBJECT(dest)	dest - zval*
O	Z_PARAM_OBJECT_OF_CLASS(dest, ce)	dest - zval*
p	Z_PARAM_PATH(dest, dest_len)	dest - char*, dest_len - int
P	Z_PARAM_PATH_STR(dest)	dest - zend_string*
r	Z_PARAM_RESOURCE(dest)	dest - zval*
s	Z_PARAM_STRING(dest, dest_len)	dest - char*, dest_len - int
S	Z_PARAM_STR(dest)	dest - zend_string*
z	Z_PARAM_ZVAL(dest)	dest - zval*
    Z_PARAM_ZVAL_DEREF(dest)	dest - zval*
+	Z_PARAM_VARIADIC('+', dest, num)	dest - zval*, num int
*	Z_PARAM_VARIADIC('*', dest, num)	dest - zval*, num int
```
