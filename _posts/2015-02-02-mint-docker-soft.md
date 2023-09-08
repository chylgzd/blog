---
layout: post
title: Docker下安装各种软件
category: [Linux-Mint,Docker,Redis,Mongodb,PostgreSQL,MySQL,Elasticsearch,GitLab,RabbitMQ,JenKins,Nexus]
comments: false
---

* content
{:toc}

更多参考：

https://hub.docker.com/_/postgres/

https://hub.docker.com/_/mysql/

https://hub.docker.com/_/mongo/

https://hub.docker.com/_/mongo-express/

https://hub.docker.com/_/elasticsearch/

https://hub.docker.com/_/redis/

https://hub.docker.com/r/vieux/redmon/

https://hub.docker.com/_/rabbitmq/

http://www.rabbitmq.com/download.html

https://github.com/rabbitmq/rabbitmq-java-client

https://yq.aliyun.com/articles/57839

### 快速执行安装

```
docker run --name mysql5 -p 3306:3306  -e MYSQL_ROOT_PASSWORD=123456 -d daocloud.io/library/mysql:5.7.4

docker run --name rbt --hostname localhost -p 15672:15672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=123456 -d daocloud.io/rabbitmq:3-management

docker run --name redis -p 6379:6379 -d daocloud.io/library/redis:3.2.9 redis-server --appendonly yes

docker run --name es2 -p 9200:9200 -p 9300:9300 -d index.tenxcloud.com/docker_library/elasticsearch elasticsearch -Des.node.name="my-application"

```

### 安装 MySQL

```
下载 mysql：
docker pull mysql:5.5

启动 mysql：
docker run --name mysql55 -p 3306:3306 -v /etc/localtime:/etc/localtime:ro -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.5
（说明：--name为别名，
		以后启动/停止用docker start/stop mysql55即可,查看docker ps -a,删除docker rm ID或别名
		-p为端口映射
		-e为参数设置
		-d为镜像目标指向
）
通过Navicat或者HeidiSQL建立连接访问测试是否成功


如果镜像是mysql8以上版本：
> docker run --name mysql8 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.17
> docker exec -it mysql8 bash
> mysql -uroot -p123456
mysql> use mysql;
mysql> ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
mysql> flush privileges;
mysql> exit;


ARM架构系统下载mariadb:
docker pull mariadb:10.6.3

docker run -p 13306:3306 --name mariadb -v /etc/localtime:/etc/localtime:ro -v /data/dev/docker_data/mariadb:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=123456 -d mariadb:10.6.3

```

### 安装 Mongodb

```
下载 mongo：
docker pull mongo:4.2

下载客户端GUI：
https://robomongo.org/download
https://download-test.robomongo.org/linux/robo3t-1.3.1-linux-x86_64-7419c406.tar.gz
或
下载 mongo-express：
docker pull mongo-express:latest

启动 mongo：

docker run --name mongodb -p 27007:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=123456 -d mongo:4.2 

docker exec -it mongodb mongo admin
db.createUser({ user: 'superAdmin', pwd: '123456', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

> docker exec -it mongodb mongo
> use admin;
> db.auth("root","123456");
> show dbs;
> use mytest; //不存在则自动创建
> db.createUser({ user: 'myuser', pwd: '123456', roles: [ { role: "dbOwner", db: "mytest" } ] });  //给mytest创建访问用户
> db.getUsers(); // 每个db有独立的user，如果该db下不存在则无法使用程序访问
> db.dropUser("myuser"); //删除用户
> db.dropDatabase("mytest"); // 删除db

启动 robomongo 客户端连接访问即可
或
启动 mongo-express：
docker run --name mongodb-web  --link mongodb:mongo  -p 8081:8081 -d mongo-express

docker run --name mongodb-web  --link mongodb:mongo  -p 8081:8081 -e ME_CONFIG_MONGODB_AUTH_DATABASE=admin  -e ME_CONFIG_MONGODB_AUTH_USERNAME=root -e ME_CONFIG_MONGODB_AUTH_PASSWORD=123456 -d mongo-express
通过浏览器访问：
http://localhost:8081

```

### 安装 PostgreSQL

```
下载 postgres：
docker pull postgres:latest

启动 postgres：
docker run --name postgresql -p 5432:5432 -v /data/dev/docker_data/postgresql/data:/var/lib/postgresql/data -e POSTGRES_USER=root -e POSTGRES_PASSWORD=123456 -d postgres
通过Navicat或者HeidiSQL建立连接访问测试是否成功

```

### 安装 Oracle

```
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
#### 创建用户表空间

```
docker exec -it oracle_11g bash
source /home/oracle/.bash_profile
sqlplus /nolog
SQL> connect /as sysdba;
SQL> CREATE USER mytestusr IDENTIFIED BY 123456;
SQL> GRANT connect,resource,dba,UNLIMITED TABLESPACE TO mytestusr;
SQL> GRANT debug any procedure, debug connect session TO mytestusr;

SQL> CREATE OR REPLACE DIRECTORY dump_dir AS '/home/oracle/app/oracle/oradata/dump_dir';
SQL> GRANT READ, WRITE ON DIRECTORY  dump_dir TO  mytestusr;

SQL> CREATE TABLESPACE tbs_mytestusr DATAFILE '/home/oracle/app/oracle/oradata/data_dir/tbs_mytest.dbf' SIZE 10G AUTOEXTEND ON NEXT 500M MAXSIZE UNLIMITED;
宿主机连接：
rlwrap sqlplus64 mytestusr/123456@//127.0.0.1:1521/helowin

-- 查询所有锁的sid, pid等信息
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

-- alter system kill session '145,20723';
```

#### 备份

```
docker exec -it oracle_11g bash
source /home/oracle/.bash_profile

expdp mytestusr/123456@helowin compression=all schemas=mytestusr directory=dumpdir dumpfile=FULL_BACKUP%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.dmp logfile=expdp_%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.log

```

#### 导入expdp数据库

```
把备份文件上传到宿主机dump_dir目录：/data/dev/oracle/dump_dir/expdp_mytestusr_full.dmp

查看与要导入的数据库字符串是否一致,不一致需要修改字符串一致：
docker exec -it oracle_11g bash
source /home/oracle/.bash_profile
sqlplus /nolog
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
impdp 'mytestusr/123456@helowin' full=Y directory=dump_dir dumpfile=expdp_mytestusr_full.dmp logfile=impdp.log TABLE_EXISTS_ACTION=REPLACE
或
impdp mytestusr/123456@helowin directory=dump_dir dumpfile=expdp_mytestusr_full.dmp tables=mytestusr.tb_test remap_schema=mytestusr_prod:mytestusr logfile=impdp.log

导入结束后日志里有警告的sql，需要手动执行那些报错的sql

```

### 安装 Elasticsearch2.x

```
https://www.elastic.co/guide/en/elasticsearch/reference/2.4/setup.html

下载 Elasticsearch：
docker pull elasticsearch:2.4

启动 elasticsearch：
docker run --name search  -p 9200:9200  -d elasticsearch elasticsearch -Des.node.name="TestNode"
通过浏览器访问：
http://localhost:9200

创建文档（test1:建立的数据库名字，table1:建立的type名字，type与关系数据库的table对应）：
curl -XPUT 'http://localhost:9200/test1/table1/1' -d '{ "first":"dewmobile", "last":"technology", "age":3000, "about":"hello,world", "interest":["basketball","music"]}'

响应：
{"_index":"test1","_type":"table1","_id":"1","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}

访问：
http://localhost:9200/test1/table1/1

修改密码(依赖X-Pack):
宿主> docker exec -it elasticsearch /bin/bash
es > ./bin/elasticsearch-setup-passwords -h #查看密码帮助
es > ./bin/elasticsearch-setup-passwords auto #随机密码
忘记密码(依赖X-Pack):
es > ./bin/elasticsearch-users useradd root -r superuser #创建一个临时超管root
es> #用临时超管权限去修改用户elastic的密码
curl -XPUT -u root:123456 http://localhost:9200/_xpack/security/user/elastic/_password -H "Content-Type: application/json" -d ' { "password": "123456" }'

```

### 安装 Elasticsearch5.x

```
https://www.elastic.co/guide/en/elasticsearch/reference/5.6/install-elasticsearch.html

docker pull docker.elastic.co/elasticsearch/elasticsearch:5.6.9
(废弃，推荐6.8.12
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.8.12
)

/data/es5/config/elasticsearch.yml：
# cluster.name
cluster.name: test-name
# network.host
network.host: 0.0.0.0
# security
xpack.security.enabled: false
# cors
http.port: 9200
http.cors.allow-origin: "*"
http.cors.enabled: true
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
http.cors.allow-credentials: true

docker run --name es5 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /data/es5/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -d docker.elastic.co/elasticsearch/elasticsearch:5.6.9

推荐6.8.12 (参考https://www.elastic.co/guide/en/elasticsearch/reference/6.8/docker.html#docker)
docker run --name es6 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /opt/docker/es6/data:/usr/share/elasticsearch/data -v /opt/docker/es6/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml  -d docker.elastic.co/elasticsearch/elasticsearch:6.8.12

The Missing Web UI for Elasticsearch：
https://github.com/appbaseio/dejaVu
installed as a chrome extension：
https://chrome.google.com/webstore/detail/dejavu/jopjeaiilkcibeohjdmejhoifenbnmlh

输入http://localhost:9200获取下拉列表

安装中文分词插件(注意ES版本号必须对应 https://github.com/medcl/elasticsearch-analysis-ik):
宿主> docker exec -it es6.8.12 /bin/bash
es6> ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.12/elasticsearch-analysis-ik-6.8.12.zip
es6> cd plugins/ #检查是否安装,当然上面安装成功后会也会提示 -> Installed analysis-ik
es6> exit # 退出
宿主> docker restart es6.8.12 #重启
最后spring框架(不需要依赖ik)里设置实体表(@Document)需要分词的字段(analyzer指定存储时分词,searchAnalyzer指定搜索时对参数的分词)如:
@Field(name="remark",type=FieldType.Text,analyzer="ik_max_word",searchAnalyzer="ik_smart")
private String remark;
补充:
ik_max_word和ik_smart的区别:
ik_max_word: 会将文本做最细粒度的拆分,会穷尽各种可能的组合,适合Term Query;
ik_smart: 会做最粗粒度的拆分适合Phrase查询
词库自动热更新(索引需要用户重建,已建立的索引不会使用新词):
宿主> vim nginx/www/elasticsearch_dic.txt
测试动态新增自定义词库
es6> vi /usr/share/elasticsearch/config/analysis-ik/IKAnalyzer.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer Dic</comment>
        <entry key="ext_dict"></entry>
        <entry key="ext_stopwords"></entry>
        <entry key="remote_ext_dict">http://宿主ip:nginx端口(80)/elasticsearch_dic.txt</entry>
        <entry key="remote_ext_stopwords"></entry>
</properties>

es6> exit # 退出重启 
宿主> docker restart es6.8.12 #重启
宿主> docker logs es6 | grep 词典 #查看是否出现:加载词典完毕...
注意: 词库为热更新，但是索引需要用户自己重建,旧的索引不会使用新词,可能导致的问题：
比如'海尔冰箱',原数据以海尔和冰箱拆分,搜索'海尔冰箱'时候可以匹配到'海尔'或'冰箱',
但如果新增热词把'海尔冰箱'作为一个词立即更新了,这时候索引不更新,可能导致'海尔冰箱'无法搜出结果,
因为搜索分词时'海尔冰箱'是个最小Term了,但索引里依旧是旧的'海尔'和'冰箱'

API相关:

分词测试：
POST xxx_index/_analyze
{
	"field": "属性",
	"text": "测试文本"
}
如:
curl -H "Content-Type:application/json" -X POST -d '{"text":"测试文本","analyzer":"ik_smart"}' "http://localhost:19200/_analyze"
或
curl -H "Content-Type:application/json" -X POST -d '{"text":"测试文本","field":"title"}' "http://localhost:19200/xxx_index/_analyze"

修改配置：
PUT _cluster/settings
{
  "persistent": {
    "action.auto_create_index": "false" 
  }
}

创建索引(reindex时先设置action.auto_create_index为false防止自动创建空的mappings然后复制已有配置):
PUT 
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "xx_entity": {
      "properties": {
        "age": {
          "type": "integer"
        },
        "id": {
          "type": "long"
        }
        "remark": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_smart"
        }
      }
    }
  }
}

删除索引
curl -H "Content-Type:application/json" -X DELETE "http://localhost:19200/xxx_index"

```

### 安装Superset数据可视化
```
https://superset.apache.org/docs/intro
https://hub.docker.com/r/apache/superset

> docker pull apache/superset
> docker run -d -p 8080:8088 --name superset apache/superset
> docker exec -it superset superset fab create-admin \
               --username root \
               --firstname Superset \
               --lastname Admin \
               --email admin@superset.com \
               --password 123456
> docker exec -it superset superset db upgrade
> docker exec -it superset superset init
#打开 http://localhost:8080/login/(root/123456)
# 安装驱动包(先提升查看mint-docker更新为apt为国内源方便后续安装驱动)
查看支持驱动列表: https://superset.apache.org/docs/databases/installing-database-drivers/
宿主> docker exec --user root -it superset bash
容器> pip install clickhouse-driver==0.2.0 && pip install clickhouse-sqlalchemy==0.1.6
宿主> docker restart superset
重启后新增数据源链接的时候就有clickhouse选项了
SQLAlchemy URI(native协议或http):
clickhouse+native://{username}:{password}@{hostname}:{port}/{database}[?options…]
或
clickhouse+http://{username}:{password}@{hostname}:{port}/{database}[?options…]
如:
clickhouse+native://root:123456@127.0.0.1:3322/db_test?secure=true
```


### 安装Grafana数据可视化

```
下载地址(可选择平台与版本oss为开源版)：
https://grafana.com/grafana/download?edition=oss&platform=mac

安装文档(如果是压缩包的可解压运行即可,选择对应下载的版本安装文档):
https://grafana.com/docs/grafana/latest/installation/mac/
#可修改默认登录用户名密码: 
> vim ./conf/defaults.ini
[security]
admin_user = admin
admin_password = admin

default_theme = light #默认主题

[snapshots] #快照安全设置
external_enabled = false
public_mode = false

allow_embedding = true #设置可嵌入其它系统(分享快照或链接的时候跨域问题需要设置为true)
[auth.anonymous]
enabled = false #是否允许不需要登录查看(分享链接的时候默认要求登录查看,分享链接的参数带?kiosk&表示全屏)
[auth.gitlab]#设置gitlab认证相关
# 目录: 日志./data/log;插件./data/plugins
mac下启动: 
> ./bin/grafana-server

clickhouse插件(注意不是grafana-clickhouse-datasource插件而是vertamedia-clickhouse-datasource):
https://grafana.com/grafana/plugins/?type=datasource&utm_source=grafana_add_ds
https://grafana.com/grafana/plugins/vertamedia-clickhouse-datasource/
启动后打开设置-插件列表在线安装(推荐)：http://localhost:3000/plugins/vertamedia-clickhouse-datasource
本地手动安装clickhouse插件(不推荐,插件安装完成需重启):
> ./bin/grafana-cli plugins install vertamedia-clickhouse-datasource

nginx配置(需修改[server]的domain和root_url配置,之后需重启):
> vim ./conf/defaults.ini
[server]
#domain = localhost
domain = xx.info
#root_url = %(protocol)s://%(domain)s:%(http_port)s/
root_url = %(protocol)s://%(domain)s/grafana/
#serve_from_sub_path = false
serve_from_sub_path = true
> vim /etc/nginx/conf.d/xx.info.conf
server {
    listen       80;
    server_name xx.info;
    location /grafana/ { #访问首页需包含子路径 -> http://xx.info/grafana/login
        proxy_pass http://localhost:3000/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Real-Port    $remote_port;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
    }
    ....
}

#[auth.jwt]认证
https://grafana.com/docs/grafana/latest/auth/jwt/
https://github.com/grafana/grafana/issues/8198
> vim /data/jwks.json #(新建jwks文件内容可在线随机生成https://8gwifi.org/jwkfunctions.jsp选择HS256)
{
  "keys": [
  {
  "kty": "oct",
  "kid": "74c2028b-148e-4dab-afbc-5cac3df340f2",
  "k": "bTB3F0pzirzvyB1-pR86YugFVOg9P_IapSGbciTJN1M",
  "alg": "HS256"
  }]
}
> vim ./conf/defaults.ini
allow_embedding = true
[auth.jwt]
enabled = true #启用auth.jwt模式
header_name = X-JWT-HEADER
email_claim = sub
username_claim = sub
jwk_set_file = /data/jwks.json #新建的jwks文件
cache_ttl = 60m
expected_claims = {}
key_file =
auto_sign_up = true #自动注册
[auth.anonymous]
enabled = true # 分享Embed HTML iframe时必须
org_name = local_demo_org #这里注意组织名必须一致或提前建好
> 配置完成后校验[auth.jwt]是否生效:
打开https://jwt.io/#debugger-io右边写入
HEADER内容:
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "74c2028b-148e-4dab-afbc-5cac3df340f2"
}
PAYLOAD:
{
  "sub": "demo" #用户名
}
VERIFY SIGNATURE:
{
密钥:bTB3F0pzirzvyB1-pR86YugFVOg9P_IapSGbciTJN1M
}勾选secret base64 encoded(不勾选则error in cryptographic primitive)
完成后把左边生成的JWT字符串放在postman里的分享Embed HTML的head里,key为[auth.jwt]的header_name
发送请求即可在用户管理里看到用户列表新增了demo
 post -> http://localhost:3000/grafana/d-solo/xxx

#其它权限集成自有系统参考：
https://blog.csdn.net/koppel/article/details/106269684
https://grafana.com/docs/grafana/latest/http_api/dashboard_permissions/
https://www.cnblogs.com/yinxy/p/14893595.html
grafana代理:
https://gitee.com/jscode/grafana-proxy?_from=gitee_search
https://www.cnblogs.com/xiaoqi/p/grafana.html
Grafana Oauth统一认证:
https://blog.51cto.com/784687488/2701783
```

### 安装 Redis

```
下载 redis：
docker pull redis:latest
下载 redmon：
docker pull vieux/redmon:latest

启动 redis：
docker run --name redis -d -p 6379:6379 redis redis-server --appendonly yes
启动 redmon：
docker run --name redis-web  -d --link redis:redis -p 4567:4567 vieux/redmon -r redis://redis:6379
通过浏览器访问：
http://localhost:4567

通过RedisClient.jar访问：
新建server，密码空，本地IP端口4567填好即可

其它命令：
redis-server --protected-mode no &
/etc/init.d/redis-server restart
redis-cli shutdown
```

### 通过nodejs应用程序访问Docker内 Redis

先安装node_redis包：

```
npm install redis
```

创建test.js并执行node test.js即可查看结果：

```javascript

var redis   = require('redis');
var client  = redis.createClient('6379', '127.0.0.1');
client.on("error", function(error) {
    console.log(error);
});
client.set("string key", "string val", redis.print);
client.hset("hash key", "hashtest 1", "some value", redis.print);
client.hset(["hash key", "hashtest 2", "some other value"], redis.print);
client.hkeys("hash key", function (err, replies) {
    console.log(replies.length + " replies:");
    replies.forEach(function (reply, i) {
        console.log("    " + i + ": " + reply);
    });
    client.quit();
});

```

### 安装 GitLab

#### 快速安装

```
(redis和postgresql没有映射端口出来，如果有共享的其它程序用可以共享端口共用)
docker run --name redis -d redis:5.0.4 redis-server --appendonly yes
docker run --name postgresql -v /data/dev/docker_data/postgresql/data:/var/lib/postgresql/data -e POSTGRES_USER=root -e POSTGRES_PASSWORD=123456 -d postgres:10.7
docker run --name gitlab11.10.1 -d \
    --link postgresql:postgresql --link redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_HOST=git.yourdomain.com' --env 'GITLAB_PORT=80' --env 'GITLAB_SSH_PORT=10022' --env 'GITLAB_ROOT_EMAIL=test@test.com' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_TIMEZONE=Beijing' \
	--volume /data/dev/docker_data/gitlab/data:/home/git/data \
    sameersbn/gitlab:11.10.1

> vim /etc/nginx/conf.d/git.yourdomain.com.conf
server {
    listen       80;
    server_name git.yourdomain.com;
    location / {
        index index.jsp index.html index.htm;
        proxy_pass http://127.0.0.1:10080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}


ARM机器安装(注意限制内存大小至少4096M)： 
> docker run \
  --detach \
  --restart always \
  --name gitlab-ce \
  --privileged \
  --memory 4096M \
  --publish 10022:22 \
  --publish 10080:80 \
  --publish 10043:443 \
  --hostname gitlab.xx.com \
  --env GITLAB_OMNIBUS_CONFIG=" \
    nginx['redirect_http_to_https'] = false; "\
  --volume /data/dev/docker_data/gitlab/conf:/etc/gitlab:z \
  --volume /data/dev/docker_data/gitlab/logs:/var/log/gitlab:z \
  --volume /data/dev/docker_data/gitlab/data:/var/opt/gitlab:z \
  yrzr/gitlab-ce-arm64v8:14.0.5-ce.0
  

```

#### 安装说明

```
（依赖于已经安装好了的postgresql和redis）
更多参数配置参考：https://github.com/sameersbn/docker-gitlab

下载 gitlab：
docker pull sameersbn/gitlab:11.4.3

启动 gitlab（如果有nginx代理到10080端口，则GITLAB_PORT=80）：
docker run --name gitlab11.8.3 -d \
    --link postgresql:postgresql --link redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_HOST=git.yourdomain.com' --env 'GITLAB_PORT=80' --env 'GITLAB_SSH_PORT=10022' --env 'GITLAB_ROOT_EMAIL=test@test.com' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alpha-numeric-string' \
    --env 'GITLAB_TIMEZONE=Beijing' \
	--volume /data/dev/docker_data/gitlab/data:/home/git/data \
    sameersbn/gitlab:11.8.3

用浏览器访问：
http://localhost:10080

会提示修改密码，比如设置为12345678，然后登陆，默认账户是：admin@example.com或设置的GITLAB_ROOT_EMAIL

登陆进去后创建工程，参考（http://localhost:10080/help/ssh/README）完成SSH Keys添加：
ssh-keygen -t rsa -C "admin@example.com"
复制生成的keygen到Add an SSH key即可:
cat ~/.ssh/id_rsa.pub 

查看git版本及文件存储相关情况：docker inspect gitlab
```

#### 修改git默认域名与端口
```
启动参数GITLAB_HOST和GITLAB_PORT已经设置过需要变更：

进入容器使用：docker exec -it gitlab bash
（如果GITLAB_HOST有新的域名需要更换）：
vim /etc/docker-gitlab/runtime/env-defaults
GITLAB_HOST=${GITLAB_HOST:-git.yourdomain.com}

（如果有nginx代理到10080需要更换gitlab端口80）
vim /etc/docker-gitlab/runtime/config/gitlabhq/gitlab.yml
## GitLab settings
gitlab:
	port:80
```

#### docker迁移根目录需要注意的问题
```
通过docker inspect gitlab查找默认_data进行打包备份
mv  /var/lib/docker/*  /data/docker/root/
chown -R root:root  /data/docker/root
chmod 777 -R /data/docker/root
mount --bind  /data/docker/root  /var/lib/docker
解压备份_data目录到/home/git/data对应--volume映射目录下
如果出现gitlab相关sudouser.so问题，重新安装gitlab镜像即可
```

#### 查找docker数据目录默认文件夹(方便迁移复制数据目录，gitlab-docker版本需要保持一致)

```
> docker inspect postgresql
找到Mounts节点，Destination表示docker容器内目录，Source表示宿主机目录
"Mounts": [
    {
        "Type": "volume",
        "Name": "5a1a07a6cd8d997cb7f86cfe44",
        "Source": "/var/lib/docker/volumes/5a1a07a6cd8d997cb7f86cfe44/_data",
        "Destination": "/var/lib/postgresql/data",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
]

gitlab 迁移新域名成功后,git remote set-url origin ssh://git@git.yourdomain.com:10022/you/test.git

```


### 安装 RabbitMQ

```
下载 RabbitMQ：
docker pull rabbitmq:3.8
docker pull rabbitmq:3.8-management

启动 rabbitmq（5672映射出来供外部java客户端访问，15672供7070web访问）：
docker run -d --hostname localhost --name rabbit -p 7070:15672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=123456 rabbitmq:3.8-management

如果碰到：
Unable to find image 'rabbitmq:3.8-management' locally

会自动下载安装该插件，然后再一次执行启动命令，

然后通过浏览器访问，（登陆用户名密码为上面命令里所设置的root,123456）：
http://localhost:7070

```

通过java客户端访问docker内的RabbitMQ：

maven配置：

```
<dependency>
	<groupId>com.rabbitmq</groupId>
	<artifactId>amqp-client</artifactId>
	<version>3.6.1</version>
</dependency>
```

发送消息-->>：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {

	private final static String QUEUE_NAME = "hello";

	public static void main(String[] argv) throws Exception {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost("localhost");
		factory.setPort(5672);
		factory.setUsername("root");
		factory.setPassword("123456");

		Connection connection = factory.newConnection();
		Channel channel = connection.createChannel();

		channel.queueDeclare(QUEUE_NAME, false, false, false, null);
		String message = "Hello World!";
		channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
		System.out.println(" [x] Sent '" + message + "'");

		channel.close();
		connection.close();
	}
}
```

运行该java客户端再访问http://localhost:7070/#/queues就可以看到Queue hello

接受消息<<---：

```java
import com.rabbitmq.client.*;
import java.io.IOException;

public class Recv {

	private final static String QUEUE_NAME = "hello";

	public static void main(String[] argv) throws Exception {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost("localhost");
		factory.setPort(5672);
		factory.setUsername("root");
		factory.setPassword("123456");

		Connection connection = factory.newConnection();
		Channel channel = connection.createChannel();

		channel.queueDeclare(QUEUE_NAME, false, false, false, null);
		System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

		Consumer consumer = new DefaultConsumer(channel) {
			@Override
			public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
				String message = new String(body, "UTF-8");
				System.out.println(" [x] Received '" + message + "'");
			}
		};
		channel.basicConsume(QUEUE_NAME, true, consumer);
	}
}
```

### 安装 RocketMQ 图形界面

```
docker pull apacherocketmq/rocketmq-console:2.0.0

初次启动
docker run --name rocketmq-console -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.0.113:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8080:8080 -t apacherocketmq/rocketmq-console:2.0.0

下次启动/停止
docker start/stop rocketmq-console
```

### 安装 Jenkins

```
参考：
https://hub.tenxcloud.com/repos/docker_library/jenkins
http://hub.daocloud.io/repos/10e50ad9-7691-4d17-a1ed-c3a0d47d66c0

下载
docker pull jenkins/jenkins(推荐)
docker pull jenkins:2.60.3
docker pull daocloud.io/library/jenkins:latest

运行 (-p 50000:50000)
docker run --name jenkins -p 8081:8080 -v /data/dev/docker_data/jenkins/data:/var/jenkins_home -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai -d jenkins:2.60.3
进入容器:
(jenkins用户)> docker exec -it jenkins bash
(root用户)> docker exec -it -u root jenkins bash

打开
localhost:8081
server {
    listen       80;
    #listen       443 ssl;
    server_name jenkins.xxx.net;
    ignore_invalid_headers off;#防错误过滤掉头参数
    location ~ ^/(WEB-INF)/ {
        deny all;
    }
    error_page 404 =404 @404;
    location @404 {
         default_type application/json;
         return 502 '{"code":"404","msg":"nginx-404","success":false,"result":null}';
    }
    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        access_log off;
        #expires 1d;#proxy_pass不能缓存
    }
}

若是war方式:
启动jenkin时的脚本为(设置JENKINS_HOME目录)：
#!/bin/bash
export JENKINS_HOME='/usr/local/soft/jenkins/jenkins_wk_home'
nohup /usr/bin/java -jar -Djava.security.egd=file:/dev/urandom -Dhudson.util.ProcessTree.disable=true jenkins.war  > /dev/null 2>&1 &
exit 0


解锁
cat /home/yourname/soft/jenkins_home/secrets/initialAdminPassword

选择自定义模式后安装插件（非推荐自动安装）：
== 搜索git选择
	Git Parameter Plug-In（根据git版本构建）
	Git plugin 
== 搜索 Maven Integration (构建maven项目)
== 搜索 nodejs plugin (构建npm项目,全局配置tool自定义下载nodejs包路径后构建选Provide Node&npm bin/folder to PATH)
== 搜索ssh选择
	Publish Over SSH
	SSH plugin （执行远程脚本）
== Build Tools不需要则去掉无用插件
	Ant Plugin
	Gradle Plugin （执行远程脚本）
== User Management and Security选择
	Role-based Authorization Strategy  （用户权限角色管理）
== Notifications and Publishing不需要则去掉无用插件
	Email Extension Template Plugin
	Email Extension Plugin
	Mailer Plugin
 
创建第一个管理员账户填写完所有信息后进入后他管理
打开插件管理/pluginManager/advanced修改更新中心URL
http://updates.jenkins-ci.org/update-center.json
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json

权限测试Configure Global Security - 访问控制 - 授权策略启用Role-Based Strategy：
系统管理 - 管理用户 - 新建用户dev和test - 填写完毕；
系统管理 - Manage and Assign Roles - Manage Roles（管理角色） ：
	- Global roles 新建访问角色勾选对应:
	Overall -> Read
	凭据 -> 不用
	Job(构建) -> Build / Cancel / Discover / Read
	Run -> 不用
	View -> Read
	
	- Global roles 新建登陆角色并勾选Overall的Read和Job的Create即可
	- Project roles 新建开发角色和测试角色Pattern分别为dev-.*|share-.*和test-.*用来区分可见项目并且勾选Job+Run+SCM项的所有
系统管理 - Manage and Assign Roles - Assign Roles（给用户分配角色） ：
	- Global roles 添加dev和test用户并且都勾选登陆角色
	- Item roles 添加dev勾选开发角色，添加test勾选测试角色

系统管理 -> System Log -> Log Levels（日志级别）：
javax.jmdns ： off （关闭日志大的问题）

保存重新登陆后，任何人新建dev-或share-开头的项目dev可见，test-开头的项目test可见


安装Maven info plugin：插件管理 - 可选插件 - 过滤Maven info plugin - 勾选直接安装空闲重启
下载http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz
并上传到MAVEN_HOME映射在磁盘的路径并新建maven_home
系统管理 - Global Tool Configuration -Maven 安装 新增maven - 取消自动安装 - 填写/var/jenkins_home/maven_home所在目录

新建maven项目：
	- 丢弃旧的构建 Discard old builds: Days to keep builds值1,Max # of builds to keep值1
	-参数化构建过程 - 添加参数 git parameter - mybranch - Parameter Type branch -源码管理 -Branch Specifier (blank for 'any')${mybranch}
	- 构建环境 Send files or execute commands over SSH after the build runs
	- SSH - 全局配置 - SSH remote hosts（提前Credentials-	(global)Add Credentials-username with password）

⚠️注意：当构建步骤Post Steps选择 Send files or execute commands over SSH 执行远程脚本时：
SSH Server需要提前在Configure System系统配置里先配置好远程机器;
(注意远程机器如果使用私钥必须使用下面方式jenkins才能识别私钥
ssh-keygen -m PEM -t rsa -b 4096 -C "xxx-jenkins@dev-docker.com"
)
⚠️git clone慢则选择Additional Behaviours：Advanced clone behaviours
 - 去掉 Fetch tags
 - 去掉 Honor refspec on initial clone
 - Timeout (in minutes) for clone and fetch operations值设置为5
 - 勾选浅克隆 Shallow clone 且 浅克隆深度Shallow clone depth值设置为1

Transfers 文件传输设置 :
	Source files填写maven构建编译后的jar: **/springboot-xxx.jar
	Remote directory: 上传上述jar到远程目录(注意会自动拼接SSH Server所配目录下)
	Exec command: sh /deploy.sh(上传文件成功后执行远程机器的一些后续如启动类的脚本,注意最好带完整路径否则失败如nohup /bin/java /xxx/springboot-xxx.jar >/dev/null 2>&1 &  ,如果需要打印日志需要勾选Advanced -> Verbose output in console)
```

### 安装 私有镜像库registry

```
参考：
http://www.tuicool.com/articles/vMZZveM
http://technolo-g.com/generate-ssl-for-docker-swarm/
https://yq.aliyun.com/articles/57310

修改 /etc/ssl/openssl.cnf
[ v3_req ]
subjectAltName = @alt_names
[ v3_ca ]
subjectAltName = @alt_names

[ alt_names ]
# The IPs of the Docker and Swarm hosts
IP.1 = 192.168.0.22

服务主机流程：
cd ~/;
mkdir registry && cd registry && mkdir certs && cd certs;
openssl req -x509 -days 3650 -subj '/CN=192.168.0.22/' -nodes -newkey rsa:2048 -keyout registry.key -out registry.crt;

cd ~/registry&& mkdir auth;
docker run --entrypoint htpasswd registry:latest -Bbn root 123456 > auth/htpasswd;

docker run --name registry -p 5000:5000 --restart=always  \
		-v `pwd`/auth:/auth  \
		-e "REGISTRY_AUTH=htpasswd"  \
		-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"  \
		-e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd"  \
		-v `pwd`/certs:/certs  \
		-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt  \
		-e REGISTRY_HTTP_TLS_KEY=/certs/registry.key  \
		-v ~/soft/_datas/registry/docker/registry:/var/lib/registry -d registry:latest


sudo mkdir -p /etc/docker/certs.d/192.168.0.22:5000
sudo cp ~/registry/certs/registry.crt /etc/docker/certs.d/192.168.0.22:5000

客户机配置
sudo mkdir -p /etc/docker/certs.d/192.168.0.22:5000
sudo scp -r bzt@192.168.0.22:~/registry/certs/registry.crt /etc/docker/certs.d/192.168.0.22:5000

/etc/default/docker
DOCKER_OPTS="--insecure-registry 192.168.0.22:5000"

客户连接到服务操作：
docker login 192.168.0.22:5000

docker tag redis localhost:5000/redis
docker push localhost:5000/redis
docker pull localhost:5000/redis

docker logout 192.168.0.22:5000

/**
sudo cp /etc/docker/certs.d/192.168.0.22:5000/registry.crt /usr/local/share/ca-certificates/192.168.0.22.crt
sudo update-ca-certificates
*/

```

### 安装nexus，maven私服

```
> docker pull sonatype/nexus3:3.14.0
> docker run -d -p 9000:8081 --name nexus -v /data/dev/docker_data/nexus:/nexus-data sonatype/nexus3:3.14.0
> vim /etc/nginx/conf.d/nexus.yourdomain.com.conf
server {
    listen       80;
    server_name nexus.yourdomain.com;
    location / {
        index index.jsp index.html index.htm;
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
默认密码: admin / admin123 (或cat /data/dev/docker_data/nexus/admin.password)
其它查看 2012-13-java-maven.md文档

ARM架构安装:
> docker run -d -p 9000:8081 --name nexus -v /data/dev/docker_data/nexus:/nexus-data klo2k/nexus3

```

### 安装showdoc，文档服务

```
> docker pull star7th/showdoc:v2.4.11
> docker run --name showdoc -p 4999:80 -v /data/dev/docker_data/showdoc/html:/var/www/html/ -d star7th/showdoc:v2.4.11
> vim /etc/nginx/conf.d/showdoc.yourdomain.com.conf
server {
    listen       80;
    server_name showdoc.yourdomain.com;
    location / {
        index index.jsp index.html index.htm;
        proxy_pass http://127.0.0.1:4999;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}

```