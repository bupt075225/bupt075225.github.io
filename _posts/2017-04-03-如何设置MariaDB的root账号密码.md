MariaDB是MySQL的一个分支,Oracle收购MySQL后，有将MySQL闭源的潜在风险，因此MySQL的创始人主导开发了MariaDB来避开这个风险。

遇到需要设置root账号密码的常见场景有：

1.新安装MariaDB通常默认root账号是没有密码，任何人都可以访问数据库；

2.忘记了root账户密码。

只要拥有服务器root访问权限，就可以访问数据库并重新设置它的root账号密码。

这篇文章描述在CentOS 7.2系统中MariaDB-5.5.52版本上设置root账号密码的方法，MariaDB 5.5以上版本使用到的命令有些改变，不在此描述范围内。

## 前提条件

设置MariaDB的root账号密码需要：

- 运行MariaDB的服务器root访问权限。

##步骤1 — 停止数据库服务

>sytemctl stop mariadb

##步骤2 — 跳过权限检查的方式重新数据库服务

在后台运行mysqld_safe命令来启动数据库，启动的时候不加载grant表，无需授权就可以连接到数据库，为了安全再加上`--skip-networking`选项屏蔽掉远程网络连接到数据库的风险：

>mysqld_safe --skip-grant-tables --skip-networking &

接下来连接到数据库:

mysql -u root

##步骤3 — 修改root账号密码

修改密码的方法是使用`ALTER USER`命令,但启动的时候没有加载grant表是用不了这个命令的，使用下面的命令让数据库加载grant表：

>MariaDB [(none)]> FLUSH PRIVILEGES;

执行修改密码的命令：

>MariaDB [(none)]> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_password');

其中`new_password`替换为你要设置的密码。

设置成功后会回显提示：

    Output

    Query OK, 0 rows affected (0.00 sec)

密码修改成功后，需要停止用mysqld_safe命令启动的数据库，再以正常的方式启动。

## 步骤4 — 重启数据库

首先停止步骤2中启动的数据库，使用如下的命令找到数据库进程ID并杀掉进程：

>cat /var/run/mariadb/mariadb.pid

>kill 20439

然后重启数据库服务：

>systemctl start mariadb

最后就可以用新设置的密码连接到数据库：

>mysql -u root -p

## 参考链接

[How To Reset Your MySQL or MariaDB Root Password](https://www.digitalocean.com/community/tutorials/how-to-reset-your-mysql-or-mariadb-root-password)



