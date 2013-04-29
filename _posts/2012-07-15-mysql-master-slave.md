---
layout: post
title: mysql主从复制配置手记
category: mysql
keywords: mysql,mysql主从
---

主从的运用背景：

由于部门CRM数据库表越来越大，从日志上看出部分功能存在读写的锁竞争比较严重，功能必须要进行优化。因此进行一主一从的优化方案，效果非常明显。

Mysql主从复制在大数据量下对数据的分流减少锁表有非常显著的效果，insert在主数据库操作，读在从数据库操作，从而降低锁表的可能。

配置主从的步骤大致如下：

1. 进去master端服务器，给slave端开通一个帐号用于replication操作

grant replication slave on *.* to 'username'@'host'  identified by 'password';

其上的username是用户名；如我给自己取了个'phper'，host是master端的host地址,如‘192.168.1.102’；password既为密码。是否新增成功可以用“select user,password,host from mysql.user;”查看

2. 然后查看是否开启binlog日志以及配置server-id=1。

“show variables like "bin"”;如log_bin一行为on即代表已经开启，如off就得进行mysql的配置文件查看log-bin=mysql-bin去除前面的注释，或在下方写上log-bin=mysql-bin也可

<a href="{{ site:url }}/uploads/2012/07/log_bin.jpg"><img class="alignnone size-full wp-image-416" title="log_bin" src="{{ site:url }}/uploads/2012/07/log_bin.jpg" alt="" width="400" height="195" /></a>

3. 重新生成一份binlog日志文件，或者初始化binlog日志，语句分表是

flush logs；reset master；

4. 查看master状态,其中的file为当前binlog记录的文件名，position为最后记录的位置值

<a href="{{ site:url }}/uploads/2012/07/master_status.jpg"><img class="alignnone size-full wp-image-417" title="master_status" src="{{ site:url }}/uploads/2012/07/master_status.jpg" alt="" width="528" height="99" /></a>

5. 对master进行读锁定，防止数据导出时写操作的不一致；

Flush tables with read lock;

6. 用mysqldump导出数据

7. 解除读锁

unlock tables;

<strong>配置从服务器的步骤：</strong>

1. 测试master开通的帐号是否可以用

2. 导入前面master端的数据

3. 配置slave端数据库配置

server-id=2；开启binlog日志log-bin=mysql-bin

<span style="color: #ff0000;">4.mysql5.1.7之后开始就不支持“master-host” 类似的参数</span>

<span style="color: #ff0000;">开启方式是：</span>

<span style="color: #ff0000;">change master to master_host='192.168.1.102', master_port=3306,master_user='phper', master_password='123456';</span>

5. 开启slave；slave start；

6. 查看slave，其中的Slave_IO_Running与Slave_SQL_Running为yes代表已经成功

<a href="{{ site:url }}/uploads/2012/07/slave_status.jpg"><img class="alignnone size-full wp-image-418" title="slave_status" src="{{ site:url }}/uploads/2012/07/slave_status.jpg" alt="" width="520" height="236" /></a>


其中要注意的地方是主从复制，一般我们都是在从上进行读。如果需要实时性高的数据，那还是的在master主上进行读取，因为数据的replication多少是有点延时的
