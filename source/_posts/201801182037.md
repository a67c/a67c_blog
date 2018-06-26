---
layout: 基于源码分析Appium服务端启动过程
title: 基于源码分析Appium服务端启动过程
date: 2018.01.18 20:37
tags:
    - Appium
---
#### 写在前面

本文档主要是通过断点跟踪对于Appium源码，从而记录的Appium服务端的启动过程，如有错误或者理解不当之处，欢迎评论提出。
Appium版本：1.7.2 客户端  appium-python-client  2018年1月
可以直接看结论，根据结论中的关键js文件即可断点跟踪出全过程。

#### appium文档

[github](https://github.com/appium)

[官方网站](http://appium.io/)

### 主体结构

![官方图片](http://upload-images.jianshu.io/upload_images/1094385-50476463c0d726b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先是官方这张图片，这张图片简直涵盖appium所有知识点！而且对于这张图片还有中文的readme！
[中文readme直通车](https://github.com/appium/appium/blob/master/docs/cn/contributing-to-appium/appium-packages.md)  
图上分类很清晰 基本上以appium为前缀的都被封装成了类库，通过npm加载，在node_modules中如下图所示，其中appium-base-driver为整个服务的基础类库。上图所示的 `jsonwp-proxy`、`mobile-json-wire-protocal`等都在里面。
![image.png](http://upload-images.jianshu.io/upload_images/1094385-aba2f76c6e89ea0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### node的express
appium的服务端采用的是express框架，[express中文文档](http://www.expressjs.com.cn/)，如果之前用express建立过项目的话，会对express中的路由比较熟悉，很多时候路由的处理大概像[Appium源码分析(3)-路由器模块](http://blog.csdn.net/itfootball/article/details/44707431)这样列出来的样子，即通过`rest.get('/wd/hub/status', controller.getStatus);`该种方式可以查看到所有路由的处理，但是到今年18年appium的代码它的路由配置经过了层层调用。

而且作为node服务之前比较习惯入口的app.js即作为服务开启，但是在appium中它将该入口作为express模块直接放在了`appium-base-driver`中，也就是说appium源码的main入口并不是服务入口，那么他们之间的关系是一个怎样的继承及调用呢？

### 源码目录分析
假设我们想实现一个与客户端通讯的服务，那么主要包括服务开启、客户端http请求，响应函数，那么在appium中我们将其细化一下，大概是
>开启服务->客户端传来请求->开启当前测试用例会话->根据配置确定ios、android等->分别处理ios或安卓请求->ios或安卓返回后->响应客户端吧->关闭会话。

首先看入口结构：

![WX20180118-194031.png](http://upload-images.jianshu.io/upload_images/1094385-3dfc475124e93591.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* build工程打包后出现的文件 程序执行来源于build中的main.js ,基本是对lib文件夹下通过babel对于代码进行的转换，所以调试的时候建议根据main.js调试
* config.js 配置文件
* logger.js 日志处理
* parser.js 终端命令的处理
* utils.js 基础函数
main.js 中除了启动的一些检查，重点代码在于
```
import { server as baseServer } from 'appium-base-driver';
let router = getAppiumRouter(args);//该函数来源于appium.js
  //此处router返回的是个函数  用于装填路由路径 该函数执行路径位于/mjsonwp/mjsonwp.js 中routeConfiguringFunction 的返回函数
  //baseServer执行之后服务开始启动
let server = await baseServer(router, args.port, args.address);
```
这两行代码可nb厉害了……就这两行基本就把基础类库溜了一圈！代码都是几行几行的，然后一调就转一大圈
在appium.js中我们找到了以下函数
```
import { BaseDriver, routeConfiguringFunction, errors,
         isSessionCommand, processCapabilities } from 'appium-base-driver';

function getAppiumRouter (args) {
  let appium = new AppiumDriver(args);//实例化的这个类继承了appium-base-driver！
  return routeConfiguringFunction(appium);//这个函数来自于appium-base-driver 
}
```
由此成功引入基础类库`appium-base-driver`，然后就去那里翻吧
#### 基础类库appium-base-driver
![没截全的地方都是配置](http://upload-images.jianshu.io/upload_images/1094385-7201454a76fa3ac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
画出的几个红框基本就是代码功能分类，其中`jsonwp-proxy`和`mjsonwp`是和协议相关，由于我不是很了解，所以不做介绍，只说明其中代码位置。
上边提到代码转向`routeConfiguringFunction `

#### routeConfiguringFunction
这出自`mjsonwp/mjsonwp.js`，**请牢牢的记住这个返回函数！！！**
```
function routeConfiguringFunction (driver) {
//...省略一些
  // return a function which will add all the routes to the driver
  return function (app) {
    for (let [path, methods] of _.toPairs(METHOD_MAP)) {
      for (let [method, spec] of _.toPairs(methods)) {
        // set up the express route handler
        buildHandler(app, method, path, spec, driver, isSessionCommand(spec.command));
      }
    }
  };
}
```
它返回了一个函数并且里面还传参了`app`,之前断点打到这里时满脑子都是**我是谁，我在哪里，我要干什么**
我们把代码往上看一下`routeConfiguringFunction`的返回值返回给`getAppiumRouter `再返回给let router  然后再传给baseServer（怪我不好好学js……看个代码艰辛非常……）
那么baseServer来自哪里呢？看import，它来自appium-base-server!
**main.js中调用了一圈再次进入appium-base-server**

#### express中的server.js
之前说到要去找baseServer来自哪里，终于在express/server.js中找到了  就是appium的http服务启动的地方！所以说服务开启的地方不在外部！在基础类库里啊！
```
async function server (configureRoutes, port, hostname = null) {
//......
//里面有这样的代码
configureServer (app, configureRoutes) {
//然后再该函数中调用了
  configureRoutes(app);
}
```
所以代码走到这里执行的就是那个返回函数，app是express的实例！那么执行这一步是为了做什么呢？这个`METHOD_MAP`是最大的全路由配置！来自于`mjsonwp/routes.js`,其中500行都是路由配置，经过该配置有效避免批量写`app.get()//balabala`
```

function (app) {
  for (let [path, methods] of _.toPairs(METHOD_MAP)) {
  //balabalbala
 }
```
至此全路由配置装填结束！

#### 服务启动
这个就没有什么波折了，既然server.js都找到了，就再这个文件中,看到这里终于看到了熟悉的node启动~
```
let app = express();
let httpServer = http.createServer(app);
```
当断点跟到这里时终端就可以跳出如下输出,服务启动啦~
```
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
```
### 总结

启动服务执行过程
`lib/main.js` 执行来自于`lib/appium.js`中的`getAppiumRouter`函数
该函数中实例化`AppiumDriver`类，同时读取路由配置文件，该类继承于`Appium-base-driver`库中暴露的基类，路由配置文件来源于`Appium-base-driver`库中`/mjsonwp/routes.js`的配置
配置文件读取之后在`/mjsonwp/mjsonwp.js`中以函数的形式返回`main.js ` 执行`baseServer`函数  该函数来自于`Appium-base-driver/express/server.js`

也就是说整个过程从
>main.js-[调用]-appium.js-[调用]-appium-base-driver/mjsonwp-[返回]-main.js-[调用]-appium-base-driver/express

其实是通过层层调用将路由已配置的方式进行装填并在基础类库`appium-base-server`中启动



