---
layout: post
title: 监控相关
category: [监控,InfluxDB,Telegraf,Grafana]
comments: false
---

* content
{:toc}

### 系统监控

```
Grafana + InfluxDB + Telegraf 构建系统监控平台
```

#### InfluxDB 时序数据库

```
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.5.2.x86_64.rpm
sudo yum localinstall influxdb-1.5.2.x86_64.rpm

systemctl start influxdb.service
systemctl status influxdb.service

http://127.0.0.1:8083
root/root !SSL

curl http://127.0.0.1:8086/query?q=CREATE+DATABASE+"test"

curl -POST http://127.0.0.1:8086/query --data-urlencode "q=CREATE DATABASE test"
```

#### Telegraf 收集系统和服务的统计数据写入到InfluxDB

```
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.6.2-1.x86_64.rpm
sudo yum localinstall telegraf-1.6.2-1.x86_64.rpm

cat /etc/telegraf/telegraf.conf

[[outputs.influxdb]]
  urls = ["http://127.0.0.1:8086"]  #infulxdb地址
  database = "telegraf" #数据库
  precision = "s"
  timeout = "5s"
  username = "admin" #帐号
  password = "admin" #密码

 [[inputs.cpu]]
  ## Whether to report per-cpu stats or not
  percpu = true
  ## Whether to report total system cpu stats or not
  totalcpu = true

[[inputs.mem]]

systemctl start telegraf.service
systemctl stop telegraf.service
```

#### Grafana 为各类数据源自定义报表显示图表等提供web界面
```
https://grafana.com/grafana/download

yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.1.1-1.x86_64.rpm
systemctl start grafana-server.service
systemctl status grafana-server.service

http://127.0.0.1:3000
admin/admin
```

