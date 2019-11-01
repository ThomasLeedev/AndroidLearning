# Android 系统原理与开发要点

## 1. Android 系统概述

### 1.1 两种形式的 Android 开发

##### a. Android 的 SDK 开发

传统的应用开发，通过 Android 提供的 SDK 进行开发。

需要掌握的知识结构：

- JAVA 语言
- 应用程序架构
- GUI 设计基础知识
- 各种视图的使用
- 2D/3D 图形 API
- 应用程序的设计思想



##### b. Android 的完全源代码开发

在 Android 的源代码基础上进行开发，可以开发应用程序，进行系统移植，或开发系统本身。

需要掌握的知识结构：

- Linux 操作系统知识
- Linux 内核知识（C 语言）
- Linux 驱动程序知识（C 语言）
- Android 底层库（C 语言，C++）
- Dalvik 虚拟机（C++，JAVA）
- Android GUI 系统（C，C++，JAVA）
- 音频，视频和多媒体（C，C++，JAVA）
- 电话部分（C，C++，JAVA）
- 连接部分（C，C++，JAVA）
- 传感器部分（C，C++，JAVA）



## 2. Android 系统的开发综述

### 2.1 Android 系统的架构

分四层，从底层到高层分别是：

- Linux 操作系统及驱动（C）

  提供核心服务，包括进程管理、内存管理、网络模型、安全性、驱动模型等，同时作为硬件和软件栈之间的抽象层。

  

- 本地框架和 Java 运行环境（C、C++）

  通过第 3 层为开发者提供服务，包括 C 的基础库、媒体库（常用音频、视频的回放和录制）、显示管理、Web 引擎、2D 图形引擎、3D 加速、位图和矢量字体显示。

  每一个 Android 应用程序都有自己的进程在运行，都拥有一个独立的 Dalvik 虚拟机用来执行 .dex 文件，.dex 针对小内存使用做了优化，Java 类经过编译后再通过 SDK 中的 “dx” 工具转化成 .dex 文件，再交给 Dalvik 虚拟机。

  

- Java 应用程序框架（JAVA）

  Views、ContentProviders、ResourceManager、NotificationManager、ActivityManager

  

- Java 应用程序（JAVA）

  

第 1 层运行于内核空间，2、3、4 层运行于用户空间，2、3 之间是本地代码层和 Java 代码层的接口（JNI），3、4 层之间是 Android 系统 API 的接口。

对于应用开发，第 3 层次以下是不可见的，**即我们平时看到的 Android 源码是第三层？**



### 2.2 Android 源代码的开发环境

##### a. 开发环境

- Git
- Repo
- JDK
- 代码编译工具（**Make？**）



##### b. 源码获取

通过 repo 和 git 获取源码：

```bash
# 初始化仓库（如果要初始化指定版本，可以在末尾加上 -b version_name）
$ repo init -u git://android.git.kernel.org/platform/manifest.git
# 获取代码
$ repo sync
# 还有部分命令等使用过以后再做记录，目前尚不了解
```

`repo init` 之后，将生成隐藏目录 .repo，其中包含 `manifest.xml` 文件，其中的代码类似：

```bash
# path 是基于当前路径下的保存路径，name 是工程的名称
<project path="dalvik" name="platform/dalvik" />
<project path="development" name="platform/development" />
<project path="frameworks/base" name="platform/frameworks/base" />
```



##### c. 源代码结构

代码分成三个部分：

- 核心工程（Core Project），系统的基础，在根目录的各个文件夹中，libc、虚拟机等。
- 扩展工程（External Project），使用其他开源项目扩展的功能，在 external 文件夹中。
- 包（Package），提供应用程序和服务，在 package 文件夹中，例如启动器、同步应用、闹钟应用、DownloadProvider 等。



##### d. 源代码的编译

根目录下执行 `make` 命令，通过递归找到各个目录中的 Android.mk 文件进行编译。

编译完成后，结果在根目录的 `out` 目录中。

编译的结果：

- 主机工具
- 目标机程序
- 目标机映像文件
- 目标机 Linux 内核（需要单独处理）

**这些都是啥？？？**



## 3. Android 的 Linux 内核与驱动程序

### 3.1 Linux 核心与驱动

...



## 4. Android 的底层库和程序

 android 启动后，系统执行的第一个进程是一个名为 init 的可执行程序，提供了以下的功能：

- 设备管理
- 解析启动脚本
- 执行基本的功能
- 启动各种服务，包括 serviceManager、zygote、mediaService



## 5. Android 的 JAVA 虚拟机和 JAVA 环境

