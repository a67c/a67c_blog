---
layout: Appium通过日志分析服务端执行过程-IOS端
title: Appium通过日志分析服务端执行过程-IOS端
date: 2018.02.22 14:49
tags:
    - Appium
---

#### 写在前面
1. 本文日志为在IOS模拟器上进行的测试 appium 1.7.2
2. 在appium服务端中，日志分为log.info和log.debug 一般每个文件夹下面都有logger.js 该js中规定当前log格式。
3. info为基础信息，debug可以看做为调试信息。
4. 本篇为初步分析，后来对日志又进行了更细致的断点查找，待整理，如有错误还请指出
5. 假如日志前面打印出的[Appium][XUCITES]等，根据该名字可以查到日志来源于哪个文件夹下，[debug][MJSONWP]代表当前是采用log.debug方式打印日志，且日志来源于`appium-base-driver/mjsonwp`的文件夹

#### 第一部分 启动服务并创建sessionID
appium启动：
```
[Appium] Welcome to Appium v1.7.2
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
```
客户端发送http请求，并传递配置参数
```
[HTTP] --> POST /wd/hub/session {"capabilities":{"alwaysMatch":{"platformName":"iOS"},"firstMatch":[{}]},"desiredCapabilities":{"platformVersion":"11.2","deviceName":"iPhone 6s","app":"/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app","platformName":"iOS"}}
```
[MJSONWP]为来自于appium服务端中封装好的类库`appium-base-driver`中mjsonwp文件夹下的mjsonwp.js，一般用于server端的交互，包括处理HTTP的请求，session会话的创建删除。以下为创建Session
```
[debug] [MJSONWP] Calling AppiumDriver.createSession() with args: [{"platformVersion":"11.2","deviceName":"iPhone 6s","app":"/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app","platformName":"iOS"},null,{"alwaysMatch":{"platformName":"iOS"},"firstMatch":[{}]}]
```
[BaseDriver]日志来源于`appium-base-driver/basedriver`文件夹。
下面日志主要用于打印整个appium中的事件触发，所有的请求都会触发各种类下的`executeCommand`函数，而这些函数全部继承driver.js中的WebDriver，所以所有的请求都会打印出`driver.js`中的logHistory。
```
[debug] [BaseDriver] Event 'newSessionRequested' logged at 1516514435268 (14:00:35 GMT+0800 (CST))
 ```
[Appium]日志来源于`lib`文件夹下
```
[Appium] Merged W3C capabilities {"alwaysMatch":{"platformName":"iOS"},"firstMat... into desiredCapabilities object {"platformVersion":"11.2","deviceName":"iPhone ...
[Appium] Requested iOS support with version >= 10, using XCUITest driver instead of UIAutomation-based driver, since the latter is unsupported on iOS 10 and up.
[Appium] Creating new XCUITestDriver (v2.64.0) session
[Appium] Capabilities:
[Appium]   platformVersion: 11.2
[Appium]   deviceName: iPhone 6s
[Appium]   app: /Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app
[Appium]   platformName: iOS
```
[BaseDriver]打印出当前sessionId创建成功，来源于`appium-base-driver/basedriver/session.js`文件。

* creatSession跳转过程说明：
创建creatSession路线：mjsonwp.js-》appium.js中的createSession-》跳转到XCUITEST中的createSession-》跳转到appium-base-driver/basedriver/driver.js中的creatSession
* 如何联系起了XCUITest？
通过appium.js中的`curSessionDataForDriver(InnerDriver)`，当判断出当前是什么系统配置时，InnerDriver此时已变成IOS或者Android的Driver

```
[BaseDriver] Session created with session id: 8790f1db-9627-41ce-a534-bc40159c7194
```
当创建SessionID之后，此时进入的是`appium-xcuitest-driver`类库中。
```
[debug] [XCUITest] Current user: '用户名'
[debug] [XCUITest] Current version of libimobiledevice: stable 1.2.0 (bottled), HEAD
[debug] [XCUITest] Xcode version set to '9.2' (tools v9.2.0.0.1.1510905681)
[debug] [XCUITest] iOS SDK Version set to '11.2'
```
来自于`appium-base-driver/basedriver/driver.js`中的logHistory函数
```
[debug] [BaseDriver] Event 'xcodeDetailsRetrieved' logged at 1516514436354 (14:00:36 GMT+0800 (CST))
```
[iOSSim]来自于`appium-ios-simulator`的日志，接下来都是在IOS检测模拟器，判断APP，安装APP的操作
```
[iOSSim] Constructing iOS simulator for Xcode version 9.2 with udid '2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5'
[XCUITest] Determining device to run tests on: udid: '2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5', real device: false
[BaseDriver] Using local app '/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app'
[debug] [BaseDriver] Event 'appConfigured' logged at 1516514436692 (14:00:36 GMT+0800 (CST))
[debug] [XCUITest] Checking whether app '/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app' is actually present on file system
[debug] [XCUITest] App is present
[debug] [iOS] Getting bundle ID from app '/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app': 'io.appium.TestApp'
[debug] [BaseDriver] Event 'resetStarted' logged at 1516514436695 (14:00:36 GMT+0800 (CST))
[XCUITest] Not scrubbing third party app in anticipation of uninstall
[debug] [BaseDriver] Event 'resetComplete' logged at 1516514436881 (14:00:36 GMT+0800 (CST))
[debug] [XCUITest] Starting log capture for iOS Simulator with udid '2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5', using 'xcrun simctl spawn 2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5 log stream --style compact'
[debug] [BaseDriver] Event 'logCaptureStarted' logged at 1516514437312 (14:00:37 GMT+0800 (CST))
[XCUITest] Setting up simulator
[debug] [iOS] No reason to set locale
[debug] [iOS] No iOS / app preferences to set
[debug] [iOSSim] Matched 1 Simulator cache item for cleanup: /Users/用户名/Library/Developer/CoreSimulator/Devices/2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5/data/Library/Caches/com.apple.mobile.installd.staging
[debug] [iOSSim] Setting common Simulator preferences to {"ConnectHardwareKeyboard":false}
[debug] [iOSSim] Updated 2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5 Simulator preferences at '/Users/用户名/Library/Preferences/com.apple.iphonesimulator.plist' with {"ConnectHardwareKeyboard":false}
[debug] [iOSSim] The count of running Simulator UI client instances is 1
[iOSSim] Both Simulator with UDID 2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5 and the UI client are currently running
[debug] [BaseDriver] Event 'simStarted' logged at 1516514437783 (14:00:37 GMT+0800 (CST))
[debug] [XCUITest] Reset requested. Removing app with id 'io.appium.TestApp' from the device
[debug] [XCUITest] Installing '/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app' on Simulator with UUID '2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5'...
[debug] [XCUITest] The app has been installed successfully.
[debug] [BaseDriver] Event 'appInstalled' logged at 1516514440337 (14:00:40 GMT+0800 (CST))
```
APP安装成功之后开始处理WDA
```
[XCUITest] Using WDA path: '/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/WebDriverAgent'
[XCUITest] Using WDA agent: '/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/WebDriverAgent/WebDriverAgent.xcodeproj'
[debug] [XCUITest] No obsolete cached processes from previous WDA sessions listening on port 8100 have been found
[debug] [JSONWP Proxy] Proxying [GET /status] to [GET http://localhost:8100/status] with no body
[debug] [JSONWP Proxy] Got response with status 200: "{\n  \"value\" : {\n    \"state\" : \"success\",\n    \"os\" : {\n      \"name\" : \"iOS\",\n      \"version\" : \"11.2\",\n      \"sdkVersion\" : \"11.2\"\n    },\n    \"ios\" : {\n      \"simulatorVersion\" : \"11.2\",\n      \"ip\" : \"192.168.0.102\"\n    },\n    \"build\" : {\n      \"time\" : \"Jan 14 2018 23:25:10\"\n    }\n  },\n  \"sessionId\" : \"17DE3FB9-3196-4C9C-9D55-3BFFDD0C1DEB\",\n  \"status\" : 0\n}"
[XCUITest] Will reuse previously cached WDA instance at 'http://localhost:8100/'. Set the wdaLocalPort capability to a value different from 8100 if this is an undesired behavior.
[debug] [BaseDriver] Event 'wdaStartAttempted' logged at 1516514440422 (14:00:40 GMT+0800 (CST))
[XCUITest] Using provided WebdriverAgent at 'http://localhost:8100/'
[debug] [BaseDriver] Event 'wdaSessionAttempted' logged at 1516514440423 (14:00:40 GMT+0800 (CST))
[debug] [XCUITest] Sending createSession command to WDA
```
在XCUITest中开启WDA之后，此时链接的request和response进入了  `appium-base-driver/lib/jsonwp-proxyproxy.js` ，这个js主要是做S/D的链接，即server服务端与devices设备的链接
```
[debug] [JSONWP Proxy] Proxying [GET /status] to [GET http://localhost:8100/status] with no body
[debug] [JSONWP Proxy] Got response with status 200: "{\n  \"value\" : {\n    \"state\" : \"success\",\n    \"os\" : {\n      \"name\" : \"iOS\",\n      \"version\" : \"11.2\",\n      \"sdkVersion\" : \"11.2\"\n    },\n    \"ios\" : {\n      \"simulatorVersion\" : \"11.2\",\n      \"ip\" : \"192.168.0.102\"\n    },\n    \"build\" : {\n      \"time\" : \"Jan 14 2018 23:25:10\"\n    }\n  },\n  \"sessionId\" : \"17DE3FB9-3196-4C9C-9D55-3BFFDD0C1DEB\",\n  \"status\" : 0\n}"
[debug] [JSONWP Proxy] Proxying [POST /session] to [POST http://localhost:8100/session] with body: {"desiredCapabilities":{"bundleId":"io.appium.TestApp","arguments":[],"environment":{},"shouldWaitForQuiescence":true,"shouldUseTestManagerForVisibilityDetection":false,"maxTypingFrequency":60,"shouldUseSingletonTestManager":true}}
[debug] [JSONWP Proxy] Got response with status 200: {"value":{"sessionId":"95403F32-CD9B-4A0E-AA89-4BE9C899FBBC","capabilities":{"device":"iphone","browserName":"TestApp","sdkVersion":"11.2","CFBundleIdentifier":"io.appium.TestApp"}},"sessionId":"95403F32-CD9B-4A0E-AA89-4BE9C899FBBC","status":0}
[debug] [BaseDriver] Event 'wdaSessionStarted' logged at 1516514443103 (14:00:43 GMT+0800 (CST))
[debug] [BaseDriver] Event 'wdaStarted' logged at 1516514443103 (14:00:43 GMT+0800 (CST))
[XCUITest] Skipping setting of the initial display orientation. Set the "orientation" capability to either "LANDSCAPE" or "PORTRAIT", if this is an undesired behavior.
[debug] [BaseDriver] Event 'orientationSet' logged at 1516514443103 (14:00:43 GMT+0800 (CST))
```
上边WDA开启完毕，session创建成功，这个session的创建过程是从

**mjsonwp.js（HTTP请求入口）-》appium.js中的createSession-》跳转到XCUITEST中的createSession-》跳转到appium-base-driver/basedriver/driver.js中的creatSession   创建成功之后回到了appium.js中将log打印，最后回到了mjsonwp.js将response返回并通知。**


```
[Appium] New XCUITestDriver session created successfully, session 8790f1db-9627-41ce-a534-bc40159c7194 added to master session list
[debug] [BaseDriver] Event 'newSessionStarted' logged at 1516514443104 (14:00:43 GMT+0800 (CST))
[debug] [MJSONWP] Responding to client with driver.createSession() result: {"webStorageEnabled":false,"locationContextEnabled":false,"browserName":"","platform":"MAC","javascriptEnabled":true,"databaseEnabled":false,"takesScreenshot":true,"networkConnectionEnabled":false,"platformVersion":"11.2","deviceName":"iPhone 6s","app":"/Users/用户名/Documents/github/python-client-master/appium/TestApp/build/release-iphonesimulator/TestApp-iphonesimulator.app","platformName":"iOS","udid":"2EF911A2-CA9C-4D28-96EB-3DBC8DF39FA5"}
[HTTP] <-- POST /wd/hub/session 200 7841 ms - 515
```
---------
#### 第二部分 发送操作请求执行并返回

客户端传来请求获取element，该请求进入mjsonwp.js进行处理，通过executeCommand函数进入appium.js 进入appium-xcuitest-driver‘类库 与JSONWP Proxy[appium-base-driver/lib/jsonwp/proxy.js]进行协议交互  交互结果返回MJSONWP[appium-base-driver/lib/mjsonwp/mjsonwp.js]
```
[HTTP] --> POST /wd/hub/session/8790f1db-9627-41ce-a534-bc40159c7194/element {"using":"accessibility id","sessionId":"8790f1db-9627-41ce-a534-bc40159c7194","value":"TextField1"}
[debug] [MJSONWP] Calling AppiumDriver.findElement() with args: ["accessibility id","TextField1","8790f1db-9627-41ce-a534-bc40159c7194"]
[debug] [XCUITest] Executing command 'findElement'
[debug] [BaseDriver] Valid locator strategies for this request: xpath, id, name, class name, -ios predicate string, -ios class chain, accessibility id
[debug] [BaseDriver] Waiting up to 0 ms for condition
[debug] [JSONWP Proxy] Proxying [POST /element] to [POST http://localhost:8100/session/95403F32-CD9B-4A0E-AA89-4BE9C899FBBC/element] with body: {"using":"accessibility id","value":"TextField1"}
[debug] [JSONWP Proxy] Got response with status 200: {"value":{"ELEMENT":"A3063789-1A9E-4FA2-A645-F2E257F0BB30"},"sessionId":"95403F32-CD9B-4A0E-AA89-4BE9C899FBBC","status":0}
[debug] [MJSONWP] Responding to client with driver.findElement() result: {"ELEMENT":"A3063789-1A9E-4FA2-A645-F2E257F0BB30"}
[HTTP] <-- POST /wd/hub/session/8790f1db-9627-41ce-a534-bc40159c7194/element 200 83 ms - 122
```
客户端再次传来请求，以下都是重复请求与返回，直到客户端传来删除会话。
当客户端单个任务执行完毕时，客户端传来删除会话，通讯与其它没有什么差别。
```
[HTTP] --> DELETE /wd/hub/session/5469f63c-9b2b-4a3a-bbdb-fa26bd6315f9 {}
[debug] [MJSONWP] Calling AppiumDriver.deleteSession() with args: ["5469f63c-9b2b-4a3a-bbdb-fa26bd6315f9"]
[debug] [BaseDriver] Event 'quitSessionRequested' logged at 1516514455122 (14:00:55 GMT+0800 (CST))
[Appium] Removing session 5469f63c-9b2b-4a3a-bbdb-fa26bd6315f9 from our master session list
[debug] [JSONWP Proxy] Proxying [DELETE /session/5469f63c-9b2b-4a3a-bbdb-fa26bd6315f9] to [DELETE http://localhost:8100/session/E9944FDA-46DC-48B4-824B-A2C99D0B3BD2] with no body
[debug] [JSONWP Proxy] Got response with status 200: "{\n  \"value\" : {\n\n  },\n  \"sessionId\" : \"334D0B8A-6B9E-4BAA-9BDA-BFEE66EBB584\",\n  \"status\" : 0\n}"
[debug] [XCUITest] Not clearing log files. Use `clearSystemFiles` capability to turn on.
[debug] [iOSLog] Stopping iOS log capture
[debug] [BaseDriver] Event 'quitSessionFinished' logged at 1516514455352 (14:00:55 GMT+0800 (CST))
[debug] [MJSONWP] Received response: null
[debug] [MJSONWP] But deleting session, so not returning
[debug] [MJSONWP] Responding to client with driver.deleteSession() result: null
[HTTP] <-- DELETE /wd/hub/session/5469f63c-9b2b-4a3a-bbdb-fa26bd6315f9 200 231 ms - 76

```