---
title: jvm类加载器
date: 2020-09-15 15:53:04
tags: java基础
categories: 编程
---
# 概述
- 类加载器子系统负责从文件系统或者网络中加载class文件，class文件在文件开头有特定的文件标识。
- ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定
- 加载的类信息存放于一块称为方法区的内存空间，除了类的信息外，方法区中还会存放运行时常量池信息,可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）

# 详述

## 类加载过程包括
加载->链接（验证->准备->解析）->初始化

### 加载
1. 通过一个类的全限定名获取定义此类的二级制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行是数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口
4. 加载.class文件的方式
 - 从本地系统中直接加载
 - 通过网络获取，典型场景：Web Applet
 - 从zip压缩包中读取,成为日后jar、war格式的基础
 - 运行时计算生成，使用最多的是：动态代理技术
 - 由其他文件生成，典型场景：JSP应用
 - 从专有数据库中提取.class文件，比较少见
 - 从加密文件中获取，典型的防Class文件被反编译的保护措施

### 链接
1. 验证
 - 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机的自身安全
 - 主要包括四种验证：文件格式验证,元数据验证，字节码验证，符号引用验证
2. 准备
 - 为类变量分配内存并且设置该类变量的默认初始值，即零值
 - 这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化
 - 这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到java堆中
3. 解析
 - 将常量池内的符号引用转换为直接引用的过程。
 - 事实上，解析操作往往是伴随着JVM在执行完初始化之后再执行
 - 符号引用就是一组符号来描述所应用的目标，符号引用的字面量形式明确定义在<java虚拟机规范>的class文件格式中，直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
 - 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等，对应常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等

### 初始化
1. 初始化阶段就是执行类构造方法＜clinit＞()的过程
2. 此方法不需要定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来
3. 构造器方法中指令按语句在源文件中出现的顺序执行
4. ＜clinit＞()不同于类的构造器。（关联：构造器是虚拟机视角下的＜init＞()）
5. 若该类具有父类，JVM会保证子类的＜clinit＞()执行前，父类的＜clinit＞()已经执行完毕
6. 虚拟机必须保证一个类的＜clinit＞()方法在多线程下被同步加锁

## 类加载器的分类
- JVM支持两种类型的类加载器，分别为引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Defined ClassLoader).
- 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是java虚拟机规范却没有这么定义，而是**将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器**
- 无论类加载器的类型如何划分，在程序中我们最常见的类加载器始终只有3个，如下所示：![图1](https://raw.githubusercontent.com/PayneZh/MarkDownPhotos/master/res/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E5%88%86%E7%B1%BB.jpg)
- 自定义类加载器继承关系如下所示：![图2](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.jpg)

### 启动类加载器（BootStrapClassLoader） 
- 这个类加载器使用C/C++语言实现的,嵌套在JVM内部
- 它用来加载Java的核心库（JAVA_HOME/jre/lib/rt.jar、resource.jar或sun.boot.class.path路径下的内容），用于提供JVM自身需要的类
- 并不继承自java.lang.ClassLoader,没有父加载器
- 加载扩展类的应用类加载器，并指定为他们的父类加载器
- 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

### 扩展类加载器（ExtensionClassLoader)
- Java语言编写，由sun.misc.Launcher$ExtClassLoader实现
- 派生于ClassLoader类
- 父类加载器为启动类加载器
- 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载。

### 应用程序类加载器（AppClassLoader)
- java语言编写，由sum.misc.Launcher$AppClassLoader实现
- 派生于ClassLoader类
- 父类加载器为扩展类加载器
- 它负责加载环境变量classpath或系统属性java.class.path指定路径下的类库
- 该类加载是程序中默认的类加载器，一般来说，Java应用的类都是由它来完成加载
- 通过ClassLoader.getSystemClassLoader()方法可以获取到该类加载器