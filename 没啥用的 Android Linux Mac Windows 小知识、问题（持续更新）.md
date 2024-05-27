---
title: 没啥用的 Android Linux Mac Windows 小知识、问题（持续更新）
date: 2023-04-23 22:30:58
tags:
- Android
- Android Studio
- Gradle
- OpenGL ES
- MacOS
- Mac
- Linux
- Cpp
- C
- Windows
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---

#### 1. AS apk 安装包安装位置

2022.3.1版本

`/app/build/intermediates/`

`/app/build/outputs/apk/debug/app-debug.apk`

#### 2. AS R 文件位置

2022.3.1版本

![R.png](没啥用的 Android Linux Mac Windows 小知识/R.png)

`/app/build/intermediates/runtime_symbol_list/debug/R.txt`

#### 3. AS OpenGL ES 相关库文件在什么位置

2022.3.1版本

在 `ndk` 目录里面，`MacOS` 在 `/Users/yourUserName/Library/Android/sdk/ndk/`

#### 4. Android 开发 OpenGL ES 找不到 GL_TEXTURE_BORDER_COLOR

在 `GLES3/gl3.h` 里面没有，在 `GLES3/gl32.h`

#### 5. AS 写 C++ 的时候，一直提示当前文件不属于任何项目

2022.3.1版本

如果你确定你已经将文件添加到 `CMakeList.txt` 里面，但依旧没作用

那大概率是下面的问题

`This file is not part of the project. Please include it in the appropriate build file (build.gradle, CMakeLists.txt or Android.mk etc.) and sync the project.`

这是因为 `CMake` 没有重新加载，你可以 `Sync Project with Gradle Files`，就是这个图标

![Sync Project with Gradle Files.png](没啥用的 Android Linux Mac Windows 小知识/Sync Project with Gradle Files.png)

如果没有效果，那大概是直接用了缓存，没有重新加载

你可以在 `CMakeList.txt` 里写几行 `message`，直到右上角出现 `Sync Project`

`External build files have changed since last project sync. A project sync may be necessary for the IDE to work properly.`

#### 6. AS C++ 没有代码提示，并且代码高亮多处爆红

2022.3.1版本

这是因为 `Android Studio` 在你没有 `#include` `.h` 文件 或者没有调用某个 `C++` 函数的时候，是不给你进行代码检查的

我还专门跟 `Google` 发了邮件，得到了如上回复

#### 7. MacOS 删除 .DS_Store

```shell
find /Users/username/dir/ -name ".DS_Store" -exec rm {} \;
```

`/Users/username/dir/` 这是你想删除 `.DS_Store` 的文件夹

#### 8. jni 中，类型映射

写这个的主要原因是因为 `Kotlin` 类型需要对应到 `Java` 包名才可以

基本类型（Primitive Type）

| Java 类型 | Kotlin 类型 | Native 类型 | 描述                             | 符号 |
| --------- | ----------- | ----------- | -------------------------------- | ---- |
| int       | Int         | jint        | 有符号32位整型（int）            | I    |
| short     | Short       | jshort      | 有符号16位整型（short）          | S    |
| long      | Long        | jlong       | 有符号64位整型（long）           | J    |
| float     | Float       | jfloat      | 32位浮点型（float）              | F    |
| double    | Double      | jdouble     | 64位浮点型（double）             | D    |
| boolean   | Boolean     | Jboolean    | 无符号8位整型（unsigned char）   | Z    |
| char      | Char        | Jchar       | 无符号16位整型（unsigned short） | C    |
| byte      | Byte        | jbyte       | 有符号8位整型（char）            | B    |

引用类型（Reference Type）

| Java 类型 | Kotlin 类型  | Native 类型   | 描述        | 符号                |
| --------- | ------------ | ------------- | ----------- | ------------------- |
| Class     | Class        | jclass        | Class类对象 | Ljava.lang.Class;   |
| Object    | Any          | jobject       | -           | Ljava.lang.Object;  |
| String    | String       | jstring       | -           | Ljava/lang/String;  |
| Object[]  | AnyArray     | jobjectArray  | -           | [Ljava.lang.Object; |
| boolean[] | BooleanArray | jbooleanArray | -           | [Z                  |
| byte[]    | ByteArray    | jbyteArray    | -           | [B                  |
| char[]    | CharArray    | jcharArray    | -           | [C                  |
| short[]   | ShortArray   | jshortArray   | -           | [S                  |
| int[]     | IntArray     | jintArray     | -           | [I                  |
| long[]    | LongArray    | jlongArray    | -           | [J                  |
| float[]   | FloatArray   | jfloatArray   | -           | [F                  |
| double[]  | DoubleArray  | jdoubleArray  | -           | [D                  |
| Void      | Unit         | Void          | -           | V                   |

除了基本数据类型，以及 `void` ，基本都是 `L+包名+;` ，数组再在开头加一个 `[` 

#### 9. Windows 的狗屎快捷键！！

总有一些“才智冠绝”、“谦虚谨慎”的人，提出让大众“赞不绝口”的设计需求

使用频率低，还无法更改的快捷键就是一种（他甚至连关闭都不让！！！）

ctrl + 空格切换中英文、win + → 改窗口位置......

这些无法更改的快捷键对于一个使用大量快捷键的程序猿简直就是噩梦
