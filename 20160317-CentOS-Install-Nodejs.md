---
title: 在CentOS 7安装Node.js
date: 2016-03-17 13:06
categories: Hexo创建博客
tags: [hexo]
description: 在CentOS 7安装Node.js开发环境
photos:
- ../../../../images/nodejs.jpg
---
## 从源码安装Node.js
首先我们要从源码安装Node.js。我真的很喜欢从源码安装软件。在你的CentOS 7机器上打开一个新的终端标签并运行以下命令 用来下载需要使用的安装文件。
``` bash
wget http://nodejs.org/dist/v0.12.0/node-v0.12.0.tar.gz 
``` 
你可以从上面的命令中看到我们怎样利用wget来操作。
然后提取tar文件，如下所示。 
``` bash
tar xvf node-v0.12.0.tar.gz
``` 
然后使用下面的命令来改变工作目录节点。  
``` bash
cd node-v**
``` 
在编译我们的代码之前，需要在CentOS机器上安装一些软件包，这样可以我们编译。所以在你打开的的终端中，输入以下内容。
``` bash
sudo yum install gcc gcc-c++
``` 
等待这些软件包的安装和运行，用以下命令来配置和编译。
``` bash
./configure
make
``` 
以上会需要一些时间来完成，别担心因为编译将需要一段时间。然后使用下面的命令来在你的系统上安装Node.js。
``` bash
sudo make install
``` 
安装完成之后，你就可以开始使用Node.js了。并为确保安装的版本正确，你可以使用以下命令检查。
``` bash
node --version
``` 
当运行上述命令时，我得到以下信息。
``` bash
v0.12.0
``` 

## 如何从EPEL库安装Node.js
另一个有效且简单的方法来安装Node.js就是从官方库。这同样确保您可以访问到EPEL库，你可以通过运行以下命令。
``` bash
sudo yum install epel-release
``` 
现在可以使用yum命令安装Node.js了。
``` bash
sudo yum install nodejs
``` 
因为在开发过程中我需要管理节点包，我还要安装新公共管理的软件包管理器，使用以下命令。
``` bash
sudo yum install npm
``` 

## 写我们的第一个Node.js程序。
在Node.js中写Hello World应用程序是很容易的！比Python简单。您需要做的就是在一个文件中写出下面这段代码并保存为something.js。
``` javascript
console.log("Hello World");
``` 
把我们刚刚创建了的保存为hello.js。然后为了运行它，我们需要使用以下命令。
``` bash
node hello.js
``` 
现在打开你的文本编辑器hello.js并加入下面这段代码。你可以拷贝和粘贴但我强烈建议你亲自输入因为这是一个很好的你可以熟悉Node.js的机会。
``` javascript
var http = require('http');
http.createServer(function (request, response) { response.writeHead(200, >{'Content-Type': 'text/plain'}); response.end('Hello World\n'); }).listen(8080);
console.log('Server started');
``` 
使用下面的命令来运行应用程序。
``` bash
node hello.js
``` 
在浏览器访问http：/ / www.localhost：8000 /时，上面的代码，它会显示“Hello World”。
 