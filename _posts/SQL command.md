## SQL常用命令

* 查询数据库名称

		show databases;

* 打开一个数据库

		use database-name
database-name替换成要访问的数据库名称。

* 显示当前数据库中已存在的表

		show tables;

* 查看数据使用的编码

		mysql> show variables like '%char%';
默认编码建议全改为utf-8，以便正确处理中文。

## FQA

Q1：安装MySQL里忘记设置root用户密码(或者忘记管理员账号的密码)，连接数据库报错：

	[root@localhost ~]# mysql -u root -p           
	Enter password: 
	ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

Linux环境下的解决方法如下(Windows请参考[重置密码][1])：

第一步：停止数据库服务

	[root@localhost ~]# service mysqld stop
	Stopping mysqld:                                           [  OK  ]

第二步：使用安全启动模式连接数据库，启动的时候不启动授权表。

	[root@localhost ~]# mysqld_safe --skip-grant-tables
	160121 08:48:10 mysqld_safe Logging to '/var/log/mysqld.log'.
	160121 08:48:10 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql

再打开一个终端，访问数据库名为mysql的数据库，使用show命令可以查询到有一张表名叫user

	[root@localhost ~]# mysql
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 3
	Server version: 5.7.10 MySQL Community Server (GPL)

	Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	mysql> use mysql
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	mysql> show tables;


第三步：使用update命令设置password字段

	mysql> update user set Password=password('123456') where USER='root';
	ERROR 1054 (42S22): Unknown column 'Password' in 'field list'

如果出现上面的报错：没有password这个字段，改用authentication_string这个字段：

	mysql> update user set authentication_string=password('123456') where user='root';
	Query OK, 1 row affected, 1 warning (0.00 sec)
	Rows matched: 1  Changed: 1  Warnings: 1

	mysql> flush privileges;
	Query OK, 0 rows affected (0.00 sec)

Q2：修改数据默认编码为utf-8

使用mysql命令，如下示例：

	set character_set_client=utf8;

## 参考链接

[1]:http://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html