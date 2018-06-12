---
layout: post
title: 消息队列RabbitMQ相关
category: [RPC,RabbitMQ]
comments: false
---

* content
{:toc}

具体安装请参考docker安装软件相关

### 简单删除队列

```bash
删除/下所有队列
rabbitmqctl list_queues | grep 0 | awk '{print $1}' | xargs -I qn rabbitmqadmin -uroot -p123456 delete queue name=qn

删除xxx-开头的队列
rabbitmqctl list_queues | grep 0 | awk '{print $1}' | grep xxx-* | xargs -I qn rabbitmqadmin -uroot -p123456 delete queue name=qn
```

### 批量清空队列脚本

```
安装成功后访问本地RabbitMQ管理界面，按提示下载rabbitmqadmin脚本：
http://localhost:15672/cli

下载脚本后copy到/usr/local/bin/目录下，调用以下命令测试是否成功：
rabbitmqadmin -H 127.0.0.1 -P 15672 -u root -p 123456 list queues

如果没有错误则在~/bin/目录下新建一个delete_queue脚本并赋权限：
chmod u+x delete_queue
sudo ln -s ~/bin/delete_queue /usr/local/bin/delete_queue

该脚本使用方法：
delete_queue 127.0.0.1 15672 root 123456

默认用户名密码一样则可以不填：
delete_queue 127.0.0.1 15672
```

#### delete_queue脚本

```bash
#!/bin/bash

_host=127.0.0.1
if [[ -n "$1" ]]
then
 _host=$1
fi

_port=15672
if [[ -n "$2" ]]
then
 _port=$2
fi

_user=root
if [[ -n "$3" ]]
then
 _user=$3
fi

_password=123456
if [[ -n "$4" ]]
then
 _password=$4
fi

connection_list=`rabbitmqadmin -H $_host -P $_port -u $_user -p $_password list connections | gawk '{print $2$3$4}'`
queue_list=`rabbitmqadmin -H $_host -P $_port -u $_user -p $_password list queues | gawk '{print $2}'`
exchange_list=`rabbitmqadmin -H $_host -P $_port -u $_user -p $_password list exchanges | gawk '{print $2}'`

for _connection in $connection_list
do
  if [[ $_connection != 'name|user' ]] && [[ $_connection != 'items' ]]
  then
    _connection=`echo $_connection | sed -e 's@->@ -> @g'`
    rabbitmqadmin -H $_host -P $_port -u $_user -p $_password close connection name="$_connection"
    echo '关闭连接：'$_connection
  fi
done

for _queue in $queue_list
do
 if [[ $_queue == queue-* ]]
 then  
    rabbitmqadmin -H $_host -P $_port -u $_user -p $_password delete queue name=$_queue
    echo '删除队列：'$_queue
 fi
done

for _exchange in $exchange_list
do
 if [[ $_exchange == exchange-* ]]
 then
    rabbitmqadmin -H $_host -P $_port -u $_user -p $_password delete exchange name=$_exchange
    echo '删除交换：'$_exchange
 fi
done
```


