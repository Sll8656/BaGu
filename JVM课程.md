# 类加载

1、加载：根据类的全限定名将字节码文件的内容转换成合适的数据放入内存的方法区和堆中。

* 类加载器根据类的全限定名以二进制流的方式获取字节码信息
* 完成加载后，jvm将字节码中的信息保存到内存的方法区中
* 生成一个InstanceKlass对象，保存类的所有信息
* 在堆中生成一份与方法区中数据类似的java.lang.Class对象

2、连接

* 验证：验证内容是否符合jvm规范（CAFEBABE、父类不空、字节码跳转指令的目的地正确）
* 准备：初始化静态变量
* 解析：将常量池中的符号引用替换成指向内存的直接引用

3、初始化

* 执行静态代码块中的代码，为静态变量赋值
* 执行字节码文件中clinit部分的字节码指令

​	**以下几种方式会导致类的初始化：**

* 访问一个类的静态变量或者静态方法（前提是没有final修饰）
* 调用class.forName(String className)
* new一个该类的对象时
* 执行Main方法的当前类 

例题：

```java
public class Test{
  public static void main(String[] args) {
  	System.out.println("A");
    new Test();
    new Test();
  }
  
  public Test() {
    System.out.println("B");
  }
  
  {
    System.out.println("C");
  }
  static {
    System.out.println("D");
  }
}

程序执行结果：DACBCB
```

```java
public class Demo02{
  public static void main(String[] args) {
    new B02();
    System.out.println(B02.a);
  }
  
  class A02 {
    static int a = 0;
    static {
      a = 1;
    }
  }
  
  class B02 extends A02 {
    static {
      a = 2;
    }
  }
}
程序执行结果：
  1、调用new创建对象，需要初始化B02，优先初始化父类 a = 1
  2、子类初始化a = 2
  最终a = 2；
```

```java
public class Test2 {
  public static void main(String[] args) {
    Test2_A[] arr = new Test2_A[10];
  }
}

class Test2_A {
  static {
    System.out.println("Test2_A的静态代码块运行");
  }
}
结果：数组的创建不会导致数组中元素的类进行初始化
```

```java
public class Test4 {
  public static void main(String[] args) {
		System.out.println(Test4_A.a);
  }
}

class Test4_A {
  public static final int a = Integer.valueOf(1);
  static {
    System.out.println("Test3_A的静态代码块运行");
  }
}
```

## GPT版本

Java类加载过程是将Java类文件（.class文件）加载到内存中，使其可供JVM使用的过程。这个过程主要包括三个阶段：加载、链接和初始化。以下是每个阶段的详细说明，用简单易懂的语言描述。

### 1. 加载（Loading）

**加载**是将类的二进制数据（.class文件）读入内存，并创建一个代表这个类的`Class`对象的过程。这是类加载过程的第一步。

- **查找和加载类的二进制数据**：
  - JVM通过类加载器（ClassLoader）找到类的二进制数据。
  - 类加载器从文件系统、网络、JAR包等位置读取类文件。
- **创建Class对象**：
  - JVM为每个加载的类创建一个`Class`对象，并将其存储在方法区中。

示例：当你运行一个Java程序时，JVM首先加载主类（包含`main`方法的类）。

### 2. 链接（Linking）

**链接**是将类的二进制数据合并到JVM的运行时环境中的过程。链接分为三个步骤：验证、准备和解析。

#### 验证（Verification）

- 检查类文件的正确性
  - 确保类文件的格式正确，没有安全问题。
  - 验证字节码是否符合JVM规范，防止恶意代码破坏JVM。

示例：检查方法是否有正确的返回类型。

#### 准备（Preparation）

- 为类的静态变量分配内存，并初始化默认值
  - 分配内存给静态变量，但不执行初始化代码。
  - 设置默认值，如`0`、`null`或`false`。

示例：如果类有一个`static int x`变量，在这个阶段`x`被分配内存并初始化为`0`。

#### 解析（Resolution）

- 将符号引用转换为直接引用
  - 将类、方法、字段等符号引用解析为可以直接访问的内存地址。
  - 这一步可能在链接阶段完成，也可能在类使用时延迟完成。

示例：将方法调用从符号引用转换为方法在内存中的实际地址。

### 3. 初始化（Initialization）

**初始化**是执行类的静态初始化块和静态变量初始化的过程。

- 执行静态初始化块

  ：

  - 运行类中的静态代码块。
  - 初始化静态变量的值。

示例：执行类中的静态代码块和静态变量赋值语句。

```java
class Example {
    static int x = 10;
    static {
        x = 20;
    }
}
```

在初始化阶段，`x`被赋值为`10`，然后静态块被执行，`x`最终被赋值为`20`。

### 总结

1. **加载**：读取类文件，创建`Class`对象。
2. **链接**：包括验证（确保类文件正确性）、准备（为静态变量分配内存并初始化默认值）和解析（将符号引用转换为直接引用）。
3. **初始化**：执行静态初始化块和静态变量的初始化。

这个过程确保类在运行时被正确地加载、链接和初始化，使其能够被安全和高效地使用。



---



类加载器（ClassLoader）是Java虚拟机提供给应用程序去实现获取类和接口字节码数据的技术

应用场景：

SPI机制、类的热部署、Tomcat类的隔离、打破双亲委派、自定义类加载器、使用Arthas不停机解决线上故障

分类：

> ​	分为两类，一类是java代码。一类是虚拟机底层源码实现的

* 底层的类加载器：保证Java程序运行中基础类被正确加载，比如java.lang.String，确保可靠性
* ==Java实现的类加载器==：所有Java中实现的类加载器都需要继承ClassLoader这个抽象类

|        虚拟机底层实现的         |              java实现的               |
| :-----------------------------: | :-----------------------------------: |
| Bootstrap：加载java中最核心的类 | Extension：允许扩展Java中比较通用的类 |
|                                 |     Application：加载应用使用的类     |

1、Bootstrap ClassLoader：

​	由Hotspot虚拟机提供、使用c++编写的类加载器

​	默认加载Java安装目录/jre/lib下的类文件，比如rt.jar, tools.jar, resource.jar等

2、java类加载器

​	扩展类和应用程序类加载器都是JDK提供的，使用Java编写的类加载器。他们的源码位于sun.misc.Launcher中，是一个静态内部类。继承自URLClassLoader。具备通过目录或者指定jar包将字节码文件加载到内存中。

**扩展类加载器：**默认加载Java安装目录/jre/lib/ext下的类文件。 

**应用程序类加载器：**加载classpath下的类文件。项目和第三方依赖中的。  

---

双亲委派机制：核心是解决在有多个类加载器的情况下，一个类到底由谁加载的问题

作用：

* 保证类加载的安全性：java.lang.String，确保核心类库的完整性和安全性
* 避免重复加载：避免同一个类被多次加载

解释：当一个类加载器收到加载类的任务时，会**自底向上查找是否加载过，再自顶向下进行加载。顺口溜：向上查找（我是不是加载过，如果加载过，那么就由这个类加载器加载，流程结束），向下加载 （是不是在我的加载目录中，在就加载，不在就继续往下）**

---

如果一个类重复出现在三个类加载器的加载位置，应该由谁来加载？

答：启动类加载器

---

String类能不能被覆盖？在自己的项目中创建java.lang.String，会被加载吗？

答：不会，因为会先向上查找，查找的时候，就已经由Bootstrap类加载器加载了。

---

![image-20240517145506294](https://s2.loli.net/2024/09/18/qp6M3hYwlTj7nOI.png)

---

打破双亲委派机制：

1.自定义类加载器

2.线程上下文类加载器

---

**自定义类加载器**

![image-20240517155204195](https://s2.loli.net/2024/09/18/8DEym4viOMqGB6w.png)



 ![image-20240517155343405](https://s2.loli.net/2024/09/18/4QOYxXh2jDgSVoU.png)

![image-20240517155718971](https://s2.loli.net/2024/09/18/jwXF95i8YDvdOGe.png)

![image-20240517160603607](https://s2.loli.net/2024/09/18/8nf2TKhWIjkpvBR.png)

---

如果自己实现了自定义类加载器，那么他的默认父类加载器是applicationClassLoader。

---

![image-20240517161526929](https://s2.loli.net/2024/09/18/sSzPl4VqWwrdO2y.png)

---



![image-20240517162047453](https://s2.loli.net/2024/09/18/Ql27mNPWDsMGdAo.png)

---

![](https://s2.loli.net/2024/09/18/QS2UjgiXF3uqwyL.png)

DriverManager怎么知道jar包中要加载的驱动在哪儿？

答：SPI机制

![image-20240517163137115](https://s2.loli.net/2024/09/18/ZpbivTeSCxIJ9XK.png)

---

SPI是如何获取到应用程序类加载器的？

答：

![image-20240517163616385](https://s2.loli.net/2024/09/18/s4Vyl5gbA61zX3C.png)

小结：

![image-20240517164130347](https://s2.loli.net/2024/09/18/J4rDj3bGQlYEcm2.png)

---



![image-20240517164324871](https://s2.loli.net/2024/09/18/oRprPJeUIj9dbnL.png)

jdbc只是在DriverManager加载完之后，通过初始化阶段触发了驱动类的加载，而类的加载流程依然遵循双亲委派机制。 

3、osgi模块化，了解即可。

---

jdk9（包括jdk9）以后的类加载器。（了解即可）

  ![image-20240517180710634](https://s2.loli.net/2024/09/18/AN5a9igLFboPQIW.png)

![image-20240517181007415](https://s2.loli.net/2024/09/18/X4b8KmZ67B3lhNz.png)

---

总结：

**1、类加载器的作用：**

​	获取在类加载过程中的字节码，并将其加载到内存。通过加载字节码数据放入内存转换成byte[]，接下来调用虚拟机底层方法将byte[]转换成方法区和堆中的数据。

2、有几种类加载器？

* 启动类加载器：加载核心类
* 扩展类加载器：加载扩展类
* 应用程序类加载器：加载项目中和第三方jar包中的类
* 自定义类加载器：重写`findClass`方法

jdk9之后，扩展类加载器变成了平台类加载器。且启动类加载器从c++实现改为由java实现。

3、什么是双亲委派机制？

​	每个Java实现的类加载器中保存了一个成员变量叫parent。自底向上查找是否加载过，再自顶向下进行加载。避免了核心类被应用程序重写并覆盖的问题，提升了安全性。

<img src="https://s2.loli.net/2024/09/18/dCPUjO768pz9DNf.png" style="zoom:50%;" />

4、打破双亲委派机制？

1）重写loadClass方法

2）JDBC，Dubbo等框架使用了SPI机制 + 线程上下文类加载器

3）OSGi实现了一整套的类加载机制，允许同级类加载器之间相互调用

---



# 内存区域

java虚拟机在运行Java程序过程中管理的内存区域，称之为运行时数据区

![image-20240517182129042](https://s2.loli.net/2024/09/18/Ht5syihDYnBUSbQ.png)

## **程序计数器**

* 控制程序指令的执行，实现分支、跳转、异常等逻辑
* 在多线程执行下，jvm会通过pc来记录线程执行的位置，方便多线程切换找到原来断点的位置。

问：程序计数器在运行中会出现内存溢出吗？

答：不会，每个线程只存储一个固定长度的内存地址。

---

## **虚拟机栈**

随着线程的创建而创建，回收会在线程的销毁时进行，由于方法可能会在不同线程中执行，每个线程都会包含一个自己的虚拟机栈。

 虚拟机栈中有很多栈帧。栈帧中保存着局部变量表、操作数栈、帧数据

![image-20240517185142893](https://s2.loli.net/2024/09/18/vinBdyaXoE2YLUM.png)

---

**栈内存溢出：**

​	Java虚拟机栈如果栈帧过多，会出现内存溢出，错误为StackOverFlowError

 修改虚拟机栈的大小：-Xss<u>栈大小</u>

注意事项：

* HotSpot虚拟机对大小有限制要求，jdk8最小为180K，最大为1024M

* 局部变量过多、操作数栈深度过大也会影响栈内存的大小

一般情况下，手动指定为-Xss256K即可

---

## **本地方法栈**

虚拟机栈是Java方法调用时的栈帧，本地方法栈存储的是native本地方法的栈帧。

在Hotspot虚拟机中，==Java虚拟机栈和本地方法栈使用的是同一个栈空间==。

---

## **堆**

堆内存是jvm中最大的一块内存，创建出来的对象都存储在堆中。

栈帧的局部变量表中，可以存放堆中对象的引用。静态变量也可以存放堆中对象的引用，通过静态变量就可以实现对象在线程之间共享。

---

在堆中有三个区域，used，total，max，顾名思义即可。total不够的时候，会扩容 used < total < max。

问：是不是当used = max = total 的时候，堆内存就溢出了？

答：不是，堆内存溢出的判断条件比较复杂。在垃圾回收期这一章再解释

---

配置total和max：

如果不设置任何虚拟机参数，max默认是系统内存的1/4，total默认是系统内存的1/64。在实际应用中，**一般都需要设置total和max的值。**

```java
total:  -Xms___
max:    -Xmx___
限制： Xmx > 2MB, Xms > 1MB
```

---

问：为什么arthas中显示的heap堆大小与设置的值不一样？

答：arthas中的heap堆内存使用了JMX技术中内存获取方式，这种方式与垃圾回收器有关，计算的是可以分配对象的内存，而不是整个内存。

---

建议将-Xmx和-Xms设置为相同的值，这样程序就不需要向java虚拟机再次申请内存。减少申请和分配内存时间上的开销，同时也不会出现内存过剩之后堆收缩的情况。

-Xmx具体设置的值与实际的应用程序运行环境有关，在实战篇中会给出设置方案。

---

## **方法区**

方法区存放基础信息的位置，线程共享，主要包含3个部分：类的元信息、运行时常量池、字符串常量池。

jdk7以及之前：方法区在堆中，叫做永久代。

jdk8及之后：方法区在元空间中，元空间是本地内存的。只要不超过操作系统的上限，就可以一直分配。

---

**模拟方法区溢出**

用过ByteBuddy框架，动态生成字节码数据加载到内存中。

![image-20240517195200344](https://s2.loli.net/2024/09/18/pYoOA2XHsJKQZtI.png)

通过实验发现：JDK7方法区存放在堆中，使用-XX:MaxPermSize=__ 来设置。JDK8方法区存放在元空间中，使用-XX:MaxMetaspaceSize=__来设置最大大小。

---

**字符串常量池：**

* 方法区中除了类的元信息、运行时常量池之外，还有一块区域叫字符串常量池(StringTable)。
* StringTable中存储在代码中定义的常量字符串内容。

在早期，字符串常量池属于运行时常量池的一部分，他们存储的位置也是一致的，后续做出了调整，将字符串常量池和运行时常量池**做了拆分**。

![image-20240517202330408](https://s2.loli.net/2024/09/18/VGt4XKhDqSHyQ9f.png)

---

问题：分析下面代码的运行结果

```java
public static void main(String[] args) {
  String a = "1";
  String b = "2";
  String c = "12";
  String d = a + b;
  String e = "1" + "2";
  System.out.println(c == d);    //使用了StringBuilder
  System.out.println(c == e);    //常量，编译阶段直接连接
}
```

答： c == d :输出false，因为d在堆中，c在StringTable中。 c == e：输出true，都是StringTable中。

---

![image-20240517203248801](https://s2.loli.net/2024/09/18/d8XVabW34mrMySq.png)

执行结果：true，false

---

**静态变量的存储**

JDK7之前，存放在方法区，也就是永久代中。JDK7及之后的版本中，静态变量存放在堆中的Class对象中，脱离了永久代。

---

## 直接内存

直接内存不属于Java运行时的内存区域

在JDK1.4中引入了NIO机制，使用了直接内存，主要为了解决以下两个问题：

​	1、Java堆中的对象如果不再使用要回收，回收时会影响对象的创建和使用

​	2、IO操作比如读文件，需要先把文件读入直接内存（缓冲区）再把数据复制到Java堆中。

现在直接放入直接内存即可，同时Java堆上维护直接内存的引用，减少了数据复制的开销，写文件也是类似的思路。

![image-20240517204015070](https://s2.loli.net/2024/09/18/7uRnZdbApmGBhCk.png)

---

![image-20240517204041468](https://s2.loli.net/2024/09/18/gQtmk1w9vyh6nfE.png)

---

# 垃圾回收

c和c++要手动释放，java引入了自动的垃圾回收GC机制。

垃圾回收器主要负责堆上的回收

应用场景：

![image-20240518084736960](https://s2.loli.net/2024/09/18/fGcRjI7DZ4HhTgb.png)

## 方法区的回收

pc，java虚拟机栈，本地方法栈不需要垃圾回收。因为这部分区域是**线程不共享的，他们随着线程的创建而创建，线程的销毁而销毁**。这些区域不需要垃圾回收。

判定==一个类==可以被回收，需要同时满足三个条件：

1、**该类的所有实例对象已经被回收，在堆中不存在任何该类的实例对象以及子类对象**

2、加载该类的类加载器已经被回收

3、该类对应的java.lang.Class对象没有在任何地方被引用



## 堆回收

### **引用计数**

Java中的对象是否能被回收，是根据对象**是否被引用**来决定的，如果对象被引用了，说明该对象还在使用，不允许被回收。

缺点：

1、每次引用和取消引用都需要维护计数器，对性能有一定的影响

2、如果发生了循环引用，引用计数法就失效了。

### **可达性分析**

​	Java使用的是可达性分析算法来判断对象是否可以被回收，可达性分析将对象分为两类：**垃圾回收的根对象（GC Root）和普通对象**，对象与对象之间存在引用关系。

可以作为GC Root的对象：

* 虚拟机栈和本地方法栈中引用的对象
* 方法区中类静态属性引用和常量引用的对象
* 监视器对象，用来保存同步锁synchronized关键字持有的对象
* 本地方法调用时使用的全局对象

不容易理解的分类：

* 系统类加载器加载的java.lang.Class对象，引用类中的静态变量
* 线程Thread对象
* 监视器对象，用来保存同步锁synchronized关键字持有的对象
* 本地方法调用时使用的全局对象

![image-20240518091732408](https://s2.loli.net/2024/09/18/NTyiZrUdL645AP8.png)

问：是不是 只有沿着GCRoot找不到的对象  这种情况才能被回收？

### 对象引用

​	可达性分析法中描述的对象引用指的是强引用，即是GCRoot对象对普通对象有引用关系，只要这层关系存在，普通对象就不会被回收。除了强引用之外Java中还设计了几种其他引用方式：软引用、弱引用、虚引用。

**软引用：**当程序==内存不足==时会被回收，常用于缓存中。

![image-20240518093924119](https://s2.loli.net/2024/09/18/BmXCSGDyshAeuzL.png)

软引用中的对象在内存不足时会被回收，SoftReference**对象本身也需要被回收**，如何知道哪些SoftReference对象需要回收？

SoftReference提供了一套队列机制：

1、软引用创建时，通过构造器传入引用队列

2、在软引用中包含的对象被回收时，该软引用对象会被放入引用队列

3、通过代码遍历引用队列，将SoftReference的强引用删除

---

软引用的使用场景-缓存

![image-20240518095303575](https://s2.loli.net/2024/09/18/tP8wKQrlTUbujmA.png)

---

**弱引用：**==不管内存够不够==，只要垃圾回收了，都会被回收。主要在ThreadLocal中被使用

弱引用对象本身也可以使用引用队列进行回收

![image-20240518100039216](https://s2.loli.net/2024/09/18/mZeB8MaVylUckdC.png)

![image-20240518100219109](https://s2.loli.net/2024/09/18/Wga3yYfBmFKAiIL.png)

执行结果：第一次有数据，第二次为null。可以证明，即使内存足够，也会被回收。

---

**虚引用和终结器引用：**这两种引用在常规开发中不会使用

![1](https://s2.loli.net/2024/09/18/7slOcqNTv9JAVtI.png) 

---

### 垃圾回收算法

 核心思想：找到内存中存活的对象，释放不存活的对象占用的空间。

4种分类：标记清除、复制、标记整理、分代GC

评判标准：Java垃圾回收会通过单独的GC线程来完成，但是不管使用哪一种，都会有部分阶段需要停止所有的用户线程。也就是STW。STW时间过长会影响用户的使用。

* 吞吐量：`执行用户代码时间 / (执行用户代码时间 + 总时间)`
* 最大暂停时间：垃圾回收过程中，STW时间的最大值
* 堆使用效率：复制算法中堆使用效率只有50%

上述三个指标，不可兼得。

**标记清除**

1、标记阶段：使用可达性分析，从GC Root开始遍历出所有存活的对象

2、清除阶段：从内存中删除非存活的对象

优点：内存使用率高。缺点：有内存碎片，且分配速度慢，因为需要链表来维护内存碎片。

**复制算法**

准备两块空间From和To，将From存活的对象全部放入To中，然后To和From名称互换。

优点：吞吐量高，没碎片。缺点：内存只能用一半

**标记整理**

1、标记阶段：使用可达性分析，从GC Root开始遍历出所有存活的对象

2、整理阶段：**将存活对象移动到堆的另一端**，清理掉非存活对象的空间

优点：无碎片，内存使用率高。缺点：要整理

==**分代GC**==

堆内存 = 年轻代 + 老年代。 年轻代 = Eden + Survivor0 + Survivor1

过程：新对象放在Eden，存活后放进Survivor，每次存活年龄++，最大15岁后进入老年代。

![image-20240518133048816](https://s2.loli.net/2024/09/18/ezx58rQh3IdO9pf.png)

当老年代中空间不足，无法放入新的对象，先尝试minor gc，如果还是不够，就会触发Full GC。Full GC会对整个堆进行垃圾回收。

---

![image-20240518133901836](https://s2.loli.net/2024/09/18/zsalhoKPfqc75Uj.png)

 

### 垃圾回收器

学习路线：

1、垃圾回收器的种类：常见的垃圾回收器的使用场景

2、使用垃圾回收器：如何指定垃圾回收器，进行简单的测试

3、垃圾回收器实战：针对应用常见进行垃圾回收器的调优、解决垃圾回收导致的CPU占用率高的问题

4、原理：垃圾回收器实现的底层原理

---

问：为什么分代GC算法要把堆分成年轻代和老年代。

![image-20240518134841920](https://s2.loli.net/2024/09/26/q8rBTHgta9bzdex.png)

**垃圾回收器的组合关系**

![image-20240518134954046](https://s2.loli.net/2024/09/18/F7UA9KO1s45c2TR.png)

---

组合1

**年轻代-Serial垃圾回收器**

<img src="https://s2.loli.net/2024/09/18/AnfTq5X2bPElpRS.png" style="zoom:30%;" />

**老年代-SerialOld垃圾回收器**

<img src="https://s2.loli.net/2024/09/18/8YtzRDH95sgNfL4.png" alt="image-20240518135313998" style="zoom:30%;" />

---

组合2

**年轻代-ParNew垃圾回收器**（==多线程==回收年轻代的垃圾）

<img src="https://s2.loli.net/2024/09/18/SpLFOdbfAEPBUQr.png" alt="image-20240518135633538" style="zoom:30%;" />

**老年代-CMS（Concurrent Mark Sweep）垃圾回收器**（减少SWT对用户线程的影响）

 <img src="https://s2.loli.net/2024/09/18/JoImEKOn4kFTyLf.png" alt="image-20240518135848820" style="zoom:30%;" />

**cms的执行步骤**

<img src="https://s2.loli.net/2024/09/18/RzsSTiIj3CgU2G4.png" alt="image-20240518140056199" style="zoom:33%;" />

---

组合3

**年轻代-Parallel Scavenge垃圾回收器**（关注吞吐量）

<img src="https://s2.loli.net/2024/09/18/THfY2E3Be9hOLv6.png" alt="image-20240518140524868" style="zoom:33%;" />

**老年代-Parallel Old垃圾回收器**

<img src="https://s2.loli.net/2024/09/18/OUe1TQ3wMS69krN.png" alt="image-20240518140654735" style="zoom:33%;" />

---

==**G1**==

<img src="https://s2.loli.net/2024/09/18/uQlCzTRVwkFAg1m.png" alt="image-20240518141622339" style="zoom:33%;" />

G1出现之前的内存结构：

<img src="https://s2.loli.net/2024/09/18/FNurcJKoGg2LA3p.png" alt="image-20240518141748988" style="zoom:33%;" />

G1的内存结构：

<img src="https://s2.loli.net/2024/09/18/u7Uf5IrMJsyqk4g.png" alt="image-20240518141820904" style="zoom:33%;" />

G1垃圾回收器有两种方式：年轻代回收(Young GC)和混合回收(Mixed GC)

Young GC：回收Eden和Survivor区中不用的对象，会导致STW，可以通过参数`-XX:MaxGCPauseMillis=n`(默认200)设置每次垃圾回收时的最大暂停时间毫秒数，G1垃圾回收器会尽可能地2保证暂停时间。

1、新创建的对象会存放在Eden区，当G1判断年轻代区不足（max默认60%），无法分配对象时会执行Young GC

2、标记出Eden和Survivor区域中存活的对象

3、根据配置的最大暂停时间选择某些区域将存活对象复制到一个新的Survivor区中（年龄+1），清空这些区域



<img src="https://s2.loli.net/2024/09/18/t34BKhYgbHL9IwO.png" alt="image-20240518142459367" style="zoom:33%;" />

4、后续Young GC与之前的相同，只不过Survivor区中存活对象会被搬运到另一个Survivor区

5、当某个存活对象的年龄达到阈值，就会被放入老年代

6、部分对象如果大小超过Region的一般，会直接放入老年代，这类老年被称为Humongous区。比如堆内存4G，每个Region是2M，只要一个大对象超过了1M就被放入 区，如果对象过大会横跨多个Region。

7、多次回收之后，会出现很多Old老年代区，此时总堆占有率达到阈值时，会触发混合回收Mixed GC（默认45%），回收所有年轻代和部分老年代的对象以及大对象区。采用复制算法完成。

MixGC:

<img src="https://s2.loli.net/2024/09/18/upNd9YmLQJPEcKO.png" alt="image-20240518143314320" style="zoom: 33%;" />

<img src="https://s2.loli.net/2024/09/18/z6p7WMVXeQthTg3.png" alt="image-20240518143503882" style="zoom:33%;" />

<img src="https://s2.loli.net/2024/09/18/SkBIgFePyMRa2ho.png" alt="image-20240518143531142" style="zoom:33%;" />

小结：

![image-20240518143638870](https://s2.loli.net/2024/09/18/v9Xp5DlQ7AjzCsF.png)



---

总结：

问：垃圾回收器这么多，怎么记？

答：

<img src="https://s2.loli.net/2024/09/18/yANstEaVm2YfQTM.png" alt="image-20240518143919775" style="zoom:33%;" />

问：Java中有哪几块内存需要进行垃圾回收？

答：pc，虚拟机栈，本地方法栈是线程不共享的，不用管理。方法区一般不需要回收，堆由垃圾回收器进行回收。

问：有哪几种常见的引用类型？

答：

* 强引用：最常见的引用方式，由可达性分析法来判断
* 软引用：对象在没有强引用情况下，内存不足会回收
* 弱引用：对象在没有强引用情况下，会直接回收
* 虚引用：通过虚引用知道对象被回收了
* 终结器引用：对象回收时可以自救，不建议使用

问：有哪几种常见的垃圾回收算法？

答：标记清除，标记整理，复制，分代GC

问：常见的垃圾回收器有哪些？

![image-20240518144412517](https://s2.loli.net/2024/09/18/GLy1djwRXBKx9Up.png)

serial组合：单线程回收，适合单核cpu场景

parnew  + cms：暂停时间短，适合大型互联网应用中与用户交互的部分

ps+po：吞吐量高，适用于后台进行大量数据操作

G1：适用于较大的堆，具有可控的暂停时间

---

# 实战篇

## 内存调优

学习路线：

* 什么是内存泄漏
* 监控java内存的常用工具
* 内存泄漏的常见场景
* 内存泄漏的结局方案

---

内存泄漏：在Java中如果不再使用一个对象，但是该对象依然在GC ROOT引用链上，这个对象就不会被垃圾回收期回收，这种情况叫做内存泄漏。内存泄漏大多数情况都是**堆内存**泄漏引起的。

少量的内存泄漏可以容忍，但是持续内存泄漏最终会导致内存溢出，但是产生内存溢出并不是只有内存泄漏这一种原因。

Top命令：

![image-20240521095734949](https://s2.loli.net/2024/09/18/aFTX58Jz3iQcWPe.png)