# iOS-Monkey-Test-Tool

---

### 1. 简要说明

1. 支持真机测试和模拟器测试

2. 支持测试报告截图

3. 支持测试执行过程中跳出app，跳回继续跑monkey

4. 支持登陆等业务流程


### 2. 优势

1. 无需插桩
2. 高效率，每秒4-5个action
3. 轻量极简




### 3.原理

1. 在Xcode中执行xcuitest时启动一个监听server，外部命令时可引导启动app并执行monkey
2. 增加一个定时执行的action，检测当前app，如果不是待测则重新launcher
3. 增加一个定时的action，检测当某个关键点出现时，往事件队列中插入一个业务事件




### 4.安装说明

1. ```$ git clone https://github.com/zhangzhao4444/Fastmonkey.git```


2. ```$ cd /Users/xxxxx/Fastmonkey/XCTestWD-master```

必须依赖安装：

3. ```$ brew install  carthage 或 $ carthage update```

iOS 11必须依赖安装：

4. ```$  brew uninstall ideviceinstaller```

5. ```$  brew uninstall libimobiledevice```

6. ```$  brew install --HEAD libimobiledevice```

7. ```$  brew link --overwrite libimobiledevice```

8. ```$  brew install ideviceinstaller```

9. ```$  brew link --overwrite ideviceinstaller```


### 5.配置XCTestWD项目工程

1. 打开XCTestWD工程

2. 设置签名账号

3. 右键点击server目录，选择"Add Files to "XCTestWD"，选择【XCTestWDMonkey.swift】文件添加到server

4. 右键点击PrivateHeaders目录，选择"Add Files to "XCTestWD"，选择【XCTestWDApplication.h】与【XCTestWDApplication.m】文件添加到PrivateHeaders

5. 清空XCTestWD的Objective-C Bridging Header选项中的值（在build setting中）

6. 修改运行模式，将XCTestWDRunner.swift文件中的serverMode设置为fals，(修改此项为false后，这样Monkey就可以直接从Xcode中运行，不需要再使用额外的命令了)

7. 修改XCTestWDMonkey.swift文件中的bundleID为被测App的值

8. 修改Monkey.swift文件中的elapsedTime值确定你需要运行多长时间的Monkey，注意单位是秒

9. 登录插桩<br>
(1).修改 XCtestWDMonekyController.swift 第70行注释放开<br>
(2).修改XCTestWDMonkey.swift 第42行放开注释<br>
(3).修改XCTestWDMonkey.swift  第104行
```let tag = "//XCUIElementTypeApplication[@name='壹诊所']/XCUIElementTypeWindow[1]/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther[2]/XCUIElementTypeTextField"```

(4) 修改 MonkeyXCTestPrivate.swift，替换166行附近的 username，passwd，button 的 xpath

```let username = "//XCUIElementTypeApplication[@name='壹诊所']/XCUIElementTypeWindow[1]/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther[2]/XCUIElementTypeTextField"
let passwd = "//XCUIElementTypeApplication[@name='壹诊所']/XCUIElementTypeWindow[1]/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther/XCUIElementTypeOther[2]/XCUIElementTypeSecureTextField"
let button = "//XCUIElementTypeButton[@name='开始']"

let value = "1111111111"  设置账号
let value = "111111" 设置密码
```




### 6.使用说明

1. 选择设备
2. ⌘+U



### 7.命令行

```$ cd ~/Desktop/Doctor

//build app  放到./tmp目录下
$ xcodebuild -scheme Doctor -workspace Doctor.xcworkspace -destination 'platform=iOS Simulator,name=iPhone 8’ -derivedDataPath ./tmp

//查看模拟器的信息
$ instruments -s

//指定模拟器
$ xcrun simctl boot "8D3DA864-E725-4D64-822C-E0CBBE103993"

//启动模拟器
$ open `xcode-select -p`/Applications/Simulator.app

//卸载app
$ xcrun simctl uninstall booted 'com.yibao120.doctors'

//安装app
$ xcrun simctl install booted ./tmp/Build/Products/Debug-iphonesimulator/Doctor.app

//打开app
$ xcrun simctl launch booted 'com.yibao120.doctors'

$ cd ~/Documents/Fastmonkey/XCTestWD-master/XCTestWD/

//运行fastmonkey的测试
$ xcodebuild -project XCTestWD.xcodeproj -scheme XCTestWDUITests -destination 'platform=iOS Simulator,name=iPhone 8’ XCTESTWD_PORT=8001 clean test

//关闭所有模拟器
$ killall Simulator
```



### 8.查看屏幕截图

```$ open /Users/sunbaoyin//Library/Developer/Xcode/DerivedData/```

依次进入XCTestWD 开头文件夹 -> Logs 文件夹 -> Test 文件夹 -> Attachments文件夹



### 9.Jenkins部署

1. 新建任务
2. 配置构建脚本，供参考的shell

```echo "\033[31m Setp1:update git \033[0m"
cd /Users/dengxudong/Desktop/Doctor

git pull origin monkeytest

echo  "\033[31m Setp2:build app \033[0m"
xcodebuild -scheme Doctor -workspace Doctor.xcworkspace -destination 'platform=iOS Simulator,OS=11.1,name=iPhone 8' -derivedDataPath ./tmp

echo  "\033[31m Setp3:start iPhone Simulator \033[0m"
xcrun simctl boot "8D3DA864-E725-4D64-822C-E0CBBE103993"
open `xcode-select -p`/Applications/Simulator.app

sleep 20s

echo -e "\033[31m Setp4:uninstall app \033[0m"
xcrun simctl uninstall booted 'com.yibao120.doctors'

sleep 10s

echo  "\033[31m Setp4:install app \033[0m"
xcrun simctl install booted ./tmp/Build/Products/Debug-iphonesimulator/Doctor.app

sleep 10s

echo  "\033[31m Setp5:run fashmonkey! \033[0m"
cd /Users/dengxudong/Documents/Fastmonkey/XCTestWD-master/XCTestWD

xcodebuild -project XCTestWD.xcodeproj -scheme XCTestWDUITests -destination 'platform=iOS Simulator,OS=11.1,name=iPhone 8' XCTESTWD_PORT=8001 clean test```
3. 使用 Post build task 构建后操作，供参考的shell

```echo  "\033[31m final:killall Simulator \033[0m"

killall Simulator
```

4. 配置邮件通知：点击任务 -> 配置 -> 构建后操作添加 【E-mail Notification】(输入邮箱，多人以空格符分隔)



参考链接：

1. https://testerhome.com/topics/9524（fastmonkey）
2. http://blog.csdn.net/hlybest24/article/details/60753538（Jenkins邮件通知）

