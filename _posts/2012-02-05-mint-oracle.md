---
layout: post
title: Oracle相关
category: [Linux-Mint,Oracle,HeidiSql,Navicat-Oracle,Oracle-SqlDeveloper]
comments: false
---

* content
{:toc}

### 安装Oracle


####  [docker下安装oracle](/blog/2015/02/02/mint-docker-soft/#%E5%AE%89%E8%A3%85-oracle)
```
官方原版下载地址：
https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html

更多参考：https://dev.aliyun.com/detail.html?spm=5176.1972343.2.8.E6Cbr1&repoId=1969

> mkdir -p /data/dev/oracle/dump_dir
> mkdir -p /data/dev/oracle/data_dir

> docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
> docker run --name oracle_11g -p 1521:1521 -v /data/dev/oracle/dump_dir:/home/oracle/app/oracle/oradata/dump_dir -v /data/dev/oracle/data_dir:/home/oracle/app/oracle/oradata/data_dir -d registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

> docker exec -it oracle_11g bash
[oracle@126dx7c /]$ > source /home/oracle/.bash_profile
[oracle@126dx7c /]$ > cd /home/oracle/app/oracle/oradata
[oracle@126dx7c /]$ > su root
Password:helowin
[oracle@126dx7c /]$ > chown -R oracle:oinstall dump_dir data_dir

```

#### 备份

```
> docker exec -it oracle_11g bash
[oracle@126dx7c /]$ > source /home/oracle/.bash_profile

[oracle@126dx7c /]$ > expdp mytestusr/123456@helowin compression=all schemas=mytestusr directory=dumpdir dumpfile=FULL_BACKUP%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.dmp logfile=expdp_%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.log

```

#### 导入expdp数据库

```
把备份文件上传到宿主机dump_dir目录：/data/dev/oracle/dump_dir/expdp_mytestusr_full.dmp

查看与要导入的数据库字符串是否一致,不一致需要修改字符串一致：
> docker exec -it oracle_11g bash
[oracle@126dx7c /]$ > source /home/oracle/.bash_profile
[oracle@126dx7c /]$ > sqlplus /nolog
SQL> connect /as sysdba;
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
[oracle@126dx7c /]$ > impdp 'mytestusr/123456@helowin' full=Y directory=dump_dir dumpfile=expdp.dmp logfile=impdp.log TABLE_EXISTS_ACTION=REPLACE
或
[oracle@126dx7c /]$ > impdp 'mytestusr/123456@helowin' directory=dump_dir dumpfile=expdp.dmp tables=mytestusr.tb_test,tb_test2 TABLE_EXISTS_ACTION=REPLACE remap_schema=produsr:localusr logfile=impdp.log
或
[oracle@126dx7c /]$ > impdp 'mytestusr/123456@helowin' directory=dump_dir dumpfile=expdp.dmp tables=mytestusr.tb_test,tb_test2 TABLE_EXISTS_ACTION=REPLACE logfile=impdp.log

导入结束后日志里有警告的sql，需要手动执行那些报错的sql

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
选择instantclient-basic-win32-11.1.0.7.0

打开Navicat for oracle选择tools - options - OCI 选择解压oracle-install-client后的oci文件重启即可

```

#### Oracle SQL Developer(推荐) [官方页面](https://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)
```
版本限制：无
选择：[Other Platforms](http://download.oracle.com/otn/java/sqldeveloper/sqldeveloper-18.4.0-376.1900-no-jre.zip)

解压安装即可(yx163-Ch17)

比较好用的功能：
工具 - 监视会话
工具 - 实时SQL监视（可以看到哪些sql执行很慢）
查看 - DBA - local_oracle - 存储 - 数据文件
查看 - DBA - local_oracle - 数据泵 - 导入/导出作业
导入向导：表/表空间/方案（用户） - 输入文件目录(连接 - 目录里可以添加定义目录)

```

### Oracle表空间迁移

#### 表空间物理文件从一个磁盘迁移到另一个磁盘
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

#### 单个表从旧表空间迁移到新的表空间
```
tb_test 表 迁移到 表空间 tablespace_new：

迁移tb_test表空间：
ALTER TABLE tb_test MOVE TABLESPACE tablespace_new;

重建tb_test表相关索引：
ALTER INDEX tb_test_primary_key REBUILD TABLESPACE tablespace_new;
ALTER INDEX tb_test_time_index REBUILD TABLESPACE tablespace_new;

```

### Oracle数据备份

#### 数据泵方式备份
```
expdp mytestusr/123456@helowin compression=all schemas=mytestusr directory=dumpdir dumpfile=FULL_BACKUP%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.dmp logfile=expdp_%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.log

如果出现UDE-00010: 已请求多个作业模式, schema 和 tables
这个说明参数schema、参数tablespace、参数table（以及其他数据库对象类型）三者不能同时出现，一次只能出现一个参数的作业模式
有可能是schema和tables同时出现导致

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

### Oracle 创建用户赋权限
```
-- 创建用户
CREATE USER myuser IDENTIFIED BY "test@123456"

-- 赋予登录权限
GRANT CREATE SESSION TO myuser

-- 将另外一个用户的表权限复制给该用户，（dba用户登录执行下列查询导出为sql用dba权限执行即可）
SELECT 'GRANT SELECT ON user1.' || t.table_name  || ' to myuser;'  from user_tables t
SELECT 'GRANT UPDATE ON user1.' || t.table_name  || ' to myuser;'  from user_tables t
SELECT 'GRANT INSERT ON user1.' || t.table_name  || ' to myuser;'  from user_tables t
SELECT 'GRANT DELETE ON user1.' || t.table_name  || ' to myuser;'  from user_tables t

-- 删除用户
DROP USER myuser

```

### Oracle11.2修改密码（有效期，重试被锁等引起）
```
(由于密码有效期的问题,超过期限不能再登录，生产环境数据连接用户密码应该设置长期)

>> 查看密码有效期并修改为永久：
SELECT * FROM dba_profiles WHERE profile='DEFAULT' AND resource_name='PASSWORD_LIFE_TIME';
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
ALTER USER mytest IDENTIFIED BY "123@456";

>> 修改密码：
> sqlplus / as sysdba
或
> sqlplus sys/sysdba
(只能连接本机数据库不需要listener进程)

> sqlplus /nolog;
> conn sys/sys as sysdba;
-- 查看密码重试次数
SQL> SELECT RESOURCE_NAME, LIMIT FROM DBA_PROFILES WHERE RESOURCE_NAME = 'FAILED_LOGIN_ATTEMPTS';
-- 修改密码错误重试次数为无限制，防止用户密码错误被锁
SQL> ALTER profile default limit failed_login_attempts UNLIMITED;
-- 11.2密码修改后延迟很慢，修改后需要重启服务
SQL> ALTER SYSTEM SET EVENT = '28401 TRACE NAME CONTEXT FOREVER, LEVEL 1' SCOPE = SPFILE;
SQL> ALTER USER mytest IDENTIFIED BY "123@456";
SQL> ALTER USER mytest ACCOUNT UNLOCK;
SQL> commit;
SQL> exit;

重启 oracleMytestService服务
重启 oracle监听服务

```

### Oracle 连接超时所产生的问题
```
> tnsping 数据库IP

查看数据库中listener.ora中的inbound_connect_timeout参数值

> lsnrctl (或 lsnrctl status)

LSNRCTL> show inbound_connect_time

LSNRCTL> set inbound_connect_time 0

> vim /xxx/orcl/network/admin/sqlnet.ora

SQLNET.INBOUND_CONNECT_TIMEOUT = 0

LSNRCTL> reload

```

### Oracle相关语句
```
-- 查看会话session相关信息（SERVER如果为DEDICATED则是专有模式）
SELECT SID,SERVER,PORT,TYPE,MACHINE,OSUSER,SERVICE_NAME,SCHEMANAME,USERNAME,PROGRAM,MODULE,EVENT,STATUS,STATE,SQL_ID,SQL_CHILD_NUMBER FROM v$session WHERE TYPE='USER' ORDER BY MACHINE,OSUSER
或
SELECT SID,SERVER,PORT,TYPE,MACHINE,OSUSER,SERVICE_NAME,SCHEMANAME,USERNAME,PROGRAM,MODULE,EVENT,STATUS,STATE,SQL_ID,SQL_CHILD_NUMBER FROM v$session ORDER BY MACHINE,OSUSER

-- 刷新共享池碎片恢复共享池的性能(注意没有使用DBMS_SHARED_POOL.KEEP固定的对象全部被清除)
ALTER SYSTEM FLUSH SHARED_POOL;
防止序列号高速缓存跳号：
exec sys.DBMS_SHARED_POOL.KEEP('seq_myorder_id', 'Q');

--当前的数据库连接数
select count(*) from v$process where program='ORACLE.EXE(SHAD)';

--数据库允许的最大连接数
select value from v$parameter where name ='processes';

--修改最大连接数
alter system set processes = 500 scope = spfile;

-- 最大连接
show parameter processes;

--并发连接数
select count(*) from v$session where status='ACTIVE';

--查询所有锁的sid, pid等信息
SELECT 
  l.inst_id, 
  SUBSTR(L.ORACLE_USERNAME, 1, 8) ORA_USER, 
  SUBSTR(L.SESSION_ID, 1, 3) SID, 
  S.serial#, SUBSTR(O.OWNER||'.'||O.OBJECT_NAME, 1, 40) OBJECT,
  P.SPID OS_PID, 
  DECODE(
    L.LOCKED_MODE, 0, 'NONE', 1, 'NULL', 
    2, 'ROW SHARE', 3, 'ROW EXCLUSIVE', 
    4, 'SHARE', 5, 'SHARE ROW EXCLUSIVE', 
    6, 'EXCLUSIVE', NULL
  ) LOCK_MODE 
FROM 
  sys.GV_$LOCKED_OBJECT L, 
  DBA_OBJECTS O, 
  sys.GV_$SESSION S, 
  sys.GV_$PROCESS P 
WHERE 
  L.OBJECT_ID = O.OBJECT_ID 
  AND l.inst_id = s.inst_id 
  AND L.SESSION_ID = S.SID 
  AND s.inst_id = p.inst_id 
  AND S.PADDR = P.ADDR(+) 
ORDER BY 
  l.inst_id;

--杀掉session多个id,号分隔
alter system kill session '11,22';

--查询触发器
select * from all_triggers where table_name = 'tb_my_demo'
select * from USER_TRIGGERS

-- 添加新列
ALTER TABLE TB_XXX ADD (col_xx NUMBER(12) );
COMMENT ON COLUMN TB_XXX.col_xx IS '测试列';

-- 通过其它表数据创建中间表
CREATE TABLE TMP1_XXX AS 
SELECT * FROM ....

-- 创建表空间

CREATE TABLESPACE tbs_my_test DATAFILE '/home/oracle/app/oracle/oradata/data_dir/tbs_my_test.dbf' SIZE 10G AUTOEXTEND ON NEXT 500M MAXSIZE UNLIMITED;

-- 创建临时表
CREATE GLOBAL TEMPORARY TABLE TMP2_XXX ON COMMIT PRESERVE ROWS AS 
SELECT * FROM ....

-- 创建视图
CREATE OR REPLACE FORCE VIEW "VIEW_XXX" WITH SCHEMABINDING AS 
SELECT * FROM ....

-- 删除表
DROP TABLE TB_XXX cascade constraints PURGE

-- 创建表
CREATE TABLE "TB_XXX" 
(	
    "ID" NUMBER(32) NOT NULL ENABLE, 
    "NAME" VARCHAR2(100 BYTE), 
    "ADDRESS" VARCHAR2(32 BYTE) NOT NULL ENABLE, 
     CONSTRAINT "TB_XXX_PK" PRIMARY KEY ("ID") 
) 
TABLESPACE "TBSP_XXX" ;
COMMENT ON COLUMN "TB_XXX"."ID" IS '编号';
COMMENT ON COLUMN "TB_XXX"."NAME" IS '姓名';
COMMENT ON COLUMN "TB_XXX"."ADDRESS" IS '地址';

-- 使用MERGE INTO关联其它表更新当前表数据(关联关系id必须一对一)
MERGE INTO TB_XX1 x1 USING TB_XX2 x2 ON(x1.id=x2.id) 
WHEN MATCHED THEN UPDATE SET x1.name=x2.name,x1.age=x2.age WHERE x1.id>12

-- 复制表数据从tb1到tb2
INSERT INTO tb1(a,b,c) SELECT a,b,c FROM tb2

-- 查询上一行/下一行数据 
lead/lag/结合over,partition by(相当于group by分组)
SELECT 
    lead(order_price) over(partition by order_type ORDER BY order_id,order_date DESC) last_price,
    lag(order_price) over(partition by order_type ORDER BY order_id,order_date DESC) last_price 
FROM order

-- 查询系统磁盘表空间使用情况（kb）
SELECT Upper(F.TABLESPACE_NAME) tbs_name,D.TOT_GROOTTE_MB total_space,D.TOT_GROOTTE_MB - F.TOTAL_BYTES use_space,F.TOTAL_BYTES free_space FROM
(
    SELECT TABLESPACE_NAME, Round(Sum(BYTES), 2) TOTAL_BYTES 
    FROM SYS.DBA_FREE_SPACE 
    GROUP BY TABLESPACE_NAME
) F, 
(
    SELECT DD.TABLESPACE_NAME, Round(Sum(DD.BYTES), 2) TOT_GROOTTE_MB 
    FROM SYS.DBA_DATA_FILES DD 
    GROUP BY DD.TABLESPACE_NAME
) D 
WHERE D.TABLESPACE_NAME = F.TABLESPACE_NAME 
ORDER  BY 1

-- 替换字符串
SELECT replace(name,'aabb','') FROM test;

-- 替换字符串正则表达式
SELECT REGEXP_REPLACE(photo,'\?t=\w+','') FROM test;


```

#### 树递归查询相关
```
-- 树递归查询（https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries003.htm）
SELECT employee_id, last_name, manager_id, LEVEL
FROM employees 
START WITH employee_id = 100
CONNECT BY PRIOR employee_id = manager_id;

-- 查询某个节点上级第一个数据
SELECT u.uid,u.name,m.mid,m.name FROM user u LEFT JOIN merchant m ON u.uid=m.user_id  
WHERE u.uid != '100'  AND m.mid IS NOT NULL AND rownum =1 
START WITH u.uid='100' 
CONNECT BY NOCYCLE PRIOR u.pid = u.uid

-- 查询某个节点下级第一级节点数据
SELECT u.uid,u.name,m.mid,m.name,level FROM user u LEFT JOIN merchant m ON u.uid=m.user_id  
WHERE m.mid IS NOT NULL AND level<=2  
START WITH u.uid='100' 
CONNECT BY NOCYCLE u.pid = PRIOR u.uid 

```

### 分区表相关
```
1. 原始表 TB_TEST1：
CREATE TABLE "TEST"."TB_TEST1" 
(	
    "ID" VARCHAR2(32 BYTE) NOT NULL ENABLE, 
    "CREATE_TIME" DATE, 
    
    CONSTRAINT "TB_TEST1_PK" PRIMARY KEY ("ID")
)
TABLESPACE "TBS_MYTEST" ;

COMMENT ON COLUMN "TEST"."TB_TEST1"."ID" IS '编号';
COMMENT ON COLUMN "TEST"."TB_TEST1"."CREATE_TIME" IS '创建时间';
COMMENT ON TABLE  "TEST"."TB_TEST1"  IS '测试表';

2. 分区表 TB_TEST2 (创建时间小于2018年5月1号的数据放在P1分区，其它数据按月自动分区)：
CREATE TABLE "TEST"."TB_TEST2" 
(	
    "ID" VARCHAR2(32 BYTE) NOT NULL ENABLE, 
    "CREATE_TIME" DATE, 
    
    CONSTRAINT "TB_TEST2_PK" PRIMARY KEY ("ID")
)
PARTITION BY RANGE(CREATE_TIME)
INTERVAL(NUMTOYMINTERVAL(1，'MONTH'))
(
  PARTITION P1 VALUES LESS THAN(TO_DATE('2018-05-01','YYYY-MM-DD'))
)
TABLESPACE "TBS_MYTEST" ;

COMMENT ON COLUMN "TEST"."TB_TEST2"."ID" IS '编号';
COMMENT ON COLUMN "TEST"."TB_TEST2"."CREATE_TIME" IS '创建时间';
COMMENT ON TABLE  "TEST"."TB_TEST2"  IS '分区表';

3. 复制数据完成数据迁移到分区表 TB_TEST1 > TB_TEST2 (如果有日期分割，最好按条件先<再>=操作一次，避免时间为空或null导致复制数据失败)：

INSERT INTO TB_TEST2(ID, CREATE_TIME) SELECT ID, CREATE_TIME FROM TB_TEST1 WHERE CREATE_TIME < TO_DATE('2018-05-01','YYYY-MM-DD');
INSERT INTO TB_TEST2(ID, CREATE_TIME) SELECT ID, CREATE_TIME FROM TB_TEST1 WHERE CREATE_TIME >= TO_DATE('2018-05-01','YYYY-MM-DD');


4. 分区表相关查询

确定该表是否已经添加了表分区
SELECT partition_name,high_value FROM user_tab_partitions WHERE table_name='TB_TEST2';

查询表分区绑定的字段名称
SELECT * FROM user_part_key_columns WHERE name='TB_TEST2';

查看当前表分区的具体情况
SELECT * FROM user_tab_partitions WHERE table_name='TB_TEST2';

查询表分区绑定的字段的最大值
SELECT max(CREATE_TIME) FROM TB_TEST2;
```

### Oracle 相关问题
```
1053:该服务没有响应启动或控制请求
> sqlplus /nolog;
> conn sys/sys as sysdba;
> select value from v$diag_info where name ='Diag Alert';
diag\rdbms\newdb\newdb\alert

加载修改过后的参数文件tnsnames.ora 和 listener.ora
host=localhost改成计算机名

> lsnrctl stop / start



```




