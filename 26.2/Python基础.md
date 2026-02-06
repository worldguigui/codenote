# Python基础

## 一.基础语法
### 1.变量与数据类型
```python
# 变量定义（无需声明类型）
name = "Python"
age = 25
price = 19.99
is_valid = True

# 一次赋值多个变量
a,b,c = 0,1,2

# 常见数据类型
int_num = 10        # 整型
float_num = 3.14    # 浮点型
string = "hello"    # 字符串
boolean = True      # 布尔型
none_type = None    # 空值
```

### 2.注释
```python
# 单行注释

"""
多行注释
可以写多行内容
"""
```

### 3.方法定义与参数类型
```python
# 指定参数类型
def func(a:int):
    print(a)

# 带返回值类型提示的方法
def func(a:int) -> int:
    return a + 1

# 位置参数
def func(a, b, c):
    return sum(a,b,c)

# 关键字参数
def func(name, age=18):     # 默认参数
    print(f"{key}: {age}")

# 可变参数
def sum_all(*args):         # 任意数量的位置参数
    return sum(args)

def print_info(**kwargs):   # 任意数量的关键字参数
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# 混合使用
def complex_func(a, b=10, *args, **kwargs):
    pass
```

## 二.流程控制
### 1.条件控制
```python
if x > 10:
    print("大于10")
elif x > 5:
    print("大于5")
else:
    print("小于等于5")

# 三元表达式
result = "正数" if x > 0 else "非正数"
```

### 2.循环语句
```python
# for循环
for i in range(5):          # 0,1,2,3,4
    print(i)

for item in my_list:        # 遍历列表
    print(item)

for key, value in my_dict.items():  # 遍历字典
    print(key, value)

# while循环
count = 0
while count < 5:
    print(count)
    count += 1

# 循环控制
for i in range(10):
    if i == 3:
        continue    # 跳过本次循环
    if i == 7:
        break       # 终止循环
```

## 三.数据结构-初始化与增删改查
### 1.列表 (List) - 有序、可重复、可变
#### 初始化
```python
# 空列表
empty_list = []
empty_list2 = list()

# 给定大小的初始化
size = 5

# 方法1：使用乘号初始化相同值
list1 = [0] * size             # [0, 0, 0, 0, 0]
list2 = [None] * size          # [None, None, None, None, None]

# 方法2：列表推导式
list3 = [0 for _ in range(size)]      # [0, 0, 0, 0, 0]
list4 = [i * 2 for i in range(size)]  # [0, 2, 4, 6, 8]

# 方法3：循环追加
list5 = []
for i in range(size):
    list5.append(i)  # [0, 1, 2, 3, 4]

# 多维列表初始化
matrix = [[0] * 3 for _ in range(3)]  # [[0,0,0], [0,0,0], [0,0,0]]

# 注意：不要用这种方式初始化多维列表
wrong_matrix = [[0] * 3] * 3  # 有问题！内层列表是同一个引用
```
#### 增删改查
```python
# 创建测试列表
my_list = [1, 2, 3, 4, 5]

# ---------- 查询操作 ----------
# 索引访问
print(my_list[0])      # 1（正向索引，从0开始）
print(my_list[-1])     # 5（负向索引，-1表示最后一个）

# 切片访问
print(my_list[1:3])    # [2, 3]（索引1到2）
print(my_list[:3])     # [1, 2, 3]（从开始到索引2）
print(my_list[2:])     # [3, 4, 5]（从索引2到最后）
print(my_list[::2])    # [1, 3, 5]（步长为2）

# 查找元素
print(3 in my_list)           # True（判断元素是否存在）
print(my_list.index(3))       # 2（返回元素索引，找不到会报错）
print(my_list.index(3, 1, 4)) # 2（在索引1-4范围内查找）
print(my_list.count(3))       # 1（统计元素出现次数）

# ---------- 增加操作 ----------
# 追加元素
my_list.append(6)            # [1, 2, 3, 4, 5, 6]

# 插入元素
my_list.insert(2, 99)        # [1, 2, 99, 3, 4, 5, 6]（在索引2处插入）

# 扩展列表
my_list.extend([7, 8, 9])    # [1, 2, 99, 3, 4, 5, 6, 7, 8, 9]
my_list += [10, 11]          # [1, 2, 99, 3, 4, 5, 6, 7, 8, 9, 10, 11]

# ---------- 修改操作 ----------
# 修改单个元素
my_list[2] = 100             # [1, 2, 100, 3, 4, 5, 6, 7, 8, 9, 10, 11]

# 修改切片
my_list[2:5] = [200, 300, 400]  # [1, 2, 200, 300, 400, 5, 6, 7, 8, 9, 10, 11]

# ---------- 删除操作 ----------
# 根据索引删除
del my_list[2]                # [1, 2, 300, 400, 5, 6, 7, 8, 9, 10, 11]
del my_list[2:5]              # [1, 2, 6, 7, 8, 9, 10, 11]（删除切片）

# 根据值删除
my_list.remove(2)             # [1, 6, 7, 8, 9, 10, 11]（删除第一个匹配值）

# 弹出元素
popped = my_list.pop()        # 11, my_list = [1, 6, 7, 8, 9, 10]（弹出最后一个）
popped2 = my_list.pop(2)      # 7, my_list = [1, 6, 8, 9, 10]（弹出指定索引）

# 清空列表
my_list.clear()               # []

# ---------- 其他常用操作 ----------
my_list = [3, 1, 4, 1, 5, 9, 2]
copy_list = my_list.copy()    # 浅拷贝
my_list.sort()                # [1, 1, 2, 3, 4, 5, 9]（升序排序）
my_list.sort(reverse=True)    # [9, 5, 4, 3, 2, 1, 1]（降序排序）
my_list.reverse()             # [1, 1, 2, 3, 4, 5, 9]反转成[9, 5, 4, 3, 2, 1, 1]

# 排序返回新列表
sorted_list = sorted(my_list)  # 不改变原列表
```

### 2.元组 (Tuple) - 有序、可重复、不可变
#### 初始化
```python
# 空元组
empty_tuple = ()
empty_tuple2 = tuple()

# 单元素元组（必须有逗号）
single_tuple = (1,)
wrong_single = (1)  # 这是整数1，不是元组！

# 给定大小的初始化
size = 5

# 元组不可变，但可以这样初始化
tuple1 = (0,) * size                     # (0, 0, 0, 0, 0)
tuple2 = tuple(0 for _ in range(size))   # (0, 0, 0, 0, 0)
tuple3 = tuple(range(size))              # (0, 1, 2, 3, 4)

# 元组推导式（实际上生成器表达式）
tuple4 = tuple(i * 2 for i in range(size))  # (0, 2, 4, 6, 8)

# 从其他序列创建
tuple5 = tuple([1, 2, 3])  # (1, 2, 3)
tuple6 = tuple("hello")    # ('h', 'e', 'l', 'l', 'o')
```
#### 增删改查
```python
# 创建测试元组
my_tuple = (10, 20, 30, 40, 50, 20)

# ---------- 查询操作 ----------
# 索引访问（同列表）
print(my_tuple[0])      # 10
print(my_tuple[-1])     # 50

# 切片访问（同列表）
print(my_tuple[1:4])    # (20, 30, 40)
print(my_tuple[:3])     # (10, 20, 30)
print(my_tuple[2:])     # (30, 40, 50, 20)

# 查找元素
print(30 in my_tuple)           # True
print(my_tuple.index(20))       # 1（第一次出现的索引）
print(my_tuple.index(20, 2))    # 5（从索引2开始查找）
print(my_tuple.count(20))       # 2（统计出现次数）

# ---------- 元组不可修改 ----------
# 以下操作会报错！
# my_tuple[0] = 100      # TypeError
# my_tuple.append(60)    # AttributeError
# del my_tuple[0]        # TypeError

# 但可以创建新元组
new_tuple = my_tuple + (60, 70)  # (10, 20, 30, 40, 50, 20, 60, 70)

# ---------- 元组解包 ----------
# 基本解包
a, b, c = (1, 2, 3)     # a=1, b=2, c=3

# 扩展解包
first, *middle, last = (1, 2, 3, 4, 5)  # first=1, middle=[2,3,4], last=5

# 交换变量
x, y = 10, 20
x, y = y, x              # x=20, y=10（利用元组解包）

# ---------- 其他操作 ----------
# 长度和最大值最小值
print(len(my_tuple))     # 6
print(max(my_tuple))     # 50
print(min(my_tuple))     # 10

# 排序（返回列表）
sorted_tuple = tuple(sorted(my_tuple))  # (10, 20, 20, 30, 40, 50)

# 转换为列表进行修改
temp_list = list(my_tuple)
temp_list.append(60)
modified_tuple = tuple(temp_list)  # (10, 20, 30, 40, 50, 20, 60)
```

### 3.字典 (Dictionary) - 无序、键值对、键不可重复、可变
#### 初始化
```python
# 空字典
empty_dict = {}
empty_dict2 = dict()

# 给定大小的初始化
size = 5

# 方法1：fromkeys方法（所有键使用相同的值）
dict1 = dict.fromkeys(range(size), 0)      # {0:0, 1:0, 2:0, 3:0, 4:0}
dict2 = dict.fromkeys(["a", "b", "c"], None)  # {'a':None, 'b':None, 'c':None}

# 方法2：字典推导式
dict3 = {i: i**2 for i in range(size)}      # {0:0, 1:1, 2:4, 3:9, 4:16}
dict4 = {f"key_{i}": None for i in range(size)}  # {'key_0':None, 'key_1':None, ...}

# 方法3：直接创建
dict5 = {"name": "Alice", "age": 25, "city": "Beijing"}

# 从键值对列表创建
pairs = [("a", 1), ("b", 2), ("c", 3)]
dict6 = dict(pairs)  # {'a':1, 'b':2, 'c':3}

# 从两个列表创建
keys = ["name", "age", "city"]
values = ["Bob", 30, "Shanghai"]
dict7 = dict(zip(keys, values))  # {'name':'Bob', 'age':30, 'city':'Shanghai'}
```

#### 增删改查
```python
# 创建测试字典
my_dict = {"name": "Alice", "age": 25, "city": "Beijing"}

# ---------- 查询操作 ----------
# 直接访问（键不存在会报错）
print(my_dict["name"])      # "Alice"

# get方法访问（键不存在返回None或默认值）
print(my_dict.get("age"))          # 25
print(my_dict.get("country"))      # None
print(my_dict.get("country", "CN")) # "CN"（默认值）

# 检查键是否存在
print("name" in my_dict)    # True
print("country" in my_dict) # False

# 获取所有键、值、键值对
keys = my_dict.keys()       # dict_keys(['name', 'age', 'city'])
values = my_dict.values()   # dict_values(['Alice', 25, 'Beijing'])
items = my_dict.items()     # dict_items([('name', 'Alice'), ('age', 25), ...])

# ---------- 增加操作 ----------
# 添加新键值对
my_dict["gender"] = "female"  # {'name':'Alice', 'age':25, 'city':'Beijing', 'gender':'female'}

# setdefault（键不存在时添加并返回值）
value = my_dict.setdefault("country", "CN")  # value="CN"，同时添加了该键值对

# update方法（批量添加或更新）
my_dict.update({"age": 26, "job": "Engineer"})  # 更新age，添加job

# ---------- 修改操作 ----------
# 直接修改
my_dict["age"] = 27  # 修改现有键的值

# update方法也可用于修改
my_dict.update({"city": "Shanghai", "age": 28})

# ---------- 删除操作 ----------
# pop方法（删除指定键并返回值）
age = my_dict.pop("age")     # age=28，删除'age'键

# popitem方法（删除最后一个键值对并返回）
last_item = my_dict.popitem()  # 随机删除一个（Python 3.7+删除最后一个）

# del语句
del my_dict["city"]          # 删除'city'键

# clear方法
my_dict.clear()              # {}（清空字典）

# ---------- 重新初始化 ----------
my_dict = {"name": "Bob", "age": 30, "city": "Guangzhou"}

# ---------- 遍历字典 ----------
# 遍历键
for key in my_dict:
    print(key, my_dict[key])

# 遍历键值对
for key, value in my_dict.items():
    print(f"{key}: {value}")

# 遍历值
for value in my_dict.values():
    print(value)

# 创建新字典
squared_dict = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}

# 过滤字典
filtered = {k: v for k, v in my_dict.items() if k != "age"}  # 排除'age'键

# 大小写转换
lowercase_dict = {k.lower(): v for k, v in my_dict.items()}
```

### 4.集合 (Set) - 无序、不重复、可变
#### 初始化
```python
# 空集合（注意：不能用{}，那是空字典）
empty_set = set()

# 给定大小的初始化
size = 5

# 方法1：从范围创建
set1 = set(range(size))          # {0, 1, 2, 3, 4}

# 方法2：集合推导式
set2 = {i for i in range(size)}  # {0, 1, 2, 3, 4}
set3 = {i * 2 for i in range(size)}  # {0, 2, 4, 6, 8}

# 方法3：从列表创建（自动去重）
set4 = set([1, 2, 2, 3, 3, 3])   # {1, 2, 3}

# 方法4：直接创建
set5 = {1, 2, 3, 4, 5}

# 注意：集合中的元素必须是不可变类型
valid_set = {1, 2, 3, "hello", (1, 2)}  # 可以包含数字、字符串、元组
# invalid_set = {1, 2, [3, 4]}  # 错误！不能包含列表
```
#### 增删改查
```python
# 创建测试集合
my_set = {1, 2, 3, 4, 5}

# ---------- 查询操作 ----------
# 检查元素是否存在
print(3 in my_set)      # True
print(10 in my_set)     # False

# 获取集合长度
print(len(my_set))      # 5

# ---------- 增加操作 ----------
# 添加单个元素
my_set.add(6)           # {1, 2, 3, 4, 5, 6}

# 添加多个元素
my_set.update([7, 8, 9])  # {1, 2, 3, 4, 5, 6, 7, 8, 9}
my_set.update({10, 11})   # 也可以添加另一个集合

# ---------- 删除操作 ----------
# remove（元素不存在会报错）
my_set.remove(3)        # 删除3，{1, 2, 4, 5, 6, 7, 8, 9, 10, 11}

# discard（元素不存在不会报错）
my_set.discard(100)     # 不报错，即使100不存在

# pop（随机删除一个元素）
popped = my_set.pop()   # 随机删除并返回一个元素

# clear（清空集合）
my_set.clear()          # set()

# ---------- 重新初始化 ----------
set_a = {1, 2, 3, 4, 5}
set_b = {4, 5, 6, 7, 8}

# ---------- 集合运算 ----------
# 并集
union_set = set_a | set_b            # {1, 2, 3, 4, 5, 6, 7, 8}
union_set2 = set_a.union(set_b)      # 同上

# 交集
intersection_set = set_a & set_b     # {4, 5}
intersection_set2 = set_a.intersection(set_b)  # 同上

# 差集（在A中但不在B中）
difference_set = set_a - set_b       # {1, 2, 3}
difference_set2 = set_a.difference(set_b)  # 同上

# 对称差集（仅在A或仅在B中）
symmetric_diff = set_a ^ set_b       # {1, 2, 3, 6, 7, 8}
symmetric_diff2 = set_a.symmetric_difference(set_b)  # 同上

# ---------- 集合关系判断 ----------
set_c = {1, 2}
set_d = {1, 2, 3, 4}

print(set_c <= set_d)   # True（子集）
print(set_c < set_d)    # True（真子集）
print(set_d >= set_c)   # True（超集）
print(set_d > set_c)    # True（真超集）
print(set_c.isdisjoint({5, 6}))  # True（没有交集）

# ---------- 集合推导式 ----------
# 创建新集合
even_set = {x for x in range(10) if x % 2 == 0}  # {0, 2, 4, 6, 8}

# 去重应用
duplicates = [1, 2, 2, 3, 4, 4, 4, 5]
unique_set = set(duplicates)  # {1, 2, 3, 4, 5}
unique_list = list(unique_set)  # [1, 2, 3, 4, 5]（注意：顺序可能改变）
```

### 5.双端队列（Deque）
#### 初始化
```python
# 创建一个空 deque
d = deque()
print(d)  # deque([])

# 从可迭代对象初始化
d = deque([1, 2, 3, 4])
print(d)  # deque([1, 2, 3, 4])

d = deque("hello")
print(d)  # deque(['h', 'e', 'l', 'l', 'o'])

# 限制最大长度为3
d = deque(maxlen=3)
d.extend([1, 2, 3, 4])  # 自动丢弃最早的元素
print(d)  # deque([2, 3, 4], maxlen=3)

# 初始化时指定最大长度
d = deque([1, 2, 3], maxlen=5)
```

#### 增删改查
```python
# 右端添加
d = deque([1, 2, 3])
d.append(4)          # 添加到右端
print(d)             # deque([1, 2, 3, 4])

d.appendright(5)     # 同上，添加到右端（实际上是 append 的别名）

# 左端添加
d = deque([1, 2, 3])
d.appendleft(0)      # 添加到左端
print(d)             # deque([0, 1, 2, 3])

d.appendleft(-1)
print(d)             # deque([-1, 0, 1, 2, 3])

# 批量添加
d = deque([1, 2, 3])
d.extend([4, 5, 6])  # 从右端批量添加
print(d)             # deque([1, 2, 3, 4, 5, 6])

d.extendleft([0, -1])  # 从左端批量添加（注意顺序！）
print(d)              # deque([-1, 0, 1, 2, 3, 4, 5, 6])
# extendleft 是逆序添加的，相当于 for item in iterable: d.appendleft(item)

# 右端删除
d = deque([1, 2, 3, 4])
value = d.pop()      # 从右端删除并返回
print(value)         # 4
print(d)             # deque([1, 2, 3])

# 左端删除
d = deque([1, 2, 3, 4])
value = d.popleft()  # 从左端删除并返回
print(value)         # 1
print(d)             # deque([2, 3, 4])

# 删除指定
d = deque([1, 2, 3, 2, 4])
d.remove(2)          # 删除第一个匹配的元素，如果要删除的元素不存在会抛异常
print(d)             # deque([1, 3, 2, 4])

# 清空
d.clear()

# 修改元素
d = deque([1, 2, 3, 4, 5])

# 通过索引修改
d[0] = 100
print(d)  # deque([100, 2, 3, 4, 5])

d[-1] = 500
print(d)  # deque([100, 2, 3, 4, 500])

# 注意：deque 不支持切片赋值
try:
    d[1:3] = [200, 300]
except TypeError as e:
    print(f"错误: {e}")  # 'deque' object does not support item assignment

# 查询元素
d = deque([10, 20, 30, 40, 50])

# 正索引（从左到右）
print(d[0])   # 10
print(d[2])   # 30
print(d[-1])  # 50（最后一个元素）
print(d[-2])  # 40（倒数第二个）

# 遍历
for item in d:
    print(item, end=' ')  # 10 20 30 40 50

# 切片（需要转换为列表）
print(list(d)[1:3])  # [20, 30]

# 旋转
d = deque([1, 2, 3, 4, 5])

d.rotate(2)   # 向右旋转2步
print(d)      # deque([4, 5, 1, 2, 3])

d.rotate(-1)  # 向左旋转1步
print(d)      # deque([5, 1, 2, 3, 4])
```
## 四.字符串操作
### 1.字符串创建
```python
# 单引号
str1 = 'Hello World'

# 双引号
str2 = "Hello World"

# 三引号
str3 = '''这是一个
多行字符串
可以换行'''

str4 = """这也是一个
多行字符串"""

# 字符串构造函数
str5 = str(123)  # "123"
```

### 2.字符串索引与切片
```python
s = "Python Programming"

# 索引（从0开始）
print(s[0])    # 'P'
print(s[-1])   # 'g'（负数表示从末尾开始）

# 切片 [start:end:step]
print(s[0:6])     # "Python"（包含start，不包含end）
print(s[7:])      # "Programming"（从索引7到最后）
print(s[:6])      # "Python"（从开始到索引6）
print(s[::2])     # "Pto rgamn"（每隔一个字符）
print(s[::-1])    # "gnimmargorP nohtyP"（反转字符串）
```

### 3.字符串大小写转换
```python
s = "Python Programming"

print(s.upper())          # "PYTHON PROGRAMMING"
print(s.lower())          # "python programming"
print(s.capitalize())     # "Python programming"（首字母大写）
print(s.title())          # "Python Programming"（每个单词首字母大写）
print(s.swapcase())       # "pYTHON pROGRAMMING"（大小写互换）
```

### 4.字符串查找和替换
```python
s = "hello world hello"

# 查找
print(s.find("world"))      # 6（返回第一次出现的索引，找不到返回-1）
print(s.rfind("hello"))     # 12（从右边开始查找）
print(s.index("world"))     # 6（类似find，但找不到会报错）
print(s.rindex("hello"))    # 12（从右边开始）

# 判断
print(s.startswith("hello"))  # True
print(s.endswith("world"))    # False
print("world" in s)          # True

# 替换
print(s.replace("hello", "HELLO"))      # "HELLO world HELLO"
print(s.replace("hello", "HELLO", 1))   # "HELLO world hello"（只替换第一个）
```

### 5.字符串分割与连接
```python
# 分割
s = "apple,banana,orange"
print(s.split(","))          # ['apple', 'banana', 'orange']
print(s.split(",", 1))       # ['apple', 'banana,orange']（最大分割次数）

s2 = "hello world\npython"
print(s2.split())            # ['hello', 'world', 'python']（默认按空白字符分割）
print(s2.splitlines())       # ['hello world', 'python']（按行分割）

# + 连接
print("a" + "b")             # "ab"

# join 连接
list1 = ['a', 'b', 'c']
print("-".join(list1))       # "a-b-c"

tuple1 = ('2023', '12', '31')
print("/".join(tuple1))      # "2023/12/31"

# 分割为三部分
s3 = "python.is.awesome.for.programming"
print(s3.partition("."))     # ('python', '.', 'is.awesome.for.programming')
print(s3.rpartition("."))    # ('python.is.awesome.for', '.', 'programming')
```

### 6.去除空白字符
```python
s = "   hello world   \n\t"

print(s.strip())       # "hello world"（去除两端空白）
print(s.lstrip())      # "hello world   \n\t"（去除左侧空白）
print(s.rstrip())      # "   hello world"（去除右侧空白）

# 去除指定字符
s2 = "***hello***"
print(s2.strip("*"))   # "hello"
print(s2.lstrip("*"))  # "hello***"
print(s2.rstrip("*"))  # "***hello"
```

### 7.判断方法
```python
s1 = "Hello123"
s2 = "12345"
s3 = "HELLO"
s4 = "hello"
s5 = "Hello World"
s6 = " \t\n"
s7 = "HelloWorld"
s8 = "123.45"

print(s1.isalnum())    # True（字母或数字）
print(s2.isdigit())    # True（数字）
print(s3.isupper())    # True（全大写）
print(s4.islower())    # True（全小写）
print(s5.istitle())    # True（每个单词首字母大写）
print(s6.isspace())    # True（空白字符）
print(s7.isalpha())    # True（字母）
print(s8.isdecimal())  # False（十进制数字）

# 检查字符串是否可打印
print("Hello".isprintable())  # True
print("\n".isprintable())     # False

### 判断字符是否相等
print(s1 == s2)    # False - 不相同

# 按字符的Unicode编码逐个比较
print("apple" < "banana")    # True - 'a' < 'b'
```

### 8.计数与统计
```python
s = "hello hello world"

print(s.count("hello"))     # 2
print(s.count("l"))         # 5
print(s.count("l", 5, 10))  # 2（在索引5-10范围内）

print(len(s))               # 17（字符串长度）
```
### 9.Unicode编码
```python
print(ord("a"))             # 97
print(chr(97))              # a
```
### 10.数字类型转换
```python
# 默认十进制
num = int("11")
print(nums)             # 11

# 二进制（以 0b 或 0B 开头）
num = int("11", 2)      # 二进制 11 → 十进制 3
print(num)              # 3

# 八进制（以 0o 或 0O 开头）
num = int("11", 8)      # 八进制 11 → 十进制 9
print(num)              # 9

# 十六进制（以 0x 或 0X 开头）
num = int("11", 16)     # 十六进制 11 → 十进制 17
print(num)              # 17

# 自动识别进制
print(int("0b11", 0))   # 3（二进制）
print(int("0o11", 0))   # 9（八进制）
print(int("0x11", 0))   # 17（十六进制）

# 浮点数转换
num = float("11")
print(num)           # 11.0
print(type(num))     # <class 'float'>

num = float("11.5")
print(num)           # 11.5

num = float("-11.25")
print(num)           # -11.25

# 科学计数法
num = float("1.23e-4")
print(num)           # 0.000123
```
## 五.面向对象
### 1.类定义
```python
class Person:
    # 类属性
    species = "Human"
    
    # 构造方法
    def __init__(self, name, age):
        self.name = name    # 实例属性
        self.age = age
        self.__secret = "私有属性"  # 私有属性（名称改写）
    
    # 实例方法
    def introduce(self):
        return f"我叫{self.name}，今年{self.age}岁"
    
    # 类方法
    @classmethod
    def create_baby(cls, name):
        return cls(name, 0)
    
    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18
    
    # 属性装饰器
    @property
    def birth_year(self):
        import datetime
        return datetime.datetime.now().year - self.age
```

### 2.继承
```python
class Student(Person):
    def __init__(self, name, age, student_id):
        super().__init__(name, age)  # 调用父类构造方法
        self.student_id = student_id
    
    # 方法重写
    def introduce(self):
        return f"{super().introduce()}，学号{self.student_id}"
```
## 六.文件操作
```python
# 读取文件
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()      # 读取全部
    lines = f.readlines()   # 读取所有行
    line = f.readline()     # 读取一行

# 写入文件
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello\n")
    f.writelines(["Line1\n", "Line2\n"])

# 追加模式
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("New log entry\n")
```

## 七.异常处理
```python
try:
    result = 10 / 0
    with open("file.txt") as f:
        content = f.read()
except ZeroDivisionError:
    print("不能除以零")
except FileNotFoundError as e:
    print(f"文件未找到: {e}")
except Exception as e:  # 捕获所有异常
    print(f"发生错误: {e}")
else:
    print("没有发生异常时执行")
finally:
    print("无论是否异常都会执行")

# 抛出异常
if x < 0:
    raise ValueError("x不能为负数")

# 自定义异常
class MyError(Exception):
    pass
```
### 八.模块与包
```python
# 导入整个模块
import math
print(math.sqrt(16))

# 导入特定内容
from math import pi, sin
from math import *  # 导入所有（不推荐）

# 别名
import numpy as np
from datetime import datetime as dt

# 包的使用
# package/
#   __init__.py
#   module1.py
#   subpackage/
#     __init__.py
#     module2.py
```
