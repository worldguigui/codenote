# Java快速读写

在算法竞赛中，出于读写性能考虑，往往会使用C++来进行解答。

对于Java来讲，由于严格的格式匹配、异常处理和臃肿的包装，Scanner和System.out.println的读写效率极其低下。

这对于大规模的输入输出相当不友好——太慢了！

对于1e6的数据量
```java
// 写
System.out.println 耗时: 3544ms
PrintWriter 耗时: 298ms

// 读
FastReaderPlus 耗时: 23ms
FastReader 耗时: 111ms
Scanner 耗时: 475ms

```

## 一.解决方案
```java

public class LeetCode {

    public static void main(String[] args) {
        String filename = "testdata.txt";
        long start = 0l;
        long end = 0l;

        // ********* 快速读 *********
        // 测试 FastReaderPlus
        FastReaderPlus frp = null;
        try {
            // 指定文件流作为输入流，模拟实际输入使用System.in代替
            frp = new FastReaderPlus(new FileInputStream(filename));
            start = System.currentTimeMillis();
            // 读取1e6次
            for (int i = 0; i < 1000000; i++) {
                frp.nextInt();
            }
            end = System.currentTimeMillis();
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        long frpTime = end - start;

        // 测试 FastReader
        FastReader fr = null;
        try {
            fr = new FastReader(new FileInputStream(filename));
            start = System.currentTimeMillis();
            for (int i = 0; i < 1000000; i++) {
                fr.nextInt();
            }
            end = System.currentTimeMillis();
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        long frTime = end - start;

        // 测试 Scanner
        Scanner sc = null;
        try {
            sc = new Scanner(new FileInputStream(filename));
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
        start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            sc.nextInt();
        }
        long scTime = System.currentTimeMillis() - start;


        // ********* 快速写 *********

        //测试 PrintWriter
        PrintWriter out = new PrintWriter(System.out);

        start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            out.println(i);
        }

        // 全部打印完统一刷新，只一次IO
        out.flush();
        long pwTime = System.currentTimeMillis() - start;
        System.out.println("PrintWriter 耗时: " + (pwTime) + "ms");


        // 测试直接打印
        start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            System.out.println(i);
        }

        System.out.println("System.out.println 耗时: " + (System.currentTimeMillis() - start) + "ms");
        System.out.println("PrintWriter 耗时: " + (pwTime) + "ms");

        System.out.println("FastReaderStream 耗时: " + (frpTime) + "ms");
        System.out.println("FastReader 耗时: " + (frTime) + "ms");
        System.out.println("Scanner 耗时: " + (scTime) + "ms");
    }
}

class FastReader {
    BufferedReader br;
    StringTokenizer st;

    public FastReader() {
        br = new BufferedReader(new InputStreamReader(System.in));
    }

    public FastReader(InputStream is) {
        br = new BufferedReader(new InputStreamReader(is));
    }

    String next() {
        while (st == null || !st.hasMoreTokens()) {
            try {
                st = new StringTokenizer(br.readLine());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return st.nextToken();
    }

    int nextInt() { return Integer.parseInt(next()); }
    long nextLong() { return Long.parseLong(next()); }
    double nextDouble() { return Double.parseDouble(next()); }
}

class FastReaderPlus {
    BufferedInputStream bis;
    byte[] buf = new byte[1 << 16];
    int ptr, len;

    public FastReaderPlus() {
        bis = new BufferedInputStream(System.in);
    }

    public FastReaderPlus(InputStream is) {
        bis = new BufferedInputStream(is);
    }

    private int read() throws IOException {
        if (ptr >= len) {
            len = bis.read(buf);
            ptr = 0;
            if (len <= 0) return -1;
        }
        return buf[ptr++];
    }

    int nextInt() throws IOException {
        int res = 0, c = read();
        boolean neg = false;
        while (c <= 32) c = read();
        if (c == '-') {neg=true;c=read();}
        while (c >= '0' && c <= '9') {
            res = res * 10 + c - '0';
            c = read();
        }
        return neg ? -res : res;
    }
}
```

## 二.解决原理
Java程序运行在JVM（Java Virtual Machine）之上。

Java为了屏蔽不同操作系统之间的差异，将底层输入输出抽象为统一的 **Stream（流）模型**。

Java IO 中最基础的抽象是：

* `InputStream`：字节输入流，它内部封装了读取标准输入的能力。
* `OutputStream`：字节输出流，它内部封装了标准输出的能力。

Java中的很多高级输入输出类，都是在这些基础流之上进行包装。

### 1.输入

1.对于 Scanner(System.in) 输入来说，数据首先由操作系统接收并缓存在输入缓冲区中。

如果输入来源是文件，则可能经过操作系统的页缓存；如果输入来源是键盘，则经过操作系统的标准输入缓冲区。

2.JVM通过native方法调用操作系统接口，将底层数据读取到JVM堆内存的缓冲区（字节数组）中。

3.Java通过InputStream抽象类对底层输入来源进行封装，System.in本身就是一个InputStream对象；

Scanner会进一步将字节流转换为字符流（InputStreamReader，字节流->字符流），再进行字符处理。

4.通过 Matcher 进行正则匹配，这一过程会增加解析开销，并可能创建较多临时对象，造成额外的GC压力，因此耗时较多。

5.通过转换方法，将解析得到的字符串转换成要求的数据类型。

6.以及保证整体流程正常进行的各种异常处理。

---

首先，底层操作系统与JVM之间的数据传输属于Java运行环境的一部分，不建议从这里进行优化。

因此，我们可以从两个方面来进行优化。

针对过程4，轻度优化，不进行复杂的正则匹配，只通过 StringTokenizer 进行简单的切割（FastReader）。

针对过程3，重度优化，避免字符流转换和字符串解析，我们直接对字节流进行处理（FastReaderPlus）。

### 2.输出

1.对于 System.out.println() 输出来说，会先将待输出的数据缓存在字符数组中。

2.接收到字符串后，数据通过 PrintStream 写入输出流，然后清空缓冲区。

3.如果涉及字符转换，则通过 OutputStreamWriter 完成字符到字节的转换（字符流->字节流）。

4.JVM通过native方法调用操作系统接口，将数据写入操作系统的输出缓冲区。

5.操作系统根据输出目标，将数据输出到屏幕、文件或其他设备。

6.以及保证整体流程正常进行的各种异常处理。

---

这里的优化主要是减少输出调用次数和刷新次数。

System.out.println 每次调用都会产生一次方法调用并且刷新缓冲区（具体看向println传递的参数autoFlash，通常默认为true）。

因此，针对过程2，可以将多个输出先保存到缓冲区，最后统一调用 flush() 写出，从而减少IO调用次数。

```java
PrintWriter out = new PrintWriter(System.out);

for(int i = 0; i < n; i++){
    out.println(i);
}

out.flush();
```
值得注意的是。

PrintWriter 可以自动管理缓冲区，当缓冲区满时会自动向底层输出，不需要我们手动处理。

