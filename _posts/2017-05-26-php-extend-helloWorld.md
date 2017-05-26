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
