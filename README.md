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




