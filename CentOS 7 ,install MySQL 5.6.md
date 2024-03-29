### CentOS 7 ，安装MySQL 5.6
由于MySQL已经被Oracle收购了，而且有收费趋势，
所以CentOS 7已经把MySQL从预装的软件包中删除，并使用mariadb这个开源数据库替代。
mariadb与MySQL 是同一厂家的，所以底层会用到有冲突的依赖库，需要卸载mariadb再安装MySQL。

1. 卸载mariadb
    - 列出已安装的mariadb
	rpm -qa | grep mariadb

    - 卸载mariadb
	rpm -e mariadb-XXXXXXXX

	如果报错：依赖检测失败：
	```
	libmysqlclient.so.18()(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要
	libmysqlclient.so.18(libmysqlclient_18)(64bit) 被 (已安裝) postfix-2:2.10.1-6.el7.x86_64 需要
	```
    - 强制卸载 nodeps
	```
	rpm -e --nodeps mariadb-XXXXXXX
	```
    - 清理残余垃圾文件
	```
	rm -rf /var/lib/mysql
	rm -rf /etc/my.cnf
	```


2. 从mysql官网下载rpm安装包(server与client)，并安装

    rpm -ivh MySQL-server-5.6.30-1.linux_glibc2.5.i386.rpm 

    - 如果报告类似以下错误：
	```
	warning: MySQL-server-5.6.30-1.linux_glibc2.5.i386.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
	error: Failed dependencies:
		libaio.so.1 is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
		libaio.so.1(LIBAIO_0.1) is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
		libaio.so.1(LIBAIO_0.4) is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
		libc.so.6 is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
		libc.so.6(GLIBC_2.0) is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
		libc.so.6(GLIBC_2.1) is needed by MySQL-server-5.6.30-1.linux_glibc2.5.i386
	```
	只需挨个安装所需依赖即可
	```
	yum install  libaio.so.1
	yum install  libc.so.6
	........
	```

    - 安装依赖库的时候，可能会出现某个库的多个版本相互冲突(经常发生在64位的系统中安装32位的软件时)：
	```
	Protected multilib versions: libstdc++-4.8.5-44.el7.i686 != libstdc++-4.8.5-16.el7.x86_64
	```
	i686是32的系统(存放在在lib目录下)，x86_64是64位系统(在lib64目录下)：
	yum  update libstdc++-4.8.5-16.el7.x86_64
	或者
	yum -y install  xxxxx库 --setopt=protected_multilib=false

    - 如果报告以下错误：
	```
	FATAL ERROR: please install the following Perl modules before executing /usr/bin/mysql_install_db:
	Data::Dumper
	```
	缺少autoconf库：
	yum -y install autoconf
	继续安装MySQL：
	/usr/bin/mysql_install_db --user=mysql



3. 启动mysql
    - 启动服务
	service mysql start

    - 登录
	首次登陆
	mysql -uroot -p
    - 找不到mysql命令:
	安装MySQL的client客户端
	rpm -ivh MySQL-client-5.6.30-1.linux_glibc2.5.i386.rpm
    - 如果报错：依赖检测失败：
	yum install libncurses.so.5

    - 安装后可以登录
	首次登陆需要密码在：/root/.mysql_secret，安装server的过程中会有提示信息
	登录之后，必须先改密码，否则任何功能都不能用：
	set password for 'root'@'localhost'=password('123456');
	root换成你的用户名，123456换成你的密码

	也可以在首次登陆之前修改密码：
	/usr/bin/mysqladmin -u root password '你的new-password'
	/usr/bin/mysqladmin -u root -h 192.168.10.11 password '你的new-password'



4. 远程访问：
    - 设置远程访问权限：
grant all privileges on *.* to root@'远程的hostIP' identified by '远程用户密码';

    - 设置之后，如果没有起效，可以在mysql中刷新缓存：
flush privileges;

    - 关闭防火墙：
临时关闭
systemctl stop firewalld
永久关闭
systemctl disable firewalld
查看状态
systemctl status firewalld
开启
service firewalld start
重启
service firewalld restart
