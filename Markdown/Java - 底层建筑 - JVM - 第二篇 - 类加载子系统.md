###  Java - 底层建筑 - JVM - 第二篇 - 类加载子系统

#### JVM内存结构

![JVM详细图](images/JVM详细图.png)

#### 类加载器子系统作用

- 类加载器子系统负责从文件系统或者网络中加载Class文件，Class文件在文件开头有特定的文件标识。
- ClassLoader只负责class文件的加载，至于它是否可以运行，则由ExectionEngine决定。
- 加载的类信息存放于一块成为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）

#### 类加载器ClassLoader角色

例子:

![类加载器ClassLoader例子](images/类加载器ClassLoader例子.png)

1. class file存在与本地硬盘上，可以理解为设计师画在纸上的模板，而最终这个模板在执行时是要加载到JVM当中来根据这个文件实例化出N个一摸一样的实例。
2. class file加载到JVM中，被称为DNA元数据模板，放在方法区。
3. 在.class文件--->JVM--->最终成为元数据模板，此过程就要一个运输工具（类加载器Class Loader）,扮演一个快递员的角色。

#### 类的加载过程

![类的加载过程2](images/类的加载过程2.png)

![类的加载过程](images/类的加载过程.png)

##### 一、加载

1. 通过一个类的全限定名获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. **在内存中生成一个代表这个类的java.lang.Class对象**，作为方法区这个类的各种数据的访问入口

###### 加载.class文件的方式

- 从本地系统中直接加载
- 通过网络获取，典型场景：Web Applet
- 从zip压缩包中获取，成为日后jar、war格式的基础
- 运行时计算生成，使用最多的是：动态代理技术
- 有其他文件生成，典型场景：JSP应用
- 从专有数据库中提取.class文件，比较少见
- 从加密文件中获取，典型的防Class文件被反编译的保护措施

##### 二、链接

###### 验证（Verify）:

- 目的在于确保Class文件中的字节流中保函信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全。
- 主要包括四种验证：文件格式验证，元数据验证，字节码验证，符号引用验证。

###### 准备（Prepare）:

- 为类变量分配内存并且设置该类变量的默认初始值，即零值。
- **这里不包含final修饰的static,因为final在编译的时候就会分配了，准备阶段会显示初始化**。
- **这里不会为实例变量分配初始化**，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。

###### 解析（Resolve）:

- 将常量池内的符号引用转换为直接引用的过程。
- 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行。
- 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或者一个间接定位到目标的句柄。
- 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methidref_info等。

``

```java
public class HelloApp {

    private static int a = 1;
    //prepare准备环节会赋值a = 0 ---初始值
    //initial初始化环节会赋值a = 1;
    //数据类型不同，初始值不同

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```

##### 三、初始化

- 初始化阶段就是执行类构造器方法<clinit>()的过程。
- 此方法不需要定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。
- 构造器方法中指令按语句在源文件中出现的顺序执行。
- <clinit> ()不同于类的构造器。（关联：构造器是虚拟机视角下的<init>()）
- 若该类具有父类，JVM会保证子类的<clinit>()执行前，父类的<clinit>()已经执行完毕。
- 虚拟机必须保证一个类的<clinit>()方法在多线程下被同步加锁。

``

```java
public class StackTest {
    private static int a = 1;
    static {
        a = 3;
    }
    public static void main(String[] args) {
        System.out.println(StackTest.a);
    }
}
```

![初始化](images/初始化.png)

​							<!--<clinit>（）只有在类中有static类型变量或静态代码块时才会生成。-->

​							<!--任何一个类声明后，内部至少存在一个类的构造器--<init>()-->

```java
public class ClinitTest {
    static class Father{
        public static int A = 1;
        static {
            A = 2;
        }
    }
    static class Son extends Father{
        public static int B = A;
    }

    public static void main(String[] args) {
        //加载Father类，其次加载Son类
        System.out.println(Son.B);
    }
}

```

![获取父类的值](images/获取父类的值.png)

<!--获取类中的值前先获取其父类的值：保证子类的<clinit>()执行前，父类的<clinit>()已经执行完毕-->

#### 类加载器的分类

- JVM支持两种类型的类加载器，分别为引导类加载器（Bootstrap ClassLoader）和**自定义类加载器（User-Defined ClassLoader）**。
- 从概念上讲，自定义类加载器一般指的是程序中有开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是**将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。**
- 无论类加载器的类型如何划分，在程序中我们最常见的类加载器始终只有3个，如下：

![类加载器](images/类加载器.png)

<!--这里的关系为包含关系-->

```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //获取其上层:扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@1b6d3586

        //获取其上层:获取不到引导类加载器
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        System.out.println(bootstrapClassLoader);//null

        //对于用户自定义类来说:使用系统类加载器加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //string类使用引导类加载器加载---->java核心类库都是使用引导类加载器加载
        ClassLoader stringClassLoader = String.class.getClassLoader();
        System.out.println(stringClassLoader); //null
    }
}
```

- ##### 启动类加载器（引导类加载器，Bootstrap ClassLoader）

1. 这个类加载器使用C/C++语言实现的，嵌套在JVM内部
2. 它用来假爱Java的核心库（JAVA_HOME/jre/lib/rt.jar、resources.jar或sun.boot.class.path路径下的内容），用于提供JVM自身需要的类
3. 并不继承自java.lang.ClassLoader,没有父加载器
4. 加载扩展类和应用程序类加载器，应制定为他们的父类加载器
5. 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

- ##### 扩展类加载器（Extension ClassLoader）

1. java语言编写，由sun.misc.Launcher$ExtClassLoader实现
2. 派生于ClassLoader类
3. 父类加载器为启动类加载器
4. 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载

- ##### 应用程序类加载器（系统类加载器，AppClassLoader）

1. java语言编写，由sun.misc.Lanucher$AppClassLoader实现
2. 派生于ClassLoader类
3. 父类加载器为扩展类加载器
4. 它负责加载环境变量classpath或系统属性java.class.path指定路径下的类库
5. **该类加载时程序中默认的类加载器**，一般来说，Java应用的类都是由它来完成加载
6. 通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

```java
public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("********启动类加载器***********");
        //获取BootstrapClassLoader能够加载的api的路径
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        for (URL element:urls){
            System.out.println(element.toExternalForm());
        }
        //从上面的路径中选取任意一个类，来看看他的类加载器是什么
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);//null


        System.out.println("*********扩展类加载器*************");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path : extDirs.split(";")){
            System.out.println(path);
        }

        //从上面的路径中选取任意一个类，来看看他的类加载器是什么:扩展类加载器
        ClassLoader classLoader1 = ECDHKeyAgreement.class.getClassLoader();
        System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@677327b6
    }
}
```

```java
********启动类加载器***********
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/resources.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/rt.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/sunrsasign.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/jsse.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/jce.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/charsets.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/lib/jfr.jar
file:/C:/Users/14667/.jdks/corretto-1.8.0_302/jre/classes
null
*********扩展类加载器*************
C:\Users\14667\.jdks\corretto-1.8.0_302\jre\lib\ext
C:\WINDOWS\Sun\Java\lib\ext
sun.misc.Launcher$ExtClassLoader@677327b6
```

- ##### 用户自定义类加载器

1. 在Java的日常应用程序开发中，类加载器几乎是由上述3种类加载器相互配合执行的，在必要时，我们还可以自定义类加载器，来制定类的加载方式。

2. ###### 为什么要自定义类加载器？

   - 隔离加载类
   - 修改类加载的方式
   - 扩展加载源
   - 方式源码泄露

3. ###### 用户自定义加载器实现步骤

   - 开发人员可以通过继承抽象类java.lang.ClassLoader类的方式，实现自己的类加载器，以满足一些特殊的需求
   - 在JDK1.2之前，自定义类加载器时，总会去继承ClassLoader类并重写loadClass()方法，从而实现自定义类加载器，但是在JDK1.2之后，已不再建议用户去覆盖loadClass()方法，而是建议把自定义的类加载逻辑卸载findClass()方法中
   - 在编写自定义类加载器时，如果没有太古复杂的需求，可以直接继承URLClassLoader类，这样就可以避免自己去编写findClass()方法及其获取字节码流的方式，使自定义类加载器编写更加简洁

   ##### 获取ClassLoader的途径

   一、获取当前类的ClassLoader: class.getClassLoader()

   二、获取当前线程上下文的ClassLoader: Thread.currentThread().getContextClassLoader()

   三、获取系统的ClassLoader: ClassLoader.getSystemClassLoader()

   四、获取调用者的ClassLoader: DriverManger.getCallerClassLoader()

#### 双亲委派机制

- Java虚拟机对class文件采用的时**按需加载**的方式，也就是说当需要使用该类时才会将它的class文件加载到内存中生成class对象。而且加载某个类的class文件时，Java虚拟机采用的时**双亲委派模式**，即把请求交由父类处理，它是一种任务委派模式。
- **工作原理**
  1. 如果一个类加载器收到了类加载请求，并不会自己先去加载，而是把这个请求委托给父类的加载器去执行；
  2. 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器；
  3. 如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。

![双亲委派机制](images/双亲委派机制.png)

- 接口是由引导类加载器加载，而接口的实现类是由系统类加载器加载

```java
public class String {
    static {
        System.out.println("自定义的string");
    }

    public static void main(String[] args) {
        System.out.println("main");
        /**
         * 错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
         *    public static void main(String[] args)
         * 否则 JavaFX 应用程序类必须扩展javafx.application.Application
         */
    }
}

```

<!--自定义一个java.lang.String ,使用main方法，会找到引导类加载器，引导类加载器中不能处理main方法，所以报错-->

```java
public class ShkStart {
    public static void main(String[] args) {
        System.out.println("ShkStart ");
        /**java.lang.SecurityException: Prohibited package name: java.lang
         * Error: A JNI error has occurred, please check your installation and try again
         * Exception in thread "main"
         */
    }
}
```

<!--在自建的java.lang包下随意创建一个类，运行会由引导类加载器进行加载-->

- 双亲委派机制的优势
  1. 避免类的重复加载
  2. 保护程序安全，防止核心API被随意篡改
     - 自定义类：java.lang.String
     - 自定义类：java.lang.ShkStart <!--抛出异常 java.lang.SecurityException: Prohibited package name: java.lang-->

#### 沙箱安全机制

- 自定义String类，但是在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载JDK自带的文件(rt.jar包中的java\lang\String.class)，报错信息是没有main方法，就是因为加载的是 rt.jar 包中的String类。这样就可以保证对Java核心源代码的保护，这就是**沙箱安全机制**

#### 其他

- 在JVM中表示两个Class对象是否为同一个类存在的两个必要条件
  - 类的完整类名必须一致 - 也就是**全限定类名必须完全一致**
  - 加载这个类的ClassLoader(指的是ClassLoader实例对象)必须相同
- 换句话说，在JVM中，即使这两个类对象(Class对象)来源于同一个Class文件，被同一个虚拟机加载，只要加载它们的ClassLoader实例对象不同，那么这两个类对象也是不相等的
- JVM必须知道一个类型是由启动加载器的还是由用户加载器加载的。如果一个类型是由用户类加载器加载的，那么JVM会**将这个类加载器的一个引用作为类型信息的一部分保存在方法区内**，当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的加载器是相同的。

####  类的主动使用和被动使用

- 主动使用的七种情况
  - 创建类的实例
  - 访问某个类的或者接口的静态变量，或者对该静态变量赋值
  - 调用类的静态方法
  - 反射(Class.forName("xxx.xxx.xxx"))
  - 初始化一个类的子类
  - Java虚拟机启动的时候被标记为启动类的类
  - Java7开始支持动态的语言支持
    - java.lang.invoke.MethodHandle 实例的解析结果
    - REF_getStatic、REF_putStatic、REF_invokeStatic对应的类没有初始化，则初始化
- 除了以上的七种情况，其他使用Java类的方式都被看作是对**类的被动使用**，都**不会导致类的初始化**
