---
layout: post
title: Docker下安装各种软件
category: [Linux-Mint,Docker,Redis,Mongodb,PostgreSQL,MySQL,Elasticsearch,GitLab,RabbitMQ,JenKins]
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
docker run --name mysql55 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.5
（说明：--name为别名，
		以后启动/停止用docker start/stop mysql55即可,查看docker ps -a,删除docker rm ID或别名
		-p为端口映射
		-e为参数设置
		-d为镜像目标指向
）
通过Navicat或者HeidiSQL建立连接访问测试是否成功

```

### 安装 Mongodb

```
下载 mongo：
docker pull mongo:latest
下载 mongo-express：
docker pull mongo-express:latest

启动 mongo：
docker run --name mongodb -p 27007:27017 -d mongo
docker run --name mongodb -p 27007:27017 -d mongo --auth

docker exec -it mongodb mongo admin
db.createUser({ user: 'root', pwd: '123456', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

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
docker run --name postgresql -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=123456 -d postgres
通过Navicat或者HeidiSQL建立连接访问测试是否成功

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

```

### 安装 Elasticsearch5.x

```
https://www.elastic.co/guide/en/elasticsearch/reference/5.6/install-elasticsearch.html

docker pull docker.elastic.co/elasticsearch/elasticsearch:5.6.9

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

The Missing Web UI for Elasticsearch：
https://github.com/appbaseio/dejaVu
installed as a chrome extension：
https://chrome.google.com/webstore/detail/dejavu/jopjeaiilkcibeohjdmejhoifenbnmlh

输入http://localhost:9200获取下拉列表
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

### 安装 GitLab（依赖于已经安装好了的postgresql和redis）

```
下载 gitlab：
docker pull sameersbn/gitlab:latest

启动 gitlab：
docker run --name gitlab -d \
    --link postgresql:postgresql --link redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=aaa0bbb1ccc2ddd3eee4' \
    sameersbn/gitlab:latest

用浏览器访问：
http://localhost:10080

会提示修改密码，比如设置为12345678，然后登陆，默认账户是：admin@example.com

登陆进去后创建工程，参考（http://localhost:10080/help/ssh/README）完成SSH Keys添加：
ssh-keygen -t rsa -C "admin@example.com"
复制生成的keygen到Add an SSH key即可:
cat ~/.ssh/id_rsa.pub

```

### 安装 RabbitMQ

```
下载 RabbitMQ：
docker pull rabbitmq:latest

启动 rabbitmq（5672映射出来供外部java客户端访问，15672供7070web访问）：
docker run -d --hostname localhost --name rabbit -p 7070:15672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=123456 rabbitmq:3-management

如果碰到：
Unable to find image 'rabbitmq:3-management' locally

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

### 安装 Jenkins

```
参考：
https://hub.tenxcloud.com/repos/docker_library/jenkins
http://hub.daocloud.io/repos/10e50ad9-7691-4d17-a1ed-c3a0d47d66c0

下载
docker pull daocloud.io/library/jenkins:latest

运行
docker run --name jenkins -p 8081:8080 -p 50000:50000 -v /home/yourname/soft/jenkins_home:/var/jenkins_home -d daocloud.io/library/jenkins:latest

打开
localhost:8081

解锁
cat /home/yourname/soft/jenkins_home/secrets/initialAdminPassword

选择自定义模式后安装插件（非推荐自动安装）：
== 搜索git选择
	Git Parameter Plug-In（根据git版本构建）
	Git plugin 
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
	- Global roles 新建登陆角色并勾选Overall的Read和Job的Create即可
	- Project roles 新建开发角色和测试角色Pattern分别为dev-.*|share-.*和test-.*用来区分可见项目并且勾选Job+Run+SCM项的所有
系统管理 - Manage and Assign Roles - Assign Roles（给用户分配角色） ：
	- Global roles 添加dev和test用户并且都勾选登陆角色
	- Item roles 添加dev勾选开发角色，添加test勾选测试角色
保存重新登陆后，任何人新建dev-或share-开头的项目dev可见，test-开头的项目test可见


安装Maven info plugin：插件管理 - 可选插件 - 过滤Maven info plugin - 勾选直接安装空闲重启
下载http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz
并上传到MAVEN_HOME映射在磁盘的路径并新建maven_home
系统管理 - Global Tool Configuration -Maven 安装 新增maven - 取消自动安装 - 填写/var/jenkins_home/maven_home所在目录

新建maven项目：
	- 丢弃旧的构建 1,1
	-参数化构建过程 - 添加参数 git parameter - mybranch - Parameter Type branch -源码管理 -Branch Specifier (blank for 'any')${mybranch}
	- 构建环境 Send files or execute commands over SSH after the build runs
	- SSH - 全局配置 - SSH remote hosts（提前Credentials-	(global)Add Credentials-username with password）
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
