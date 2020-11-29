# MySQL 8 安装-CentOS 7

先下载[https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar](https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar)。并上传到CentOS处。

	mkdir mysql
	cd mysql
	
	#将下载的文件上传此目录
	
	tar xvf mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar
	mysql-community-libs-8.0.18-1.el7.x86_64.rpm
	mysql-community-devel-8.0.18-1.el7.x86_64.rpm
	mysql-community-embedded-compat-8.0.18-1.el7.x86_64.rpm
	mysql-community-libs-compat-8.0.18-1.el7.x86_64.rpm
	mysql-community-common-8.0.18-1.el7.x86_64.rpm
	mysql-community-test-8.0.18-1.el7.x86_64.rpm
	mysql-community-server-8.0.18-1.el7.x86_64.rpm
	mysql-community-client-8.0.18-1.el7.x86_64.rpm

先卸载MariaDB。

	#查询
	rpm -pa | grep mariadb
	mariadb-libs-5.5.60-1.el7_5.x86_64
	
	#卸载
	rpm -e mariadb-libs-5.5.60-1.el7_5.x86_64

安装。

	yum install -y net-tools
	rpm -ivh mysql-community-common-8.0.18-1.el7.x86_64.rpm
	rpm -ivh mysql-community-libs-8.0.18-1.el7.x86_64.rpm
	rpm -ivh mysql-community-client-8.0.18-1.el7.x86_64.rpm
	rpm -ivh mysql-community-server-8.0.18-1.el7.x86_64.rpm
	rpm -ivh mysql-community-libs-compat-8.0.18-1.el7.x86_64.rpm

初始化mysql，并启动mysqld服务。

	mysqld initialize --console
	
	systemctl start mysqld

查找临时密码，并登陆mysql。

	grep "temporary password" /var/log/mysqld.log
	2019-12-13T14:26:42.238771Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: D/-A9bpU6)+!
	
	#此处输入密码：D/-A9bpU6)+!
	mysql -uroot -p
	Enter password:
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 8
	Server version: 8.0.18
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql>
	
	#修改临时密码
	mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MySQL8--';
	Query OK, 0 rows affected (0.03 sec)
	
	mysql> exit
	
	#重新登陆
	mysql -uroot -p
	Enter password:
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 8
	Server version: 8.0.18
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql>

因为安全原因，MySQL安装完成后初始只能本地登陆，不能远程登陆。

	#在另一台CentOS上可以在安装一个MySQL客户端测试。
	mysql -h192.168.1.11 -uroot -p
	Enter password:
	ERROR 1130 (HY000): Host '192.168.1.30' is not allowed to connect to this MySQL server

需要修改配置支持远程登陆。

	use mysql;
	
	select user, authentication_string, host from user;
	+------------------+------------------------------------------------------------------------+-----------+
	| user             | authentication_string                                                  | host      |
	+------------------+------------------------------------------------------------------------+-----------+
	| mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	| mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	| mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	| root             | $A$005$5K,]b.7CQUC5wInjKCpCJF6qGgp4yhtsL4NJD7DupqgXTdPBB1zk2 | localhost |
	+------------------+------------------------------------------------------------------------+-----------+
	4 rows in set (0.00 sec)
	
	update user set host='%' where user='root';
	
	mysql> select user, authentication_string, host from user;
	+------------------+------------------------------------------------------------------------+-----------+
	| user             | authentication_string                                                  | host      |
	+------------------+------------------------------------------------------------------------+-----------+
	| root             | $A$005$5K,]b.7CQUC5wInjKCpCJF6qGgp4yhtsL4NJD7DupqgXTdPBB1zk2 | %         |
	| mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	| mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	| mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
	+------------------+------------------------------------------------------------------------+-----------+
	4 rows in set (0.00 sec)
	
	FLUSH PRIVILEGES;

在另一台CentOS的MySQL客户端重新登录一次。

	mysql -h192.168.1.11 -uroot -p
	Enter password:
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 12
	Server version: 8.0.18 MySQL Community Server - GPL
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql>




