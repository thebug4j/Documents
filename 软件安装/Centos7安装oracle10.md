

[TOC]



# 1、安装docker

参考印象笔记中的记录



# 2、拉取oracle

[参考](http://t.csdn.cn/78BFr)

```
docker pull vkanjilal/oracle10g
docker run -p 2222:22 -p 1521:1521 -d -t --name oracle10g klwang/oracle10g
```



# 3、配置修改

```
docker exec -it oracle10g bash
su - oracle
sqlplus / as sysdba
SQL> conn /as sysdba
#授权 (出现ORA-01034: ORACLE not available等一会 5分钟，docker初始化没完成)
SQL> grant sysdba to sys;

#创建用户
SQL> create user tmsUsr identified by tms123;
SQL> grant connect,resource,dba to tmsUsr;

#修改密码规则策略为密码永不过期
SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;

#修改数据库最大连接数据；
SQL> alter system set processes=1000 scope=spfile;

#重启数据库并设置字符集
SQL> shutdown immediate;
SQL> startup mount;
SQL> alter system enable restricted session;
SQL> alter system set job_queue_processes=0;
SQL> alter database open;
SQL> alter database character set internal_use UTF8;
SQL> shutdown immediate;
SQL> startup;
SQL> alter system disable restricted session;
SQL> quit;

#退出sqlplus后执行
lsnrctl start;
lsnrctl reload;
```



# 4、问题排查

#### 1、navicate连不上，报错：ORA-12541: TNS:no listener

```
docker exec -it oracle10g bash
su oracle
查看监听器，看报错信息
lsnrctl start/stop/status

```



#### 2、报错 ORA-01034: ORACLE not available

```
主要是因为oracle安装程序没有给oracle这个可执行程序设置正确的setuid（su - oracle再操作就不会有这个问题）。这样设置一下：
$ cd $ORACLE_HOME/bin
$ chmod 6751 oracle
```

#### 3、中文乱码（在导入数据前）

```
二、修改Oracle数据库编码SQL> select userenv('language') from dual;    #先查看数据库的字符集

SQL> shutdown immediate;

SQL> startup mount;

SQL> alter system enable restricted session;

SQL> alter system set job_queue_processes=0;

SQL> alter database open;

SQL> alter database character set internal_use UTF8;

SQL> shutdown immediate;

SQL> startup

SQL> alter system disable restricted session;
```



# 5、其他

#### 1、导入数据

```
#拷贝dmp到宿主机上，并docker cp命令拷贝到容器内部
docker cp /root/test/oracledb.dmp  oracle10g:/home/oracle/
#登录永不，导入数据
su oracle
sqlplus / as sysdba
SQL> conn /as sysdba

#先创建需要的表空间否则导入会失败
mkdir /u01/app/oracle/product/10.2.0/dbhome2/oracledata
SQL> create tablespace TS_BJWMS datafile '/u01/app/oracle/product/10.2.0/dbhome2/oracledata/TS_BJWMS.DBF' size 2500M autoextend on next 500M maxsize 12000M;

#分配表空间给用户
SQL> ALTER USER tmsUsr DEFAULT TABLESPACE TS_BJWMS;

#退出sqlplus后
imp tmsUsr/tms123 file=/home/oracle/oracledb.dmp log=/home/oracle/oracledb.log full=y ignore=y;
```

#### 2、每天14点30到16点关闭oracle，否则腾讯云数据跑不掉

- 停止oracle10g

```
docker stop oracle10g
```

- 启动oracle10g

```
docker start oracle10g
docker exec -it oracle10g bash
su - oracle

#启动oracle
sqlplus / as sysdba
SQL> conn /as sysdba
SQL> startup;
SQL> quit;

#退出sqlplus后执行，用来启动监听器并重新加载oracle注册信息
lsnrctl start;
lsnrctl reload;

#等30秒后navicate进行连接

```

