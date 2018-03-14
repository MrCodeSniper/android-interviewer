# 2018android
1.请写出ArrayList,LinkedList,HashMap之间的区别和联系
本题侧重与对android集合框架的认识程度 这里进行解析

java集合框架Collection
collection是集合框架的根，定义了集合操作的通用行为
继他之后存在四个子接口 list,map,set,queue

list是有序collection接口
实现：
arraylist是基于数组实现list接口的 线程不同步  
vector也是实现list的接口 他的特点在与线程同步 在函数中添加了synchronized实现 所以性能上 比arraylist低
因为基于数组实现 存取时间复杂度o(1) 增删时间复杂度o(n)
linkedlist是基于链表实现list接口的 故存取时间复杂度o(n) 增删时间复杂度o(1)

Set是不包含重复的元素的无序Collection
实现：
Hashset是基于hashmap的set集合 通过键值对存储 保证不含重复元素
treeset基于sortedmap实现是有序的

map是以键值对形式存在的collection集合 并保证键不重复
实现：
hashmap是继承abstractmap类实现map接口的 不同步元素可以为空
treemap是继承abstractmap类实现map接口的 通过数据结构树来实现键位有序排列 
hashtable是继承自dictionary类实现map接口的 同步且元素不能为空

queue队列 先进先出
stack栈 先进后出


![集合框架](https://upload-images.jianshu.io/upload_images/898312-a5d959e2a6d5b353.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/613)

———————————————————————————————————————

2.请概述下activity的生命周期和启动模式 与fragment生命周期有什么不同?
本题侧重与android组件的运作原理 是如何产生如何结束 启动的形式如何 如下分析

activity启动时通常会回调oncreat()函数 在这个函数中常做一些初始化的操作
之后执行onstart()函数 执行之后acitivity已经可见在后台准备好但是没有获取焦点无法与用户交互
之后执行onresume()函数 activity出现在前台并且获取到焦点可以与用户交互
当需要其他activity时 会执行onPause()函数 这个函数是个过渡态 用来保存一些 全局持久性数据  以防内存不足时杀死程序
activity从前台切换到后台时 执行onStop()函数
activity走向终点准备销毁时会执行 onDestory()函数  在此时做一些 进行一些回收工作和资源的释放工作 执行完之后会被gc回收

在onPause()之后若从其他activity返回原activity 会重新返回执行onResume()函数
在onStop()之后若从 后台返回前台 会执行onRestart()函数返回onStart()函数重新执行

activity启动模式
每启动一个activity会把此放到一个stack的顶端 启动模式是用来处理activity与栈的分配关系
启动模式有四种:standrad,singletop,singleinstance,singletask

standrad 每启动一个activity创建一个实例 哪个activity启动他就进那个activity所在的stack顶部
singletop 栈顶复用模式 当要启动的acitivity处于栈顶 不产生实例而是调用onNewIntent()函数
singletask 栈内复用模式 当要启动的acitivity处于栈内 不产生实例而是调用onNewIntent()函数
singleinstance 单实例模式  在栈内复用的基础上 一个activity占领一个栈  

———————————————————————————————————————

3.你能说出几种设计模式？能写出单例模式的几种方式？
这个题考验的是基本功-设计模式 设计模式

part1.单例模式-保证app中只有此一个实例
单例的实现方式：
1.懒汉式 在第一次使用时构建
2.饿汉式 在类被装载时构建

懒汉式具体代码
```
public class Singleton(){
    private static Singleton instance;
    private Singleton(){}//构造方法私有化
    public static Singleton getInstance(){
        if(instance==null){
          instance=new Singleton();
        }
        return instance;
    }
}
```
但是当多线程同时执行这个函数且实例都为空时会创建各自的实例 这样就不会是单例了 所以需要线程同步    
public static synchronized Singleton getInstance() 但是如此的话加了锁之后会引起其他线程等待锁释放大大影响效率——同步单锁懒汉式
所以又在加锁之前进行判断使得有非空实例的线程无需经过锁机制 增加了效率
但java编译器 在执行时会有指令重排序会发生，所以需对变量加volatile 以防其他线程读 当本线程写时 防止出错发生
```
//最终懒汉式 双重检查锁
public class Singleton(){
    private static volatile Singleton instance;
    private Singleton(){}//构造方法私有化
    public static Singleton getInstance(){
        if(instance==null){
          synchronized (Singleton.class) {
               if (instance == null) {
                    instance = new Singleton();
                }
           }
        }
        return instance;
    }
}
```
饿汉式具体代码

```
public class Singleton(){
    private static Singleton instance=new Singleton();//类装载时创建实例 缺点初始化的时机很难把控 适合占用资源少的实例
    private Singleton(){}
    public static Singleton getInstance(){
        return instance;
    }
}
```

其他实现方式

1.静态内部类
```
public static Singleton(){// 使得第一此使用时才类装载 两种方式得结合
     private static class SingleHolder(){
         private static final Singleton INSTANCE=new Singleton();
      }
       private Singleton(){}
       public static Singleton getInstance(){
        return SingleHolder.INSTANCE;
     }

}

```
2.枚举
```
public enum SingleInstance {// 使用SingleInstance.INSTANCE.fun1(); 枚举内部 线程安全且不会出现指令重排序 且也是两种方式结合 在需要继承得方式不适用
    INSTANCE;
    public void fun1() {
        // do something
    }
```

———————————————————————————————————————

4.android事件分发机制

事件在三大层中传递 activity - viewgroup -view
要传递的MotionEvent中包含了一系列用户的事件
处理事件有三个重要方法
onDispatchTouchEvent 分发事件 
onInterceptTouchEvent 拦截事件   只存在与viewgroup中 并在分发事件中被调用
onTouchEvent 触摸事件
三个函数返回值的意义
return true  消费事件 自己处理
return false 停止向下传递 开始往父类传递
return super 按默认流程传递

DecorView setContentView()的view的父view

首先需要了解默认情况下事件是如何传递的：
当一个事件产生时会传给当前activity 执行当前activity的分发事件函数 传递到DecorView底层view的ViewGroup
执行viewgroup的事件分发函数 若返回true则消费事件 若为false则传给activity处理触摸事件函数执行
继续执行会执行到 拦截事件函数 若返回true 即在viewgroup中处理事件函数执行 若返回false 继续往下执行
继续执行会传递给view的事件分发函数 返回true消费 返回false 传给viewgroup 处理事件函数执行
继续执行由view的事件函数执行 返回true消费结束 返回false or super 继续上传给viewgroup和activity触摸事件处理

![事件分发](https://upload-images.jianshu.io/upload_images/3985563-d7646a08adcc7df7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

—————————————————————————————————————————

5.ANR原理及源码解析

ANR- Activity Not Response 应用程序未响应

安卓中ActivityManageService，WindowManageService
会检测app响应时间 若app在一段时间内无法响应触摸或者键盘 或者事件未处理完 就会出现

满足以下任一条件均会产生anr： 

5秒内无法响应屏幕触摸事件或键盘输入事件

在执行广播接收者的onReceive()函数时10秒没有处理完成，后台为60秒。

前台服务20秒内，后台服务在200秒内没有执行完毕。

内容提供者的publish在10s内没进行完。

产生的原因：

1.主线程阻塞或主线程数据读取 （避免死锁 由自线程处理耗时操作）

2.CPU满负荷，I/O阻塞        （IO操作在子线程异步执行）

3.内存不足                   （优化内存占用）

4.各大组件ANR              （各大组件生命周期不要做耗时操作）

从源码角度分析：


service进程在绑定后会发送延时消息
//发送delay消息(SERVICE_TIMEOUT_MSG)
bumpServiceExecutingLocked(r, execInFg, "create");

```
private final void bumpServiceExecutingLocked(ServiceRecord r, boolean fg, String why) {
    ... 
    scheduleServiceTimeoutLocked(r.app);
}
```

```
void scheduleServiceTimeoutLocked(ProcessRecord proc) {
    if (proc.executingServices.size() == 0 || proc.thread == null) {
        return;
    }
    long now = SystemClock.uptimeMillis();
    Message msg = mAm.mHandler.obtainMessage(
            ActivityManagerService.SERVICE_TIMEOUT_MSG);
    msg.obj = proc;
    
    //当超时后仍没有remove该SERVICE_TIMEOUT_MSG消息，则执行service Timeout流程
    mAm.mHandler.sendMessageAtTime(msg,
        proc.execServicesFg ? (now+SERVICE_TIMEOUT) : (now+ SERVICE_BACKGROUND_TIMEOUT));
}
```

总结：
都是设置当前时间并发送一个固定时间的延时消息，然后进入目标线程创建或执行，若在这段延迟时间内不执行取消延时消息的函数，
最后都会执行ActivityManageService.appNotResponding() 产生anr

—————————————————————————————————————————

6.android动画机制与原理

android有三类动画

1.补间动画 
早在android3.0之前就存在 能进行 一些简单的动画 例如 旋转 缩放 平移等 但是只是改变在界面上的效果
实现 动画类来实现

2.帧动画
通过让一系列图片按顺序播放产生动画效果
实现 xml或代码

3.属性动画
通过控件属性值进行修改 不断地对值进行操作来实现的 达到动画的效果 不再是只针对view的动画

原理：

—————————————————————————————————————————

7.android编译打包过程

首先了解一下项目中 经常出现的却在脑子里不清楚的文件

classes.dex

android有属于自己的代码处理虚拟机 并针对手机的特性做了最佳的优化
android虚拟机dalvik支持的字节码格式 android上可执行的文件 由java.class编译而来

编译打包流程 

大概步骤：应用的模块（源码，资源文件，aidl文件） 依赖文件（依赖模块，库，依赖资源） 经过编译 生成dex文件 和编译过的资源 
并结合项目的测试或者发布的key 通过加密打包 生成apk文件(实质上为压缩文件)

具体步骤：

1.应用的资源文件通过aapt android assets package tool 打包为R.java类和编译好的资源文件（我们经常通过R.来找到编译好的资源文件）

2.aidl文件通过aidl生成java interface代码

3.R.java,java interface,source通过jvm 转化为.class文件

4. class文件和第三方的库经过dex编译器生成 .dex文件

5. dex文件和aapt编译好的资源通过apkbuilder 生成未签名的apk

6. 未签名的apk通过jarsinger 将apk与key 签名

7. 对签名后的apk进行对齐处理

—————————————————————————————————————————

8.jni ndk 

jni -java native interface 能实现java 和其他语言通信
Java 需要与 本地代码 进行交互 通常用jni实现

ndk-Native Development Kit android开发的一个工具包
快速开发C、 C++的动态库，并自动将so和应用一起打包成 APK
android中用ndk能帮助快速开发 实现jni功能

ndk实际应用场景:
优化密集运算和消耗资源较大模块的性能，如音视频解码，图像操作等
需要提高安全性的地方，编译成 so 库不容易被反编译；如文件加密、核心算法模块等；

C/C++代码被编译成库文件之后，才能执行，库文件分为动态库和静态库两种：
动态库 在程序运行期载入库
静态库 程序编译期就被载入


————————————————————————————————————

9.String,StringBuffer与StringBuilder的区别?

String 字符串常量，是不可变的对象, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String

StringBuffer 字符串变量,当对此对象进行改变 改变的是自身 线程安全 可将字符缓冲区在多个线程中安全的使用

StringBuilder 字符串变量 线程不安全的 适合字符串缓冲区被单个线程使用时候使用

————————————————————————————————————

10.java四种引用（对象回收 ，引用进入引用队列）

强引用

Object object=new Object() 只有在object这个引用被释放后，对象才会被释放掉 即使程序报错崩溃也不会回收
object是存在内存空间中的一个连续的地址 当我们声明实例后会有个强引用指针指向这个内存空间 只有这个引用释放掉 才行
即使对这个对象赋值为null 不会立即回收 回收器会在适当的时机回收


软引用

如果一个对象持有软引用，那么当内存充足时，GC不会回收它，当内存不足时，GC会对其回收 
软引用主要用户实现类似缓存的功能 常和一个引用队列共同使用 当软引用的对象被回收时 jvm会把这个软引用放入相关联的队列中

弱引用

弱引用相较于软引用有更短的生命周期，一旦GC发现只具有弱引用的对象，无论当前内存是否不足，都会对其进行回收
弱引用主要用于监控对象是否被标记为即将被回收的垃圾

虚引用
如果一个对象仅存在一个虚引用，那么就和没有引用相同，任何时候都可能被GC回收
虚引用主要用于跟踪GC的活动

————————————————————————————————————

11.MVC-MVP-MVVM （代码组织模式）

mvc是最经典的模式 m包含数据逻辑状态 v包含视图显示 而c层就是分开控制m和v层的控制中枢 但是c层是activity和fragment 经常在其中做一些操作ui和保存数据的操作 耦合度高 项目大起来不容易维护 缺点还是很明显的

mvp把layout和activity作为了view层 增加了presenter层做控制层 针对接口进行编程，所以业务逻辑工作交给presenter去操作 而view，model只做属于自己的工作
消除了对特定view的耦合 灵活性大大提升
缺点是随着业务增多 Presenter层会越变越大

自身实现：

1.先针对业务逻辑与界面显示划分 建立contract接口层 接口中包含 调用请求的接口 和界面变化的接口

2.再写继承contract界面接口和实现contract业务接口的presenter层 在实现的业务函数中进行http请求 在请求后调用contract界面函数

3.在view层实现contract的view接口，创建presenter并在合适的地方请求网络 在完善实现view的函数


mvvm
使用了数据绑定机制 视图可以绑定到变量和表达式
view绑定了由viewModel暴露的变量和操作
ViewModel负责包装模型并准备视图所需的可观察数据

问题 会随着时间 xml的代码会越多 无关的展示逻辑应该直接获取值 而不是绑定
————————————————————————————————————

12. android消息机制 -源码分析

成员介绍

Handler:消息处理类 handlemessage函数用来处理消息 sendmessage函数向消息队列发送消息

Message:消息的载体 数据结构为单链表 包含handler实例target 

MessageQueue:消息队列 fifo数据结构消息的存储体 包含消息实例 通过next函数得到下一个消息 可以通过enqueue函数让消息入队

Looper:消息循环类 包换消息队列实例 调用Looper.loop（）就能进入到消息循环中

如何相互运作的？

用户创建的线程是没有对应的looper的
而ui线程在apk启动时 activitymanagerservice会创建actvity和对应的ui线程，在启动主线程时默认创建一个looper 并执行looper.loop进入循环中


当我们在子线程中加载完数据需要向主线程发送信息时
首先创建handler实例 它是我们传递消息和处理消息的关键 在此时我们还会创建handler对应的messagequeque ，但handler需依赖于looper 这样mq就与handler和looper相关联
通过handler.obtainMessage()从消息池（也是个单链表缓冲区）里取出一个消息并装入想发送的信息
通过handler.sendMessage() 我们把消息传入消息队列中 等待looper循环到我们消息
looper.loop函数主要循环干了3件事情 1.读取mq的下一个消息2.把消息分给相应的target也就是handler3.再把分发后的消息回放的消息池里
然后是handler处理消息
在处理之前首先需要分发消息
若消息中带了回调 则调用回调 >若handler有回调则执行回调中的处理消息方法，然后再执行自身handler处理数据

```
/**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {//是一个Runnable对象，实际就是Handler的post方法所传递的Runnable参数
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);//处理消息
        }
    }
```

![handle机制框架](https://upload-images.jianshu.io/upload_images/5051594-156bb5b9d2cfba32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

————————————————————————————————————

13.进程和线程的区别？

进程是 资源分配的基本单位

线程是 处理器调度的基本单位

一个进程中可以包含多个线程 多个线程共享这个进程的资源

多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用而是看作一个进程整体，来实现进程的调度和管理


————————————————————————————————————

14.图片相关面试题

图片一直是android中处理的重点 处理不好就会出现oom等内存异常出现 回收复用压缩这些都是图片必须掌握的操作

一。使用bitmap时要注意什么？

bitmap对象经常占用大量的空间 
为此需要高效加载，使用bitmapfactory类 不把图片加入内存 而是读取 并decode图片资源于我们需要的option配置 并加载我们处理后的小图
异步处理图片，执行结束后及时释放资源 
在需要大量加载图片的场景 listview,viewpager 需要图片复用 还要避免图片被重复处理 这时候就需要缓存

图片的三级缓存

内存缓存 LruCache 最近最少使用的优先回收原则 本质使用哈希链表 内部hashmap进行存储 bitmap的list 
ps：垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠

本地硬盘缓存 以md5加密的url为文件地址 创建缓存图片 并压缩存储

网络缓存 从网络获取图片流 并由bitmapfactory decode生成bitmap对象


————————————————————————————————————

15.项目优化方面




————————————————————————————————————








