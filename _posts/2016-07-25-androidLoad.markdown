---
layout:     post
date: 		2016-07-25
title: 		"Android启动流程"
subtitle:   "Android入门笔记整理"
author:     "Halo"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - Linux
	- OS
---

##Android启动分为三个阶段：bootloader引导，linux kernel启动，android启动。
    Bootloader启动阶段: bootloader是在操作系统运行之前运行的一段程序，可以初始化硬件设备、建立内存空间映射图，从而将系统的软硬件环境带到一个合适状态，以便为最终调用操作系统内核准备好正确的环境。当你开机时，机器首先要启动，CPU最先执行的一段程序就是BootLoader，这和电脑上的BIOS是一个玩意儿。Bootloader下可以进入recovery和fastboot模式。
    
    linux kernel启动阶段：设置缓存、被保护存储器、计划列表、加载硬件的驱动。当内核完成系统设置，它首先在系统文件中寻找”init”文件，然后启动root进程或者系统的第一个进程。
    
    启动Init进程：源码位于system/core/init目录，在进程中执行了：文件夹建立，挂载，rc文件解析，属性设置，启动服务，执行动作，socket监听……init.rc文件可以在/system/core/rootdir/init.rc找到，readme.tx可以在/system/core/init/readme.txt找到。对于init.rc文件，Android中有特定的格式以及规则，Android初始化语言由四大类型的声明组成，即Actions（动作）、Commands（命令）、Services（服务）、以及Options（选项）
    
    启动Zygote进程：Android系统创造了”Zygote”。Zygote让Dalvik虚拟机共享代码、低内存占用以及最小的启动时间成为可能。Zygote是一个虚拟器进程，正如我们在前一个步骤所说的在系统引导的时候启动。Zygote预加载以及初始化核心库类。
    
    启动Runtime进程: 在Zygote进程启动完成后，Init进程会启动Runtime进程。Runtime进程首先初始化服务管理器（Service Manager），并把它注册为绑定服务（Binder Service）的默认上下文管理器，负责绑定服务的注册与查找。然后Runtime进程会向Zygote进程发送启动系统服务（System Service）的请求，Zygote进程在收到请求后，会“孵化”出一个新的Davlik VM实例并启动系统服务进程。
    
    启动本地服务: System Service会首先启动两个本地服务（由C或C++编写的Native服务）：Surface Flinger和Audio Flinger，这两个本地服务向服务管理器注册成为IPC服务对象，以便在需要它们的时候很容易查找到。然后System Service会启动一些Android系统管理服务，包括硬件服务和系统框架核心平台服务，并注册它们成为IPC服务对象。

    启动Home Launcher：当System Service加载完所有的系统服务后就意味着系统准备好了，它会向所有服务发送一个系统准备完毕（systemready）广播。当Activity Manager Service （由System Service启动）接收到systemready广播后，会向Zygote进程发送创建Dalvik VM实例的请求，Zygote进程会负责生成一个新的Dalvik虚拟机实例，然后ActivityManagerService在系统中查找具有<category android:name="android.intent.category.HOME"/>属性的Activity，并启动它。