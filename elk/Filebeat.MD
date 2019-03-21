## Filebeat 

### 安装


### 启动
测试filebeat启动后，查看相关输出信息：
./filebeat -e -c filebeat.yml -d "publish" 

后台方式启动filebeat：

nohup ./filebeat -e -c filebeat.yml >/dev/null 2>&1 &  将所有标准输出及标准错误输出到/dev/null空设备，即没有任何输出 
nohup ./filebeat -e -c filebeat.yml > filebeat.log &

停止filebeat：

查找进程ID并kill掉：
ps -ef |grep filebeat
kill -9  进程号



----- 有报错
修复/lib/ld-linux.so.2: bad ELF interpreter: No such file or directory问题
1、在64系统里执行32位程序如果出现/lib/ld-linux.so.2: 
bad ELF interpreter: No such file or directory，安装下glic即可

sudo yum install glibc.i686


2、error while loading shared libraries: libz.so.1: 
cannot open shared object file: No such file or directory

sudo yum install zlib.i686 



### 配置文件解析

1. 主配置文件 /etc/filebeat/filebeat.yml

``` bash

###################### Filebeat Configuration Example #########################

#=========================== Filebeat inputs =============================

filebeat.inputs:

# Each - is an input. Most options can be set at the input level","so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

# 这里每一个 type 表示定义了一个日志读取源，这个源是收集 Nginx 的访问日志
- type: log

  enabled: true

  paths:
    - /usr/log/nginx/access/access.log
  fields_under_root: true
  fields: 
    alilogtype: nginxacclog

# 收集某一个服务的错误日志
- type: log

  enabled: true

  paths:
    - /var/www/service/storage/logs/error.log
  fields_under_root: true
  fields: 
    alilogtype: service_error
    serverip: ${serverip}

# 收集某一个服务的错误日志，并且使用了多行合并
- type: log

  enabled: true

  paths:
    - /var/www/user_center/storage/logs/SERVER*.log
  fields_under_root: true
  fields: 
    alilogtype: usercenter_serverlog
    serverip: ${serverip}

  multiline.pattern: "^\["
  multiline.negate: true
  multiline.match: after


- type: log 
  enabled: true 
  paths: 
    - /home/apps/newIndex/logs/platform.error.log.* 
  documnet_type: "newIndex"
  fields:
    review: 2

  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3}'
  multiline.negate: true
  multiline.match: after
  multiline.timeout: 10s


#============================= Filebeat modules ===============================

filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml

  # Set to true to enable config reloading
  reload.enabled: false

  # Period on which files under path should be checked for changes
  #reload.period: 10s

#================================ Outputs =====================================

#-------------------------- Elasticsearch output ------------------------------

# 这部分是用于配置日志输出到 Elasticsearch 的部分

#----------------------------- Logstash output --------------------------------

# 将日志发送到 logstash 主机的 5044 端口，对应的这台 logstash 主机需要配置一个 input 监听于 5044 （配置过程，参考 Logstash 文档）
output.logstash:
  hosts: ["10.26.10.15:5044"]

```
2. 参数详解

``` bash

## 默认值 log ，表示一个日志读取源
type : log

## 该配置是否生效，如果设置为 false 将不会收集该配置的日志
enabled: true

## 要抓取的日志路径，写绝对路径
paths: /to/file.log

## fields 表示自定义字段，在下面缩进两格处写要自己添加的字段。如： alilogtype: usercenter_serverlog  表示在输出的每条日志中加入该字段，key：alilogtype ， value：usercenter_serverlog 用于标识该日志源的类别，在传输到下一层 logstash 时可以根据该字段分类处理。
  fields: 
    alilogtype: usercenter_serverlog
##     意思相同，增加一个自定义字段，key：serverip ，value: ${serverip} 这个值是读取的系统环境变量，如果系统中没有定义这个环境变量，那么启动 filebeat 的时候会报错，找到这个值.
        serverip: ${serverip}

## 设置系统环境变量，创建文件  /etc/profile.d/serverip.sh  加入内容：
export serverip=`ifconfig eth0 | grep "inet addr" | awk "{print $2}" | cut -d":" -f2`
## 这里拿的是本机 IP

## 多行合并参数，正则表达式
multiline.pattern: "^\["
## true 或 false；默认是false，匹配pattern的行合并到上一行；true，不匹配pattern的行合并到上一行
multiline.negate: true
## after 或 before，合并到上一行的末尾或开头
multiline.match: after

##  ["ERROR","WARN"] 该属性可以配置只收集error级别和warn级别的日志,如果有配置多行收集,一定要将这个配置放在多行的后面
include_lines:
## ["DEBUG"] 该属性配置不收集DEBUG级别的日志,如果配置多行 这个配置也要放在多行的后面
exclude_lines: 

## Logstash所在的主机
hosts: 

## 如果设置为TRUE和配置了多台logstash主机，输出插件将负载均衡的发布事件到所有logstash主机。如果设置为false，输出插件发送所有事件到随机的一台主机上，如果选择的不可达将切换到另一台主机。默认是false。
loadbalance

## 每个配置的主机发布事件到Logstash的工作者数量。这最适用于启用负载平衡模式。示例：如果您有2个主机和3个工作人员，则共有6个工作人员启动（每个主机3个）。
worker

```


### 另一个配置解释

``` bash

############### Filebeat #############
filebeat:
  # List of prospectors to fetch data.
  prospectors:
    -
      # paths指定要监控的日志
      paths:
        - /var/log/*.log
 
      #指定被监控的文件的编码类型使用plain和utf-8都是可以处理中文日志的。
      # Some sample encodings:
      #   plain","utf-8","utf-16be-bom","utf-16be","utf-16le","big5","gb18030","gbk,
      #    hz-gb-2312","euc-kr","euc-jp","iso-2022-jp","shift-jis","...
      #encoding: plain
 
      #指定文件的输入类型log(默认)或者stdin。
      input_type: log
 
      # 在输入中排除符合正则表达式列表的那些行
      # exclude_lines: ["^DBG"]
 
      # 包含输入中符合正则表达式列表的那些行默认包含所有行include_lines执行完毕之后会执行exclude_lines。
      # include_lines: ["^ERR"",""^WARN"]
 
      # 忽略掉符合正则表达式列表的文件默认为每一个符合paths定义的文件都创建一个harvester。
      # exclude_files: [".gz$"]
 
      # 向输出的每一条日志添加额外的信息比如“level:debug”方便后续对日志进行分组统计。默认情况下会在输出信息的fields子目录下以指定的新增fields建立子目录例如fields.level。
      #fields:
      #  level: debug
      #  review: 1
 
      # 如果该选项设置为true则新增fields成为顶级目录而不是将其放在fields目录下。自定义的field会覆盖filebeat默认的field。
      #fields_under_root: false
 
      # 可以指定Filebeat忽略指定时间段以外修改的日志内容比如2h两个小时或者5m(5分钟)。
      #ignore_older: 0
 
      # 如果一个文件在某个时间段内没有发生过更新则关闭监控的文件handle。默认1h,change只会在下一次scan才会被发现
      #close_older: 1h
 
      # i设定Elasticsearch输出时的document的type字段也可以用来给日志进行分类。Default: log
      #document_type: log
 
      # Filebeat以多快的频率去prospector指定的目录下面检测文件更新比如是否有新增文件如果设置为0s则Filebeat会尽可能快地感知更新占用的CPU会变高。默认是10s。
      #scan_frequency: 10s
 
      # 每个harvester监控文件时使用的buffer的大小。
      #harvester_buffer_size: 16384
 
      # 日志文件中增加一行算一个日志事件max_bytes限制在一次日志事件中最多上传的字节数多出的字节会被丢弃。The default is 10MB.
      #max_bytes: 10485760
 
      # 适用于日志中每一条日志占据多行的情况比如各种语言的报错信息调用栈。这个配置的下面包含如下配置
      #multiline:
 
        # The regexp Pattern that has to be matched. The example pattern matches all lines starting with [
        #pattern: ^\[
 
        # Defines if the pattern set under pattern should be negated or not. Default is false.
        #negate: false
 
        # Match can be set to "after" or "before". It is used to define if lines should be append to a pattern
        # that was (not) matched before or after or as long as a pattern is not matched based on negate.
        # Note: After is the equivalent to previous and before is the equivalent to to next in Logstash
        #match: after
 
        # The maximum number of lines that are combined to one event.
        # In case there are more the max_lines the additional lines are discarded.
        # Default is 500
        #max_lines: 500
 
        # After the defined timeout","an multiline event is sent even if no new pattern was found to start a new event
        # Default is 5s.
        #timeout: 5s
 
      # 如果设置为trueFilebeat从文件尾开始监控文件新增内容把新增的每一行文件作为一个事件依次发送而不是从文件开始处重新发送所有内容。
      #tail_files: false
 
      # Filebeat检测到某个文件到了EOF之后每次等待多久再去检测文件是否有更新默认为1s。
      #backoff: 1s
 
      # Filebeat检测到某个文件到了EOF之后等待检测文件更新的最大时间默认是10秒。
      #max_backoff: 10s
 
      # 定义到达max_backoff的速度默认因子是2到达max_backoff后变成每次等待max_backoff那么长的时间才backoff一次直到文件有更新才会重置为backoff。
      #backoff_factor: 2
 
      # 这个选项关闭一个文件,当文件名称的变化。#该配置选项建议只在windows。
      #force_close_files: false
 
    # Additional prospector
    #-
      # Configuration to use stdin input
      #input_type: stdin
 
  # spooler的大小spooler中的事件数量超过这个阈值的时候会清空发送出去不论是否到达超时时间。
  #spool_size: 2048
 
  # 是否采用异步发送模式(实验!)
  #publish_async: false
 
  # spooler的超时时间如果到了超时时间spooler也会清空发送出去不论是否到达容量的阈值。
  #idle_timeout: 5s
 
  # 记录filebeat处理日志文件的位置的文件
  registry_file: /var/lib/filebeat/registry
 
  # 如果要在本配置文件中引入其他位置的配置文件可以写在这里需要写完整路径但是只处理prospector的部分。
  #config_dir:
 
 
############################# Output ##########################################
 
# 输出到数据配置.单个实例数据可以输出到elasticsearch或者logstash选择其中一种注释掉另外一组输出配置。
output:
 
  ### 输出数据到Elasticsearch
  elasticsearch:
    # IPv6 addresses should always be defined as: https://[2001:db8::1]:9200
    hosts: ["localhost:9200"]
 
    # 输出认证.
    #protocol: "https"
    #username: "admin"
    #password: "s3cr3t"
 
    # 启动进程数.
    #worker: 1
 
    # 输出数据到指定index default is "filebeat"  可以使用变量[filebeat-]YYYY.MM.DD keys.
    #index: "filebeat"
 
    # 一个模板用于设置在Elasticsearch映射默认模板加载是禁用的,没有加载模板这些设置可以调整或者覆盖现有的加载自己的模板
    #template:
 
      # Template name. default is filebeat.
      #name: "filebeat"
 
      # Path to template file
      #path: "filebeat.template.json"
 
      # Overwrite existing template
      #overwrite: false
 
    # Optional HTTP Path
    #path: "/elasticsearch"
 
    # Proxy server url
    #proxy_url: http://proxy:3128
 
    # 发送重试的次数取决于max_retries的设置默认为3
    #max_retries: 3
 
    # 单个elasticsearch批量API索引请求的最大事件数。默认是50。
    #bulk_max_size: 50
 
    # elasticsearch请求超时事件。默认90秒.
    #timeout: 90
 
    # 新事件两个批量API索引请求之间需要等待的秒数。如果bulk_max_size在该值之前到达额外的批量索引请求生效。
    #flush_interval: 1
 
    # elasticsearch是否保持拓扑。默认false。该值只支持Packetbeat。
    #save_topology: false
 
    # elasticsearch保存拓扑信息的有效时间。默认15秒。
    #topology_expire: 15
 
    # 配置TLS参数选项如证书颁发机构等用于基于https的连接。如果tls丢失主机的CAs用于https连接elasticsearch。
    #tls:
      # List of root certificates for HTTPS server verifications
      #certificate_authorities: ["/etc/pki/root/ca.pem"]
 
      # Certificate for TLS client authentication
      #certificate: "/etc/pki/client/cert.pem"
 
      # Client Certificate Key
      #certificate_key: "/etc/pki/client/cert.key"
 
      # Controls whether the client verifies server certificates and host name.
      # If insecure is set to true","all server host names and certificates will be
      # accepted. In this mode TLS based connections are susceptible to
      # man-in-the-middle attacks. Use only for testing.
      #insecure: true
 
      # Configure cipher suites to be used for TLS connections
      #cipher_suites: []
 
      # Configure curve types for ECDHE based cipher suites
      #curve_types: []
 
      # Configure minimum TLS version allowed for connection to logstash
      #min_version: 1.0
 
      # Configure maximum TLS version allowed for connection to logstash
      #max_version: 1.2
 
 
  ### 发送数据到logstash 单个实例数据可以输出到elasticsearch或者logstash选择其中一种注释掉另外一组输出配置。
  #logstash:
    # Logstash 主机地址
    #hosts: ["localhost:5044"]
 
    # 配置每个主机发布事件的worker数量。在负载均衡模式下最好启用。
    #worker: 1
 
    # #发送数据压缩级别
    #compression_level: 3
 
    # 如果设置为TRUE和配置了多台logstash主机输出插件将负载均衡的发布事件到所有logstash主机。
    #如果设置为false输出插件发送所有事件到随机的一台主机上如果选择的不可达将切换到另一台主机。默认是false。
    #loadbalance: true
 
    # 输出数据到指定index default is "filebeat"  可以使用变量[filebeat-]YYYY.MM.DD keys.
    #index: filebeat
 
    # Optional TLS. By default is off.
    #配置TLS参数选项如证书颁发机构等用于基于https的连接。如果tls丢失主机的CAs用于https连接elasticsearch。
    #tls:
      # List of root certificates for HTTPS server verifications
      #certificate_authorities: ["/etc/pki/root/ca.pem"]
 
      # Certificate for TLS client authentication
      #certificate: "/etc/pki/client/cert.pem"
 
      # Client Certificate Key
      #certificate_key: "/etc/pki/client/cert.key"
 
      # Controls whether the client verifies server certificates and host name.
      # If insecure is set to true","all server host names and certificates will be
      # accepted. In this mode TLS based connections are susceptible to
      # man-in-the-middle attacks. Use only for testing.
      #insecure: true
 
      # Configure cipher suites to be used for TLS connections
      #cipher_suites: []
 
      # Configure curve types for ECDHE based cipher suites
      #curve_types: []
 
 
  ### 文件输出将事务转存到一个文件每个事务是一个JSON格式。主要用于测试。也可以用作logstash输入。
  #file:
    # 指定文件保存的路径。
    #path: "/tmp/filebeat"
 
    # 文件名。默认是 Beat 名称。上面配置将生成 packetbeat","packetbeat.1","packetbeat.2 等文件。
    #filename: filebeat
 
    # 定义每个文件最大大小。当大小到达该值文件将轮滚。默认值是1000 KB
    #rotate_every_kb: 10000
 
    # 保留文件最大数量。文件数量到达该值将删除最旧的文件。默认是7一星期。
    #number_of_files: 7
 
 
  ### Console output 标准输出JSON 格式。
  # console:
    #如果设置为TRUE事件将很友好的格式化标准输出。默认false。
    #pretty: false
 
 
############################# Shipper #########################################
 
shipper:
  # #日志发送者信息标示
  # 如果没设置以hostname名自居。该名字包含在每个发布事务的shipper字段。可以以该名字对单个beat发送的所有事务分组。
  #name:
 
  # beat标签列表包含在每个发布事务的tags字段。标签可用很容易的按照不同的逻辑分组服务器。
  #例如一个web集群服务器可以对beat添加上webservers标签然后在kibana的visualisation界面以该标签过滤和查询整组服务器。
  #tags: ["service-X"",""web-tier"]
 
  # 如果启用了ignore_outgoing选项beat将忽略从运行beat服务器上所有事务。
  #ignore_outgoing: true
 
  # 拓扑图刷新的间隔。也就是设置每个beat向拓扑图发布其IP地址的频率。默认是10秒。
  #refresh_topology_freq: 10
 
  # 拓扑的过期时间。在beat停止发布其IP地址时非常有用。当过期后IP地址将自动的从拓扑图中删除。默认是15秒。
  #topology_expire: 15
 
  # Internal queue size for single events in processing pipeline
  #queue_size: 1000
 
  # GeoIP数据库的搜索路径。beat找到GeoIP数据库后加载然后对每个事务输出client的GeoIP位置目前只有Packetbeat使用该选项。
  #geoip:
    #paths:
    #  - "/usr/share/GeoIP/GeoLiteCity.dat"
    #  - "/usr/local/var/GeoIP/GeoLiteCity.dat"
 
 
############################# Logging #########################################
 
 
# 配置beats日志。日志可以写入到syslog也可以是轮滚日志文件。默认是syslog。
logging:
 
  # 如果启用发送所有日志到系统日志。
  #to_syslog: true
 
  # 日志发送到轮滚文件。
  #to_files: false
 
  # 
  files:
    # 日志文件目录。
    #path: /var/log/mybeat
 
    # 日志文件名称
    #name: mybeat
 
    # 日志文件的最大大小。默认 10485760 (10 MB)。
    rotateeverybytes: 10485760 # = 10MB
 
    # 保留日志周期。 默认 7。值范围为2 到 1024。
    #keepfiles: 7
 
  # Enable debug output for selected components. To enable all selectors use ["*"]
  # Other available selectors are beat","publish","service
  # Multiple selectors can be chained.
  #selectors: [ ]
 
  # 日志级别。debug","info","warning","error 或 critical。如果使用debug但没有配置selectors* selectors将被使用。默认error。
  #level: error

```


### demo


filebeat.yml中output的配置将除了logstash之外注释掉

``` bash

output:
    logstash:
        hosts: ["192.168.197.130:5044"]
        worker: 2
        loadbalance: true
        index: filebeat

这里  worker  的含义，是 beat 连到每个 host 的线程数。
在  loadbalance  开启的情况下，意味着有 4 个worker 轮训发送数据

```

beat 写入 Logstash 时，会配合 Logstash-1.5 后新增的 metadata 特性。将 beat 名和 type 名 记录在 metadata 里。所以对应的 Logstash 配置应该是这样 


``` js

input {
    beats {
        port => 5044
    }
}
output {
    elasticsearch {
        hosts => ["http://192.168.197.128:9200"]
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
    }
    stdout{
        codec=>rubydebug
    }
} 


```


运行logstash
``` bash

nohup bin/logstash -f config/logstash-dev.conf &


nohup bin/logstash -f config/logstash-prod.conf &

```


### 遇到的问题
1. 版本一定要一致

2. 这种类型转换失败的错误
[2019-03-20T17:24:39,404][WARN ][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"filebeat-6.6.1", :_type=>"doc", :routing=>nil}, #<LogStash::Event:0xeed570e>], :response=>{"index"=>{"_index"=>"filebeat-6.6.1", "_type"=>"doc", "_id"=>"LopqmmkBh3B0wGLquTZS", "status"=>400, "error"=>{"type"=>"mapper_parsing_exception", "reason"=>"failed to parse [log_json_content.message]", "caused_by"=>{"type"=>"illegal_state_exception", "reason"=>"Can't get text on a START_OBJECT at 1:1211"}}}}}

这条日志的某个字段的类型跟以前的不一致，json解析后不能按照原来的类型存储到elastic中。

The message "Can't get text on a START_OBJECT" means that Elasticsearch was expecting an entry of type "string" but you are trying to give an object as input.

If you check Kibana you will find that the field "products" exists there and is defined as a string. But since you are entering a list of dictionaries then the field "products" should have been defined from the beginning as an object (preferably with dynamic fields in it).


### logstash

多行合并

"^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3} [http-nio-"
 
 
http://192.168.70.10:1234/newIndex/v1/assess/composition


http://192.168.70.10:1234/indexV2/cache/clear/IndexV2_

http://localhost:8989/indexV2/cache/clear/IndexV2_



\s*%{TIMESTAMP_ISO8601:time} \s*(?<info>([\s\S]*))\s*\[\]

 

<appender name="errorFile" class="ch.qos.logback.core.rolling.RollingFileAppender">

   <filter class="ch.qos.logback.classic.filter.LevelFilter">

      <level>ERROR</level>

      <onMatch>ACCEPT</onMatch>

      <onMismatch>DENY</onMismatch>

   </filter>

   <file>${log.dir}/elk/error.log</file> <!-- 当前的日志文件文件放在 elk文件下，该日志的内容会被filebeat传送到es --> 

   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  <! -- 历史日志会放到 bak 文件下，最多保存7天的历史，最多占用 1G的空间 -->

      <fileNamePattern>${log.dir}/bak/error.%d{yyyy-MM-dd}.log</fileNamePattern>

      <maxHistory>7</maxHistory>

      <totalSizeCap>1GB</totalSizeCap>

   </rollingPolicy>

   <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">

  json { 
    source =>"message" 
    target => "log" 
    }

      <providers>

         <pattern>

            <pattern>

              {

　　　　　　　　　　　　　　"tags": ["errorlog"],

　　　　　　　　　　　　　　"project": "myproject",

　　　　　　　　　　　　　　"timestamp": "%date{\"yyyy-MM-dd'T'HH:mm:ss,SSSZ\"}",

　　　　　　　　　　　　　　"log_level": "%level",

　　　　　　　　　　　　　　"thread": "%thread",

　　　　　　　　　　　　　　"class_name": "%class",

　　　　　　　　　　　　　　"line_number": "%line",

　　　　　　　　　　　　　　"message": "%message",

　　　　　　　　　　　　　　"stack_trace": "%exception{5}",

　　　　　　　　　　　　　　"req_id": "%X{reqId}",

　　　　　　　　　　　　　　"elapsed_time": "#asLong{%X{elapsedTime}}"

　　　　　　　　　　　　　　}

            </pattern>

         </pattern>

      </providers>

   </encoder>

</appender>



## 解析 
1. 设置log4j日志格式

2016-09-22 10:37:16  [ http-nio-1568-exec-1:0 ] - [ ERROR ]  {"threadName":"http-nio-1568-exec-1","typeName":"org.apache.tomcat.util.threads.TaskThread","description":"Create folder fail!","exceptionCause":"java.lang.ThreadGroup[name=main,maxpri=10]","trace":[{"methodName":"getStackTrace","fileName":"Thread.java","lineNumber":1552,"className":"java.lang.Thread","nativeMethod":false},{"methodName":"<init>","fileName":"BaseLogBean.java","lineNumber":33,"className":"com.iscas.datasong.domain.logger.BaseLogBean","nativeMethod":false},

%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-4level] [%logger{36}] [%msg]%n


filter{
  grok{
    match=>{ 
      "message" => "%{DATA:createtime}  \[%{DATA:thread}\] - \[%{DATA:level}\]  \[%{DATA:logger}\]  %{GREEDYDATA:log_json}"   
      }
  }
  json {
        source => "log_json"
        target => "log_json_content"
        remove_field=>["logjson"]
  }
