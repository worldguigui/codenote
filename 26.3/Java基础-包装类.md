# Java基础-包装类

## 一.Java的八种基本数据类型

|类型|字节数|默认值|包装类|范围|
|:-----|:-----|:-------|:--------|:------|
| byte    | 1      | 0        | Byte      | -128 ~ 127 |
| short   | 2      | 0        | Short     | -32768 ~ 32767 |
| int     | 4      | 0        | Integer   | -2^31 ~ 2^31-1 |
| long    | 8      | 0L       | Long      | -2^63 ~ 2^63-1 |
| float   | 4      | 0.0f     | Float     | 单精度浮点数 |
| double  | 8      | 0.0d     | Double    | 双精度浮点数 |
| char    | 2      | '\u0000' | Character | 0 ~ 65535 |
| boolean | ~1     | false    | Boolean   | true / false |

Java中的基本类型主要存储在栈（Stack）或堆（Heap）中的对象内部，取决于它的使用场景。

JVM中有一个名为“虚拟机栈”的区域，负责分配每个线程运行时所需要的内存，先进后出

每个栈由多个栈帧（frame）组成，对应着每次方法调用时所占用的内存。

每个线程同一时刻只能有一个活动栈帧，对应着当前正在执行的那个方法。

栈帧随着方法的结束而自动销毁，无需GC。

### 1.存储在栈中
```java
void method() {
    int a = 10;
}
```
局部变量a存在于栈帧（Stack Frame）的局部变量表中。

对于 int 等基本类型，值直接存储在局部变量表中；对于引用类型，存储的是对象地址（引用），对象本身仍在堆中。

### 2.存储在堆中
```java
class User {
    int age = 18;
}
```
Java中的对象存储在堆中，作为实例字段的age在对象中，自然也存储在堆中。

### 3.存储在元空间中
对于static变量，在JDK8及以后，其字段定义信息属于类元数据，存储在元空间中。

而变量的实际值在类加载完成后就已经存在，存储在堆中的Class对象实例中，不依赖于对象实例的创建。

## 二.包装类
包装类，就是基本数据类型对应的引用数据类型。

### 1.包装类的缓存
包装类在初始化时，会创建一个缓存数组，用于缓存一定范围内的对象，以减少对象创建。

比如说，Integer类在初始化时，会创建一个缓存数组（IntegerCache），用于缓存一定范围内的对象（默认-128~127）。

创建对象时，如果值在这个区间内，就会直接引用，避免重复创建。

```java
Integer i1 = Integer.valueOf(127);
Integer i2 = Integer.valueOf(127);
System.out.println(i1 == i2);
// true

Integer i3 = Integer.valueOf(128);
Integer i4 = Integer.valueOf(128);
System.out.println(i3 == i4);
// false

Integer i5 = Integer.valueOf(127);
Integer i6 = Integer.valueOf(127);
System.out.println(i5 == i6);
// false
// 现在不推荐直接new，因为直接在堆中创建，无法利用内部的缓存机制。
```

### 2.自动拆箱装箱
自动装箱：把基本数据类型自动变成其对应的包装类。

自动拆箱：把包装类自动变成其对应的基本数据类型。

自动拆装箱是Java编译时的语法糖，运行时表现为方法调用。

```java
// 自动装箱
Integer i = 10;

// 自动拆箱
int a = i;
```

## 三.==和equals
对于基础数据类型，==比较的是具体数值的大小。

对于引用数据类型，==比较的是它们在内存中的地址。

所有的类都可以重写equals方法，实现自己的比较逻辑；如果没重写，则默认比较地址。

如果重写equals方法，通常也需要重写hashCode方法，以保证在HashMap等集合中的行为一致。
