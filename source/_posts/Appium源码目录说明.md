---
layout: Appium源码目录说明
title: Appium源码目录说明
date: 2018.01.29 15:22
tags:
    - Appium
---
Appium版本 1.7.2
Appium源码主要由appium的入口文件js及一些引用的基础类库组成，以下举例说明源码目录大致功能，对于类库以appium-base-driver为例，类库中源码整体结构基本一致。
#### appium文件夹下内容

```
├── AUTHORS
├── CHANGELOG.md
├── CONDUCT.md
├── CONTRIBUTING.md
├── GOVERNANCE.md
├── IDEAS.md
├── LICENSE
├── README.md
├── RELEASE.pdf
├── ROADMAP.md
├── build//打包后生成的文件夹
├── node_modules//引用的类库文件夹
├── docs//文档文件夹
├── commands-yml//文档相关文件夹
├── bin
│   └── ios-webkit-debug-proxy-launcher.js
├── gulpfile.js//程度打包文件
├── lib//主程序执行入口
│   ├── appium.js //
│   ├── config.js//基础配置相关
│   ├── grid-register.js
│   ├── logger.js//日志
│   ├── logsink.js//日志
│   ├── main.js //程序执行入口
│   ├── parser.js //appium终端命令相关js
│   └── utils.js//基础类库
├── npm-shrinkwrap.json
├── package.json
├── packweb.json
└── triagers.json
```
#### 基础类库appium-base-driver
```
├── LICENSE
├── README.md
├── build//打包后生成的文件夹
├── node_modules//引用的类库文件夹
├── docs//文档文件夹
├── gulpfile.js//gulp打包文件
├── index.js//程序入口
├── lib//主要执行代码文件夹
│   ├── basedriver
│   │   ├── README.md
│   │   ├── capabilities.js//配置文件
│   │   ├── commands
│   │   │   ├── find.js
│   │   │   ├── index.js
│   │   │   ├── session.js//客户端与服务端创建sessionId
│   │   │   ├── settings.js
│   │   │   └── timeout.js
│   │   ├── desired-caps.js
│   │   ├── device-settings.js
│   │   ├── driver.js//基础类库
│   │   ├── helpers.js
│   │   └── logger.js
│   ├── express
│   │   ├── README.md
│   │   ├── crash.js
│   │   ├── express-logging.js
│   │   ├── logger.js
│   │   ├── middleware.js//中间件
│   │   ├── server.js//appium服务启动
│   │   └── static.js
│   ├── jsonwp-proxy
│   │   ├── README.md
│   │   └── proxy.js//服务端与设备端通信
│   ├── jsonwp-status
│   │   ├── README.md
│   │   └── status.js//遵循json wire protocal的code返回值 
│   └── mjsonwp
│       ├── README.md
│       ├── errors.js
│       ├── helper.js
│       ├── index.js
│       ├── mjsonwp.js//接收客户端res并返回res
│       ├── routes.js//路由配置
│       ├── validators.js//参数验证
├── package-lock.json
├── package.json
├── static//静态资源
└── test//测试用例
```