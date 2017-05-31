---
layout: post
title: php7扩展开发-语言中的超级全局变量
category: php
keywords: 扩展
---
全局作用域的符号表是在调用扩展的RINIT方法(一般都是MINIT方法里)前创建的，并在RSHUTDOWN方法执行后自动销毁  
当用户在PHP中调用一个函数或者类的方法时，内核会创建一个新的符号表并激活之， 这也就是为什么我们无法在函数中使用在函数外定义的变量的原因 （因为它们分属两个符号表，一个当前作用域的，一个全局作用域的）  
symbol _ table元素可以通过EG宏来访问  
当前的符号表可以通过zend _ rebuild _ symbol _ table()取得

```
zend_bool php_hisenking_autoglobal_callback(zend_string *name)
{
    zval hisenking_val;
    ZVAL_STRING(&hisenking_val, "laoniangke.com" );
    zend_set_local_var_str("_HISENKING", sizeof("_HISENKING")-1, & hisenking_val, 0);
    zend_hash_update(&EG(symbol_table), name, &hisenking_val);
    return 0;
}

PHP_MINIT_FUNCTION(helloWorld)
{
        zend_register_auto_global(zend_string_init("_HISENKING", sizeof("_HISENKING") - 1, 1), 0, php_hisenking_autoglobal_callback);

        return SUCCESS;
}
```

具体可以查看php源码的/mian/php _ variables.c中“_ GET”,“_ POST”,“_ COOKIE”,“_ SERVER”,“_ ENV”,“_ REQUEST”,“_ FILES”,超级全局变量的实现方式

Accessor Macro | Associated Data
---- | ---
EG() | 这个宏可以用来访问符号表，函数，资源信息和常量。
CG() | 用来访问核心全局变量。
PG() | PHP全局变量。我们知道php.ini会映射一个或者多个PHP全局结构。举几个使用这个宏的例子：PG(register _ globals), PG(safe _ mode), PG(memory _ limit)
FG() | 文件全局变量。大多数文件I/O或相关的全局变量的数据流都塞进标准扩展出口结构。
