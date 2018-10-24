---
layout: post
title: Oracle相关
category: [Linux-Mint,Oracle,HeidiSql,Navicat-Oracle,Oracle-SqlDeveloper]
comments: false
---

* content
{:toc}

### 安装Oracle
```
官方原版下载地址：
https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html

或者参考docker下如何安装oracle：
https://chylgzd.github.io/blog/2015/02/02/mint-docker-soft/#%E5%AE%89%E8%A3%85-oracle

```

###  安装Oracle图形化客户端


#### Navicat for oracle [官方页面](https://navicat.com/en/download/navicat-for-oracle)
```
版本限制：
Navicat for oracle:11.0.5-linux
Oracle-instant-client:Version 11.1.0.7.0

Oracle-instant-client:

OS-linux：https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html
(OS-ALL:https://www.oracle.com/technetwork/database/database-technologies/instant-client/downloads/index.html)
选择instantclient-basic-win32-11.1.0.7.0 (yx163-Ch17)

打开Navicat for oracle选择tools - options - OCI 选择解压oracle-install-client后的oci文件重启即可

```

#### Oracle SQL Developer(推荐) [官方页面](https://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)
```
版本限制：无
选择：[Other Platforms](http://download.oracle.com/otn/java/sqldeveloper/sqldeveloper-18.3.0.277.2354-no-jre.zip)
解压安装即可

比较好用的功能：
工具 - 监视会话
工具 - 实时SQL监视（可以看到哪些sql执行很慢）
查看 - DBA - local_oracle - 存储 - 数据文件
查看 - DBA - local_oracle - 数据泵 - 导入/导出作业
导入向导：表/表空间/方案（用户） - 输入文件目录(连接 - 目录里可以添加定义目录)

```

### Oracle表空间迁移
```
sqlplus /nolog
SQL> connect /as sysdba;

-- mount model;
SQL> shutdown immediate;
SQL> startup mount;

(归档模式采用offline;非归档模式则采用offline drop;)
SQL> alter database datafile 'C:/mydb/orcl/data/MYTEST/TBS_MYTEST' offline;
SQL> alter database datafile 'C:/mydb/orcl/data/MYTEST/SYSAUX01.DBF' offline;
SQL> alter database datafile 'C:/mydb/orcl/data/MYTEST/SYSTEM01.DBF' offline;
SQL> alter database datafile 'C:/mydb/orcl/data/MYTEST/USERS01.DBF' offline;
SQL> alter database datafile 'C:/mydb/orcl/data/MYTEST/UNDOTBS01.DBF' offline;

SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/TBS_MYTEST' to 'D:/mydb/data/MYTEST/TBS_MYTEST';
SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/SYSAUX01.DBF' to 'D:/mydb/data/MYTEST/SYSAUX01.DBF';
SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/SYSTEM01.DBF' to 'D:/mydb/data/MYTEST/SYSTEM01.DBF';
SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/USERS01.DBF' to 'D:/mydb/data/MYTEST/USERS01.DBF';
SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/UNDOTBS01.DBF' to 'D:/mydb/data/MYTEST/UNDOTBS01.DBF';
SQL> alter database rename file 'C:/mydb/orcl/data/MYTEST/TEMP01.DBF' to 'D:/mydb/data/MYTEST/TEMP01.DBF';(临时表空间只需要这一句)


SQL> alter database open; 

(SYSTEM 表空间文件 1 处于脱机状态 goto ORA-01147)
SQL> shutdown normal;
SQL> startup;

SQL> alter database datafile 1 online;
SQL> alter database open;
(ORA-01113: 文件 1 需要介质恢复)
SQL> recover datafile 'D:/mydb/data/MYTEST/SYSTEM01.DBF';
SQL> alter database open;

SQL> recover datafile 'D:/mydb/data/MYTEST/SYSAUX01.DBF';
SQL> recover datafile 'D:/mydb/data/MYTEST/USERS01.DBF';
SQL> recover datafile 'D:/mydb/data/MYTEST/TBS_MYTEST';
SQL> recover datafile 'D:/mydb/data/MYTEST/UNDOTBS01.DBF';

SQL> alter tablespace SYSAUX online;
SQL> alter tablespace USERS online;
SQL> alter tablespace TBS_MYTEST online;
SQL> alter tablespace UNDOTBS1 online;

```

### Oracle数据备份

#### 数据泵方式备份
```
expdp mytestusr/123456@helowin compression=all schemas=mytestusr directory=dumpdir dumpfile=FULL_BACKUP%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.dmp logfile=expdp_%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.log

```

#### 备份的数据同步到其它机器(win)
```
WinSCP命令式FTP工具：https://winscp.net/eng/index.php

@echo off 
D:
cd D:\myorclbak\sync
winscp /console /command "option batch continue" "option confirm off" "open ftp://yourftpname:yourftppwd@127.0.0.1 -passive" "option transfer binary" "synchronize local D:\myorclbak\sync\data\ /" "exit" /log="D:\myorclbak\sync\log.txt"
cd D:\myorclbak\sync
rem pause
```

#### 恢复导入数据泵的备份
```
把备份文件移动到oracle里定义的dump_dir目录：/data/dev/oracle/dump_dir/expdp_mytestusr_full.dmp

查看与要导入的数据库字符串是否一致,不一致需要修改字符串一致：
(
docker里：
docker exec -it oracle_11g bash
source /home/oracle/.bash_profile
)
sqlplus /nolog
SQL> conn /as sysdba;
SQL> select * from V$NLS_PARAMETERS;
如果线上字符串为ZHS16GBK本地不是则需要修改本地docker数据库：
SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION;
SQL> ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
SQL> ALTER SYSTEM SET AQ_TM_PROCESSES=0;
SQL> ALTER DATABASE OPEN;
SQL> ALTER DATABASE ﻿CHARACTER SET ZHS16GBK ;
ERROR at line 1:
ORA-02231: missing or invalid option to ALTER DATABASE
SQL> ALTER DATABASE CHARACTER SET INTERNAL_USE ZHS16GBK;
Database altered.
SQL> exit;

全部导入
impdp 'mytestusr/123456@helowin' full=Y directory=dump_dir dumpfile=expdp_mytestusr_full.dmp logfile=impdp.log TABLE_EXISTS_ACTION=REPLACE
或
impdp mytestusr/123456@helowin directory=dump_dir dumpfile=expdp_mytestusr_full.dmp tables=mytestusr.tb_test remap_schema=mytestusr_prod:mytestusr logfile=impdp.log

导入结束后日志里有警告的sql，需要手动执行那些报错的sql

```

### Oracle11.2修改密码过程
```
sqlplus /nolog;
SQL> conn sys/sys as sysdba;
-- 查看密码重试次数
SQL> SELECT RESOURCE_NAME, LIMIT FROM DBA_PROFILES WHERE RESOURCE_NAME = 'FAILED_LOGIN_ATTEMPTS';
-- 修改密码错误重试次数为无限制，防止用户密码错误被锁
SQL> ALTER profile default limit failed_login_attempts  UNLIMITED;
-- 11.2密码修改后延迟很慢，修改后需要重启服务
SQL> ALTER SYSTEM SET EVENT = '28401 TRACE NAME CONTEXT FOREVER, LEVEL 1' SCOPE = SPFILE;
SQL> ALTER USER mytest IDENTIFIED BY "123@456";
SQL> ALTER USER mytest ACCOUNT UNLOCK;
SQL> commit;
SQL> exit;

重启 oracleMytestService服务
重启 oracle监听服务

```






