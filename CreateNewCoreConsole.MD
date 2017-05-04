# 测试创建一个.Net Core Console App

## 工具
1. 安装 .Net Core的电脑
2. Vs Code 

## 1. 创建项目
打开命令行 cd到项目目录
``` bash
mkdir CoreConsole
dotnet new Console
```
然后用Vs Code 打开文件

## 2. 添加依赖
在dependencies节点中添加。

## 发布程序

``` cs
{
  "version": "1.0.0-*",
  "buildOptions": {
    "emitEntryPoint": true
  },
  "dependencies": {
    "Microsoft.NETCore.App": {
      "type": "platform", //去掉
      "version": "1.0.0-rc2-*"
    }
  },
  "frameworks": {
    "netcoreapp1.0": {
      "imports": "dnxcore50"
    }
  },
  "runtimes":{//加入runtime
    "win7-x64": { },
    "win7-x86": { },
    "osx.10.10-x64": { },
    "osx.10.11-x64": { },
    "ubuntu.14.04-x64":{ }
  }
}
``` 

dotnet restore 

dotnet publish 

这回默认发布第一个runtimes内的配置
dotnet publish -r ubuntu.14.04-x64
这是发布ubuntu版本的

还有另一个命令（这个应该是跨平台的）
dotnet publish -c release

> ref > https://github.com/dotnet/core/blob/master/Documentation/prereqs.md