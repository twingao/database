下载RPM Repositorie，地址[https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm](https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm)。上传到CentOS。

	mkdir postgresql
	cd postgresql/
	
	rpm -ivh pgdg-redhat-repo-latest.noarch.rpm
	
	yum list | grep postgresql12
	postgresql12.x86_64                         12.1-2PGDG.rhel7           pgdg12
	postgresql12-contrib.x86_64                 12.1-2PGDG.rhel7           pgdg12
	postgresql12-debuginfo.x86_64               12.1-1PGDG.rhel7           pgdg12
	postgresql12-devel.x86_64                   12.1-2PGDG.rhel7           pgdg12
	postgresql12-docs.x86_64                    12.1-2PGDG.rhel7           pgdg12
	postgresql12-libs.x86_64                    12.1-2PGDG.rhel7           pgdg12
	postgresql12-llvmjit.x86_64                 12.1-2PGDG.rhel7           pgdg12
	postgresql12-odbc.x86_64                    12.00.0000-1PGDG.rhel7     pgdg12
	postgresql12-plperl.x86_64                  12.1-2PGDG.rhel7           pgdg12
	postgresql12-plpython.x86_64                12.1-2PGDG.rhel7           pgdg12
	postgresql12-plpython3.x86_64               12.1-2PGDG.rhel7           pgdg12
	postgresql12-pltcl.x86_64                   12.1-2PGDG.rhel7           pgdg12
	postgresql12-server.x86_64                  12.1-2PGDG.rhel7           pgdg12
	postgresql12-test.x86_64                    12.1-2PGDG.rhel7           pgdg12

安装PostgreSQL 12。

	yum install -y postgresql12.x86_64 postgresql12-contrib.x86_64 postgresql12-server.x86_64

初始化数据库。

	/usr/pgsql-12/bin/postgresql-12-setup initdb
	Initializing database ... OK

启动数据库。

	systemctl start postgresql-12
	
	systemctl enable postgresql-12

修改postgreq用户的密码。

	passwd postgres
	更改用户 postgres 的密码 。
	新的 密码：
	无效的密码： 密码是一个回文
	重新输入新的 密码：
	passwd：所有的身份验证令牌已经成功更新。
	
	
	su - postgres
	-bash-4.2$ psql
	psql (12.1)
	输入 "help" 来获取帮助信息.
	
	postgres=# ALTER USER postgres with password '1111';
	ALTER ROLE
	postgres=# \q
	-bash-4.2$ exit
	登出

修改数据库访问权限。

	vi /var/lib/pgsql/12/data/pg_hba.conf
	......
	# TYPE  DATABASE        USER            ADDRESS                 METHOD
	
	# "local" is for Unix domain socket connections only
	local   all             all                                     md5
	# IPv4 local connections:
	host    all             all             127.0.0.1/32            md5
	# IPv6 local connections:
	host    all             all             ::1/128                 md5
	# Allow replication connections from localhost, by a user with the
	# replication privilege.
	#local   replication     all                                     peer
	#host    replication     all             127.0.0.1/32            ident
	#host    replication     all             ::1/128                 ident
	host    all              all             0.0.0.0/0               md5

允许远程访问数据库。

	vi /var/lib/pgsql/12/data/postgresql.conf
	# - Connection Settings -
	
	listen_addresses = '*'                  # what IP address(es) to listen on;
	                                        # comma-separated list of addresses;
	                                        # defaults to 'localhost'; use '*' for all
	                                        # (change requires restart)
	port = 5432                             # (change requires restart)

重启数据库。

	systemctl restart postgresql-12