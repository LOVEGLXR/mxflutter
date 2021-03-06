# MXFlutter

MXFlutter 是一套基于 JavaScript 的 Flutter 框架，它用极类似Dart的开发方式，通过编写JavaScript 代码，来开发 Flutter 应用。更多细节在 [基于JavaScript的Flutter框架详细介绍](https://juejin.im/post/5d11a4f06fb9a07ec63b21ea)。

## MXFlutter 接入指南
mxflutter 是一个标准的 Dart package，可以按照 Dart 引入 package 的方式接入，步骤非常简单。在开始接入之前，运行体验 mxfltuter 的两个例子，对接入会有帮助，一个是示例丰富但比较复杂的例子，在 https://github.com/mxflutter/mxflutter.git 主库根目录，一个是最简化接入示例，在主库 mxflutter/example/ 目录下 ，推荐第一次接入按照第二个例子来。

### 三步接入MXFlutter

#### 1. 添加依赖

推荐 fork 在 github 的主库 https://github.com/mxflutter/mxflutter.git 来接入，方面自己修改和定期从主库的更新。

```
  dependencies:
    mxflutter:
      git:
        url: https://github.com/mxflutter/mxflutter.git
        path: mxflutter/

```

#### 2. 拷贝示例JS代码文件，配置JS代码资源引入

第一步拷贝JS代码文件：mxflutter 主库提供了JS代码模版，拷贝主库 mxflutter/example/mxflutter_js_src (https://github.com/mxflutter/mxflutter/tree/master/mxflutter/example/mxflutter_js_src) 文件夹到你的工程目录，和pubspec.yaml文件同级。


第二步在 pubspec.yaml 文件中引入 mxflutter_js_src 代码资源文件夹


```
 flutter:
   assets:
     - mxflutter_js_src/

```

*特别注意：第一步拷贝的文件夹和第二步导入的资源是配套的，因为 pubspec.yaml 导入资源时，不会自动导入子文件夹， 如果你是拷贝的主库根目录 https://github.com/mxflutter/mxflutter/tree/master/mxflutter_js_src 文件夹，要配套按照主库 pubspec.yaml 的资源配置来引入，学习接入建议使用 mxflutter/example/mxflutter_js_src 的示例*

完成后目录结构应该是这样的

```
my_flutter/
├── lib/
│   └── main.dart
└── pubspec.yaml
└── mxflutter_js_src/
│   └── main.js
│   └── home_page.js
│   └── js_dev_demo.js
│   └── mxjsbuilder_demo.js
```

#### 3. 在Flutter代码中，运行MXFlutter，打开由JS编写的页面

在 main.dart 文件中，调用 runJSApp 启动JSApp，runJSApp 函数如果不传任何参数，默认会运行 mxflutter_js_src/mian.js 文件


```
//mxflutter
import 'package:mxflutter/mxflutter.dart';

void main() {
  //-------1. MXFlutter 启动---------
  MXJSFlutter.getInstance().runJSApp();
  runApp(MyApp());
}

```

在合适时机，比如用户点击界面时，打开JS页面。



```
  onTap: () {
                //-------2. MXFlutter push 一个使用MXFlutter框架编写的页面
                Navigator.push(
                    context,
                    MaterialPageRoute(
                        builder: (context) => MXJSPageWidget(
                            jsWidgetName: "MXJSWidgetHomePage")));
              }

```

上面代码 MXJSPageWidget 的参数 jsWidgetName: "MXJSWidgetHomePage",在mxflutter_js_src/main.js  MyApp::createJSWidgetWithName 函数中使用，用来标示打开哪个JS页面。

##### bingo 如果顺利的话，基本得接入工作已经完成，你应该可以打开一个经典的 Flutter 示例页面了。接下来可以尝试修改下 mxflutter_js_src/ 文件夹下的JS文件，可以看到 UI 变化。


### 更多易用的配置

#### iOS模拟器热重载支持

在 https://github.com/mxflutter/mxflutter.git 主库iOS工程或者主库 mxflutter/example/ 目录的例子中，[AppDelegate.m](https://git.code.oa.com/MXFlutter/MXFlutter/blob/master/mxflutter/example/ios/Runner/AppDelegate.m) 文件详细注释了如果打开模拟器热重载。拷贝setupMXFlutterJSPath 函数，在 Flutter 插件注册  [GeneratedPluginRegistrant registerWithRegistry:self]; 之前调用即可


```
- (void)setupMXFlutterJSPath{
    
    ///mxflutter  js代码模拟器热重载支持
    ///把JS文件重定向到本地地址，简单支持模拟器JS文件热重载，
    ///仅支持模拟器环境使用
    ///*发布版本的热更新，也使用同样API设置为你的JS代码下载目录
    
#if TARGET_IPHONE_SIMULATOR
    
   //业务JS代码的热重载
    NSString *jsAppPath = nil;
    NSArray *jsAppSearchPathList = nil;
    
    ///mxflutter framework的开发者用来加快调试，如不需要修改mxflutter jsframework，无需设置jsFramewrokPath
    NSString *jsFramewrokPath = nil;
    
    ///1. 可以直接修改定位到绝对路径,但多人协作开发时，每个人路径不同需要，需要都配置，推荐第二种方法，配置PROJECT_DIR
    ///jsFramewrokPath = @“/Volumes/Data/Work/mxflutter2/mxflutter/js_lib”;
    ///jsAppPath =  @"/Volumes/Data/Work/mxflutter2/mxflutter_js_src/"
    
    ///2. 可以配置PROJECT_DIR，把JS代码路径地址重定向到开发机iMac地址，用来支持模拟器热重载，支持多人协同，配置PROJECT_DIR方法如下。
    ///XCode -> Build Settings -> Preprocessor Macros ： Debug下增加 PROJECT_DIR=@\""$PROJECT_DIR\/"\"
    ///这是github https://github.com/TGIF-iMatrix/mxflutter.git 下的 flutter/example/ios 工程为例子，js_lib的
    ///物理路径配置如下
    
#ifdef PROJECT_DIR
    
    //可以根据自己的工程部署调整相对路径
    jsAppPath = [PROJECT_DIR stringByAppendingPathComponent:@"../mxflutter_js_src/"];
    
    //
    jsAppSearchPathList = @[
        [jsAppPath  stringByAppendingPathComponent:@"app_demo/"],
        [jsAppPath  stringByAppendingPathComponent:@"mxjsbuilder_demo/"]
    ];
    
    jsFramewrokPath = [PROJECT_DIR stringByAppendingPathComponent:@"../../js_lib/"];
    
#endif
    
    //可以调用[MXFlutterPlugin setJSFramewrokPath:jsFramewrokPath]; 设置jsframework到Documents你的下载目录，支持framework的热更新
    if (jsFramewrokPath.length > 0) {
         [MXFlutterPlugin setJSFramewrokPath:jsFramewrokPath];
    }
    
    if (jsAppPath.length >  0) {
        [MXFlutterPlugin setJSAppPath:jsAppPath jsAppSearchPathList:jsAppSearchPathList];
    }
   
#endif
 
}

```





