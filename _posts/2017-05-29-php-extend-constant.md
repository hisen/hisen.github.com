---
layout: post
title: php7扩展开发-常量
category: php
keywords: 扩展
---

```
PHP_MINIT_FUNCTION(say)
{

		REGISTER_STRINGL_CONSTANT("__SITE__", "www.laoniangke.com", 12, CONST_CS | CONST_PERSISTENT);
        return SUCCESS;
}
```
除了CONST_CS标记，常量的flags字段通常还可以用CONST_PERSISTENT和CONST_CT_SUBST  

CONST_CS标识是否大小写敏感  
在|后的标识位中的标识符说明了该常量的作用域和生命周期。当我们在MINIT中定义常量时，你可能需要在多个请求中使用这个常量，当你在RINIT中定义常量时，这个 常量会在当前请求结束的时候销毁。  
CONST_CT_SUBST我们看注释可以知道其表示Allow compile-time substitution（在编译时可被替换）。 在PHP内核中这些常量包括：TRUE、FALSE、NULL、ZEND_THREAD_SAFE和ZEND_DEBUG_BUILD五个  

为什么要加在PHP_MINIT_FUNCTION内，请参考php生命周期


