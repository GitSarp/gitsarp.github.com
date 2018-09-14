---
title: Java基础
date: 2018-07-01 21:31:48
tags: Java基础
categories: Java
---
### Java基础
 1. 基本数据类型

Name | 字节数 | 初始值 
- | :-: | -: 
byte | 1| 0
char | 2 | 空格 
short | 2 | 0
int | 4| 0
float | 4 | 0.0f
long | 8 | 0L
double | 8| 0.0d
char | 2 | 空格 
boolean | 不明确 | false

2. 获得变量类型：
1.System.out.println(TypeToolsTest2.class.getDeclaredField("ii").getType()); //成员变量
2.System.out.println(Integer.class.isInstance(ii));
3.泛型T.getClass().getName();


3. Java 会对 -128~127 的整数进行缓存
    因此Integer a=128;Integer b=128;xx c=127;xx d=127
    a!=b but c==d     (IntegerCache.class)
    == 它比较的是对象的地址;equals 比较的是对象的内容

4. IO:
    ` BufferedReader br = new BufferedReader(new InputStreamReader(System.in));br.read();/br.readLine();`
    ![类图](http://www.runoob.com/wp-content/uploads/2013/12/iostream2xx.png)

5. 类的实例化顺序：父类static，子类static，父类普通代码，父类构造，子类普通，子类构造

6. 强引用：用关键词new出来的对象，强引用锁指向的对象在任何时候都不会被回收 ，即使内存不足，抛出错误。

7. 集合：根接口Collection，List（有序可重复）,Set（无序不重复）接口都继承他。Map是单独的。

    LinkedList：底层用双链表实现:增加删除效率高
    ArrayList：底层用数组实现:查找效率高
    Vector：实现了一个动态数组。和ArrayList相似，但是有以下不同：vector是同步的； vector包含了许多传统方法，或者只是需要一个可以改变大小的数组的情况。
        Stack:先进后出
    HashSet:HashSet不存入重复元素的规则.使用hashcode和equals
        LinkedHashSet:是对LinkedHashMap的简单包装，对LinkedHashSet的函数调用都会转换成合适的LinkedHashMap方法
    TreeSet:让存入的元素自定义比较规则;给TreeSet指定排序规则
    7.HashMap和HashSet的区别
    前者可以接受键和值为null，线程不安全，单线程效率高；后者线程安全，Java5后可以由CurrentHashMap替代。前者不能保证数据的顺序随着时间的推移不变。