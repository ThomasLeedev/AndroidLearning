# 《Android 系统软件开发》

> 主要记录之前没有接触过的知识，接触过且记得的就不再记录了

## 1. 概述

### 1.1 Android 7 个子系统

##### RIL（Radio Interface Layer） 子系统

无线电接口系统，用于短信、电话、数据通信等。



##### Input 子系统

处理所有来自用户的输入数据，如触摸屏、声音控制、物理按键等。



##### GUI 子系统

图形界面，功能包括绘制 2D 图形、通过 OpenGL 库处理 3D 游戏、通过 SurfaceFlinger 来重叠几个图形界面。



##### Audio 子系统

音频处理。



##### Media 子系统

Android 系统中最庞大的子系统，与硬件编解码、OpenCore 多媒体框架、Android 多媒体框架等相关，如音视频播放器、Camera 摄像预览等。



##### Connectivity 子系统

处理以太网、WIFI、蓝牙连接、GPS 定位连接、NFC 等。



##### Sensor 子系统

传感器子系统，gyroscope（陀螺仪）、accelerometer（加速度计）、proximity（距离感应器）、magnetic（磁力感应器）等。



### :)

- 应用框架层中，PackageManager 主要用来管理程序安装包的安装、更新、删除等操作。



## 2. Android 源码开发环境搭建

> 书中使用的虚拟机 + Ubuntu 系统，我使用的是服务器的 CentOs 系统

### 2.1 搭建 Linux 编译环境

##### 需要安装的程序包

Python、GNU Make、JDK、Git。



##### 安装必备的工具包



### :)

- Android 版本的升级对我们学习 Android 底层没有很大的影响



---

> 由于书籍出版时间在 2015 年，这章内容主要是环境的搭建和源码的编译过程，所以暂时跳过，待之后在网络上查找新的资料进行实践。



## 3. Android 系统的启动

### 3.1 Android init 进程启动

##### 启动过程

Linux 被 bootloader 加载进内存，然后开始运行。

初始化 Linux 运行环境，挂载 ramdisk.img 根文件系统映射。

运行其中的 init 程序（Android 系统的根目录下），这是 Linux 的第一个用户程序，pid 为 1。



init 进程对应的代码在 Android 源码目录中的 system/core/init/init.c 中。



##### init 进程主要完成四大功能

- 解析 init.rc 及 init.{hardware}.rc 初始化脚本文件，使用初始化脚本语言来完成 Android 系统关键进程和服务的启动
- 监听 Android 设备的组合按键
- 监听属性服务，类似于系统配置 config，例如设置 persist.service.adb.enable = 0 表示手机关闭 adb 调试桥
- 监听并处理子进程死亡事件



##### Android 初始化语言

Android 初始化语言包含四种类型的声明：Actions、Commands、Services、Options。



- Actions

  ```bash
  # Actions 是一系列 command 的集合，相当于一个执行脚本，trigger 是脚本执行的触发器
  on<trigger>
  <command 1>
  <command 2>
  <command 3>
  
  eg.
      on boot
          ifup lo
          ...
  
      on init
          ...
          
      on property:ro.secure = 0
      	start console
      	
      on property:persist.service.adb.enable=1
      	start adbd
      
      on property:persist.service.adb.enable=0
      	stop adbd
  ```

  

- Triggers

  匹配特定事件类型的字符串。

  - 命名触发器

    如 boot，这是 init 执行后的第一个被触发的 Triggers。

  - 属性触发器

    <key> = <value>，当 <key> 被设置为指定的 <value> 是被触发。

  - 设备变化触发器

    device-added-<path>、device-removed-<path>，会在一个设备结点文件被增删时触发。

  - 服务退出触发器

    service-exited-<name>，在一个特定的服务退出时触发。



- Commands

  由很多的命令组成，具体参考 P202



- Services

  Services 是一个程序，一个本地守护进程，被 init 进程启动，在退出时可选择让其重启。

  服务的形式：

  ```
  service <name><pathname> [<argument>]*
  <option>
  <option>
  ...
  
  name: 服务名
  pathname: 当前服务对应的程序位置
  option: 当前服务设置的选项
  ```



- Options

  参考 P204

  

##### init.rc 中 Actions 的执行流程

参考 P207



### 3.2 Android 本地守护进程

init.rc 中最后一个 Action boot 的最后一个 Command 为 class_start default，用来启动所有 class 为 default 的 Service，而在 init.rc 里定义的 Service 的 class 类别都没有定义，使用默认设置，所以所有的 Service 都会被启动。P209 列出了 Android 中主要的 Service。



下面是一些重要的 Service。



##### ueventd 进程

Android 使用类似 Linux 系统中的 udev 方式来实现对设备的管理，实现这一功能的进程称为 ueventd 守护进程，其源码为 system/core/init/devices.c。



Android 中，init 进程通过两种方式创建设备结点文件（我的理解是它是设备的标志，用户通过结点文件访问设备，驱动程序通过结点文件给用户提供操作的方法）。

1.ColdPlug（冷插拔）

init 进程被启动时，统一创建设备结点文件。

2.HotPlug（热插拔）

当有设备插入 USB 端口时，init 进程收到事件，动态创建设备结点文件。

插入设备，内核加载相关的驱动程序，设备驱动通过 device_create 将设备信息写入到 sysfs 文件系统中并向内核发出 uevent 事件。ueventd 守护进程用来接收 uevent 事件，然后读取 sysfs 里的设备信息，自动创建设备结点。



设备结点信息保存在 Android 根文件系统中的 /ueventd.rc 中。



##### adbd 进程

它的实现在 system/core/adb 目录下，和开发电脑端的 adb 通过 Socket 进行通信。



##### servicemanager 进程

服务管理器，用于管理服务的注册、查找、检查等操作。这里的服务指 Android 应用框架层的相关服务，例如 ActivityManagerService、PowerManagerService 等。



##### vold（volume daemon） 进程

负责对外部存储设备的操作，和框架层的 MountService 紧密联系。

vold 接收到 linux 内核的 ueventd 消息，然后转发给 MountService，而后 MountService 将具体对外部设备的操作发送给 vold，让 vold 做出最终的挂载、卸载、格式化等处理。



##### ril-daemon（Radio Interface Layer） 进程

无线设备的抽象层，实现通信功能必须使用的 Modem 硬件，而硬件之间存在差异，代码也会存在差异。

rild 和框架层通过 socket 进行通信，具体的操作（打电话、信息等）交给 rild 来实现，rild 通过 ril 将上层操作指令转换成 Modem 可识别的语言。



##### surfaceflinger 进程

Android 系统中任何在屏幕中显示的界面都要经过 SurfaceFlinger 的整合，是显示核心处理单元。

Android 中，二维、3D 图像都要在一个 Surface 上绘制，像一个画布，而有时屏幕同时显示两个应用程序的界面，所以屏幕上显示的界面要经过一个“整合器”，整合到一个屏幕上显示出来，这个整合器叫做 SurfaceFlinger，最终通过 FrameBuffer 显示出来。



### 3.3 Zygote 守护进程与 system_server 进程

[jvm 和 dvm 的区别]( https://www.cnblogs.com/haihai88/p/7656495.html )

Zygote 守护进程的启动是 Android 运行环境启动的开始阶段，Zygote 进程通过 Linux 系统特有的 Fork 机制分裂克隆出完全相同的运行环境，所有的 Android 应用程序都是 Zygote 进程的子进程。

system_server 进程作为 Zygote 进程的嫡长子进程，



##### Zygote 守护进程的启动

在 init 进程中，启动了 Zygote 服务，对应的程序为 /system/bin/app_process，服务名为 zygote，参数列表为：`-Xzygote/system/bin --zygote --start-system-server` ，**P229 还需要好好看看**。

> AndroidRuntime::start 方法实现了下面的功能：
>
> （1）通过 startVM 启动 dvm，并注册了一些本地 JNI 函数。
>
> （2）它的参数，在 JNI 代码中运行了第一个 Java 程序 ZygoteInit，将其作为 DVM 的主线程，同时给它传递两个在 JNI 中构建的参数：“com/android/internal/os/ZygoteInit" 和 "true"
>
> Zygote 进程由 init 进程作为 Service 启动，在 Zygote 进程里通过 startVM 启动了 VM 执行环境，通过 JNI 代码在 VM 环境中运行 ZygoteInit.java，作为 VM 中的主线程。



##### ZygoteInit 类的功能与 system_server 进程的创建

Zygote 将 DVM 运行环境准备好了，并开始调用执行 ZygoteInit.java 代码。

（1）在 Zygote 进程中启动 DVM，DVM 中通过 JNI 调用 ZygoteInit.main()，启动 DVM 第一个主线程

（2）在 ZygoteInit.java 中绑定了 ZygoteSocket 并预加载类和资源。

（3）通过 Zygote.forkSystemServer() 本地方法，在 Linux 中分裂出 Zygote 的第一个子进程，取名为 system_server，该进程中的代码和数据与父进程 Zygote 完全一样。

（4）在 Zygote 进程中通过 RuntimeInit.zygoteInit()，调用 SystemServer。main()，从而在 system_server 中运行 SystemServer.java 代码。

> 在 Zygote 进程创建时，init.rc 创建了 /dev/socket/zygote 文件，它在 ZygoteInit 中被绑定为服务器端，用来接收克隆 Zygote 进程的请求。
>
> ① Zygote 进程绑定并通过 select 监听 ZygoteSocket(/dev/socket/zygote)
>
> ② 其他进程发出 socket 连接请求，用于创建新应用程序
>
> ③ Zygote 和请求发起者建立用于通信的新连接，接收待创建应用程序的信息
>
> ④ Zygote 进程根据客户端数据请求新的子进程

在 ZygoteInit 中预加载的类是指大量的 Android 框架层代码，这些类有数千个，预加载资源是指一些系统的图片、图标、字符串资源。在 fork 进程时，由于将类和资源提前加载到父进程中，提高了应用程序的启动速度。



##### system_server 进程的运行

> system_server 进程加载了 libandroid_servers.so 本地库，Java 通过 System.load 一个 so 时，会自动调用该动态库中的 JNI_Onload 方法，通常在 JNI_Onload 方法中注册本地函数与 Java 方法映射关系。
>
> # 待完善 ...



##### HOME 桌面的启动

当 Android 系统服务启动完毕，system_service 进程会通知 Android 系统服务系统启动完毕，在 ActivityManagerService.systemReady 方法里会启动 Android 系统桌面应用程序：launcher。



### 3.4 实践：开机启动应用程序

> # 待完善 ...



## 4. Android 编译系统与定制 Android 平台系统



## 8. Android HAL 硬件抽象层

为了支持更多的设备类型，提出了 HAL 的概念。

Android 系统的功能不依赖于某一个具体的硬件驱动，而是依赖于 HAL 代码，相当于将 Linux 的设备驱动分为两部分，一部分在 Kernel 中使用 GPL 协议开源一些非常核心的接口访问代码，另外一部分在 Kernel 上的应用层使用 Apache 协议。



### 8.1 HAL 的架构形式

Android HAL 有两种架构形式：Module 架构和 Stub 代理架构。

Module 架构：08 年以前的旧架构，HAL 的代码被编译生成动态模板库，Android 应用程序和框架层使用 JNI 并调用 HAL Module 库代码，在 HAL Module 库中再去访问设备驱动。

Stub：存根或桩的意思，指一个对象代表。Stub 架构也要加载 module 库，但是这个 module 不包含操作底层硬件驱动的功能，只保存了底层 Stub 提供的操作接口。Stub 第一次被使用时加载到内存，后续再使用时仅返回硬件对象操作接口，不会存在设备多次打开问题。