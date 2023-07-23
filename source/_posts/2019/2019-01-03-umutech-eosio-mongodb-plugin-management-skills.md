---
layout: post
title: EOSIO MongoDB 插件系列：管理技巧
date: 2019-01-03 14:45:22
categories: UMUTech
tags:
- 调研
- blockchain
---
> 作者: UMU @ MEET.ONE 实验室

总结同步主网数据到 MongoDB 时的常用操作，大部分以 transaction_traces 表为例。

## 1. nodeos 配置优化

```ini
read-mode = read-only
validation-mode = light

mongodb-queue-size = 2048
abi-serializer-max-time-ms = 15000
```

## 2. 首次启动 nodeos

从 `https://eosnode.tools/blocks` 下载最新 blocks data，以减少网络同步时间。

首次启动，应使用 `--replay-blockchain` 参数。

## 3. 守护 nodeos 进程

目前 nodeos 1.5+ 版本如果优雅退出，下次启动可以无需痛苦的 replay 过程，所以可以监控 nodeos 进程，如果退出就调用。

启动脚本 /home/ubuntu/shell/continue.sh：

```sh
nohup /usr/local/eosio/bin/nodeos --config-dir /home/ubuntu/nodeos/config-dir --data-dir /home/ubuntu/nodeos/data-dir > /home/ubuntu/shell/`date +%Y-%m-%d_%H-%M`.log 2>&1 &
```

守护脚本 /home/ubuntu/shell/autorun.sh：

```sh
ps -C nodeos || /home/ubuntu/shell/continue.sh
```

添加到计划任务，运行 `sudo crontab -e`，输入下行并保存、退出：

```sh
* * * * * /home/ubuntu/shell/autorun.sh
```

## 4. 读写用户分离

nodeos 需要写入，使用有写入权限的 EOS 用户，其余情况使用只读权限的 EOSReader 用户，数据库安装之后就尽量不使用管理员用户。

```js
use EOS
db.createUser({"user" : "EOS", "pwd" : "Password", "roles" : [{role : "readWrite", "db" : "EOS"},"dbOwner"]});
db.createUser({"user" : "EOSReader", "pwd" : "password", "roles" : [{role : "read", "db" : "EOS"}]});
```

## 5. 查询同步进度

```js
use EOS
db.transaction_traces.find({}, {"block_num" : 1, "block_time" : 1}, -1).sort({$natural:-1}).pretty()
```
## 6. 修复丢失数据

参考《[EOSIO MongoDB 插件系列：从 log 中找回丢](/2019/01/03/umutech-eosio-mongodb-plugin-find-lost-insertion-from-logs/
)》