---
layout: post
title: sql语句优化之update
category: mysql
keywords: mysql,mysql优化
---

在处理批量更新数据库的信息时，一般我们将sql语句写在一个循环体内。如：
<pre>
foreach($arr as $key=&gt;$val)
{
            $sql = “UPDATE table SET field='{$val}' WHERE id='{$key}'”;
            mysql_query($sql)；
}
</pre>

好的做发可能用pdo或mysqli，预先进行prepare一条sql语句，然后在循环进行执行。如:
<pre>
$sql = “UPDATE table SET field=:field WHERE id=:id”;
$stmt = $db-&gt;prepare($sql);
foreach($arr as $key=&gt;$val)
{
        $stmt-&gt;execute(array(':field'=&gt;$val,':id'=&gt;$key));
}
</pre>

以上或多或少都给数据库增加负担。其实update本省就可以进行批量操作。sql语法如下：
<pre>
UPDATE table
SET field = CASE id
WHEN 1 THEN 3
WHEN 2 THEN 4
WHEN 3 THEN 5
END
WHERE id IN (1,2,3)
</pre>
