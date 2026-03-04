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
