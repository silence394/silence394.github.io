---
title: 第三方手游的CaptureFrame
categories: 工具
---
平时看到效果和效率比较惊艳的手游的时候，忍不住地想像GPA分析Windows下的游戏一样，学习一番。对安卓的应用可以通过SnapDragon达到目的。另外尝试了通过GPA运行手游模拟器，虽然没有成功，但是网上有运行成功的帖子，说明这个方法是可行的。苹果的app目前只能想到以通过GPA运行苹果模拟器来分析。
用SnapDragon分析安卓应用的操作如下：
1. 连接手机
手机开启开发者模式，用SnapDragon连接。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fwst714m4ij20b403x3ye.jpg)

<!-- more --> 

2. 截取Frame
点击“New Snapshot Capture”，并选择要截取的app，执行“Take snapshot”捕获帧。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fwst714oa0j20at0b9q38.jpg)
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fwst714xf0j209w07qmx8.jpg)
3. 分析Frame
捕获的frames在下方的DataExplore中，对感兴趣的进行分析即可。
![](http://ww1.sinaimg.cn/large/c5c3a364ly1fwst715520j20vy080jsh.jpg)
