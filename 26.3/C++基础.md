# C++基础

## 一.基础语法
```c++
// 整型
int a = 10;                // 4字节
short s = 5;                // 2字节
long l = 100000L;           // 4或8字节
long long ll = 1000000000;  // 8字节

// 浮点型
float f = 3.14f;           // 4字节，需加f后缀
double d = 3.14159;        // 8字节

// 字符型
char c = 'A';              // 1字节
wchar_t wc = L'中';         // 宽字符

// 布尔型
bool flag = true;          // true/false

// 无符号类型
unsigned int u = 100;
```

## 二.变量声明与初始化
```c++
int a = 10;                 // 拷贝初始化
int b(20);                  // 直接初始化  
int c{30};                  // 列表初始化 (C++11)
int d = {40};               // 拷贝列表初始化

// auto类型推导 (C++11)
auto x = 5;                 // int
auto y = 3.14;              // double
auto z = "hello";           // const char*

// decltype查询表达式的类型 (C++11)
int i = 42;
decltype(i) j = 100;        // int类型
```

## 三.流程控制
```c++
if (condition) {
    // ...
} else if (condition) {
    // ...
} else {
    // ...
}

// switch语句（支持整数和枚举类型）
switch (value) {
    case 1:
        break;
    default:
        break;
}

// 循环
for (int i = 0; i < 10; i++) { }
while (condition) { }
do { } while (condition);
```

## 四.函数
```c++
// 普通函数
返回类型 函数名(参数列表) {
    // 函数体
    return 返回值;
}

// 内联函数，一种请求编译器在调用处直接展开函数代码的机制，
// 目的是消除函数调用的开销（压栈、跳转、返回等）
// 只是在编译层面有优化，其他地方与普通函数没有区别
inline int max(int a, int b) {
    return a > b ? a : b;
}

// 带默认参数的函数
void func(int a, int b = 10, int c = 20);

// 函数重载
void print(int i);
void print(double d);
void print(string s);

// Lambda表达式 (C++11)
auto lambda = [](int x, int y) -> int {
    return x + y;
};

// 函数指针
int (*funcPtr)(int, int);  // 指向返回int、接受两个int参数的函数

int add(int x,int y) {
    return x + y;
}

void processNumbers(int x, int y, int (*operation)(int, int)) {
    cout << "结果: " << operation(x, y) << endl;
}

processNumbers(1,2,add)   // 输出 3
```

## 五.数组与字符串
### 1.数组
```c++
// 一维数组
int arr[5] = {1, 2, 3, 4, 5};
int arr2[] = {1, 2, 3, 4, 5};
int size = sizeof(arr) / sizeof(arr[0]);

// 二维数组
int matrix[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};

// 动态数组
int* dynArr = new int[10];
delete[] dynArr;  // 释放内存
```

### 2.字符串
```c++
// C风格字符串
char str1[] = "Hello";
char* str2 = "World";

// C++字符串
#include <string>
string s1 = "Hello";
string s2("World");
string s3 = s1 + " " + s2;

// 字符串方法
s.length();     // 长度
s.size();       // 大小
s.empty();      // 是否为空
s.substr(pos, n);  // 子串
s.find("ll");   // 查找
```
## 六.指针与引用
### 1.指针
```c++
int var = 10;
int* ptr = &var;        // 指针声明
*ptr = 20;              // 解引用

// 空指针
int* p1 = nullptr;      // C++11
int* p2 = NULL;         // 传统方式

// 指针运算
int arr[5] = {1, 2, 3, 4, 5};
int* p = arr;
p++;                    // 指向下一个元素

// 指针数组 vs 数组指针
int* arrPtr[5];         // 指针数组，存放指针
int (*ptrArr)[5];       // 数组指针，指向数组
```

### 2.引用
```c++
int x = 10;
int& ref = x;           // 引用必须初始化
ref = 20;               // 改变x的值

// 常量引用
const int& cref = x;    // 不能通过cref修改

// 右值引用 (C++11)
int&& rref = 10;        // 可绑定到临时对象
```

## 七.面向对象编程
### 1.类与对象
```c++
class MyClass {
private:                // 私有成员
    int privateVar;
    
protected:              // 保护成员
    int protectedVar;
    
public:                 // 公有成员
    // 构造函数
    MyClass() : privateVar(0) {}  // 初始化列表
    MyClass(int val) : privateVar(val) {}
    
    // 拷贝构造函数
    MyClass(const MyClass& other) : privateVar(other.privateVar) {}
    
    // 移动构造函数 (C++11)
    MyClass(MyClass&& other) noexcept : privateVar(other.privateVar) {
        other.privateVar = 0;
    }
    
    // 析构函数
    ~MyClass() {}
    
    // 成员函数
    void setValue(int val) { privateVar = val; }
    int getValue() const { return privateVar; }  // const成员函数
    
    // 静态成员函数
    static void staticFunc() {}
    
private:
    static int staticVar;  // 静态成员变量
};

// 创建对象
MyClass obj1;
MyClass obj2(10);
MyClass obj3 = obj2;     // 拷贝构造
MyClass obj4 = std::move(obj3);  // 移动构造
```

### 2.继承
```c++
class Base {
public:
    virtual void func() { cout << "Base" << endl; }  // 虚函数
    virtual ~Base() {}  // 虚析构函数
};

class Derived : public Base {
public:
    void func() override { cout << "Derived" << endl; }  // 重写
};

// 多重继承
class A {};
class B {};
class C : public A, public B {};
```

### 3.多态
```c++
Base* ptr = new Derived();
ptr->func();  // 调用Derived的func
delete ptr;

// 纯虚函数和抽象类
class Abstract {
public:
    virtual void pureFunc() = 0;  // 纯虚函数
    virtual ~Abstract() {}
};
```

### 4.访问控制
```c++
class Example {
public:     // 任何地方可访问
protected:  // 派生类可访问
private:    // 仅本类可访问
};
```

## 八.运算符重载
```c++
class Complex {
private:
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // 成员函数形式
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    // 友元函数形式
    friend Complex operator-(const Complex& a, const Complex& b);
    
    // 输入输出重载
    friend ostream& operator<<(ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os;
    }
};
```

## 九.模板编程

### 1.函数模板
```c++
template <typename T>
T max(T a, T b) {
    return a > b ? a : b;
}

// 使用
int m1 = max<int>(10, 20);
double m2 = max(3.14, 2.71);  // 类型推导
```

### 2.类模板
```c++
template <typename T, int size>
class Array {
private:
    T data[size];
    
public:
    T& operator[](int index) { return data[index]; }
};

Array<int, 10> arr;
```

### 3.模板特化
```c++
template <>
class Array<bool, 10> {
    // 特化实现
};
```

## 十.STL容器
```c++
#include <vector>
#include <list>
#include <map>
#include <set>
#include <unordered_map>
#include <string>

// 顺序容器
vector<int> vec = {1, 2, 3, 4, 5};
list<int> lst = {1, 2, 3, 4, 5};
deque<int> deq = {1, 2, 3, 4, 5};

// 关联容器
map<string, int> mp;
mp["one"] = 1;
set<int> s = {1, 2, 3, 4, 5};

// 无序容器 (C++11)
unordered_map<string, int> umap;
unordered_set<int> uset;

// 容器适配器
stack<int> stk;
queue<int> que;
priority_queue<int> pq;
```

## 十一.迭代器
```c++
vector<int> vec = {1, 2, 3, 4, 5};

// 正向迭代
for (auto it = vec.begin(); it != vec.end(); ++it) {
    cout << *it << endl;
}

// 反向迭代
for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
    cout << *it << endl;
}

// 常量迭代器
for (auto cit = vec.cbegin(); cit != vec.cend(); ++cit) {
    // *cit = 10;  // 错误，不能修改
}
```

## 十二.异常处理
```c++
try {
    if (error) {
        throw runtime_error("发生错误");
    }
} catch (const runtime_error& e) {
    cout << "捕获异常: " << e.what() << endl;
} catch (...) {
    cout << "捕获未知异常" << endl;
}

// 自定义异常
class MyException : public exception {
public:
    const char* what() const noexcept override {
        return "我的自定义异常";
    }
};
```

### 十三.输入输出
```c++
#include <iostream>
#include <fstream>
#include <sstream>

using namespace std;

// 控制台IO
cout << "输出" << endl;
cin >> input;
cerr << "错误信息" << endl;

// 文件IO
ofstream outFile("output.txt");
outFile << "写入文件" << endl;
outFile.close();

ifstream inFile("input.txt");
string line;
getline(inFile, line);

// 字符串流
stringstream ss;
ss << "Hello " << 123;
string str = ss.str();
```
