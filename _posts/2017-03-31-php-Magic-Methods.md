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

	 // 在对象中调用一个不可访问方法时，__call() 会被调用
	 // $a = new hisen();
	 // $a->showName('king');
    public function __call($name, $arguments) {
        echo 'call';
        echo $name;
        echo var_dump($arguments);
    }
    
    // hisen::doit('king');
    // 在静态上下文中调用一个不可访问方法时，__callStatic() 会被调用
    public static function  __callStatic($name, $arguments) {
        echo '__callStatic';
    }
    
    // 在给不可访问属性赋值时，__set() 会被调用
    // $a->name = 'king';
    public function __set($name, $value)
    {
        echo '__set' . $name . $value;
    }
    
    // 当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用
    // isset($a->name);
    public function __isset($name) 
    {
        echo 'isset' . $name;
    }
    
    // 当对不可访问属性调用 unset() 时，__unset() 会被调用
    // unset($a->name);
    public function __unset($name)
    {
        echo 'unset' . $name;
    }
    
    // serialize() 函数会检查类中是否存在一个魔术方法 __sleep()。如果存在，该方法会先被调用，然后才执行序列化操作
    // $b = serialize($a)
    public function __sleep()
    {
        return array('name');
    }
    
    // unserialize() 会检查是否存在一个 __wakeup() 方法
    // unserialize($b);
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
