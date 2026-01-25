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

## 二.BitSet源码
### 1.设计思想
BitSet 使用 long[] 来存储数据，1 个 long 占据 64 bit ，可以用来表示 64 个 0 或 1。<br/>
|整型数组|实际字面量大小|表示范围|逻辑上表示的01序列|
|:-----|:-----|:-----|:-----|
|words[2]|4611686018427387904|191-128|10000000_00000000_00000000_00000000_00000000_00000000_00000000_00000000|
|words[1]|2147483647|127-64|00000000_00000000_00000000_00000000_01111111_11111111_11111111_11111111|
|words[0]|512|63-0|00000000_00000000_00000000_00000000_00000000_00000000_00000000_11111111|

该表表示BitSet的191位、94\~64位、7\~0位为1。<br/>

### 2.静态量
```
private static final int ADDRESS_BITS_PER_WORD = 6;
 
private static final int BITS_PER_WORD = 1 << ADDRESS_BITS_PER_WORD;

private static final int BIT_INDEX_MASK = BITS_PER_WORD - 1;

/* Used to shift left or right for a partial word mask */
private static final long WORD_MASK = 0xffffffffffffffffL;

/* use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = 7997698588986878753L;
```
|静态量|作用|
|:-----|:-----|
|ADDRESS_BITS_PER_WORD|每个word需要多少地址位。|
|BITS_PER_WORD|每个word的位数，1 << 6 = 64，即long有64位|
|BIT_INDEX_MASK|位索引掩码 - 63 (0x3F)，用于计算位在words内的位置。|
|WORD_MASK|全1的word掩码，64位全为1 (0xffffffffffffffffL)|
|serialVersionUID|序列化版本UID。|

<br/>
对于 index=150 来说，计算它在words内的位置，150 / 64 = 2，即 words[2] 中的 150 % 64 = 22 位上。<br/>
可以用 BIT_INDEX_MASK & 150 来计算，这实际上就是取模运算。<br/>

### 3.变量
```
/**
 * The internal field corresponding to the serialField "bits".
 */
private long[] words;

/**
 * The number of words in the logical size of this BitSet.
 */
private transient int wordsInUse = 0;

/**
 * Whether the size of "words" is user-specified.  If so, we assume
 * the user knows what he's doing and try harder to preserve it.
 */
private transient boolean sizeIsSticky = false;
```
|变量|作用|
|:-----|:-----|
|words|存储数据的long类型数组|
|wordsInUse|数组的逻辑长度，即words实际使用的数量。该索引及其之后的word都是0，这部分数据是无用的。|
|sizeIsSticky|标记数组大小是否用户指定。|

### 4.构造方法
```
public BitSet() {
    initWords(BITS_PER_WORD);
    sizeIsSticky = false;
}

/**
 * Creates a bit set whose initial size is large enough to explicitly
 * represent bits with indices in the range {@code 0} through
 * {@code nbits-1}. All bits are initially {@code false}.
 *
 * @param  nbits the initial size of the bit set
 * @throws NegativeArraySizeException if the specified initial size
 *         is negative
 */
public BitSet(int nbits) {
    // nbits can't be negative; size 0 is OK
    if (nbits < 0)
        throw new NegativeArraySizeException("nbits < 0: " + nbits);

    initWords(nbits);
    sizeIsSticky = true;
}

private void initWords(int nbits) {
    words = new long[wordIndex(nbits-1) + 1];
}

/**
 * Creates a bit set using words as the internal representation.
 * The last word (if there is one) must be non-zero.
 */
private BitSet(long[] words) {
    this.words = words;
    this.wordsInUse = words.length;
    checkInvariants();
}

/**
 * Given a bit index, return word index containing it.
 */
private static int wordIndex(int bitIndex) {
    return bitIndex >> ADDRESS_BITS_PER_WORD;
}
```
BitSet提供了三种构造方法。<br/>
1.默认的无参构造，words数组的长度为1。<br/>
2.参数为nbits的有参构造，words数组的长度由nbits决定。<br/>
3.参数为words的有参构造，this.words = words，直接赋值。<br/>

方法 wordIndex(int bitIndex)，根据 bitIndex 计算它的绝对索引。<br/>

### 5.查询方法
```
/**
 * Returns the value of the bit with the specified index. The value
 * is {@code true} if the bit with the index {@code bitIndex}
 * is currently set in this {@code BitSet}; otherwise, the result
 * is {@code false}.
 *
 * @param  bitIndex   the bit index
 * @return the value of the bit with the specified index
 * @throws IndexOutOfBoundsException if the specified index is negative
 */
public boolean get(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

    checkInvariants();

    int wordIndex = wordIndex(bitIndex);
    return (wordIndex < wordsInUse)
        && ((words[wordIndex] & (1L << bitIndex)) != 0);
}
```
通过 wordIndex(bitIndex) 计算出绝对索引，然后 1L << bitIndex 计算相对索引，最后相与得到相对索引上的值。<br/>

在 Java 中，对于 int：x << y 等价于 x << (y & 0x1F)（使用低5位）；<br/>
对于 long：x << y 等价于 x << (y & 0x3F)（使用低6位）；<br/>
取 bitIndex 的低6位，相当于得到了 bitIndex % 64 的结果。<br/>

### 6.wordInUse
该变量用来记录数组的逻辑长度。<br/>

数组的逻辑长度可能会在这几种情况下改变，1.set()时；2.clear()时；3.flip()时；4.范围操作时。<br/>

|操作类型|wordsInUse的变化|处理方法|
|:-----|:-----|:-----|
|set()|只可能增加|expandTo()|
|clear()|只可能减少|recalculateWordsInUse()|
|flip()|可能增减|recalculateWordsInUse()|
|范围操作（范围set、clear、flip）|可能增减|recalculateWordsInUse()|

关于 wordsInUse 增减很好理解。<br/>
set 置一、 clear 置零、flip 取反。<br/>
如果在逻辑长度内置一，wordsInUse 不改变；若在逻辑长度之外置一，则可能增加。<br/>
如果在逻辑长度外置零，wordsInUse 不改变；若在逻辑长度之内置零，则可能减少。<br/>
如果是取反和范围操作，则要视情况而定。<br/>
### 7.set方法
```
/**
 * Sets the bit at the specified index to {@code true}.
 *
 * @param  bitIndex a bit index
 * @throws IndexOutOfBoundsException if the specified index is negative
 * @since  1.0
 */
public void set(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

    int wordIndex = wordIndex(bitIndex);
    expandTo(wordIndex);

    words[wordIndex] |= (1L << bitIndex); // Restores invariants

    checkInvariants();
}

/**
 * Ensures that the BitSet can accommodate a given wordIndex,
 * temporarily violating the invariants.  The caller must
 * restore the invariants before returning to the user,
 * possibly using recalculateWordsInUse().
 * @param wordIndex the index to be accommodated.
 */
private void expandTo(int wordIndex) {
    int wordsRequired = wordIndex+1;
    if (wordsInUse < wordsRequired) {
        ensureCapacity(wordsRequired);
        wordsInUse = wordsRequired;
    }
}

/**
 * Ensures that the BitSet can hold enough words.
 * @param wordsRequired the minimum acceptable number of words.
 */
private void ensureCapacity(int wordsRequired) {
    if (words.length < wordsRequired) {
        // Allocate larger of doubled size or required size
        int request = Math.max(2 * words.length, wordsRequired);
        words = Arrays.copyOf(words, request);
        sizeIsSticky = false;
    }
}
```

