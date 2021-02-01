## ninja简介
早期的Android系统都是采用Android.mk的配置来编译源码，从Android 7.0开始引入ninja。ninja是一个编译框架，会根据相应的ninja格式的配置文件进行编译，使用ninja的主要目的是用来加快Android源码的编译速度

## Android中的ninja程序
```java
prebuilts/build-tools/linux-x86/bin/ninja
```

## Android中编译生成的ninja文件
```java
out/soong/build.ninja
out/soong/.bootstrap/build.ninja
out/soong/.bootstrap/build-globs.ninja
out/soong/.minibootstrap/build.ninja
out/soong/.minibootstrap/build-globs.ninja
out/combined-<product_name>.ninja
out/build-87939e401eb63e7a3ee5d558a8d326e2.ninja
out/build-<product_name>-package.ninja
out/build-87939e401eb63e7a3ee5d558a8d326e2-cleanspec.ninja
```

## 转换关系
```
通过Kati将Android.mk转换成ninja格式的文件
通过Blueprint + Soong将Android.bp转换成ninja格式的文件
通过androidmk将Android.mk转换成Android.bp，但针对没有分支、循环等流程控制的Android.mk才有效。
```

转换关系图如下：<br><br>
![](assets/markdown-img-paste-2021013021410063.png)


```java
Android.bp
-------------------------------------
cc_library_shared {
    name: "libinbp",
}

cc_binary {
    name: "binaryinbp",
    srcs: ["main.cpp"],
    shared_libs: [
        "libinbp",
        "libinmake"       // This is not possible, building gives error: binaryinbp" depends on undefined module "libinmake"
    ]
}

Android.mk
-------------------------------------
include $(CLEAR_VARS)
LOCAL_MODULE := libinmake
include $(BUILD_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_SRC_FILES := main.cpp
LOCAL_MODULE := binaryinmake
LOCAL_SHARED_LIBRARIES := \
    libinmake \
    libinbp
include $(BUILD_EXECUTABLE)
```


## 常见用法

```java
// 在增量编译方面，ninja比make单独编译某个模块快很多，常见的模块有apk jar包等
prebuilts/build-tools/linux-x86/bin/ninja -f out/build-<product_name>.ninja ${model_name}
// example 1
prebuilts/build-tools/linux-x86/bin/ninja -f out/build-<product_name>.ninja services
// example 2
prebuilts/build-tools/linux-x86/bin/ninja -f out/build-<product_name>.ninja Launcher3

// 链接文件方便编译
ln -s prebuilts/build-tools/linux-x86/bin/ninja ninja
ln -s out/combined-aosp_arm64.ninja build.ninja
ninja -f build.ninja ${model_name}
```

## google Soong介绍
https://source.android.google.cn/setup/build
