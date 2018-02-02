## spring boot jar 方式部署

### 运行环境
1. java
2. CentOS7

### 具体步骤
1. 在mvn中引入标签并修改发布类型为jar
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

-- 修改发布类型
<packaging>jar</packaging>

-- 如果需要可以再application.properties 中修改端口号
server.port=89
```

2. 使用mvn打包文件
```
mvn clean package
```

3. 在centos中建立专属springboot发布后的jar包文件夹，将jar文件上传到此目录下，并创建另一个专门放脚本的文件夹，在脚本文件夹下创建运行的脚本文件。
```
-- 创建文件
vi week.service

-- 编辑文件
 
[Unit]
Description=week //程序的名称
After=syslog.target

[Service]
ExecStart=/opt/java/bin/java -jar /home/springboot/apps/week-0.0.1-SNAPSHOT.jar // 分别是java路径和jar路径

[Install]
WantedBy=multi-user.target

```
4. 在/etc/systemd/system/目录下创建连接文件，连接到刚才创建的week.service文件上

```
// 切换到/etc/systemd/system/
cd /etc/systemd/system/

// 创建连接文件
ln -s /home/springboot/week.service week.service
 
```

5. 启动
```
systemctl start week.service
或者
systemctl start week

停止服务
systemctl stop week.service
或者
systemctl stop week

服务状态
systemctl status week.service
或者
systemctl status week

开机启动
systemctl enable week.service
或者
systemctl enable week

项目日志
journalctl -u week.service
或者
journalctl -u week

```




