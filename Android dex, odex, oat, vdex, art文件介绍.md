Android dex, odex, oat, vdex, art文件介绍

下图可以简化将Java源转换为OAT的过程：
![](assets/markdown-img-paste-20210202155720870.png)

## .dex
```
Dalvik虚拟机字节码文件
```

## .vdex
```
其中包含APK的未压缩DEX代码，以及一些旨在加快验证速度的元数据
```
## .odex .oat
```java
// JIT，Just-in-time，即时编译，边运行边编译；
// AOT，Ahead Of Time，提前编译，指运行前编译。
其中包含APK中已经过AOT编译的方法代码。
原始DEX文件（例如classes.dex）被转换为另一个包含本机代码的文件。这个新文件通常具有.odex。.oat扩展名，并由ELF格式包装。
```

## .art
```
其中包含APK中列出的某些字符串和类的ART内部表示，用于加快应用启动速度。
```

## apk生成
```
./priv-app/MediaProvider/MediaProvider.apk
./priv-app/MediaProvider/oat/arm64/MediaProvider.vdex
./priv-app/MediaProvider/oat/arm64/MediaProvider.odex
```

## jar包生成
```
./framework/services.jar
./framework/oat/arm64/services.odex
./framework/oat/arm64/services.vdex
./framework/oat/arm64/services.art
```
## bootjar生成
```
./framework/boot.vdex
./framework/arm/boot.vdex
./framework/arm/boot.oat
./framework/arm/boot.art
./framework/arm64/boot.vdex
./framework/arm64/boot.oat
./framework/arm64/boot.art
```


https://lief.quarkslab.com/doc/latest/tutorials/10_android_formats.html
https://source.android.com/devices/tech/dalvik/configure
