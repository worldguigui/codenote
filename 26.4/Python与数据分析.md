# Python与数据分析

## 一.数据读取、


## 数据获取

掌握了**文件对象是字节流上的读写指针**和**网络请求的本质是发送符合协议的报文**，就大概掌握了数据获取！

无论是本地文件还是网络数据，底层逻辑都是“建立连接→读取/写入数据→关闭连接”。本地文件操作中，`seek()` 和读写方法控制着指针在字节流上的位置；网络数据获取中，`requests` 库帮我们封装了 HTTP 协议的请求与响应，返回的 `Response` 对象就是远程服务器上某个资源的“文件句柄”。

---

### 1. 本地数据获取（文件操作）

#### 1.1 文件打开与关闭
```python
# 打开文件
f = open('data.txt', 'r', encoding='utf-8')

# with 语句自动管理文件关闭
with open('data.txt', 'r') as f:
    content = f.read()
# 代码块结束，文件自动关闭
```

**打开模式速查**：

| 模式 | 含义 | 补充说明 |
|------|------|----------|
| `'r'` | 只读 | 文件必须存在 |
| `'w'` | 只写 | 文件不存在则新建，存在则**清空** |
| `'a'` | 追加 | 文件不存在则新建，存在则在末尾追加 |
| `'x'` | 新建只写 | 文件已存在则失败 |
| `'r+'` | 读写 | 文件必须存在 |
| `'b'` | 二进制模式 | 与上述模式组合，如 `'rb'`、`'wb'` |

#### 1.2 读文件
三种方法，按需选择：

```python
with open('data.txt', 'r') as f:
    s1 = f.read()           # 读全部内容
    s2 = f.read(5)          # 读 5 个字符

    line = f.readline()     # 读一行（含换行符）
    line = f.readline(20)   # 读本行最多 20 个字符

    lines = f.readlines()   # 读全部行，返回字符串列表
```

#### 1.3 写文件
```python
with open('out.txt', 'w') as f:
    f.write("Hello, World!\n")        # 写入字符串，返回字节数

    lines = ['line1\n', 'line2\n']
    f.writelines(lines)               # 写入字符串列表（不会自动加换行）
```

#### 1.4 文件指针定位 seek()
`seek(offset, whence)` 移动读写指针的位置。

```python
f.seek(0)        # 移到文件开头（whence=0）
f.seek(-5, 2)    # 从文件末尾（whence=2）向前移动 5 个字节
f.seek(3, 1)     # 从当前位置（whence=1）向后移动 3 个字节
```
whence 参数：`0`=文件头，`1`=当前位置，`2`=文件尾。

#### 1.5 CSV 文件读写
```python
import csv

# 读取
with open('consumer.csv', 'r', newline='') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)          # 每行是一个列表

# 写入
with open('consumer.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['col1', 'col2', 'col3'])   # 写一行
    writer.writerows([('a','b','c'), ('d','e','f')])  # 写多行
```

---

### 2. 网络数据获取

#### 2.1 requests 库：发送 HTTP 请求
```python
import requests

r = requests.get('http://www.baidu.com')
print(r.status_code)    # 200 表示成功，404 表示未找到
```

**Response 对象关键属性**：

| 属性 | 说明 |
|------|------|
| `r.status_code` | HTTP 状态码（200=成功） |
| `r.text` | 响应内容的字符串形式 |
| `r.content` | 响应内容的二进制形式 |
| `r.encoding` | 从 HTTP header 推测的编码方式 |
| `r.apparent_encoding` | 从内容分析出的编码方式（备选，更准确） |

#### 2.2 典型爬取流程
```python
import requests

url = 'https://item.jd.com/14260998.html'
try:
    r = requests.get(url)
    r.raise_for_status()              # 若状态码不是 200，抛出异常
    r.encoding = r.apparent_encoding  # 用更准确的编码
    print(r.text[:1000])
except:
    print('爬取失败')
```

#### 2.3 设置请求头（反爬基础手段）
很多网站会拒绝没有 User-Agent 的请求：
```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...'
}
r = requests.get(url, headers=headers)
```

#### 2.4 Beautiful Soup：解析 HTML
```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(r.text, 'html.parser')

# 查找元素
soup.find_all('a')                                    # 找所有 a 标签
soup.find_all('a', attrs={'class': 'mnav'})           # 找 class='mnav' 的 a 标签
soup.find(attrs={'id': 'book'})                       # 找 id='book' 的单个元素

# 提取信息
for link in soup.find_all('a'):
    print(link.string)     # 标签内的纯文本
```

#### 2.5 Web API（获取结构化数据）
很多网站提供 JSON 格式的 API：
```python
import requests
import pandas as pd

url = 'https://api.github.com/repos/pandas-dev/pandas/issues'
r = requests.get(url)
data = r.json()  # 解析为 Python 字典/列表

# 直接转为 DataFrame
issues = pd.DataFrame(data, columns=['number', 'title', 'state'])
```

#### 2.6 网络爬虫规范
- **Robots 协议**：网站根目录的 `robots.txt` 声明了允许/禁止爬取的页面。`Disallow: /` 表示禁止爬取所有内容。
- **注意事项**：爬取频率不宜过高（避免造成服务器负担），数据有版权归属，注意隐私保护。

---

## 二.NumPy

掌握了**轴（axis）**和**广播（broadcasting）**，就大概掌握了 NumPy。

### 1. 为什么用 NumPy？
- **快**：NumPy 的运算在 C 语言层面执行，比纯 Python 循环快 10~100 倍。
- **省**：数据存储在连续内存块上，比列表更节省空间。
- **强**：不用写 `for` 循环，就能对整块数据做运算（向量化计算）。

### 2. 核心数据结构：ndarray
ndarray 是一个**多维同质数据容器**——所有元素必须是同一类型。

```python
import numpy as np

# 从列表创建
arr = np.array([1, 2, 3])          # 一维
arr2d = np.array([[1,2,3], [4,5,6]])  # 二维

# 常⽤属性
print(arr2d.ndim)    # 秩（维度数）：2
print(arr2d.shape)   # 形状：(2, 3)
print(arr2d.size)    # 元素总数：6
print(arr2d.dtype)   # 元素类型：int64
```

### 3. 快速生成数组
```python
np.zeros((3, 4))        # 全 0 数组
np.ones((2, 3))         # 全 1 数组
np.eye(4)               # 单位矩阵
np.arange(0, 10, 2)     # 等差数组：[0,2,4,6,8]
np.linspace(0, 1, 5)    # 等间距数组：[0. 0.25 0.5 0.75 1.]
np.random.randn(3, 3)   # 标准正态分布随机数
np.random.randint(0, 10, size=5)  # 随机整数
```

### 4. 索引与切片
NumPy 的切片是**视图（view）**，不是拷贝。修改切片会直接影响原数组。这不同于 Python 列表。

```python
arr = np.arange(10)
s = arr[5:8]    # 切片
s[:] = 12       # 修改切片
print(arr)      # [ 0  1  2  3  4 12 12 12  8  9] —— 原数组变了！

# 二维切片
arr2d = np.array([[1,2,3], [4,5,6]])
print(arr2d[0, :])      # 第0行：[1 2 3]
print(arr2d[:, 1])      # 第1列：[2 5]
```

- **布尔索引**：用条件直接筛选
```python
arr = np.array([1, 2, 3, 4, 5])
print(arr[arr > 3])     # [4 5]
```

- **花式索引**：用整数数组选特定位置，结果总是**拷贝**
```python
print(arr[[0, 2, 4]])   # [1 3 5]
```

### 5. 核心概念一：轴（axis）
- `axis=0`：**垂直方向**操作，跨行（把列压扁）
- `axis=1`：**水平方向**操作，跨列（把行压扁）

```python
arr = np.array([[1,2,3], [4,5,6]])
print(arr.sum())        # 21 （全部加总）
print(arr.sum(axis=0))  # [5 7 9]（每列求和）
print(arr.sum(axis=1))  # [6 15] （每行求和）
```

### 6. 核心概念二：广播（broadcasting）
不同形状的数组运算时，NumPy 自动“拉伸”较小的一方。

**规则**：从尾部对齐维度，兼容条件是——对应维度要么相等，要么其中一个是 1。

```python
a = np.array([[1,2,3], [4,5,6]])  # shape (2,3)

# 标量 → 广播到每个元素
print(a + 10)           # 每个元素+10

# 行向量 (3,) → 视为 (1,3)，广播到 (2,3)
b = np.array([10, 20, 30])
print(a + b)

# 列向量 (2,1) → 广播到 (2,3)
c = np.array([[100], [200]])
print(a + c)
```

### 7. 改变形状与拼接
```python
arr.reshape(3, 2)      # 返回新形状的视图（不改变原数组）
arr.resize(3, 2)       # 原地改变形状

np.vstack([a, b])      # 垂直拼接（加行）
np.hstack([a, b])      # 水平拼接（加列）
```

### 8. ufunc 与统计函数
ufunc 是一种对数组逐元素运算的快速函数。

```python
np.sqrt(arr)      # 逐元素开方
np.exp(arr)       # 逐元素指数

arr.mean()        # 均值
arr.std()         # 标准差
arr.min()         # 最小值
arr.argmax()      # 最大值的索引
```

---

## 三.Matplotlib

掌握了**图形是画在画布（Figure）的坐标系（Axes）上的**，就大概掌握了 Matplotlib。

### 1. 两种绘图风格
pyplot和面向对象风格，后者的行为更加可控，推荐使用。

```python
import matplotlib.pyplot as plt
import numpy as np

# pyplot 风格
plt.plot([1,2,3], [1,4,9])
plt.show()

# 面向对象风格
fig, ax = plt.subplots()
ax.plot([1,2,3], [1,4,9])
ax.set_title("My Plot")
ax.set_xlabel("X")
ax.set_ylabel("Y")
plt.show()
```

### 2. 绘图三步流程
1. **创建画布**：`fig = plt.figure()` 或 `fig, ax = plt.subplots()`
2. **添加内容**：`ax.plot()`, `ax.bar()`, 设置标题/轴标签等
3. **保存与显示**：`plt.savefig()` / `plt.show()`

### 3. 常用图表速查

#### 折线图
```python
plt.plot(x, y, 'ko--')  # 'k'=黑色, 'o'=圆点标记, '--'=虚线
# 更清晰的写法：
plt.plot(x, y, color='green', linestyle='--', marker='s')
```
格式字符串记忆：`[颜色][标记][线型]`

#### 散点图
```python
plt.scatter(x, y, s=area, c=colors, alpha=0.6)
```
可设置点的大小(s)、颜色(c)、透明度(alpha)。

#### 柱状图
```python
plt.bar(x, height, width=0.8, color='black')
plt.barh(y, width)  # 水平条形图
```

#### 直方图
```python
plt.hist(data, bins=20)  # bins=分组数
```

#### 饼图
```python
plt.pie(sizes, labels=labels, autopct='%1.1f%%',
        explode=explode, startangle=90)
```

### 4. 多子图
三种创建方式，按需选择：

- **`plt.subplots()`（推荐）**：一次创建规则网格
```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8))
axes[0, 0].plot(x, y)
axes[0, 1].bar(x, height)
plt.tight_layout()  # 自动调整间距
```

- **`plt.subplot(行,列,序号)`**：逐个添加，编号从 1 开始
```python
plt.subplot(211)  # 2行1列第1个
plt.plot(x, y1)
plt.subplot(212)
plt.plot(x, y2)
```

- **`fig.add_subplot(行,列,序号)`**：先生成 Figure，再添加子图
```python
fig = plt.figure()
ax1 = fig.add_subplot(2, 1, 1)
ax1.plot(x, y)
```

### 5. 中文显示
```python
# 方法1：全局设置字体
plt.rcParams['font.family'] = 'Heiti TC'

# 方法2：局部设置（仅指定位置）
plt.xlabel("横轴", fontproperties='PingFang HK')
```

### 6. 颜色、线型、标记速查

| 颜色 | 代码 | 线型 | 代码 | 标记 | 代码 |
|------|------|------|------|------|------|
| 蓝   | b    | 实线 | -    | 圆点 | o    |
| 绿   | g    | 虚线 | --   | 方块 | s    |
| 红   | r    | 点划线 | -. | 三角 | v    |
| 黑   | k    | 点线 | :    | 星号 | *  |

---

## 四.pandas

掌握了**一切数据操作，本质上都是在处理索引（Index）对齐**，就大概掌握了 pandas。

### 1. 两大核心结构

```python
import pandas as pd
import numpy as np

# Series：带标签的一维数组
s = pd.Series([3, 5, 7], index=['a', 'b', 'c'])
s = pd.Series({'Ohio': 350, 'Texas': 710})  # 字典创建，键=索引

# DataFrame：带行列标签的二维表格
data = {'name': ['Mayue', 'Lilin', 'Wuyun'],
        'pay': [3000, 4500, 8000]}
df = pd.DataFrame(data)
```

属性和方法速览：`df.index`（行索引）、`df.columns`（列索引）、`df.values`（值数组）、`df.head()`（前5行）、`df.info()`（类型概览）、`df.describe()`（数值统计摘要）。

### 2. 核心理解：索引对齐
两个对象运算时，是按**标签对齐**的，不匹配的位置会产生 `NaN`。

```python
aSer = pd.Series({'AXP': '86.40', 'CSCO': '122.64'}, index=['AXP', 'CSCO', 'BA'])
bSer = pd.Series({'AXP': '86.40', 'CSCO': '122.64', 'CVX': '23.78'})
print(aSer + bSer)
# AXP     86.4086.40
# CSCO    122.64122.64
# BA      NaN           ← 没有对齐，产生缺失值
# CVX     NaN
```
先对齐，再计算——这就是 pandas 一切运算的底层逻辑。

### 3. 数据选择： loc 与 iloc
```python
# 选列（三种方式）
df['name']      # 返回 Series
df[['name', 'pay']]  # 返回 DataFrame

# 选行（按位置或标签切片）
df[0:3]         # 按位置
df['a':'c']     # 按标签（包含末尾！）

# 选区域（最常用）
df.loc['b':'d', '语文':'英语']   # 按标签（包含末尾）
df.iloc[1:4, 1:4]               # 按位置（不包含末尾）

# 选单个值
df.at['b', '数学']    # 标签
df.iat[1, 2]          # 位置
```

**条件筛选**（布尔索引）：
```python
df[(df.index >= 'b') & (df.index <= 'd') & (df.数学 >= 90)]
```
注意：多个条件用 `&`（与）、`|`（或），每个条件必须加括号。

### 4. 修改数据：增·删·改
#### 增
```python
df['tax'] = [0.05, 0.05, 0.1]  # 新增一列
df.loc[5] = {'name': 'Liuxi', 'pay': 5000}  # 新增一行
df2 = pd.concat([df1, df2])    # 或 append（拼接）
```

#### 删
```python
del df['tax']             # 直接删列（修改原数据）
df.drop(5)                # 删行（默认 axis=0，返回新对象）
df.drop('tax', axis=1)    # 删列（axis=1 或 axis='columns'）
```

#### 改
```python
df['tax'] = 0.03          # 整列赋值标量
df.loc[5] = ['Liuxi', 9800, 0.05]  # 修改某行
```
注意：DataFrame 的选取通常是**视图**，修改会反映到原数据上。

drop的选取是视图，不会改变原数据。

del的选取会改变原数据。

### 5. 数据对齐与合并
```python
# 内部运算自动对齐
df1 + df2  # 返回行列并集，不匹配处为 NaN

# 合并（类似 SQL JOIN）
pd.merge(left, right, on='key', how='left')
```
how 参数就是 SQL 的连接类型：left、right、inner、outer。

### 6. 描述性统计
```python
df.describe()     # 一次性：count、mean、std、min、25%、50%、75%、max
df.mean(numeric_only=True)  # 每列均值
df.数学.mean()    # 单列均值
```

数值统计速查：`min`、`max`、`mean`、`median`、`std`、`var`、`ptp`（极差）、`quantile`（四分位数）、`mode`（众数）、`skew`（偏度）、`kurt`（峰度）。

类别型统计：
```python
obj.value_counts()  # 频数统计
obj.describe()      # count, unique, top, freq
```

### 7. 排序与秩次
```python
df.sort_values(by='总分')                # 按单列排序
df.sort_values(by=['语文', '英语'])       # 多列排序
df.rank()                                # 返回每列的排名
```

### 8. 数据读写
```python
# CSV
df = pd.read_csv('score.csv')
df.to_csv('score_copy.csv')

# Excel
df = pd.read_excel('users.xlsx')
df.to_excel('users_copy.xlsx', sheet_name='users1')
```

### 9. 用 pandas 快速绘图
DataFrame 自带 `plot` 属性，本质是对 Matplotlib 的封装：
```python
df.plot()                       # 默认折线图，索引作为 x 轴
df.plot.bar()                   # 柱状图
df.plot.pie(subplots=True)      # 饼图
df.plot.hist(bins=20)           # 直方图
```
快速探索数据时很方便，精细控制仍需 Matplotlib。

### 10. 分组聚合
pandas 的分组操作对应 SQL 的 `GROUP BY`：
```python
df.groupby('部门')['工资'].mean()       # 按部门分组，求平均工资

df.groupby('部门').agg({              # 多列多操作
    '工资': 'mean',
    '年龄': 'max',
    '工号': 'count'
})
```
**split-apply-combine**：先按规则分组（split），对每组应用函数（apply），结果自动合并（combine）。

---