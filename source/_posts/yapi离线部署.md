---
title: yapi离线部署
categories: web
tags:
  - 工具
date: 2018-04-19 16:27:48
---

> YApi 是一个可本地部署的、打通前后端及QA的、可视化的接口管理平台

yapi的仓库地址: [https://github.com/ymfe/yapi](https://github.com/ymfe/yapi)

本人在内网机器不允许访问外网的情况下在内网部署一套yapi, 这里记录下成功的流程.

所需准备的是一台能访问外网的机器(我这里是安装了一个ubuntu`16.04`的虚拟机, 内网机器操作系统是`RedHat6.9`), 通过这台外网机器完成所有需要网络的操作. 另外外网机器需要`gcc 4.8+`(因为在安装`yapi`依赖时需要`build node-sass`), 没有`gcc`或者版本较低需要自行安装升级.

<!-- more -->

---

### 下载`nodejs`和`mongodb`.

通过有网络的机器进行下载, 这里一定要下载编译好的压缩文件.

在[nodejs下载页](https://nodejs.org/en/download/)中选择对应平台的`Binaries`下载. 因为`nodejs`在内外网机器都需要使用, 如果有必要的话可能需要下载两份. 我这里因为都是linux平台所以只下载一份, 版本是`node-v8.11.1-linux-x64.tar.xz`.

在[mongodb下载页](https://www.mongodb.com/download-center?jmp=nav#community)选择适合内网机器的版本, 我这里用的是`mongodb-linux-x86_64-3.6.4.tgz`.


### 在外网机器安装`nodejs`

解压`nodejs`并放到想要的目录下(我这里直接把解压的内容放在当前目录的`node`文件夹下, 也可以放在其他目录但请确保读写权限).

```
tar xvJf node-v8.11.1-linux-x64.tar.xz
mv node-v8.11.1-linux-x64 node

```

将`nodejs`加入环境变量

```
vim ~/.bashrc
```

在其中加入一行

```
export PATH=<node文件夹的路径>/bin:$PATH
```

启用环境变量并验证`nodejs`安装成功

```
source ~/.bashrc
node -v
```

如果能正确显示版本信息说明安装成功.

---

### 在外网机器获取`yapi`源码并安装依赖

使用`git`获取`yapi`源码, 如果没有`git`命令请按照对应平台的安装方法安装`git`.

创建一个新文件夹`yapi`, 使用`clone`将`yapi`源码放入`vendors`中:

```
mkdir yapi
cd yapi
git clone https://github.com/YMFE/yapi.git vendors
cp vendors/config_example.json ./config.json
cd vendors
npm install --production
```

我这里还安装了`pm2`

```
npm install -S pm2
```

将创建的`yapi`文件夹打成压缩包得到`yapi.tar.gz`(其目录下有`config.json`和`vendors`)

```
tar -czf yapi.tar.gz yapi
```

至此, 所有需要外部网络的操作已经完成, 可以进行内网部署.

---

### 内网安装的准备工作

将`nodejs`,`mongodb`和上面打包的`yapi`三个压缩包传到内网机器上, 并按照之前的流程安装`nodejs`.

---

### 内网安装`mongodb`

解压`mongodb-linux-x86_64-3.6.4.tgz`并放入`mongodb`文件夹中

```
tar -zxvf mongodb-linux-x86_64-3.6.4.tgz
mv mongodb-linux-x86_64-3.6.4 mongodb
```

把`mongodb`放入环境变量中, 修改`~/.bashrc`, 加入以下内容

```
export PATH=<mongodb文件夹的路径>/bin:$PATH
```

验证安装

```
source ~/.bashrc
mongo --version
```

创建`dbdata/db`文件夹和`dblog`文件夹(请自行确保这些文件夹的读写权限)

```
mkdir -p dbdata/db
mkdir dblog
```

启动`mongodb`服务

```
sudo ./mongodb/bin/mongod --fork --dbpath ./dbdata --logpath ./dblog/log
```

---

### 启动`yapi`

解压`yapi.tar.gz`

```
tar -zxvf yapi.tar.gz
```

按需要修改`yapi/config.json`中的相关配置(例如管理员账号等)

初始化数据库:
```
cd ./yapi/vendors
npm run install-server
```

使用pm2启动
```
npx pm2 start ./server/app.js
```

启动完成后即可尝试访问`yapi`看是否成功, 具体地址要根据内网机器的ip和在`config.json`中配置的端口号

如果要关闭`yapi`服务, 可以使用

```
npx pm2 stop all
```
