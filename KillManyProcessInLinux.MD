## Linux下批量Kill多个进程的方法

``` bash
ps -ef|grep LOCAL=NO|grep -v grep|cut -c 9-15|xargs kill -9 
```

### 管道符“|”用来隔开两个命令，管道符左边命令的输出会作为管道符右边命令的输入。下面说说用管道符联接起来的几个命令： 

1. “ps - ef”是Red Hat 里查看所有进程的命令。这时检索出的进程将作为下一条命令“grep LOCAL=NO”的输入。 
2. “grep LOCAL=NO”的输出结果是，所有含有关键字“LOCAL=NO”的进程，这是Oracle数据库中远程连接进程的共同特点。 
3. “grep -v grep”是在列出的进程中去除含有关键字“grep”的进程。 
4. “cut -c 9-15”是截取输入行的第9个字符到第15个字符，而这正好是进程号PID。 
5. “xargs kill -9”中的xargs命令是用来把前面命令的输出结果（PID）作为“kill -9”命令的参数，并执行该令。 
6. “kill -9”会强行杀掉指定进程，这样就成功清除了oracle的所有远程连接进程。其它类似的任务，只需要修改“grep LOCAL=NO”中的关键字部分就可以了。 

>例如：
``` bash
ps -ef|grep /opt/dotnet/dotnet|grep -v grep|cut -c 9-15|xargs kill -9
```