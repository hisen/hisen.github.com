---
layout: post
title: php魔术方法整理
category: php
keywords: 魔术方法
---

```
class hisen{
	 public $name;
    
    public function __construct()
    {
        echo "hello";
    }

    public function __destruct()
    {
        echo "world";
    }

    public function __call($name, $arguments) {
        echo 'call';
        echo $name;
        echo var_dump($arguments);
    }
    
    public static function  __callStatic($name, $arguments) {
        echo '__callStatic';
    }
    
    public function __set($name, $value)
    {
        echo '__set' . $name . $value;
    }
    
    public function __isset($name) 
    {
        echo 'isset' . $name;
    }
    
    public function __unset($name)
    {
        echo 'unset' . $name;
    }
    
    public function __sleep()
    {
        return array('name');
    }
    
    public function __wakeup()
    {
        echo '__wakeup';
    }
    
    // 当一个类被当成字符串时调用
    // $a = new hisen('king');
    // echo $a;
    public function __toString()
    {
        return '__toString';
    }
    
    // 当尝试以调用函数的方式调用一个对象
    // $a = new hisen('king');
    // $a('jin');
    public function __invoke()
    {
        echo '__invoke';
    }
    
    // 当调用 var_export() 导出类时，此静态 方法会被调用
    // eval('$b = ' . var_export($a, true). ';');
    public static function __set_state($an_array)
    {
        echo '__set_state';
    }
    
    // 当复制完成时,如果定义了 __clone() 方法，则新创建的对象（复制生成的对象）中的 __clone() 方法会被调用
    // $b = clone $a;
    public function __clone()
    {
        echo 'clone';
    }
    
    // 当调用var_dump()导出类时，此静态 方法会被调用
    // $a = new hisen();
    // var_dump($a);
    public function __debugInfo()
    {
        return ['__debugInfo'];
    }
}
```
