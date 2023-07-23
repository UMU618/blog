---
layout: post
title: pm2 运维经验
date: 2019-07-16 17:32:00
categories: UMUTech
tags:
- ops
- nodejs
---
## 安装

比较多的文章推荐用 npm 安装，但 [UMU](https://blog.umu618.com/) 更推荐 yarn。

**理由**：[Visual Studio Code](https://code.visualstudio.com/) 脑残粉跟随 [Microsoft](https://github.com/Microsoft)/[vscode](https://github.com/Microsoft/vscode) 使用 yarn。

参考 [yarn 安装](https://yarnpkg.com/en/docs/install#debian-stable)，其中 Ubuntu 下命令为：

```sh
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install --no-install-recommends yarn
```

使用 yarn 安装 pm2：

```sh
yarn global add pm2
```

## 运行

不熟悉 yarn 的话，装完一头雾水，装到哪了？用以下命令显示：

```sh
yarn global bin
```

结果参考：

- macOS：`/usr/local/bin`
- Ubuntu：`/home/ubuntu/.yarn/bin`

启动脚本的命令为：

```sh
`yarn global bin`/pm2 start my-program.js
```

## 安全建议

**root 身份能不用则不用。**
