---
layout: post
title: "Android测试入门(一) Activity测试简介"
date: 2014-03-02 21:54:03 +0800
comments: true
categories: [Android测试, Android, 林翔宇]
---

作者：林翔宇



## 瞎扯

什么是测试？试想大家写一个程序的时候，经常要手动在命令行下输入测试数据，然后看看程序的输出与实际一不一致。而在做App等UI事件众多的程序的时候，经常要手动点击按钮等控件，看看会不会有预期一样的反应。

大家每次都这样做，累不累呢～有什么好方法可以帮我们节约这些时间从而省出更多时间来约漂亮的女孩子呢～

当然有咯。我们再写一个程序，把这些步骤自动化就行咯。这个程序就是一个测试。

当然软件测试包含的东西很多，我们就来简单扯扯Android测试相关的内容，分几期做一个简单的入门介绍。


## 简介

[Official Doc](https://developer.android.com/tools/testing/index.html)


Android SDK包含了强大的测试工具

- 一个Android Test本身是一个Android Application，并且它的AndroidManifest.xml文件里面有它测试目标的信息
- 与一个Application不同，Android Test由多个测试用例组成，而不是Android components。
- Android测试拥立扩展自JUnit TestCase类，并且可以在测试时发送触摸和键盘输入信息
- 根据不同的component (application, activity, content provider, or service)，你可以选择不同的Android test 基类
- Eclipese/ADT, Intellij IDEA, Android Studio均提供了和Android Test的良好整合

一个 Test Application包含了以下几种对于Activity相关的测试

- 启动的初始化状态测试，比如测试一个UI控件是否成功实例化，会不会返回空指针。
- UI测试，比如测试按钮点击出现的结果与预期比较一不一样
- 状态转移时候的测试，当一个Activity在Pause，Start等几个状态之间变换的时候，测试有的控件，变量。

## 新建一个测试

在Intellij IDEA中，可以在项目文件下 File - New Module - Android Test Module来创建一个测试

![Intellij New Test Module](http://i.imgur.com/z5M4eF5.png)

Intellij IDEA会生成一个类似下面的程序结构


``` 

MMDailyTest
├── AndroidManifest.xml
├── MMDailyTest.iml
├── ant.properties
├── bin
├── build.xml
├── gen
│   └── org
│       └── nupter
│           └── mmdaily
│               └── tests
│                   ├── BuildConfig.java
│                   ├── Manifest.java
│                   └── R.java
├── libs
├── local.properties
├── proguard-project.txt
├── project.properties
├── res
└── src
    └── org
        └── nupter
            └── mmdaily
                ├── api
                └── ui
                    └── ActivityListActivityTest.java
                    
```

和一个Android App目录基本一致

来看看 AndroidManifest.xml 有什么东西


```xml

<?xml version="1.0" encoding="utf-8"?>
<!-- package name must be unique so suffix with "tests" so package loader doesn't ignore us -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="org.nupter.mmdaily.tests"
          android:versionCode="1"
          android:versionName="1.0">
    <!-- We add an application tag here just so that we can indicate that
         this package needs to link against the android.test library,
         which is needed when building test cases. -->
    <application>
        <uses-library android:name="android.test.runner"/>
    </application>
    <!--
    This declares that this application uses the instrumentation test runner targeting
    the package of org.nupter.mmdaily.  To run the tests use the command:
    "adb shell am instrument -w org.nupter.mmdaily.tests/android.test.InstrumentationTestRunner"
    -->
    <instrumentation android:name="android.test.InstrumentationTestRunner"
                     android:targetPackage="org.nupter.mmdaily"
                     android:label="Tests for org.nupter.mmdaily"/>
</manifest>

```

默认生成的注释写得很清楚，相比普通的App的AndroidManifest.xml ，多了一个android:name="android.test.runner"的library依赖，以及一个instrumentation元素，制定了测试的对象的包。


## 第一个测试类

一个测试首先是一个继承自`android.test.ActivityInstrumentationTestCase2<YOUR_ACTIVITY_CLASS>` 的类

比如我们为ReadPageActivity创建一个ReadPageActivityTest.java

这个ReadPageActivity里面只有一个显示为返回的Button，我们测试按这个按钮的反应，以及这个按钮的文字是不是一直显示为Button


```java

package org.nupter.mmdaily.ui;

import android.app.Instrumentation;
import android.test.ActivityInstrumentationTestCase2;
import android.widget.Button;
import org.nupter.mmdaily.R;

/**
 * Author: linxiangyu
 * Date:   3/2/14
 * Time:   3:24 AM
 */
public class ReadPageActivityTest extends ActivityInstrumentationTestCase2<ReadPageActivity> {

    /**
     * 一个Android Activity的Example，是<a href="https://developer.android.com/tools/testing/testing_android.html">官方教程的简化版</a>
     */

    private ReadPageActivity mActivity;
    private Button mButton;


    public ReadPageActivityTest() {
        super(ReadPageActivity.class);    // 整个测试开始之前构建的初始化条件。注意这个构造函数的样子，写错了运行测试的时候这个类就不会被调用了
    }

    @Override
    public void setUp() throws Exception {
        super.setUp();       // 每个TestCase开始之前都会调用，在这里初始化一些成员变量，可以避免TestCase对变量的修改的影响

        setActivityInitialTouchMode(false); //如果你的测试要触发触摸或者键盘输入，那么必须手动关闭掉Touch Mode

        mActivity = getActivity();    // 获取activity

        mButton = (Button) mActivity.findViewById(
               R.id.ReadPageBack
        );

    }

    @Override
    public void tearDown() throws Exception {
        super.tearDown();    // 每个TestCase结束之后都会调用
    }



    public void testPreConditions() {
        assertTrue(mButton != null);
    } // 测试一些初始条件，比如你的UI控件不为空。


    public void testBaseUI(){
        assertEquals(mButton.getText(), "返回");
        mActivity.runOnUiThread(
                new Runnable() {
                    public void run() {
                        mButton.requestFocus();
                        mButton.callOnClick();
                    } //
                }); // UI控件的改变必须这样运行在一个runOnUiThread 中

    }  // 测试一些UI事件

    public void testStatePause() {
        Instrumentation mInstr = this.getInstrumentation();
        mInstr.callActivityOnPause(mActivity);
        assertEquals(mButton.getText(), "返回");
    } // 测试一些App状态转移的时候的样子
}


```

### 运行测试

![Run Test](http://i.imgur.com/1VWC6oS.png)


你会在Intellij 下面的Run窗口里面看到相关的提示，以及你的模拟器/手机迅速跳转到相应的测试Activity，然后又返回来。

![Test Result](http://i.imgur.com/oTD4PMs.png)

很高兴，测试一次通过，全绿。

### 项目代码

[项目的代码](https://github.com/mmclub/MMDaily/tree/23ee19d4b25889a5a112f4f6d39e4d306fc8d82b)

这就是一个最简单的Android测试的样例了，之后我们会介绍其他Android SDK里面测试相关的东西。


