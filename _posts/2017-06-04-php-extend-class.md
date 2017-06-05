---
layout: post
title: php7扩展开发-类
category: php
keywords: 扩展
---

```
<?php
	class myclass{
		private $name = 'hisen';
		public $web = 'laoniangke.com';
		public static $age;
		
		public funciton __construct($age)
		{
			self::$age = $age;
		}
		
		public function showName()
		{
			echo $this->name;
		}
		
		public function showAge()
		{
			echo self::$age;
		}
	}
```


```
zend_class_entry *myclass_ce;

PHP_METHOD(myclass, showName)
{
        zval *name, rv;
        name = zend_read_property(myclass_ce, getThis(), "name", sizeof("name")-1, 0, &rv );
        if (Z_TYPE_P(name) != IS_STRING) {
                convert_to_string(name);
        }
        zend_string *strg;

        strg = strpprintf(0,"%s", Z_STRVAL_P(name));
        RETURN_STR(strg);
}

PHP_METHOD(myclass, showAge)
{
        zval *val;
        zend_ulong age = 0;
        val = zend_read_static_property(myclass_ce, "age", sizeof("age")-1, 0);
        if (Z_TYPE_P(val) != IS_LONG) {
                convert_to_long(val);
                age = Z_LVAL_P(val);
        }

        RETURN_LONG(age);
}

PHP_METHOD(myclass, __construct)
{
        char *arg = NULL;
        size_t arg_len;

        if(zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE)
        {
                return;
        }
        zend_update_static_property_string(myclass_ce, "age", sizeof("age")-1, arg );
}

static zend_function_entry myclass_method[] = {
    PHP_ME(myclass, showName, NULL, ZEND_ACC_PUBLIC)
    PHP_ME(myclass, showAge, NULL, ZEND_ACC_PUBLIC)
    PHP_ME(myclass, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
    { NULL, NULL, NULL }
};

PHP_MINIT_FUNCTION(helloWorld)
{
        zend_class_entry ce;
        INIT_CLASS_ENTRY(ce, "myclass",myclass_method);
        myclass_ce = zend_register_internal_class(&ce TSRMLS_CC);
        zend_declare_property_string(myclass_ce, "web", strlen("web"), "laoniangke.com",  ZEND_ACC_PUBLIC TSRMLS_CC);
        zend_declare_property_string(myclass_ce, "name", strlen("name"), "hisen", ZEND_ACC_PUBLIC TSRMLS_CC);
        zend_declare_property_null(myclass_ce, "age", strlen("age"), ZEND_ACC_PUBLIC|ZEND_ACC_STATIC);

        return SUCCESS;
}
```

访问修饰符

```
ZEND_ACC_PUBLIC    public
ZEND_ACC_PRIVATE   private
ZEND_ACC_PROTECTED  protected
定义static的属性与方法只是在掩码标志中加入ZEND_ACC_STATIC即可
构造函数在掩码标志中加入ZEND_ACC_CTOR即可
析构函数在掩码标志中加入ZEND_ACC_DTOR即可
```



类属性读取

```
zend_read_property
zend_read_static_property  静态属性
```

更新对象的属性

```
ZEND_API int zend_declare_property_ex(zend_class_entry *ce, zend_string *name, zval *property, int access_type, zend_string *doc_comment);
ZEND_API int zend_declare_property(zend_class_entry *ce, const char *name, size_t name_length, zval *property, int access_type);
ZEND_API int zend_declare_property_null(zend_class_entry *ce, const char *name, size_t name_length, int access_type);
ZEND_API int zend_declare_property_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);
ZEND_API int zend_declare_property_long(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);
ZEND_API int zend_declare_property_double(zend_class_entry *ce, const char *name, size_t name_length, double value, int access_type);
ZEND_API int zend_declare_property_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value, int access_type);
ZEND_API int zend_declare_property_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_len, int access_type);

ZEND_API int zend_declare_class_constant(zend_class_entry *ce, const char *name, size_t name_length, zval *value);
ZEND_API int zend_declare_class_constant_null(zend_class_entry *ce, const char *name, size_t name_length);
ZEND_API int zend_declare_class_constant_long(zend_class_entry *ce, const char *name, size_t name_length, zend_long value);
ZEND_API int zend_declare_class_constant_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_bool value);
ZEND_API int zend_declare_class_constant_double(zend_class_entry *ce, const char *name, size_t name_length, double value);
ZEND_API int zend_declare_class_constant_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_length);
ZEND_API int zend_declare_class_constant_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value);

ZEND_API int zend_update_class_constants(zend_class_entry *class_type);

ZEND_API void zend_update_property_ex(zend_class_entry *scope, zval *object, zend_string *name, zval *value);
ZEND_API void zend_update_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zval *value);
ZEND_API void zend_update_property_null(zend_class_entry *scope, zval *object, const char *name, size_t name_length);
ZEND_API void zend_update_property_bool(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_long value);
ZEND_API void zend_update_property_long(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_long value);
ZEND_API void zend_update_property_double(zend_class_entry *scope, zval *object, const char *name, size_t name_length, double value);
ZEND_API void zend_update_property_str(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_string *value);
ZEND_API void zend_update_property_string(zend_class_entry *scope, zval *object, const char *name, size_t name_length, const char *value);
ZEND_API void zend_update_property_stringl(zend_class_entry *scope, zval *object, const char *name, size_t name_length, const char *value, size_t value_length);
ZEND_API void zend_unset_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length);

ZEND_API int zend_update_static_property(zend_class_entry *scope, const char *name, size_t name_length, zval *value);
ZEND_API int zend_update_static_property_null(zend_class_entry *scope, const char *name, size_t name_length);
ZEND_API int zend_update_static_property_bool(zend_class_entry *scope, const char *name, size_t name_length, zend_long value);
ZEND_API int zend_update_static_property_long(zend_class_entry *scope, const char *name, size_t name_length, zend_long value);
ZEND_API int zend_update_static_property_double(zend_class_entry *scope, const char *name, size_t name_length, double value);
ZEND_API int zend_update_static_property_string(zend_class_entry *scope, const char *name, size_t name_length, const char *value);
ZEND_API int zend_update_static_property_stringl(zend_class_entry *scope, const char *name, size_t name_length, const char *value, size_t value_length);
```
