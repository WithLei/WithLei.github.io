---
layout: post  
title:  "retrofit 有关线程调度器"  
date: 2018-12-04  
description: "写在前面 目前很多大厂APP（如淘宝、饿了么、美团等等）并不是纯原生Android&amp;amp;amp;IOS，也不是纯JS开发，而是Hybird APP开发，混合型优势很多：比如热更新，保证在一些类似双十一的活动到来时能够快速上线活动页面，用户不必再去更新APP。再来有效地减小了安装..."
tag: ReactNative
---

## 写在前面
目前很多大厂APP（如淘宝、饿了么、美团等等）并不是纯原生Android&IOS，也不是纯JS开发，而是Hybird APP开发，混合型优势很多：比如热更新，保证在一些类似双十一的活动到来时能够快速上线活动页面，用户不必再去更新APP。再来有效地减小了安装包的体积大小，大部分的界面都位于服务器端，本地只需要进行绘制。
## 1. 新建Android项目
我这里使用之前的项目
![rn01](https://img-blog.csdnimg.cn/20181204154851296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
## 2. 在项目根目录引入React-Native模块
在项目根目录打开终端（tip:按住shift + 右键，选择打开dos窗口）
输入
```
npm init
```
![rn02](https://upload-images.jianshu.io/upload_images/2516894-d01cd9efd4e07c59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/993/format/webp)
这里输入一些该项目的属性，为了生成package.json文件的项目描述。
接下来引入rn的一些模块文件
```
npm install --save react react-native
```
此时会在根目录生成一个node_modules文件夹，存的是RN的一些模块文件
结束后会输出一些日志信息，如果出现版本问题需要重新下载，因为react-native对react的版本有严格要求，高于或低于某个范围都不可以。
```
npm i -S react@某.某.某版本//此处为提示的版本号
```
接下来打开package.json文件，在scripts模块下添加
```json
"start": "node node_modules/react-native/local-cli/cli.js start"
```
我的package.json文件
```json
{
  "name": "plusclubrn",
  "version": "1.0.0",
  "description": "plusclub in rn",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/WithLei/plusClub.git"
  },
  "keywords": [
    "react",
    "native"
  ],
  "author": "renly",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/WithLei/plusClub/issues"
  },
  "homepage": "https://github.com/WithLei/plusClub#readme",
  "dependencies": {
    "express": "^4.16.4",
    "react": "^16.6.3",
    "react-native": "^0.57.7"
  },
  "devDependencies": {}
}
```
在项目根目录新建一个文件**index.android.js**，并将以下代码拷贝进去
```js
import React, {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View
} from 'react-native';
class HelloWorld extends Component {
    render() {
        return (
            <View style={styles.container }>
                <Text style={styles.hello}>Hello, World</Text >
                <Text style={styles.hello}>This is my first RN fixed Android Project</Text >
            </View >
        )
    }
}
var styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
    },
    hello: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
});
AppRegistry.registerComponent('HelloWorld', () => HelloWorld);
```
## 3.Native部分的配置
首先在app下的gradle添加依赖
```java
    implementation 'com.facebook.react:react-native:+'
```
这个+替换为你的项目根文件下node_modules/react-ative/android 下的react-native的版本

在项目目录下的gradle添加以下
```java
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            url "https://jitpack.io"
        }
        // 如果有多个maven依赖，需要分开添加
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            // 注意这里和官网的是不一样的，由于我们的node_modules在根目录下，官方为
            url "$rootDir/node_modules/react-native/android"
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
声明网络权限：
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
创建一个容器类
```java
public class MyReactActivity extends ReactActivity {
    @Nullable
    @Override
    protected String getMainComponentName() {
        return "HelloWorld";
    }
}
```
在清单文件里面声明此Activity
```xml
<activity android:name=".MyRNActivity"
           android:label="@string/app_name"
           android:theme="@style/Theme.AppCompat.Light.NoActionBar"/>
```
配置调试的界面
```xml
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
```
配置Application,继承RN的Application
```java
import android.app.Application;
import com.facebook.react.ReactApplication;
import com.facebook.react.ReactNativeHost;
import com.facebook.react.ReactPackage;
import com.facebook.react.shell.MainReactPackage;
import com.facebook.soloader.SoLoader;
import java.util.Arrays;
import java.util.List;
public class App extends Application implements ReactApplication {
    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {

        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage()
                    //将我们创建的包管理器给添加进来

            );
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        SoLoader.init(this, /* native exopackage */ false);
    }
}
```
做一个跳转，我选择绑定到了原生的button上做响应事件
```
startActivity(new Intent(this, MyRNActivity.class));
```
## 运行APP
我们可以先把APP运行起来，跑在虚拟机或者真机上
然后启动服务器：在项目根目录下打开终端运行
```
npm start
```
然后就可以运行了，中间有不少坑，碰到的时候没截图，之后参考了很多博客和官方Issue解决了，这里就不复现bug了。

参考博客：
[将RN嵌入到现有的Android应用中 - Byron_Walden](https://www.jianshu.com/p/99f2a4c21986)
[Guide文档 - React native中文网](https://reactnative.cn/#content)
[Github issues - facebook](https://github.com/facebook/react-native/issues)
