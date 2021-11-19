###  Java - 底层建筑 - JVM - 第八篇 - 堆

![堆](D:\JVM\导图\堆.png)

- 每个**进程**拥有一个**JVM实例**
- 一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域
- Java堆区在JVM启动的时候即被创建。其空间大小也就确定了。是JVM管理的最大的一块内存空间
  - 堆内存的大小是可以调节的
- 《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但是在逻辑上他应该是连续的
- 所有的线程共享Java堆，在这里还可以划分为线程私有缓冲区（Thread Local Allocation Buffer ，TLAB）
- 《Java虚拟机规范》中对Java堆的描述是：所有的对象实例以及数组都对应当在运行时分配在堆上
  - 我要说的是：“几乎“所有的对象实例都在这里进行分配内存
- 数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置
- 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候，才会被移除
- 堆，是GC（Garbage Collection 垃圾收集器）执行垃圾回收的重点区域
- 测试代码 - 只用jdk自带的工具查看堆内存分析 **在jdk的安装目录下的bin下的jvisualvm.exe工具**

```java
/**
 * -Xms10m -Xmx10m
 */
public class HeapDemo {
    public static void main(String[] args) {
        System.out.println("Start....");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end....");
    }
}
```

```java
/**
 * -Xms20m -Xmx20m
 */
public class HeapDemo1 {
    public static void main(String[] args) {
        System.out.println("Start....");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end....");
    }
}

```

![VisualVM1](D:\JVM\导图\VisualVM1.png)

![VisualVM2](D:\JVM\导图\VisualVM2.png)

<!--可以看到设置的堆大小总共为10M-->

- 实例程序

```java
public class SimpleHeap {
    private int id;

    public SimpleHeap(int id){
        this.id = id;
    }

    public void show(){
        System.out.println("My ID is"+id);
    }

    public static void main(String[] args) {
        SimpleHeap s1 = new SimpleHeap(1);
        SimpleHeap s2 = new SimpleHeap(2);
        
        int[] arr = new int[10];
        Object[] arr1 = new Object[10];
    }
}
```

- 对应的内存图

![堆的核心概述](D:\JVM\导图\堆的核心概述.png)

![堆创建对象](D:\JVM\导图\堆创建对象.png)

####  堆的内存细分

- 现代的垃圾收集器大部分都基于分代收集理论设计。堆空间细分为

- Java7以及之前堆的内存逻辑分为三部分：新生区+养老区+

  永久区

  - Young Generation Space 新生区 Young/New
    - 又被划分为Eden区和Survivor区
  - Tenure Generation Space 养老区 Old/Tenure
  - Permanent Space 永久区 Perm

- Java8及以后堆内存逻辑上分为三部分：新生区+养老区+

  元空间

  - Young Generation Space 新生区 Young/New
    - 又被划分为Eden区和Survivor区
  - Tenure Generation Space 养老区 Old/Tenure
  - Meta Space 元空间 Meta

- **约定：新生区 <=> 新生代 <=> 年轻代 养老区 <=> 老年区 <=> 老年代 永久代 <=> 永久区**

#### 堆空间大小的设置

- Java堆用来存储Java对象实例，那么堆的大小在JVM启动的时候就已经设定好了，可以通过选项设置 "-Xmx"和"-Xms"来进行设置 **（设置的是年轻代+年老代的大小）**
  - -Xms：表示堆区的启始内存大小，等价于 -XX:InitialHeapSize
  - -Xmx：表示堆区的最大内存，等价于 -XX:MaxHeapSize
  - 说明：**-X是JVM的运行参数，ms是Memory Start的缩写**
- 一旦堆区的内存大小超过 -Xmx 所指定的最大内存时，将会抛出 OutOfMemoryError异常
- 通过会将 -Xms 和 -Xmx两个参数设置为相同的值，**为了能够在垃圾回收机制清理完堆区之后不需要重新分隔计算堆区的大小，从而提升性能**
- 默认情况下
  - 初始内存大小：物理电脑内存大小 / 64
  - 最大内存大小 物理电脑内存大小 / 4

```java
public class HeapSpaceInitial {
    public static void main(String[] args) {

        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        System.out.println("-Xms :" + initialMemory + "M");
        System.out.println("-Xmx :" +maxMemory + "M");

        System.out.println("系统内存大小为:" + initialMemory * 64.0 / 1024 + "G");
        System.out.println("系统内存大小为:" + maxMemory * 4.0 / 1024 + "G");
    }
}
```

```java
-Xms :213M
-Xmx :3152M
系统内存大小为:13.3125G
系统内存大小为:12.3125G

```

![堆区使用状况](D:\JVM\导图\堆区使用状况.png)

- 在代码中我们会发现计算的结果不一致，那是因为 s0 和 s1 区在进行计算的时候，只使用了一个
- 查询参数
  - jps / jstat -gc 进程id （见上图）
  - -XX:+PrintGCDetails （见下图）

```java
-Xms :575M
-Xmx :575M
系统内存大小为:35.9375G
系统内存大小为:2.24609375G
Heap
 PSYoungGen      total 179200K, used 9216K [0x00000000f3800000, 0x0000000100000000, 0x0000000100000000)
  eden space 153600K, 6% used [0x00000000f3800000,0x00000000f41001a0,0x00000000fce00000)
  from space 25600K, 0% used [0x00000000fe700000,0x00000000fe700000,0x0000000100000000)
  to   space 25600K, 0% used [0x00000000fce00000,0x00000000fce00000,0x00000000fe700000)
 ParOldGen       total 409600K, used 0K [0x00000000da800000, 0x00000000f3800000, 0x00000000f3800000)
  object space 409600K, 0% used [0x00000000da800000,0x00000000da800000,0x00000000f3800000)
 Metaspace       used 3311K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
```

####  OutOfMemoryError 举例

- 下面举例演示：Exception in thread "main" java.lang.OutOfMemoryError: Java heap space

```java
/**
 * VM args: -Xms10m -Xmx10m
 * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
 */
public class HeapOomCase {
    public static void main(String[] args) {
        List<OomObject> list = new ArrayList<>();
        while (true){
            list.add(new OomObject());
        }
    }
}
class OomObject{

}
```

####  年轻代和老年代

- 存储在JVM中的Java对象可以被划分为两类
  - 一类是生命周期毕竟短的瞬时对象。这类对象的创建和消亡都非常迅速
  - 另外一类对象的生命周期却非常长，在某些极端的情况下还能与JVM的生命周期保持一致
- Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（OldGen）
- 其中年轻代又可以分为Eden空间，Survivor0空间和Survivor1空间（有时也叫from区和to区）

![年轻代与老年代](D:\JVM\导图\年轻代与老年代.png)

- 下面的参数在开发中一般不会调整
  - Young：Old = 1：2
  - Eden：s0：s1 = 8：1：1
- 配置新生代老年代在堆结构的占比
  - 默认 -XX:NewRatio=2 表示新生代栈1，老年代占2 新生代占整个堆的 1/3
  - 可以修改 -XX:NewRatio=4 表示新生代栈1，老年代占4 新生代占整个堆的 1/5
  - 在Hotspot中，Eden空间和另外两个Survivor空间缺省占比为8：1：1
  - 当然开发人员可以通过选项 "-XX:SurvivorRatio"调整这个比例，如 -XX:SurvivorRatio=8
    - 但是默认情况下，我们看到的并不是 8:1:1 而是 6:1:1 如果像看到，就可以设置 VM参数 -XX:SurvivorRatio=8

- **几乎所有的Java对象都是在Eden区被new出来的**
- 绝大部分的Java对象的销毁都在新生代进行了
  - IBM公司的专门研究表明：新生代 80% 的对象都是”朝生夕死“的
- 可以使用 ”-Xmn“ 设置新生代的最大内存大小
  - 这个参数一般使用默认值即可

####  图解对象的分配过程

- 为新对象分配内存是一件非常严谨和复杂的任务，JVM的设计者们不仅仅需要考虑内存如何分配，在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收之后是否会在内存空间中产生内存碎片
  - new的对象先放在伊甸园区，此区没有大小限制
  - 当伊甸园区填满的时候，程序又需要创建新的对象，JVM的垃圾回收器将对伊甸园进行垃圾回收（Minor GC），将伊甸园区中的不不再被其他对象所引用的对象进行销毁，在加载新的对象放在伊甸园区
  - 然后将伊甸园区的幸存对象移动到幸存者0区
  - 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者01区域的，如果没有回收，就放到幸存者1区
  - 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区
    - **注意：只有Eden区满了之后才会触发YGC，而幸存者区满了不会触发YGC，但是会将Eden区和幸存者区一起回收**
  - 当一个对象经历了15次 Minor GC之后，就会放到养老区
    - 可以设置参数：-XX:MaxTenuringThreshold=进行设置
  - 在养老区，相对悠闲，当养老区的内存不足的时候，再次触发GC：Major GC 进行养老区的内存清理
  - 如果养老区执行了Major GC之后依旧无法进行对象的保存，就会产生OOM异常

![对象的分配过程1](D:\JVM\导图\对象的分配过程1.png)

![对象的分配过程2](D:\JVM\导图\对象的分配过程2.png)

![对象的分配过程3](D:\JVM\导图\对象的分配过程3.png)

**总结**

- **针对幸存者s0，s1区的总结：复制之后有交换，谁空谁是to**
- **关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不在永久区/元空间收集**

![对象分配特殊过程](D:\JVM\导图\对象分配特殊过程.png)

- 监控案例

```java
/**
 * -Xms600m -Xmx600m
 */
public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024*200)];

    public static void main(String[] args) {
       List<HeapInstanceTest> list = new ArrayList<>();
       while (true){
           list.add(new HeapInstanceTest());
           try {
               Thread.sleep(10);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
    }
}
```

![VisualVM对象的分配过程](D:\JVM\导图\VisualVM对象的分配过程.png)

####  常用的调优工具

- JDK命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- VisualVM
- Jprofiler
- Java Flight Recorder
- GC Viewer
- GC Easy

#### Minor GC、Major GC和Full GC

- JVM在进行GC的时候，并非每次都对上面三个内存区域一起回收的，大部分的时候回收都是指新生代
- 针对Hotspot VM的实现，它里面的GC按照回收区域又分为2种类型：一种是部分收集（Partial GC）一种是整堆收集（Full GC）
- 部分收集：不是完整的整个Java堆收集
  - 新生代收集（Minor GC 完全等价于 YGC）：只是在新生代的垃圾收集
  - 老年代收集（Major GC / Old GC）：只是对老年代的垃圾收集
    - 目前只有CMS GC 会有单独收集老年代的行为
    - **注意很多时候Major GC 和 Full GC 混淆使用，需要具体判别老年代回收还是整堆回收**
  - 混合收集（Mixed GC）：收集整个新生代和部分老年代的垃圾收集
    - 目前 只有G1 GC 会有这种行为
- 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集

#####  年轻代GC（Minor GC/YGC）的触发机制

- 当年轻代空间不足的时候，就会触发Minor GC，这里的年轻代满指的是 Eden区满，而不是S0或者S1满。Servivor满不会触发Minor GC （但是每次Minor GC 会请i年轻代的内存，包括 Eden/S0/S1）
- 因为Java对象大多都**具备朝生夕死**的特性，所以Minor GC 非常频繁，一般回收速度也很快，
- Minor GC 会引发STW（Stop The World），暂停其他用户线程，等待垃圾回收结束，用户线程才恢复执行

![对象分配图解](D:\JVM\导图\对象分配图解.png)

##### 老年代GC（Major GC / Full GC）的触发机制

- 指发生在老年代的GC，对象从老年代消失的时候，我们说”Major GC“ 或 ”Full GC“ 发生了
- 出现了Major GC ，经常会伴随至少一次的Minor GC（但是非绝对的，在Parallel Scavenge 收集器的收集策略里就有直接进行Major GC的策略选择过程）
  - 也就是在老年代空间不足的时候，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC
- Major GC 的速度会比Minor GC 慢十倍一样，STW的时间或更长
- 如果Major GC 之后，内存还不足，就报OOM了

#####  Full GC 触发机制（后面细讲）

- 触发Full GC的情况有有下面五种
  - 调用System.gc()的时候，系统建议执行Full GC，但是不是必然执行的
  - 老年代空间不足
  - 方法区空间不足
  - 通过Minor GC 后进入老年代的平均大小大于老年代的可用内存
  - 由Eden区、Survivor space0（From Space）区向Survivor space1（To Space）区复制的时候，对象大小大于 To Space的可用内存，则把该对象转存到老年代。且老年代 的可用内存大小小于该对象大小
  - **说明：Full GC 是开发或者调优种尽量避免的，这样暂停的时间会短一些**

#### 堆空间分代思想

- 为啥需要把Java堆分代？不分代就不能工作了吗？
- 经过研究，不同对象的生命周期不同，70%~99% 都是临时对象
  - 新生代：由Eden、两块大小相同的Survivor（又称from/to，s0/s1） 构成，to总为空
  - 老年代：存放新生代中经历多次GC依然存活的对象

![堆空间分代思想](D:\JVM\导图\堆空间分代思想.png)

![堆空间分代思想2](D:\JVM\导图\堆空间分代思想2.png)

- 其实不分代也是完全可以的，分代的唯一理由就是**优化GC性能**，如果没有分代，那么所有的对象都在一起。GC的时候，就会对整堆进行全局扫描，然而很多对象都是朝生夕死 的，如果分代的话，把这些对象聚集在一起，GC先回收这部分，就会节省很多空间和资源

#### 堆内存分配策略

- 对象提升（Promotion）规则
  - 如果对象在Eden出现并经过第一次Minor GC之后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor 区中每熬过一次MinorGC，年龄就增加1岁，当他的年龄增加到一定的程度（默认为15岁）就会晋升到老年代
  - 对象晋升老年代的年龄，可以通过 ： -XX:MaxTenuringThreshold 来设置
- 针对不同的年龄段的对象分配原则如下所示
  - 优先分配待Eden
  - 大对象直接分配到老年代
    - 尽量避免程序中出现过多的大对象
  - 长期存活的对象分配到老年代
  - 动态对象年龄判断
    - 如果Survivor区中的相同年龄的所有的对象的总和大于Survivor空间的一半，年龄大于或者等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄
  - 空间分配担保
    - -XX:HandlePromotionFailure

####  为对象分配内存：TLAB

- **为什么要有TLAB（Thread Local Allocation Buffer）**
  - 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
  - 由于对象实例的创建在JVM中非常频繁。因此在并发环境下从堆区中划分内存空间是线程不安全的
  - 为避免多个线程操作同一个地址，需要使用加锁等机制，进而影响分配速度
- **什么是TLAB**
  - 从内存模型而不是垃圾收集的角度，堆Eden区继续进行划分，**JVM为每个线程分配了一个私有缓存区域**，它包含在Eden区内
  - 多线程同时分配内存的时候，使用TLAB可以避免一系列的非线程安全问题。同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为 **快速分配策略**
  - 几乎所有的OpenJDK衍生出来的JVM都提供了TLAB设计

![TLAB](D:\JVM\导图\TLAB.png)

- TLAB的再说明

  - 尽管不是所有的对象实例都能在TLAB中成功分配内存，但是 **JVM确实把TLAB作为内存分配的首选**
  - 在程序中，开发人员可以通过选项 ”-XX:UseTLAB“ 设置是否开启TLAB空间
  - 默认情况下，TLAB空间的内存非常小，**仅仅占有整个Eden空间的1%**。当然我们可以通过选项 ”-XX:TLABWasteTargetPercent“设置TLAB空间所占用的Eden空间的百分比大小
  - 一旦对象在TLAB空间分配内存失败的时候，JVM就会尝试使用 **加锁机制**来确保数据操作的原子性，从而在Eden区中直接分配内存

  **对象的分配过程**

![对象的分配过程](D:\JVM\导图\对象的分配过程.png)

####  小结堆空间的参数设置

- -XX:+PrintFlagsInitial 查看所有参数的默认初始值

- -XX:+PrintFlagsFinal 查看所有参数的最终值（可能会存在修改，不再是初始值）

  ​		具体查看某个参数的指令：jsp :查看当前运行中的进程

  ​		jinfo -flag SurvivorRatio 进程id

- -Xms：初始堆空间内存 （默认大小为 物理内存空间/64）

- -Xmx：最大堆空间内存（默认大小为 物理内存空间/4）

- -Xmn：设置新生代的大小（初始值和最大值）

- -XX:NewRatio 配置新生代与老年代在堆结构的占比

- -XX:SurvivorRatio 设置新生代中Eden和S0、S1空间的比例

- -XX:MaxTenuringThreshold 设置新生代垃圾打最大年龄

- -XX:+PrintGCDetails 输出详细的GC处理日志

- -XX:PrintGC 打印GC的简要信息 -verbose:gc

- -XX:HandlePromotionFailure 是否设置空间分配担保

------

- 在发生Minor GC之前，虚拟机会**检查老年代最大可用的连续空间是大于新生代所有对象的总空间**

  - 如果大于，则此此Minor GC 是安全的

  - 如果小于，则虚拟机会查看 -XX:HandlePromotionFailure 设置值是否允许担保失败

    - 如果HandlePromotionFailure=true，那么会继续

      检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小

      - 如果大于，则尝试进行一次Minor GC ，但这次Minor GC 仍然是有风险的
      - 如果小于，则改为一次Full GC

    - 如果HandlePromotionFailure=false 则改为进行一次 Full GC

- 在JDK6 Update24之后，HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，观察OpenJDK中的源码变化，虽然源码中定义了HandlePromotionFailure参数，但是在代码中已经不会使用它。在此版本之后规则变为 **只要老年代的连续空间大于新生待对象总大小或者历次晋升的平均大小就会进行Minor GC**，否则就执行Full GC

#### 堆是分配对象存储的唯一选择吗？

- 在《深入理解Java虚拟机》中关于Java堆内存中有这样的一段描述
  - 随着JIT编译器的发展与**逃逸分析技术**逐渐成熟，**栈上分配、标量替换优化技术**将会导致一些微妙的变化，所有的对象都分配到堆上就渐渐变得不是那么绝对了
  - 在Java虚拟机中。对象是在Java堆中分配内存的，这是一个普遍的常识。但是有一种特殊情况，那就是 **如果经过逃逸分析（Escape Analysis）后发现。一个对象并没有逃逸出方法的话，那么就有可能被优化成栈上分配**，这样就无需再堆上分配内存，也无需进行垃圾回收了，这是最常见的对外存储技术。
  - 此外，前面提到的基于OpenJDK深度定制的TaoBaoVM，其中创新的GCIH（GC Invisible heap）技术实现 off-heap，将生命周期比较长的Java对象从heap移到heap外，并且GC不能管理GCIH内部的对象，以此达到降低GC的回收频率和提升GC的回收效率的目的

**逃逸分析概述**

- 如何将堆上的对象分配到栈，需要使用逃逸分析手段
- 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流的分析算法
- 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的适用范围从而觉得是否要将这个对象分配到堆上
- 逃逸分析的基本行为就是分析对象的动态作用域
  - 当一个对象在方法中被定义之后，对象只在方法内部使用。则认为没有发生逃逸
  - 当一个对象在方法中被定义后，它被外部的方法所引用，则认为发生逃逸，例如调用参数传递到其他方法中
- 快速判断是否发生逃逸分析
  - 就看new的对象是否可以在方法外部调用
  - 简而言之，是否只是在方法内部使用此对象
- JDK 6u23版本之后，Hotspot中默认开启了逃逸分析
- 如果使用较早的版本
  - 选项 ”**-XX:+DoEscapeAnalysis**“显示开启
  - 选项 ”-XX:+PrintEscapeAnalysis“查看逃逸分析筛选
- **结论：开发中能使用局部变量的，就不要在方法外进行定义**

```java

public class EscapeAnalysis {
    

    private  EscapeAnalysis obj;
    /**
     * 方法返回EscapeAnalysis对象，发生逃逸
     * @return
     */
    public EscapeAnalysis getInstance(){
        return obj == null ? new EscapeAnalysis() : obj;
    }

    /**
     * 为成员属性赋值，发生逃逸
     */
    public  void setObj(){
        this.obj = new EscapeAnalysis();
    }

    /**
     * 对象的作用域仅在当前方法有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }

    /**
     * 引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
    }
}
```

####  逃逸分析：代码优化

- 使用逃逸分析，编译器可以堆代码做以下优化
  - **栈上分配**，将堆分配转化为栈分配，如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
  - **同步策略**，如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
  - **分离对象或者标量替换**。有的对象可能不需要作为一个联系的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中

#####  栈上分配

- JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成之后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收，这样就无需进行垃圾回收了
- 常见的栈上分配场景
  - 在逃逸分析中已经说明了。分别是给成员变量赋值、方法返回值、实例引用传递

#####  同步策略 - 锁消除

- 线程之间同步的代价是非常高的，同步的后果是降低并发性和性能
- 在动态编译的同步块的时候，JIT编译器可以借助逃逸分析来 **判断同步块所使用的锁对象是否只能被一个线程访问而没有被发布到其他线程**。如果没有，那么JIT编译器在这个同步块的时候就会取消堆这部分代码的同步。这样就可以大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫**锁消除**

```java
public void f(){
	Object hollis = new Onject();
	synchronized(hollis){
		System.out.println(hollis);
	}
}
//代码中对hollis这个对象加锁，但是hollis对象的生命周期只在f()方法中，不会被其他线程所访问到，所以在JIT编译阶段就会被优化掉。优化成：
public void f(){
    Object hollis = new Onject();
    System.out.println(hollis);
}
```

#####  分离对象或者标量替换

- **标量（Scalar）**是指一个无法在分解成更小的数据的数据，Java中的原始数据类型就是标量
- 相对的，哪些还可以分解的数据就叫做 **聚合量（Aggregate）**，Java中的对象就是聚合量，因为可以分解为其他的聚合量和标量。
- 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个成员变量来替代。这个过程就是**标量替换**

- 代码演示

```java
public class Test {
    public static void main(String[] args) {
        alloc();
    }

    public static void alloc() {
        Point point = new Point(1, 23);
        System.out.println(point.x);
        System.out.println(point.y);

    }
}

class Point {
    public int x;
    public int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
}
```

- 经过标量替换之后是这个样子

```java
public class Test {
    public static void main(String[] args) {
        alloc();
    }

    public static void alloc() {
        int x = 1;
        int y = 23;
        System.out.println(x);
        System.out.println(y);

    }
}

class Point {
    public int x;
    public int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

- 可以看到，Point这个聚合变量经过逃逸分析之后，发现他并没有逃逸，就会被替换为两个聚合量了
- 标量替换有什么好处呢？就是可以大大减少堆内存的占用，因为一旦不需要创建对象，那么就不再需要分配堆内存了
- 标量替换为栈上分配提供了很好的基础。
- 开启标量替换
  - 参数：-XX:+EliminateAllocations 开启了标量替换（默认打开），允许将对象打散分配在栈上。

##### 总结

- 关于逃逸分析的论文在1999就发表了，但是在JDK6才实现，技术不是很成熟
- 根本原因就是 **无法保证逃逸分析的性能消耗一定能高于他的消耗，虽然经过逃逸分析可以做标量替、栈上分配、和锁消除，但是逃逸分析自身也需要进行一系列复杂的分析。也是一个相对耗时的过程**
- 虽然技术不是很成熟，但是他是 **即时编译器优化技术中一个十分重要的手段**

