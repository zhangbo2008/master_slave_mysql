# master_slave_mysql
主从复制docker实现




2020-06-05,0点12

自己一定要懂架构的东西:
https://www.bilibili.com/video/BV16J411g7zH?p=9
防止以后做项目吃亏.



主从复制



https://www.cnblogs.com/songwenjie/p/9371422.html

docker pull zhangbo2008/mysql57   # 拉取 mysql 5.7
docker run -p 3339:3306 -ti  --name mymysql -e MYSQL_ROOT_PASSWORD=123456  zhangbo2008/mysql57  


vi /etc/mysql/my.cnf
		[mysqld]
		## 同一局域网内注意要唯一
		server-id=100  
		## 开启二进制日志功能，可以随便取（关键）
		log-bin=mysql-bin

/etc/init.d/mysql  start




docker inspect --format='{{.NetworkSettings.IPAddress}}' mymysql


CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';


GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456'  
 
flush privileges; 




# 然后在slave服务器上运行.



docker run -p 3340:3306 -ti --name mymysql2 -e MYSQL_ROOT_PASSWORD=123456  zhangbo2008/mysql57 

vi /etc/mysql/my.cnf
	[mysqld]
	## 设置server_id,注意要唯一
	server-id=101  
	## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
	log-bin=mysql-slave-bin   
	## relay_log配置中继日志
	relay_log=edu-mysql-relay-bin



/etc/init.d/mysql  start

 
 
 
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
 
 
 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456'  
 
flush privileges; 
 
 
stop slave;

change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000004', master_log_pos= 306, master_connect_retry=30;
# 注意上面这个地方必须写3306.值得是容器内部端口号!!!!!!!!


start slave;


show slave status \G;

看到slave_io_running slave_sql_running都是yes 就表示成功了.


