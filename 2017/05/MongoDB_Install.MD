# CentOS 7 Install MongoDB 

## 安装
``` bash

### 1. 使用wget下载安装包

wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.2.6.tgz

### 2. 解压 创建数据文件路径 并添加配置文件
tar zxvf mongodb-linux-x86_64-3.2.6.tgz
mv mongodb-linux-x86_64-3.2.6.tgz mongodb
cd mongodb
mkdir db
mkdir logs
cd bin
vi mongodb.conf

### 3. 配置文件内容开始
dbpath=/usr/local/mongodb/db
logpath=/usr/local/mongodb/logs/mongodb.log
port=27017
fork=true
nohttpinterface=true
# 配置文件内容结束 [注意dbpath和logpath的路径要根据自己的mongodb路径进行修改]

### 3. 重新绑定mongodb的配置文件地址和访问IP
/usr/local/mongodb/bin/mongod --bind_ip localhost -f /usr/local/mongodb/bin/mongodb.conf

### 4. 开机自动启动mongodb

vi /etc/rc.d/rc.local
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
重启一下系统测试下能不能自启

#进入mongodb的shell模式 
/usr/local/mongodb/bin/mongo
#查看数据库列表 
show dbs
#当前db版本 
db.version();

```

## 问题解决


# CentOS 6.4 Install MongoDB 
> 如果按照上面的方法安装MongoDB在CentOS 6.4中通常会报错，可能是因为上面的mongodb版本太高的原因，需要安装很多依赖。

所以：

### 1. 下载MongoDB 
``` bash
wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.4.9.tgz
```

### 2. 解压
``` bash
# 解压
tar zxvf mongodb-linux-x86_64-2.4.9.tgz

# 将解压后的文件复制到/usr/local/mongodb目录下面，（大多数源码包安装的程序都在/usr/local/目录下面）
mv mongodb-linux-x86_64-2.4.9 /usr/local/mongodb

# 进入/usr/local/下的mongodb目录
cd mongodb

# 创建数据存储路径
mkdir db

# 创建日志路径
mkdir logs

# 到bin目录下面创建配置文件
cd bin
vi mongodb.conf

# 在VI中输入下面的配置
dbpath=/usr/local/mongodb/db
logpath=/usr/local/mongodb/logs/mongodb.log
port=27017
fork=true
nohttpinterface=true

```

### 3. 重新绑定mongodb的配置文件地址和访问IP
``` bash
/usr/local/mongodb/bin/mongod --bind_ip localhost -f       /usr/local/mongodb/bin/mongodb.conf
```


### 4. 开机自动启动mongodb
``` bash
vi /etc/rc.d/rc.local
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
```

### 5. 使用mongodb(可以重启一下系统测试下能不能自启)
``` bash
#进入mongodb的shell模式 
/usr/local/mongodb/bin/mongo
#查看数据库列表 
show dbs
#当前db版本 
db.version();
```