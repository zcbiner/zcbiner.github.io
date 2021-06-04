---
title: Android接入aar问题汇总
date: 2021-04-27 10:15:47
tags: Android
---

以前觉得接入aar是一件非常简单的事情，然而当主工程庞大复杂，aar是外部提供且不能随意修改时，问题就变得很复杂了。本文记录接入aar时遇到的问题与解决思路。

<!--more-->

#### 1.aar接入方式

如果aar发布到了maven库，那么直接依赖maven库的地址就行。比如引入AndroidX的库：

```gr
implementation 'androidx.arch.core:core-runtime:2.1.0'
```

如果没有发布，可以考虑本地引入，将aar拷贝到libs目录下，然后：

```groovy
// 把aar拷贝到libs目录下
implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
```

或者这样定义：

```groovy
repositories {
	flatDir {
		dirs 'libs'
	}
}

dependencies {
	implementation(name:'aarname', ext:'aar')
}
```

#### 2. 库冲突

如果主工程和aar引用了相同的库，比如OKHttp，但是使用版本不一致，那么就是编译出错。解决方法就是使用相同版本的库。

#### 3. aar中库缺失

在我们的案例中，aar没有包含Kotlin协程库，然而aar里用到了协程，导致编译出错。错误日志：

```
AndroidRuntime: FATAL EXCEPTION: main Process: com.xxx.xxx, PID: 31163
java.lang.NoClassDefFoundError: Failed resolution of : Lkotlinx/coroutines/GlobalScope
```

解决方法：在主工程中引入协程相关库

#### 4.attr冲突

报错日志类似于这样：

```
error: duplicate value for resource 'attr/content' with config ''.
```

这说明是aar中定义在values中的attr属性值与主工程中有命名冲突。全局搜索：``name="content"``应该可以在主工程找到相关属性。可以修改主工程的属性值，避免冲突，或者推动aar方修改。

#### 5. aar so包问题

报错如下：

```
java.lang.UnsatisfiedLinkError: dlopen failed: library "xxx.so" not found
```

首先要检查这个so包是否包含在了aar中，如果没有说明aar打包就有问题，这个只能由aar方解决。如果有，那可能是abi指定问题。我们主工程使用的是armeabi，因此统一指定位armeabi。

在build文件中指定abi：

```groovy
android {
    defaultConfig {
        ndk {
            abiFilters 'armeabi'
        }
    }
}
```



#### 6. support包和AndroidX冲突

aar中使用的还是support包，主工程用的是AndroidX。

aar中继承的是``AppCompatActivity``，运行时报错：

```
java.lang.IllegalStateException: You need to use a Theme.AppCompat theme (or descendant) with this activity
```

Google后续只会支持AndroidX，support包已经不维护了。因此把aar中使用的support包都通过工具转换为AndroidX是一个不错的选择。参考文章：[jetifier](https://developer.android.google.cn/studio/command-line/jetifier?hl=zh_cn)。



#### 7.总结

当主工程相当庞大时，很简单的改动都会变得复杂。主工程的编译时间太久了，这是一个很头痛的事情，每次改动编译都会浪费大量的时间。



如果对外提供aar，一定要保证aar的质量，完善使用文档，不然对接入方会造成很多麻烦。

