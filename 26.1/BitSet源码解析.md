# BitSet源码解析
最近在Redis的面试题学习中，了解到缓存穿透的一种解决方案--布隆过滤器（BloomFilter），其中的一种底层实现就是 BitSet。<br/>

在进行源码分析前，先了解一下序列化 Serialization 和关键字 Transient 。<br/>

## 一.序列化与反序列化
如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。<br/>
简单说来，序列化指的就是将数据结构、对象转换为可以存储或传输的形式，通常是二进制文本流，也可以是XML、JSON等文本格式。<br/>
反序列化就是将这些文本数据重新转换回对象。<br/>

对于 Java 来说，实现 JDK 自带的序列化，可以让类实现 Serializable 接口，告诉 JVM 该类可以被序列化。<br/>
Serializable 接口是一个标记接口，没有方法。<br/>

在类中一般会手动指定 serialVersionUID ，作为类的唯一标志，序列化时保存在文本流中；反序列化时，如果 UID 不匹配，则转化失败。<br/>

序列化只针对对象，static修饰的变量不会被序列化。<br/>
serialVersionUID 是特例，序列化时会保存在文本流中。<br/>
除了静态变量以外，还可以通过关键字 transient ，来排除修饰的变量，使其不会被序列化。<br/>

示例代码：
```
import java.io.*;

// 实现Serializable接口，标记该类可以被序列化
class Person implements Serializable {

    private transient String name;
    private int age;

    private static final long serialVersionUID = 1l;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString(){
        return "name: " + this.name + "\nage: " + this.age;
    }
}

public class SerializableDemo {
    public static void main(String[] args) {
        Person person = new Person("张三", 25);

        try {
            // 1. 序列化：对象 -> 字节流
            FileOutputStream fos = new FileOutputStream("person.ser");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(person);  // 写入对象
            oos.close();

            System.out.println("对象已序列化到文件");

            // 2. 反序列化：字节流 -> 对象
            FileInputStream fis = new FileInputStream("person.ser");
            ObjectInputStream ois = new ObjectInputStream(fis);
            Person restoredPerson = (Person) ois.readObject();  // 读取对象
            ois.close();

            System.out.println("从文件恢复的对象: " +
                    restoredPerson.getClass().getName());
            System.out.println(restoredPerson.toString());

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 二.BitSet源码-设计思想
BitSet 使用 long[] 来存储数据，1 个 long 占据 64 bit ，可以用来表示 64 个 0 或 1。<br/>
|:-----|:------|
|高64位|neirong|
|:------|:------|
