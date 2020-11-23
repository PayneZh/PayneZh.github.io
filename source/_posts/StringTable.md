---
title: StringTable
date: 2020-11-19 09:16:45
tags: java基础
categories: 编程
---
## String的基本特性

- String:字符串，使用一对“”引起来表示。
- String声明为final的，不可被继承。
- String实现了Serializable接口：表示字符串是支持序列化的。实现了Comparable接口：表示String可以比较大小。
- String在jdk8及以前内部定义了final char[] value用于存储字符串数据。jdk9时改为byte[]。（https://openjdk.java.net/jeps/254）
- String:代表不可变的字符序列。简称：不可变性。
 >>当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
 >>当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
 >>当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

- 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。
- 字符串常量池中是不会存储相同内容的字符串的。 
  >>String的String Pool是一个固定大小的HashTable，默认值大小长度是1009.如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下滑。
  >>使用-XX:StringTableSize可设置StringTable的长度
  >>在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTableSize设置没有要求
  >>在jdk7中，StringTable的长度默认值是60013,在jdk8中，1009是可设置的最小值。

## String的内存分配

- 在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念。
- 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种。
 >>直接使用双引号声明出来的String对象会直接存储在常量池中。比如：String info = “abc”；
 >>如果不是用双引号声明的String对象，可以使用String提供的intern()方法。这个后面重点谈
- Java 6及以前，字符串常量池存放在永久代
- Java 7中Oracle的工程师对字符串池的逻辑做了很大的改变，即将字符串常量池的位置调整到Java堆内。
 >>所有的字符串都保存在堆（Heap）中，和其它普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
 >>字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在Java 7中使用String.intern()。
- Java8元空间。字符串常量池在堆

![图1](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/jvm%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/JDK6%E6%96%B9%E6%B3%95%E5%8C%BA%E5%9B%BE.jpg)
![图2](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/jvm%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/JDK7%E6%96%B9%E6%B3%95%E5%8C%BA%E5%9B%BE.jpg)
![图3](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/jvm%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/JDK8%E6%96%B9%E6%B3%95%E5%8C%BA%E5%9B%BE.jpg)

StringTable为什么要调整？
官网：https://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html#jdk7changes
1. permSize默认比较小，容易导致oom
2. 永久代垃圾回收频率低

## String的基本操作

```
public class Memory {
    public static void main(String[] args) {
        int i = 1;
        Object obj = new Object();
        Memory mem = new Memory();
        mem.foo(obj);
    }

    private void foo(Object param){
        String str = param.toString();
        System.out.println(str);
    }
}
```

以上代码的内存分布图如下：
![图4](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/StringTable/%E6%9F%90%E4%BB%A3%E7%A0%81%E7%9A%84%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E5%9B%BE.jpg)

## 字符串拼接操作

1. 常量与常量的拼接结果在常量池，原理是编译器优化。
2. 常量池中不会存在相同内容的常量。
3. 只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder
4. 如果拼接的结果调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

## intern()的使用

如果不是用双引号声明的String对象，可以使用String提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，若存在就会将当前字符串放入常量池中。

- 比如：String myInfo = new String("I love java").intern();

也就是说，如果在任意字符串上调用String.intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true：
("a" + "b" + "c").intern() == "abc"
通俗点讲，Interned String就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度，注意，这个值会被存放在字符串内部池（String Intern Pool）。

总结String的intern()的使用

- jdk1.6中，将这个字符串对象尝试放入串池。
	>如果串池中有，则并不会放入。返回已有的串池中的对象的地址
	>如果没有，会把**此对象复制一份**，放入串池，并返回串池中的对象地址。

- jdk1.7起，将这个字符串对象尝试放入串池。
	>如果串池中有，则并不会放入，返回已有的串池中的对象的地址
	>如果没有，则会把**对象的引用地址复制一份**，放入串池，并返回串池中的引用地址

## G1中的String去重操作

官方描述：http://openjdk.java.net/jeps/192

- 背景：对于许多java应用（有大的也有小的）做的测试得出以下结果：
	>堆存活数据集合里面String对象占了25%
	>堆存活数据集合里面重复的String对象有13.5%
	>String对象的平均长度是45

- 许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是String对象。更进一步，这里面差不多一半String对象是重复的，重复的意思是说:
string1.equals(string2)=true,堆上存在重复的String对象必然是一种内存的浪费。这个项目将在G1垃圾收集器中实现自动持续对重复的String对象进行去重，这样就能避免浪费内存。

- 实现：
	>当垃圾收集器工作的时候，会访问堆上存活的对象，对每一个访问的对象都会检查是否是候选的要去重的String对象。
	>如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理对列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象
	>使用一个hashtable来记录所有被String对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
	>如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用。最终会被垃圾收集器回收掉。
	>如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组了。

- 命令行选项
	>UseStringDeduplication(bool):开启String去重，默认是不开启的，需要手动开启
	>PrintStringDeduplicationStatistics（bool）:打印详细的去重统计信息
	>StringDeduplicationAgeThreshold（uintx）:达到这个年龄的String对象被认为是去重的候选对象
