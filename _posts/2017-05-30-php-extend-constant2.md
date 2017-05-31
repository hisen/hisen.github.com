---
layout: post
title: php7扩展开发-扩展中的全局变量
category: php
keywords: 扩展
---
php环境变量配置方式，可以通过以下方式实现配置php.ini，脚本中通过ini_set,以及现在LNMP中php-fpm.conf或者nginx.conf。  
为了学习扩展开发，下面实现类型display_errors的功能，默认是on，可以通过php.ini配置进行关闭。
扩展的全局变量是在php_*.h中定义的，打开文件搜索如下几行代码(以helloWorld为例)  

```
ZEND_BEGIN_MODULE_GLOBALS(helloWorld)
        zend_long  global_value;
        char *global_string;
ZEND_END_MODULE_GLOBALS(helloWorld)
```
扩展的全局变量都在  
ZEND_BEGIN_MODULE_GLOBALS(helloWorld)  
ZEND_END_MODULE_GLOBALS(helloWorld)  
两行代码中定义，这里直接使用默认的  

并且文件还为我们提供了HELLOWORLD_G宏来访问全局变量  

```
#define HELLOWORLD_G(v) ZEND_MODULE_GLOBALS_ACCESSOR(helloWorld, v)

#if defined(ZTS) && defined(COMPILE_DL_HELLOWORLD)
ZEND_TSRMLS_CACHE_EXTERN()
#endif

#endif
```  

接下来开始编写helloWorld.c文件  
声明在头文件中定义的这些全局变量，去掉下面这行的注释  
ZEND_DECLARE_MODULE_GLOBALS

添加自定义INI指令,搜索如下代码,去掉注释  

```
PHP_INI_BEGIN()
    STD_PHP_INI_ENTRY("helloWorld.global_value",      "42", PHP_INI_ALL, OnUpdateLong, global_value, zend_helloWorld_globals, helloWorld_globals)
    STD_PHP_INI_ENTRY("helloWorld.global_string", "foobar", PHP_INI_ALL, OnUpdateString, global_string, zend_helloWorld_globals, helloWorld_globals)
PHP_INI_END()
```

STD_PHP_INI_ENTRY 宏参数表  

```
default_value	如果没有在INI文件中指定，条目的默认值。默认值始终是一个字符串。

modifiable	设定在何种环境下INI条目可以被更改的位域。可以的值是：
• PHP_INI_SYSTEM. 能够在php.ini或http.conf等系统文件更改
• PHP_INI_PERDIR. 能够在 .htaccess中更改
• PHP_INI_USER. 能够被用户脚本更改
• PHP_INI_ALL. 能够在所有地方更改

on_modify	处理INI条目更改的回调函数。你不需自己编写处理程序，使用下面提供的函数。包括：
• OnUpdateInt
• OnUpdateString
• OnUpdateBool
• OnUpdateStringUnempty
• OnUpdateReal

property_name	 应当被更新的变量名

struct_type	变量驻留的结构类型。因为通常使用全局变量机制，所以这个类型自动被定义，类似于zend_myfile_globals。

struct_ptr	全局结构名。如果使用全局变量机制，该名为myfile_globals。
```

使自定义INI条目机制正常工作，你需要分别去掉PHP_MINIT_FUNCTION(helloWorld)中的REGISTER_INI_ENTRIES()和PHP_MSHUTDOWN_FUNCTION(myfile)中的UNREGISTER_INI_ENTRIES()的注释

在php.ini中配置如下信息，扩展中的值将被设置成99

```
; php.ini
helloWorld.global_value = 99
```

编写函数获取常量  

```
PHP_FUNCTION(helloWorld_ini)
{
    RETURN_LONG(HELLOWORLD_G(global_value));
}

PHP_FUNCTION(helloWorld_change_ini)
{
    HELLOWORLD_G(global_value)++;
}

const zend_function_entry helloWorld_functions[] = {
        PHP_FE(helloWorld,      NULL)           /* For testing, remove later. */
        PHP_FE(helloWorld_ini,  NULL)           /* For testing, remove later. */
        PHP_FE(helloWorld_change_ini,   NULL)           /* For testing, remove later. */
        PHP_FE(confirm_helloWorld_compiled,     NULL)           /* For testing, remove later. */
        PHP_FE_END      /* Must be the last line in helloWorld_functions[] */
};
```

测试  

```
<?php
var_dump(helloWorld_ini()); // 99
helloWorld_change_ini();
var_dump(helloWorld_ini()); // 100
```

访问级别    

```
PHP_INI_USER | 可在用户脚本（例如 ini_set()）或 Windows 注册表（自 PHP 5.3 起）以及 .user.ini 中设定
PHP_INI_PERDIR | 可在 php.ini，.htaccess 或 httpd.conf 中设定
PHP_INI_SYSTEM | 可在 php.ini 或 httpd.conf 中设定
PHP_INI_ALL | 可在任何地方设定
```


参考  
http://www.laruence.com/2009/04/28/719.html  
https://github.com/qzfzz/php7-extension-dev-book/blob/master/12.4.md

