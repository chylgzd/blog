---
layout: post
title: MySQL相关
category: [Linux-Mint,MySQL,HeidiSql,Navicat,Wine,自启动]
comments: false
---

* content
{:toc}

### 安装mysql
```
sudo apt-get install mysql-server mysql-common
```

#### 卸载mysql
```
sudo rm -rf /var/lib/mysql/  /etc/mysql/
sudo apt-get autoremove mysql* --purge
sudo apt-get remove apparmor
```

#### 配置文件my.cnf目录

```
一般就在以下几个目录内：
/etc/my.cnf
/etc/mysql/my.cnf
$MYSQL_HOME/my.cnf
[datadir]/my.cnf
~/.my.cnf
```

mysql5.6_my.cnf

```properties
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
[client]
default-character-set = utf8
port    = 3306
[mysqld]
character-set-server = utf8
user    = mysql
port    = 3306
max_connections = 500

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
# basedir = .....
# datadir = .....
# port = .....
# server_id = .....
# socket = .....

# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M 

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

### 安装mysql图形化客户端HeidiSql[下载页面](http://www.heidisql.com/download.php)
```
需要wine环境，先安装wine
sudo apt-get install wine
从下载页面下载后用wine方式打开和安装，需要选择系统文件目录安装，
安装成功后在该目录运行程序即可，
发送快捷方式到桌面会存在找不的文件

会自动生成桌面快捷图标(~/.local/share/applications/或者/usr/share/applications/)：
[Desktop Entry]
Name=HeidiSQL
Exec=env WINEPREFIX="/home/yourname/.wine" wine Z:\\\\home\\\\yourname\\\\HeidiSQL\\\\heidisql.exe 
Type=Application
StartupNotify=true
Path=/home/yourname/.wine/dosdevices/z:/home/yourname/HeidiSQL
Icon=768E_heidisql.0
```

###  安装mysql图形化客户端Navicat Premium [下载页面](https://www.navicat.com/download/navicat-premium)
```
从下载页面下载linux安装包并解压到/home任意目录
在解压后目录内执行下面命令安装即可，以后每次启动也是执行这句命令：
./start_navicat

会自动生成桌面快捷图标Navicat.desktop：
[Desktop Entry]
Version=1.0
Type=Application
Name=Navicat
Comment=Navicat
Exec=/home/yourname/navicat/start_navicat
Icon=7817_Navicat.0
Path=/home/yourname/navicat
Terminal=false
StartupNotify=false

修改启动文件解除使用限制（已无效）：

启动一次并配置好数据库连接再关闭退出以便生成此文件：
- ~/.navicat64/user.reg
- ~/.navicat64/system.reg

创建启动修改脚本：
sudo gedit /etc/profile.d/reset_navicat.sh

写入以下内容：
#!/bin/sh

sed -i -e "s/\s[0-9]\{8,10\}\s[0-9]\{4,8\}//g" ~/.navicat64/user.reg
sed -i -e "s/\s[0-9]\{8,10\}\s[0-9]\{4,8\}//g" ~/.navicat64/system.reg

以上方法无用则直接删除文件夹，记得备份连接配置，重新打开程序导入配置即可（有效）：
rm -rf ~/.navicat64
```

### 设置取消自启动
```
uRedhat 提供了chkconfig这个命令来管理系统在不同运行级别下的服务开启/关闭: 
chkconfig ServiceName on/off 并可以用chkconfig --list(两个杠) 查看当前的制定状况。
Ubuntu里没有这个命令，其实也可以不用任何命令简单管理系统服务, 
可以通过改变 /etc/rc*.d（*的取值是从0到6和S）下的启动脚本名来管理服务. 
比如不想让KDM自动启动, 可以这样: 
sudo find /etc/rc* -name *kdm* -exec rm {} /;
也就是把KDM的启动脚本全删掉.
Ubuntu也提供了另外一个简单的命令来实现管理。但首先服务必须已在/etc/init.d目录中存在。如：
添加一个服务: sudo update-rc.d ServiceName defaults
删除一个服务: sudo update-rc.d ServiceName remove
还可以安装另外一个比较强的工具: sudo apt-get install sysv-rc-conf sysvconfig
启动: sudo sysv-rc-conf 它可心配置各服务在各级别上的启动情况.
随时想启动某个服务, 可以这样: sudo /etc/init.d/ServiceName start
比如我要远程登录, 要用ssh服务: sudo /etcinit.d/ssh start (别的系统可能是sshd)
还可以做别的操作: 
start : 启动服务 
stop : 停止服务 
restart : 关闭服务，然后重新启动 
reload : 使服不重新启动而重读配置文件 
status : 提供服务的当前状态 
condrestart : 如果服务锁定，则这个来关闭服务，然后再次启动 
再说一下 linux 运行级别的意思: 通常有这几个 runlevel : 
runlevel　 system state 
0　 halt the system 
1 　 single user mode 
2　 basic multi user mode 
3　 multi user mode 
5　 multi user mode with gui 
6　 reboot the system 
S 　 single user mode runlevel 
命令查看当前运行级别. init 命令改变当前运行级别.

取消自启动：
sudo update-rc.d -f mysql remove
sudo update-rc.d -f docker remove
echo manual | sudo tee /etc/init/mysql.override
echo manual | sudo tee /etc/init/docker.override

开启自启动：
sudo update-rc.d -f mysql defaults
sudo update-rc.d -f docker defaults
sudo rm /etc/init/mysql.override
sudo rm /etc/init/docker.override

/etc/rc2.d/ 目录下S开头的会自启动K则相反，把S改成K就可以了
比如S01bluetooth改成K01bluetooth，就可以取消蓝牙自启动

部分版本Ubuntu中mysql会使用UpStart替代传统的/sbin/init在启动的同时运行服务和设定的任务，需要修改mysql的运行级别：
sudo gedit /etc/init/mysql.conf
将start on runlevel [2345]改为start on runlevel [!0123456]后就可以了

docker.conf:
start on (filesystem and net-device-up IFACE!=lo)改为
start on (filesystem and net-device-up IFACE!=lo) runlevel [!0123456]
```

### MYSQL 相关

#### 账户相关
```
# 查询所有账号信息
mariadb:
SELECT DISTINCT a.`User`,a.`Host`,a.password_expired,a.* FROM mysql.user a;

mysql:
SELECT DISTINCT a.`User`,a.`Host`,a.password_expired,a.password_last_changed,a.password_lifetime,a.* FROM mysql.user a;

# 创建只读用户
mysql / mariadb:
CREATE USER all_r IDENTIFIED BY 'test@1234'
GRANT SELECT ON *.* TO 'all_r'@'%'  (所有库只读)
GRANT SELECT ON mysql.* TO 'xx_r'@'%'  (某个库只读)
GRANT ALL PRIVILEGES ON testdb.* TO 'testdb_rw'@'%'  (某个库所有权限)
GRANT ALL PRIVILEGES ON testdb.tb_demo1 TO 'testdb_rw'@'%'  (某个库某个表的所有权限,注意多个表需多条语句)

# 修改用户密码
mysql / mariadb:
ALTER user 'all_r'@'%' IDENTIFIED BY 'test#1234';

# 刷新权限
mysql / mariadb:
FLUSH PRIVILEGES;

# 清空用户所有权限
REVOKE ALL PRIVILEGES ON *.* FROM 'user_name'@'%'
REVOKE ALL PRIVILEGES ON *.* FROM 'user_name'@'localhost'
(注意如果授权的时候是GRANT ALL PRIVILEGES ON xx.*,回收的时候也要对应REVOKE ALL PRIVILEGES ON xx.*，而不能是ON *.*)
REVOKE ALL PRIVILEGES ON test.* FROM 'user_name'@'%'

```

### MYSQL8相关 

#### 常用

```
更新密码
update user set authentication_string='' where user='root';
ALTER user 'root'@'%' IDENTIFIED BY 'hello@123456';
flush privileges;
```

#### 递归查询
```
#tb_user表某几列id,pid,name,递归查询根节点下的所有子节点
WITH RECURSIVE tmp(id,pid,name) AS
(
	SELECT id,pid,name FROM tb_user WHERE id = 2 -- 这里是根节点ID值
	UNION ALL 
	SELECT u.id,u.pid,u.name FROM tmp t JOIN tb_user u ON t.id = u.pid 
)
SELECT * FROM tmp -- 这里可以再联合其它表关联查询了 LEFT JOIN ...

#或者所有列
WITH RECURSIVE tmp AS
(
	SELECT * FROM tb_user WHERE id = 2  
	UNION ALL 
	SELECT u.* FROM tmp t JOIN tb_user u ON t.id = u.pid 
)
SELECT * FROM tmp

```

