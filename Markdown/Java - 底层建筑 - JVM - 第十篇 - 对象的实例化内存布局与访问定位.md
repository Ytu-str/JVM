###  Java - 底层建筑 - JVM - 第十篇 - 对象的实例化内存布局与访问定位

####  对象的的实例化

![对象的的实例化](D:\JVM\导图\对象的的实例化.png)

- 实例代码

```java
public class ObjectTest {
    public static void main(String[] args) {
        Object o = new Object();
    }
}
```

- Javap命令反编译

```java
D:\JVMDemo\out\production\JVMDemo\com\company\chapter1>javap -v ObjectTest
警告: 二进制文件ObjectTest包含com.company.chapter1.ObjectTest
Classfile /D:/JVMDemo/out/production/JVMDemo/com/company/chapter1/ObjectTest.class
  Last modified 2021-11-19; size 462 bytes
  MD5 checksum 70358c95311c74a025719432d2869079
  Compiled from "ObjectTest.java"
public class com.company.chapter1.ObjectTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #2.#19         // java/lang/Object."<init>":()V
   #2 = Class              #20            // java/lang/Object
   #3 = Class              #21            // com/company/chapter1/ObjectTest
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lcom/company/chapter1/ObjectTest;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               o
  #16 = Utf8               Ljava/lang/Object;
  #17 = Utf8               SourceFile
  #18 = Utf8               ObjectTest.java
  #19 = NameAndType        #4:#5          // "<init>":()V
  #20 = Utf8               java/lang/Object
  #21 = Utf8               com/company/chapter1/ObjectTest
{
  public com.company.chapter1.ObjectTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/company/chapter1/ObjectTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class java/lang/Object
         3: dup
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: astore_1
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            8       1     1     o   Ljava/lang/Object;
}
SourceFile: "ObjectTest.java"
```

