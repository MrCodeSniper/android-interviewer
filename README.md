# 2018android-面试题
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





