---
layout: post
title: flutter全栈的最后一步web开发
categories: Blog
description:  flutter,web
keywords:  flutter,web

---

简单记录下核心流程，原文可以[参考](https://mp.weixin.qq.com/s/7BtOOoKFW7Do6oE9ChhbEg)

## 1. Flutter 切换为beta渠道，配置web支持

```
flutter channel beta

flutter upgrade

flutter config --enable-web

flutter devices

Chrome     • chrome     • web-javascript • Google Chrome 78.0.3904.108

```

## 2. 安装express nodejs网站项目

```

mkdir node

mkdir server

cd node/server


npm install express-generator -g //安装好了略过

express --view=pug myapp

cd myapp
npm i
npm start

```

## 3. Flutter 编译web项目

```

flutter build web

```

## 4. 编写脚本自动编译，拷贝目录到www

```
mkdir bin
touch bin/test_start_node.sh
```

```

#!/usr/bin/env bash

# 构建Web
flutter build web
# 拷贝web内容到node目录
cp -r build/web/ node/server/myapp/public_flutter_web/
# 启动服务
node node/server/myapp/bin/www

```
## 5. 配置express静态目录
app.js

```
app.use(express.static(path.join(__dirname, 'public_flutter_web')));

```

## 6.运行

```
./bin/test_start_node.sh


```


