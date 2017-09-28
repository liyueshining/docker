14.04.1-Ubuntu x86_64 GNU/Linux

MySQL

1, install
            apt-get install mysql-server
			
			配置文件路径：/etc/mysql/my.cnf
			数据文件路径：/var/lib/mysql
			日志文件路径：/var/log/mysql
			
2, status
            service mysql status

3, login
            mysql -u root -p

4, remote login
            mysql -h ip -u root -p

5. 设置字符集（以utf8为例）：

　　　　1） 查看当前的编码：show variables like 'character%';

　　　　2)　修改my.cnf，在[client]下添加default-character-set=utf8

　　　　3） 在[server]下添加default-character-set=utf8，init_connect='SET NAMES utf8;'   ---好像没有这一项，加了之后 会导致MySQL 启动不了。

　　　　4） 重启mysql。service mysql restart  or   mysqld restart

6. 设置bind-address，如果是127.0.0.1的话，只能在本机访问MySQL，无法远程访问。
        bind-address = 0.0.0.0

7. 修改mysql最大连接数
        max_connections=1024  设置太大的话 会导致 MySQL 启不来
		
8. 旧数据升级到utf8（旧数据以latin1为例）：

　　　　1） 导出旧数据：mysqldump --default-character-set=latin1 -hlocalhost -uroot -B dbname --tables old_table >old.sql

　　　　2） 转换编码(Linux和UNIX)：iconv -t utf-8 -f gb2312 -c old.sql > new.sql

　　　　　　这里假定原表的数据为gb2312，也可以去掉-f，让iconv自动判断原来的字符集。

　　　　3） 导入：修改new.sql，在插入或修改语句前加一句话："SET NAMES utf8;"，并修改所有的gb2312为utf8，保存。

　　　　　　mysql -hlocalhost -uroot -p dbname < new.sql

　　　　　　如果报max_allowed_packet的错误，是因为文件太大，mysql默认的这个参数是1M，修改my.cnf中的值即可（需要重启mysql)。

9. 支持utf8的客户端：Mysql-Front,Navicat,PhpMyAdmin，Linux Shell（连接后执行SET NAMES utf8;后就可以读写utf8的数据了。5.4设置完毕后就不用再执行这句话了）

10. 备份和恢复

　　　　备份单个数据库：mysqldump -uroot -p -B dbname > dbname.sql

　　　　备份全部数据库：mysqldump -uroot -p --all-databases > all.sql

　　　　备份表： mysqldump -uroot -p -B dbname --table tablename > tablename.sql

　　　　恢复数据库：mysql -uroot -p < name.sql

　　　　恢复表：mysql -uroot -p dbname < name.sql (必须指定数据库) 

MongoDB 2.4.9

1，install
        apt-get install mongodb
		
		The MongoDB instance stores its data files in /var/lib/mongodb 
		and its log files in /var/log/mongodb by default, 
		and runs using the mongodb user account. 
		You can specify alternate log and data file directories in /etc/mongod.conf. 
		See systemLog.path and storage.dbPath for additional information.

        If you change the user that runs the MongoDB process, 
		you must modify the access control rights to the /var/lib/mongodb 
		and /var/log/mongodb directories to give this user access to these directories.
		
2,  restart mongodb
        service mongodb restart
		
3， manage：
        admin web console  http://ip:28017
		user connection port 27017
		
		mongodb shell by  input mongo
		
		and mongod


4， 设置bind_ip

       bind_ip: "0.0.0.0"
	   
5， 使用：

6， 卸载：
    service mongodb stop
	
	apt-get purge mongodb
	
	remove data and log
	sudo rm -r /var/log/mongodb
	sudo rm -r /var/lib/mongodb



		
