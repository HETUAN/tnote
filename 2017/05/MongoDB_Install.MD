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

show dbs    -- 查看数据库列表
use admin   --创建admin数据库，如果存在admin数据库则使用admin数据库
db   ---显示当前使用的数据库名称
db.getName()  ---显示当前使用的数据库名称
db.dropDatabase()  --删当前使用的数据库
db.repairDatabase()  --修复当前数据库
db.version()   --当前数据库版本
db.getMongo()  --查看当前数据库的链接机器地址 
db.stats() 显示当前数据库状态，包含数据库名称，集合个数，当前数据库大小 ...
db.getCollectionNames()   --查看数据库中有那些个集合（表）
show collections    --查看数据库中有那些个集合（表）
db.person.drop()  --删除当前集合（表）person

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

> 查看mongodb是否启动 

netstat -lanp|grep 27017
或者
netstat -nltp | grep 27017


``` bash
[2017-05-19 11:51:55]-->

System.TimeoutException: A timeout occured after 30000ms selecting a server using CompositeServerSelector{ Selectors = WritableServerSelector, LatencyLimitingServerSelector{ AllowedLatencyRange = 00:00:00.0150000 } }. Client view of cluster state is { ClusterId : "1", ConnectionMode : "Automatic", Type : "Unknown", State : "Disconnected", Servers : [{ ServerId: "{ ClusterId : 1, EndPoint : "192.168.15.146:27017" }", EndPoint: "192.168.15.146:27017", State: "Disconnected", Type: "Unknown", HeartbeatException: "MongoDB.Driver.MongoConnectionException: An exception occurred while opening a connection to the server. ---> System.Net.Internals.SocketExceptionFactory+ExtendedSocketException: 由于目标计算机积极拒绝，无法连接。 192.168.15.146:27017
   at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
   at System.Net.Sockets.Socket.Connect(EndPoint remoteEP)
   at System.Threading.Tasks.Task.Execute()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Connections.TcpStreamFactory.<ConnectAsync>d__7.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Connections.TcpStreamFactory.<CreateStreamAsync>d__4.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Connections.BinaryConnection.<OpenHelperAsync>d__48.MoveNext()
   --- End of inner exception stack trace ---
   at MongoDB.Driver.Core.Connections.BinaryConnection.<OpenHelperAsync>d__48.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Servers.ServerMonitor.<HeartbeatAsync>d__27.MoveNext()" }] }.
   at MongoDB.Driver.Core.Clusters.Cluster.ThrowTimeoutException(IServerSelector selector, ClusterDescription description)
   at MongoDB.Driver.Core.Clusters.Cluster.WaitForDescriptionChangedHelper.HandleCompletedTask(Task completedTask)
   at MongoDB.Driver.Core.Clusters.Cluster.<WaitForDescriptionChangedAsync>d__44.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Clusters.Cluster.<SelectServerAsync>d__37.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Bindings.WritableServerBinding.<GetWriteChannelSourceAsync>d__8.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.Core.Operations.BulkMixedWriteOperation.<ExecuteAsync>d__39.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.OperationExecutor.<ExecuteWriteOperationAsync>d__3`1.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.MongoCollectionImpl`1.<ExecuteWriteOperationAsync>d__62`1.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.MongoCollectionImpl`1.<BulkWriteAsync>d__22.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at MongoDB.Driver.MongoCollectionBase`1.<InsertOneAsync>d__36.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.GetResult()
   at Bruce.TestAlading.Repositories.KeyWordsMongoRepository.<Insert>d__1.MoveNext() in E:\TUAN\Vs2017\Bruce.Console\Bruce.TestAlading\Repositories\KeyWordsMongoRepository.cs:line 24


```

遇到上面的问题的话可以绑定0.0.0.0
或者在mongodb.conf文件中增加一行
bind_ip = 0.0.0.0
然后重启