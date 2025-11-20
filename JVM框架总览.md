## JVM框架总览

![image-20250618104501762](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250618104501762.png)



## 程序计数器（寄存器）

**作用**：记录下一条JVM指令的执行地址

**特点**：

​	1、是线程私有的，即每个线程都有独立的程序计数器

​	2、不存在内存溢出问题，因为jvm规范了程序寄存器是没有内存溢出的区域



## 虚拟机栈

**定义：**

​	1、**虚拟机栈**：线程运行需要的空间，每个虚拟机栈由多个栈帧组成，对应着每次方法调用所占内存，每个线程只有一个活动栈帧，即正在执行的某个方法。

​	2、**栈帧**：每个方法运行所需要的内存，由参数，局部变量，返回地址等组成。

![image-20250618110302481](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250618110302481.png)

问题辨析：

​	**1、垃圾回收是否涉及栈内存？**

​		答：不涉及，因为栈由多个栈帧组成，当调用方法时开辟新的栈帧并压入栈，当方法结束时其栈帧会弹出栈，也就是会被自动回收，因此不涉及垃圾回收。



​	**2、栈内存分配越大越好吗？**

​		答：每个线程在创建的时候都会创建一个虚拟机栈，而物理内存是固定的，当栈内存越大时，可分配的线程数越少，相反栈内存越小，可能会导致内存溢出问题。



​	**3、方法内的局部变量是否是线程安全的？（逃逸分析）**

​		答：如果方法内局部变量没有逃离方法的作用范围，线程是安全的。

​				如果是局部变量引用了对象，并逃离方法的作用范围，线程是不安全的。



**栈内存（StackOverflowError）溢出原因**：

​	1、当栈帧过多导致内存溢出

​	2、当栈帧过大导致内存溢出

​	3、对象之间的循环依赖



## 本地方法栈

**定义**：本地方法栈为虚拟机使用到的Native方法服务，也就是一些带有native关键字的方法就是需要java去调用本地的C或C++方法，因为java有时候没办法直接和操作系统底层交互，所以需要用到本地方法。





## 堆（heap）

**定义**：通过new关键字，创建的对象都会使用堆内存。

**特点**：

​	1、它是线程共享的，堆中对象需要考虑线程安全问题。

​	2、有垃圾回收机制。

**堆内存溢出（OutOfMemoryError）**

​	在创建新对象时，堆内存中的空间不足以存放新创建的对象而产生堆内存溢出现象。

​	**原因**：

​		1、程序中出现了死循环，不断创建对象；

​		2、程序占用内存太多，超过JVM堆设置的最大值。

​	绝大多数内存溢出都属于堆溢出，由于大量对象占据了堆空间，这些对象都持有强引用导致无法回收，当对象大小之和大于Xmx参数指定的堆空间时就会发生堆溢出。

​	

**堆内存诊断**

jps工具：查看当前系统中有哪些java进程

jmap工具：查看堆内存占用情况

jconsole工具：图形界面，多功能的检测工具，可以连续检测（控制台输出jconsole调出可视化界面）



## **方法区（Method Area）**

**组成**

![image-20250618165156000](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250618165156000.png)

**定义**：是所有java虚拟机线程的共享区域，存储了跟类的结构相关的一些信息，包括类的成员变量、方法数据、成员方法和构造器方法的代码，运行时常量池等。

方法区在虚拟机启动时被创建，逻辑上是堆的一个组成部分，方法区存在于JVM内存是JVM 规范的概念模型，技术实现上在jdk1.8+已经移除JVM内存（永久代），转而存储在本地内存（元空间），方法区在运行时如果内存不足，也会抛出OutOfMemoryError错误。

![image-20250618165601218](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250618165601218.png)



**运行时常量池**

1、常量池，就是一张表，虚拟机指令根据这张常量表去找到要执行的类名、方法名、参数类型、字面量等信息。

2、运行时常量池，常量池是*.class文件中的，当该类被加载，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址。



**StringTable（串池）**

**定义**：StringTable是一个类，它是一张Hash表，默认大小1009；这张表在每个HotSpot VM的实例只有一份，被所有类共享，字符串常量池在JDK6及之前版本都是存放在方法区中，JDK7之后都是存放在堆中。



**特性**：串池中的字符串仅是符号，第一次用到时才会变成对象

​			利用串池的机制，来避免重复创建字符串对象

​			字符串变量拼接的原理是StringBuilder（1.8）

​			字符串常量拼接的原理是编译期优化

​		   可以使用intern方法，主动将串池中还没有的字符串对象放入串池：

​			 1.8将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池，会把串池中的对象放回

​			 1.6将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有会把此对象复制一份，放入串池，会				把串池中的对象放回



StringBuilder调优：

​	调整 -XX:StringTableSize=桶个数

​	考虑将字符串对象是否入池



**面试题**：String s1 = "a";

​				String s2 = "b";

​				String s3 = "ab";

​				String s4 = s1 + s2;

​				String s5 = "a" + "b";

**解析**：s4其实底层调用new StringBuilder().append("a").append("b").toString();

​			而toString()方法是new String("ab");

​			尽管如此，s3的“ab”是在串池中的，而s4是最终是通过new String("ab")得到的，因此s4是在堆中的，

​			所以s3  != s4。

​			对于s5，javac在编译期间的优化，结果已经在编译期确定为"ab"，因此s5 == s3



**面试题**：String x = "ab"	

​				String s = new String("a") + new String("b");

​				String s2 = s.intern(); 	//将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串														 //池。最终都会把串池中的字符串返回

​				System.out.println(s2 == x); //true

​				System.out.println( s == x); //false

**解析**： 首先在加载阶段，“ab”、“a”、“b”会直接加载进入串池，new String("a") + new String("b")语句则会将“a”、“b”、“ab”存入堆中，所以x是串池中的值，而s是堆中变量，由于innter()方法的特性，故s2是输入串池中的。



## 直接内存（Direct Memory）

定义：

- ​	常见于NIO操作时，用于数据缓冲区
- ​	分配回收成本较高，但读写性能高
- ​	不受JVM内存回收管理

未使用直接内存之前：

![image-20250624181026009](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624181026009.png)

使用直接内存之后：

![image-20250624181054450](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624181054450.png)

可见使用直接内存会减少一次无意义的数据写入



## GC（垃圾回收）

**如何判断对象可以回收**

1、引用计数法

![image-20250624165300178](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624165300178.png)

2、可达性分析算法（JVM采用）

- 不能被当成垃圾回收的对象称为根对象

- Java虚拟机中的垃圾回收器采用可达性分析来探索所有存活的对象
- 扫描堆中的对象，看是否能够沿着GC Root对象为起点的引用链找到该对象，找不到，表示可以回收

哪些对象可以作为GC Root？

System class，Thread，Native Stack，Busy Monitor。



什么是局部变量？什么是引用对象？

![image-20250624185257886](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624185257886.png)



**五种引用**

![image-20250624183109348](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624183109348.png)

**强引用**：

- 只有所有的GC Roots对象都不通过【强引用】引用该对象，该对象才能被垃圾回收

**软引用**：

- 仅有软引用引用该对象时，在垃圾回收后，内存不足时会再次发出垃圾回收，回收软引用对象

- 可以配合引用队列来释放软引用自身

- 软引用使用于实现内存敏感的缓存，例如图片的缓存，用户信息的缓存等

  ![image-20250625090825027](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625090825027.png)

前提：JVM设置堆内存 -Xmx20m

SoftReference是软引用对象，其引用byte数组时，而该byte数组又无其他强引用对象引用，因此，当内存不足时，所有没有被强引用对象引用的弱引用对象所引用的对象都会被垃圾回收。

当然软引用对象自身也占用内存，而list又强引用了该对象，一时间无法被主动回收，所以可以通过关联引用队列，当自己所关联的byte数组被回收时，自己也会进入引用队列中，引用队列调用方法获取无用的软引用对象并移除。

**弱引用**：

- 仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象

- 可以配合引用队列来释放弱引用自身

  ![image-20250625092112017](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625092112017.png)

弱引用使用与软引用相同，被弱引用对象引用的对象，在内存无论是否充足，都可能被回收，如果**触发Full GC**则会清除所有弱引用对象。

**注**：软引用对象与弱引用对象的引用如果被回收，其自身会被压入引用队列，方便做进一步的释放。

**虚引用**：

- 必须配合引用队列使用，只要配合ByteBuffer使用，当引用对象被回收时，会将虚引用入队，由Reference Handler线程调用虚引用相关方法释放内存

![image-20250624185728361](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250624185728361.png)

**终结器引用（FinalReference）**

- 无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队（被引用对象暂时没有被回收），再由Finalizer线程通过终结器引用找到被引用对象并调用它的finalize方法，第二次GC时才能回收被引用对象。



### 1. **垃圾回收算法**

1. **标记清除算法**

   当沿着GC Root对象引用链去查找，不存在于引用链的对象会被做标记，之后做清除操作，但不是将其内存进行一个清零操作，而是记录其起始地址和终止地址放在一个空闲地址列表，当需要做内存分配时，会在该地址列表查找合适的内存进行分配，**速度较快**，**缺点是会造成内存碎片**，当需要分配的内存过大时，没有一块内存碎片可以进行分配，资源利用率不高。

   ![image-20250625092912610](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625092912610.png)

2. **标记整理算法**

   该算法标记步骤与标记清除算法一致，随后会将所有内存进行一个整理，获取一个较大且完整的内存碎片，该算法**内存碎片利用率高**，但因牵扯到对象内存的移动，**GC效率低下**。

   ![image-20250625093805470](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625093805470.png)

3. **复制算法**

   复制算法是先将可回收对象进行标记，将不可回收对象移动到另一块相同大小的内存空间中，随后将可回收对象进行一个内存清除，并将两个GC Root对象相互指向彼此那块内存区，完成一次垃圾回收，**不会存在内存碎片**，该方法是典型的空间换时间的一种算法，**缺点即占用双倍的内存空间**，造成大量内存的浪费。

   ![image-20250625094312461](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625094312461.png)

![image-20250625094329931](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625094329931.png)

![image-20250625094351674](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625094351674.png)





### 2. **分代垃圾回收机制**

堆内存大的区域一共分为两块，一块为新生代，一块为老年代。新生代又划分为伊甸园、幸存区From、幸存区To。java中有的对象需要长时间使用，这些对象放在老年代中，用完即可丢弃的对象放在新生代中。老年代的垃圾回收很久才发生一次，新生代发生垃圾回收比较频繁。

![image-20250625101100017](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625101100017.png)



**整个分代垃圾回收机制过程详细讲解**：

- 新创建的对象会存放在新生代的伊甸园区域
- 新生代空间不足，触发minor gc，伊甸园和from存活的对象通过可达性算法将不可回收的对象使用copy复制到to中，存活的对象年龄+1并且交换from to

- minor gc会触发stop the world，暂停其他用户线程，等待垃圾回收结束，用户线程才会恢复运行
- 当对象寿命超过阈值（最大15，4bit）时，会晋升至老年代
- 当老年代空间不足，会先尝试触发minor gc，如果之后空间仍不足，那么会触发full gc，其stw时间更长。
- 如果新创建对象过大造成内存溢出，会直接存入进入老年代

![image-20250625103535900](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625103535900.png)



### 3. **相关JVM参数**

![image-20250625105117536](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625105117536.png)

### 4. 垃圾回收器

1. 串行

   - 单线程

   - 堆内存较小，适合个人电脑

     ![image-20250625114119151](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625114119151.png)

2. 吞吐量优先

   - 多线程

   - 堆内存较大，多核cpu

   - 让单位时间内，stw时间最短

     ![image-20250625114317773](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625114317773.png)

3. 响应时间优先

   - 多线程

   - 堆内存较大，多核cpu

   - 尽可能让单次stw时间最短

![image-20250625144858578](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625144858578.png)



**G1（Garbage First）**:

适用场景

- 同时注重吞吐量和低延迟，默认的暂停目标是200ms
- 超大堆内存，会将堆划分为多个大小相等的Region
- 整体上是标记+整理算法，两个区域之间是复制算法

相关JVM参数

-XX:+UseG1GC

-XX:G1HeapRegionSize=size

-XX:MaxGCPauseMillis=time



**G1垃圾回收阶段**

![image-20250625151504643](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625151504643.png)



**Young Collection**

G1的新生代的内存被划分为大小相等的区域，每个区域都可作为伊甸园、幸存区、老年代，

- 会STW

  

其中E代表的是伊甸园

![image-20250625151556922](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625151556922.png)

一段工作之后，当伊甸园区满了，执行minor gc，没有被垃圾回收的对象将被移动到幸存区![image-20250625152019594](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625152019594.png)

当幸存区的对象也比较多，会触发minor GC进行垃圾回收，当一些对象没有被回收且存活年龄到一定值，会被移动到老年代，其他没有被回收的对象会被移动到幸存区from，之后再进行交换。

![image-20250625152549281](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625152549281.png)



**Young Collection + CM**（并发标记）

- 在Young GC时会进行GC Root的初始标记
- 老年代占用堆空间比例达到阈值时，进行并发标记（不会STW），由下面的JVM参数决定

-XX:InitiatingHeapOccupancyPercent=percent(默认45%)

![image-20250625153140574](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625153140574.png)

**Mixed Collection**

会对E、S、O进行全面垃圾回收

- 最终标记（Remark）会STW
- 拷贝存活（Evacuation）会STW

-XX:MaxGCPauseMillis=ms

![image-20250625153846214](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625153846214.png)



### 5. Full GC

- SerialGC
  - 新生代内存不足发生的垃圾收集 - minor gc
  - 老年代内存不足发生的垃圾收集 - full gc

- ParallelGC

  - 新生代内存不足发生的垃圾收集 - minor gc

  - 老年代内存不足发生的垃圾收集 - full gc

- CMS
  - 新生代内存不足发生的垃圾收集  - minor gc
  - 老年代内存不足
- G1 
  - 新生代内存不足发生的垃圾收集  - minor gc
  - 老年代内存不足





## 类加载与字节码技术

1. 类文件结构
2. 字节码指令
3. 编译期处理
4. 类加载阶段
5. 类加载器
6. 运行期优化

![image-20250625164022581](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625164022581.png)



图解运行流程：

1. 原始代码

![image-20250625171239934](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171239934.png)

编译后的字节码文件：

![image-20250625171318102](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171318102.png)

![image-20250625171428564](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171428564.png)

![image-20250625171450726](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171450726.png)

![image-20250625171510923](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171510923.png)

3. 常量池载入运行时常量池

   ![image-20250625171548765](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171548765.png)

4. 方法字节码载入方法区

![image-20250625171811775](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171811775.png)

5. main线程开始运行，分配栈帧内存

![image-20250625171851261](C:\Users\16225\AppData\Roaming\Typora\typora-user-images\image-20250625171851261.png)
