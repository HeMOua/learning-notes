QtScrcpy来源
---
- 发现一款优秀的开源软件scrcpy
- 需要的技术（音视频编解码渲染，安卓）和自己技术栈比较匹配
- 正在用Qt做项目

所以用Qt重写scrcpy，巩固已有技术，练习Qt

和Scrcpy比较
----
关键点|Scrcpy|QtScrcpy
--|:--:|:--:
界面|sdl|qt
视频解码|ffmpeg|ffmpeg
视频渲染|sdl|opengl
跨平台基础设施|自己封装|Qt提供
编程语言|C|C++
编程方式|同步|异步
控制方式|单点触控|单点/多点触控
编译方式|meson+gradle|Qt Creator

- 使用Qt可以非常容易的定制自己的界面
- 基于Qt的信号槽机制的异步编程提高性能
- 方便新手学习
- 增加多点触控支持
- 实现了游戏（某某战场）映射

原理介绍
---
- 视频显示的流程
- 模拟点击的流程

QtScrcpy启动流程
1. 将scrcpy-server推送到手机（adb push）
2. 启动反向代理（adb reverse）
3. pc端监代理的端口
4. 启动scrcpy-server（adb shell app_process）
5. 接收scrcpy-server的连接，并进行通信

技术难点
---
1. 减少延时

    收到手机端的视频要立即解码显示，避免任何不必要的缓冲，这就需要我们自己来编写视频的解码和显示，已有的轮子无法满足我们的需求（VLC等）。

2. 如何通过pc在android上运行应用？

    我们需要一个应用运行在安卓上，这不可避免（采集安卓屏幕，模拟点击），我们不想通过adb install那么挫的方式，还要用户手动的点击允许安装，这对于一些有应用洁癖的用户不可容忍。有没有这么一种方式，当我们pc应用启动的时候，将我们的安卓应用push到安卓手机上，然后在后台运行，为我们的pc应用提供视频采集和模拟点击的功能，当我们的pc应用退出的时候，自动删除用户安卓手机上的安卓应用，这样对于用户来讲就像什么都没有发生一样。
    
    好消息是当然有，shell通过调用app_process，可以在Android上 执行Java代码，而adb又可以执行shell命令，所以adb shell app_process就可以满足我们的需求。
    
    涉及到android相关知识，这里不过多解释，感兴趣的可以通过搜索关键词如"adb shell app_process"来了解更多知识([Android上app_process启动java进程](https://blog.csdn.net/u010651541/article/details/53163542))。

3. 一些重要的功能需要调用android隐藏的api （[android @hide](https://stackoverflow.com/questions/31908205/what-exactly-does-androids-hide-annotation-do/31908373#31908373)），
    但隐藏的方法和类不能直接访问（并且它们可能因Android版本而异。）但是可以使用java反射来调用它们。

