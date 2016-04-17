---
layout: post
title: "Testing in Android"
date: 2016-03-01 18:27:11 +0800
comments: true
published: true
sharing: true
footer: true
categories: Android
---

IDE: AndroidStudio

AndroidStudio 中将测试分为了本地测试(Local test) 和设备测试 (Instrumented Test) 两大类。

## Local Unit Tests

本地测试，直接在本地开发者电脑上运行测试，主要用来测试纯逻辑部分的代码，不涉及任何 Android 系统框架或真机。

本地测试的代码保存在 `src/test/java/` 下。

配置如下：

```
dependencies {
    ...
    testCompile (
            'junit:junit:4.12',
            'org.mockito:mockito-core:1.10.19'
    )
    ...
}
```

以上分别引入了 junit 和 mockito 两个测试用框架，前者是单元测试运行框架，后者是单元测试需要用到的模拟框架，更多信息可以参考[Mocking Android dependencies ](http://developer.android.com/training/testing/unit-testing/local-unit-tests.html#mocking-dependencies)

## Instrumented Unit Test

设备测试 [Instrumented tests](http://developer.android.com/training/testing/start/index.html#run-instrumented-tests)，可以在真机或模拟器上运行相关的测试代码，主要用来测试包含 Android 平台库的代码。

设备测试的代码保存在 `src/androidTest/java` 下。

要想运行设备测试，需要进行如下的gradle 配置：

```
dependencies {
    ...
    //!!这里是设备测试需要用到的注解，运行器和规则依赖库。
    androidTestCompile 'com.android.support:support-annotations:23.0.1'
    androidTestCompile 'com.android.support.test:runner:0.4.1'
    androidTestCompile 'com.android.support.test:rules:0.4.1'

    // Optional -- Hamcrest library
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'
    // Optional -- UI testing with Espresso
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
    // Optional -- UI testing with UI Automator
    //androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.1'
    ...
}
```

设备测试中还可以细分为两类测试，逻辑测试 和 UI 测试。逻辑测试是在 JUnit3/JUnit4 的基础上通过 Test runner 库来完成的；而UI测试是通过 Espresso 和 uiautomator 来提供支持的。

`InstrumentationTestRunner   AndroidJUnitRunner  MonitoringInstrumentation`

### Logic Unit Tests (逻辑测试)

### Automationg User Interface Tests (UI测试)

#### Espresso

```
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'

```

#### UIAutomator

### Testing App Component Integration (集成测试)
