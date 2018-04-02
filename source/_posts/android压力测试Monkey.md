---
title: android压力测试Monkey
date: 2018-04-02 15:16:25
tags: 
     - 压力测试
     - Monkey
     - android
categories: 
     - android
     
---
　　顾名思义，Monkey就是猴子，Monkey测试，就像一只猴子， 在电脑面前，乱敲键盘在测试。  猴子什么都不懂， 只知道乱敲

通过Monkey程序模拟用户触摸屏幕、滑动Trackball、 按键等操作来对设备上的程序进行压力测试，检测程序多久的时间会发生异常。

<!-- more -->
　

话不多说，直接写步骤：

　　首先，电脑上需要配置有sdk环境变量和python环境变量
自行百度环境变量配置方法

　　python 下载地址：
https://www.python.org

　　在手机上装上要测得应用
打开开发者选项中的调试模式

　　链接电脑

　　cmd打开命令行窗口

        输入  python查看python环境变量是否配置成功
        输入 adb devices 查看当前链接的设备
        

![python环境配置成功](http://upload-images.jianshu.io/upload_images/2616527-085449e46823a2e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![成功链接手机](http://upload-images.jianshu.io/upload_images/2616527-cb6f8b8b5bf90e3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　所有指令都有正确回应的时候说明环境变量配置成功，就可以接下来的压力测试了

    在命令行窗口输入

    adb shell monkey 1000
    这个就表示模拟手机1000次随机操作：
    比如说 滑动屏幕，随机点击屏幕某个坐标，音量键，截图 ====

    但是如果按照这个操作一遍就会发现，没有指定某个应用啊！
    对的。。。
    确实没有。。。

　　接下来就指定某个应用

    首先我们要获取手机上的应用的包名

    还是命令行输入
    adb logcat | grep START
    意思就是说，将还有START标签的应用通过logcat打印出来

    比如说我要测试一个手机自带的闹钟程序：
    点开手机上的闹钟app
    输入    adb logcat | grep START

　　下图为窗口打印的log日志
    
   ![](http://upload-images.jianshu.io/upload_images/2616527-9779588fce3809c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　你现在手机的显示界面就是闹钟app的首页，
其中cmp后面跟着的就是闹钟这个应用的包名
从后面也可以看出来我用的是魅族手机。


　　获取到了想测试的应用的包名之后就可以给指定应用做压力测试了：

    命令行输入
    adb shell monkey -p 应用包名 测试的事件数

　　我这里就输入
adb shell monkey -p com.android.alarmclock 1000

![压力测试结果](http://upload-images.jianshu.io/upload_images/2616527-a318e66d1a48deeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　上图为压力测试结果，可以看到，我的测试事件是1000个，injected显示的结果也是执行了1000次测试，说明，每一次都通过了没有问题


再来一个出问题的情况

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2616527-978551b6309b617b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　可以看到，如果说出了问题，也会打出来具体的错误log，比较方便


　　如果说你想在每个事件中间加点儿间隔时间的话可以用这个指令

    adb shell monkey --throttle<毫秒数>

　　monkey压力测试还有一个非常有用的功能

　　因为，每次压力测试，测试的所有事件都是随机产生的，如果遇到问题的话，怎么能让问题重现呢？

　　这时候就用到了

    adb shell monkey -p 应用包名 -s 自定义数字 事件数

　　这句代码的意思就是说把（事件数）个随机事件装进一个队列中，这个队列的编号就是你自己定义的那个数字编号，如果出了问题你想重现的话，指定同样编号的随机事件队列就可以了

　　举例：

    第一次压力测试的时候
    adb shell monkey -p 应用包名 -s 120 1000
    执行编号为120的1000次随机事件测试

    这时候你发现出了问题了，想重现
    同样再执行一遍这组随机事件就行了
    adb shell monkey -p 应用包名 -s 120 1000


　　如果你不想进行什么其他的没用的操作，比如说截屏，音量大小，只是想测试触摸点击事件的话，也可以做到

    adb shell monkey --pct-touch 事件所占百分比

　　还是以闹钟为例：

    adb shell monkey -p com.android.alarmclock --pct-touch 100 1000

    表示100%执行触摸点击测试1000次随机事件

    你也可以将测试的事件打印出来
    adb shell monkey -v -p com.android.alarmclock --pct-touch 100 1000


　　-v表示将测试的事件打印出来

　　在最顶端可以看到事件的百分比
 

![事件百分比](http://upload-images.jianshu.io/upload_images/2616527-d804e95f7c752045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




　　事件也都为touch点击事件

![action事件](http://upload-images.jianshu.io/upload_images/2616527-0d8237099542e765.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



　　如果在测试期间，应用crash了，测试默认会停止
可以使用 

    --ignore-crashes
　　它会使测试在遇到crash的情况是，不自动停止，而是继续执行完指定数目的测试，不过还是会照常打印log

　　同样，如果应用中主线程执行了耗时操作是，会遇到ANR异常，monkey也会默认终止测试，这是可以用

    --ignore-timeouts

　　来打印错误的同时，继续执行完测试


　　写到这里发现了阿标的博客：关于制定测试事件写的比较详细，直接copy过来，嘿嘿嘿~~~~

    参数：  --pct-｛+事件类别｝｛+事件类别百分比｝

    用于指定每种类别事件的数目百分比（在Monkey事件序列中，该类事件数目占总事件数目的百分比）
    

    --pct-touch ｛+百分比｝

    调整触摸事件的百分比(触摸事件是一个down-up事件，它发生在屏幕上的某单一位置)

    --pct-motion ｛+百分比｝

    调整动作事件的百分比(动作事件由屏幕上某处的一个down事件、一系列的伪随机事件和一个up事件组成)adb shell monkey -p

    --pct-trackball ｛+百分比｝

    调整轨迹事件的百分比(轨迹事件由一个或几个随机的移动组成，有时还伴随有点击)

    --pct-nav ｛+百分比｝

    调整“基本”导航事件的百分比(导航事件由来自方向输入设备的up/down/left/right组成)

    --pct-majornav ｛+百分比｝

    调整“主要”导航事件的百分比(这些导航事件通常引发图形界面中的动作，如：5-way键盘的中间按键、回退按键、菜单按键)

    --pct-syskeys ｛+百分比｝

    调整“系统”按键事件的百分比(这些按键通常被保留，由系统使用，如Home、Back、Start Call、End Call及音量控制键)

    --pct-appswitch ｛+百分比｝

    调整启动Activity的百分比。在随机间隔里，Monkey将执行一个startActivity()调用，作为最大程度覆盖包中全部Activity的一种方法

    --pct-anyevent ｛+百分比｝

    调整其它类型事件的百分比。它包罗了所有其它类型的事件，如：按键、其它不常用的设备按钮、等等

    --pct -anyevent 100 1000* 指定多个类型事件的百分比：

  　　注意：各事件类型的百分比总数不能超过100%；

　　重复指定操作测试利用monkey scipt脚本。。。。后面在写吧。。。。╮(╯▽╰)╭