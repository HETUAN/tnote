## CentOS run .NET Core scheduled task

### In order to run my .NET Core Console App everyday or weeks like windows services,
> [refence address > http://blog.csdn.net/foxman209/article/details/6759920](http://blog.csdn.net/foxman209/article/details/6759920)
### 1. Install corntab (this is default installed in centos system, if not exist or you can,t sure you can run this code in terminal,)

``` bash
yum install vixie-cron
//if fail
# yum install vixie-cron 
yum install  crontabs
```
说明：
vixie-cron软件包是cron的主程序；
crontabs软件包是用来安装、卸装、或列举用来驱动 cron 守护进程的表格的程序。
cron 是linux的内置服务，但它不自动起来，可以用以下的方法启动、关闭这个服务：
``` bush
/sbin/service crond start //启动服务
/sbin/service crond stop //关闭服务
/sbin/service crond restart //重启服务
/sbin/service crond reload //重新载入配置
``` 

查看crontab服务状态：service crond status

手动启动crontab服务：service crond start

### 2. 创建任务

#### 1.直接分配

#crontab -e

然后出来个可编辑文件：可以通过上述设置执行时间，

例如每小时执行一次任务输出日期到/home/mydatetest文件中

0 * * * *  date >> /home/mydatetests

每分钟执行一次

* * * * *  date >> /home/mydatetest

下面是复杂的写法

``` bash
编辑root下的定时任务
crontab -u root -e

每天早上6点追加一条字符串到一个文本。
0 6 * * * echo "Good morning." >> /tmp/test.txt

每两个小时追加一条字符串一个文本。
0 */2 * * * echo "Have a break now." >> /tmp/test.txt

晚上11点到早上8点之间每两个小时，早上八点
0 23-7/2，8 * * * echo "Have a good dream：）" >> /tmp/test.txt

每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点
0 11 4 * 1-3 command line

1月1日早上4点
0 4 1 1 * command line

每月每天每小时的第 0 分钟执行一次 /bin/ls
0 * * * * /bin/ls

在 12 月内, 每天的早上 6 点到 12 点中，每隔 20 分钟执行一次 /usr/bin/backup
*/20 6-12 * 12 * /usr/bin/backup

周一到周五每天下午 5:00 寄一封信给 alex_mail_name :
0 17 * * 1-5 mail -s "hi" alex_mail_name < /tmp/maildata

每月每天的午夜 0 点 20 分, 2 点 20 分, 4 点 20 分....执行 echo "haha"
20 0-23/2 * * * echo "haha"

晚上11点到早上8点之间每两个小时，早上8点,显示时间
0 23-7/2，8 * * * date
```

#### 2.通过*.sh文件

2.1建立*.sh  设置其权限为744 :chmod 744 mytask.sh

2.2 同上方法直接 * * * * * mytask.sh即可


#### 3.删除所有任务 crontab -r

> 总之在创建任务时一定要使用绝对路径。不然任务可能不执行


部署程序
1. 安装crontab
2. 创建定时任务
``` bash
# 打开任务命令输入配置
crontab -e
# 在打开的文件中输入下面的配置
# 每过一分钟将当前的时间写入到log文件
* * * * * date >> /home/www/testcrontab.log
# 每天23点30执行一次
30 23 * * * /home/www/task/publish/task.everyday 
# 每周五的23点30执行一次
30 23 * * 5 /home/www/task/publish/task.everyweek

# 保存并关闭

# 可以首先测试执行 没有权限的话按照上面的方法增加权限。

# 使用下面的命令可以查看配置的任务
crontab -l 

#重新加载配置
service crontab reload

# 查看crontab服务的状态
service crontab status

#重启服务
service crontab resatrt

```
> 配置任务时出现的错误大多是配置有问题而造成的比如说路径问题。


``` bash
 
 kill -9 线程Id
 
 //批量
 ps -ef|grep /opt/dotnet|grep -v grep|cut -c 9-15|xargs kill -9 
 

```