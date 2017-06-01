---
layout: post
title: php7扩展开发-数组
category: php
keywords: 扩展
---
实现一个简单的例子  

```
<?php
	$arr = 
	[
	  0 => null,
	  1 => 32,
	  2 => true,
	  3 => 3.14,
	  4 => 'laoniangke',
	  'name' => 'hisen'
	  'phper' => true
	];
```
扩展代码如下

```
PHP_FUNCTION(helloWorld_arr)
{
    array_init(return_value);
    add_next_index_null(return_value);
    add_next_index_long(return_value, 32);
    add_next_index_bool(return_value, 1);
    add_next_index_double(return_value, 3.14);
    add_next_index_string(return_value, "laoniangke");
    add_assoc_string(return_value, "name", "hisen");
    add_assoc_bool(return_value, "phper", 1);
}
```

再来实现一个相对复杂点的功能

```
<?php
	$b = ['a', 'b', 'c', 'd'];
	$tag = '_';
	$result = [];
	foreach($b as $val) {
		foreach($b as $v) {
			$result[] = $val . $tag . $v;
		}
	}
输出：
array (size=16)
  0 => string 'a_a' (length=3)
  1 => string 'a_b' (length=3)
  2 => string 'a_c' (length=3)
  3 => string 'a_d' (length=3)
  4 => string 'b_a' (length=3)
  5 => string 'b_b' (length=3)
  6 => string 'b_c' (length=3)
  7 => string 'b_d' (length=3)
  8 => string 'c_a' (length=3)
  9 => string 'c_b' (length=3)
  10 => string 'c_c' (length=3)
  11 => string 'c_d' (length=3)
  12 => string 'd_a' (length=3)
  13 => string 'd_b' (length=3)
  14 => string 'd_c' (length=3)
  15 => string 'd_d' (length=3)
```
扩展代码如下

```
PHP_FUNCTION(arr_tianchong)
{
    zval *arr, *prefix, *entry, *prefix_entry, value, *tag;
    zend_string *result;
    zend_ulong i;

    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "a|z", &arr, &tag) == FAILURE) {
        RETURN_NULL();
    }
    prefix = arr;
    array_init_size(return_value, zend_hash_num_elements(Z_ARRVAL_P(arr)) * zend_hash_num_elements(Z_ARRVAL_P(arr)));
    i = 0;
    ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(arr), entry) {
        ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(prefix),  prefix_entry) {
                if (Z_TYPE_P(entry) != IS_STRING) {
                        convert_to_string(entry);
                }
                if (Z_TYPE_P(prefix_entry) != IS_STRING) {
                        convert_to_string(prefix_entry);
                }
                if(tag != NULL && Z_TYPE_P(tag) == IS_STRING) {
                        result = strpprintf(0, "%s%s%s", Z_STRVAL_P(prefix_entry), Z_STRVAL_P(tag), Z_STRVAL_P(entry));
                } else {
                        result = strpprintf(0, "%s%s", Z_STRVAL_P(prefix_entry), Z_STRVAL_P(entry));
                }
            ZVAL_STR(&value, result); /*设置值为字符串*/
            zend_hash_index_add(Z_ARRVAL_P(return_value), i++, &value);
        }ZEND_HASH_FOREACH_END();
    }ZEND_HASH_FOREACH_END();
}
```

以下是php内核一些宏API的整理说明

数组

```
数组的初始化
array_init // 初始化一个数组
array_init_size // 初始化一个数组，指定初始化数组的元素个数

zend_hash_num_elements // 获取哈希/数组的元素个数

实例
$arr = array(); // array_init(arr);
$arr[] = NULL; // add_next_index_null(arr);
$arr[] = 42; // add_next_index_long(arr, 42);
$arr[] = true; // add_next_index_bool(arr, 1);
$arr[] = 3.14; // add_next_index_double(arr, 3.14);
$arr[] = 'foo'; // add_next_index_string(arr, "foo", 1);
$arr[] = $myvar; // add_next_index_zval(arr, myvar);
$arr[0] = NULL;	// add_index_null(arr, 0);
$arr[1] = 42; // add_index_long(arr, 1, 42);
$arr[2] = true; // add_index_bool(arr, 2, 1);
$arr[3] = 3.14; // add_index_double(arr, 3, 3.14);
$arr[4] = 'foo'; // add_index_string(arr, 4, "foo", 1);
$arr[5] = $myvar; // add_index_zval(arr, 5, myvar);
$arr['abc'] = NULL; // add_assoc_null(arr, "abc");
$arr['def'] = 711; // add_assoc_long(arr, "def", 711);
$arr['ghi'] = true; // add_assoc_bool(arr, "ghi", 1);
$arr['jkl'] = 1.44; // add_assoc_double(arr, "jkl", 1.44);
$arr['mno'] = 'baz'; // add_assoc_string(arr, "mno", "baz", 1);
$arr['pqr'] = $myvar; // add_assoc_zval(arr, "pqr", myvar);

数组的遍历
ZEND_HASH_FOREACH_VAL(ht, val)
ZEND_HASH_FOREACH_KEY(ht, h, key) 
ZEND_HASH_FOREACH_PTR(ht, ptr)
ZEND_HASH_FOREACH_NUM_KEY(ht, h) 
ZEND_HASH_FOREACH_STR_KEY(ht, key)
ZEND_HASH_FOREACH_STR_KEY_VAL(ht, key, val)
ZEND_HASH_FOREACH_KEY_VAL(ht, h, key, val)

实例
ZEND_HASH_FOREACH_KEY_VAL(Z_ARRVAL_P(arr), num_key, string_key, entry) {
...
Z_STRVAL_P(entry)
}ZEND_HASH_FOREACH_END()
```


类型的判断

```
获取类型的宏
#define Z_TYPE(zval)        (zval).type
#define Z_TYPE_P(zval_p)    Z_TYPE(*zval_p)
#define Z_TYPE_PP(zval_pp)  Z_TYPE(**zval_pp)

类型常量
#define IS_UNDEF                    0
#define IS_NULL                     1
#define IS_FALSE                    2
#define IS_TRUE                     3
#define IS_LONG                     4
#define IS_DOUBLE                   5
#define IS_STRING                   6
#define IS_ARRAY                    7
#define IS_OBJECT                   8
#define IS_RESOURCE                 9
#define IS_REFERENCE                10

/* constant expressions */
#define IS_CONSTANT                 11
#define IS_CONSTANT_AST             12

/* fake types */
#define _IS_BOOL                    13
#define IS_CALLABLE                 14

/* internal types */
#define IS_INDIRECT                 15
#define IS_PTR                      17
```


类型转换

```
ZEND_API void convert_to_long(zval *op);
ZEND_API void convert_to_double(zval *op);
ZEND_API void convert_to_null(zval *op);
ZEND_API void convert_to_boolean(zval *op);
ZEND_API void convert_to_array(zval *op);
ZEND_API void convert_to_object(zval *op);
ZEND_API void _convert_to_string(zval *op);
```

zval取值

```
//操作整数的
#define Z_LVAL(zval)            (zval).value.lval
#define Z_LVAL_P(zval_p)        Z_LVAL(*zval_p)
#define Z_LVAL_PP(zval_pp)      Z_LVAL(**zval_pp)
 
//操作IS_BOOL布尔型的
#define Z_BVAL(zval)            ((zend_bool)(zval).value.lval)
#define Z_BVAL_P(zval_p)        Z_BVAL(*zval_p)
#define Z_BVAL_PP(zval_pp)      Z_BVAL(**zval_pp)
 
//操作浮点数的
#define Z_DVAL(zval)            (zval).value.dval
#define Z_DVAL_P(zval_p)        Z_DVAL(*zval_p)
#define Z_DVAL_PP(zval_pp)      Z_DVAL(**zval_pp)
 
//操作字符串的值和长度的
#define Z_STRVAL(zval)          (zval).value.str.val
#define Z_STRVAL_P(zval_p)      Z_STRVAL(*zval_p)
#define Z_STRVAL_PP(zval_pp)        Z_STRVAL(**zval_pp)
 
#define Z_STRLEN(zval)          (zval).value.str.len
#define Z_STRLEN_P(zval_p)      Z_STRLEN(*zval_p)
#define Z_STRLEN_PP(zval_pp)        Z_STRLEN(**zval_pp)
 
#define Z_ARRVAL(zval)          (zval).value.ht
#define Z_ARRVAL_P(zval_p)      Z_ARRVAL(*zval_p)
#define Z_ARRVAL_PP(zval_pp)        Z_ARRVAL(**zval_pp)
 
//操作对象的
#define Z_OBJVAL(zval)          (zval).value.obj
#define Z_OBJVAL_P(zval_p)      Z_OBJVAL(*zval_p)
#define Z_OBJVAL_PP(zval_pp)        Z_OBJVAL(**zval_pp)
 
#define Z_OBJ_HANDLE(zval)      Z_OBJVAL(zval).handle
#define Z_OBJ_HANDLE_P(zval_p)      Z_OBJ_HANDLE(*zval_p)
#define Z_OBJ_HANDLE_PP(zval_p)     Z_OBJ_HANDLE(**zval_p)
 
#define Z_OBJ_HT(zval)          Z_OBJVAL(zval).handlers
#define Z_OBJ_HT_P(zval_p)      Z_OBJ_HT(*zval_p)
#define Z_OBJ_HT_PP(zval_p)     Z_OBJ_HT(**zval_p)
 
#define Z_OBJCE(zval)           zend_get_class_entry(&(zval) TSRMLS_CC)
#define Z_OBJCE_P(zval_p)       Z_OBJCE(*zval_p)
#define Z_OBJCE_PP(zval_pp)     Z_OBJCE(**zval_pp)
 
#define Z_OBJPROP(zval)         Z_OBJ_HT((zval))->get_properties(&(zval) TSRMLS_CC)
#define Z_OBJPROP_P(zval_p)     Z_OBJPROP(*zval_p)
#define Z_OBJPROP_PP(zval_pp)       Z_OBJPROP(**zval_pp)
 
#define Z_OBJ_HANDLER(zval, hf)     Z_OBJ_HT((zval))->hf
#define Z_OBJ_HANDLER_P(zval_p, h)  Z_OBJ_HANDLER(*zval_p, h)
#define Z_OBJ_HANDLER_PP(zval_p, h)     Z_OBJ_HANDLER(**zval_p, h)
 
#define Z_OBJDEBUG(zval,is_tmp)     (Z_OBJ_HANDLER((zval),get_debug_info)?  \
                        Z_OBJ_HANDLER((zval),get_debug_info)(&(zval),&is_tmp TSRMLS_CC): \
                        (is_tmp=0,Z_OBJ_HANDLER((zval),get_properties)?Z_OBJPROP(zval):NULL)) 
#define Z_OBJDEBUG_P(zval_p,is_tmp) Z_OBJDEBUG(*zval_p,is_tmp) 
#define Z_OBJDEBUG_PP(zval_pp,is_tmp)   Z_OBJDEBUG(**zval_pp,is_tmp)
 
//操作资源的
#define Z_RESVAL(zval)          (zval).value.lval
#define Z_RESVAL_P(zval_p)      Z_RESVAL(*zval_p)
#define Z_RESVAL_PP(zval_pp)        Z_RESVAL(**zval_pp)
```
