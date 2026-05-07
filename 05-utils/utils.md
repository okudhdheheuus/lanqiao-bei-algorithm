# 蓝桥杯临考函数知识点

整理自个人笔记，涵盖常用模块与技巧，便于考前快速回顾。

---

## 一、datetime 模块

### 1. 日期遍历与判断

**功能：**

从起始日期到结束日期，统计每个月的第一天是星期一的次数。

```
import datetime

st = datetime.date(2000, 1, 1)
ed = datetime.date(2025, 1, 1)
dy = datetime.timedelta(days=1)
ans = 0
while st <= ed:
    if st.day == 1 and st.isoweekday() == 1:   # isoweekday(): 1~7
        ans += 1
    st += dy
print(ans)
```

**补充：**

- `st.weekday()` 返回 0~6（0 表示周一）
- `st.isoweekday()` 返回 1~7（1 表示周一）
- 获取年、月、日：`st.year`，`st.month`，`st.day`

### 2. 计算间隔天数

```
st = datetime.date(2000, 1, 1)
ed = datetime.date(2025, 1, 1)
delta = ed - st                     # 得到 timedelta 对象
print(delta.days)                   # 包含最后一天
```

### 3. 格式化与解析日期字符串

```
d = ed.strftime('%Y-%m-%d')        # 日期 -> 字符串 "2020-10-01"
d2 = datetime.datetime.strptime("2020-10-01", "%Y-%m-%d").date()  # 字符串 -> 日期
```

---

## 二、Fraction 模块（分数处理）

### 1. 导入与创建分数

```
from fractions import Fraction
a = Fraction(1, 3)            # 分子, 分母
b = Fraction('1/3')           # 字符串
```

注意：使用 Fraction 容易超时（大数据下），建议用元组手动化简。

### 2. 手动化简分数（推荐）

```
import math

def reduce_frac(num, den):
    """将分数化为最简，分母为正"""
    if den < 0:
        num, den = -num, -den
    g = math.gcd(abs(num), den)
    return (num // g, den // g)
```

---

## 三、杨辉三角与组合数

- 杨辉三角第 n 行第 m 列（从 0 开始）对应组合数 `C(n, m)`
- 每行最大值为 `C(n, n//2)`

---

## 四、因数与分解

### 1. 质因数分解（试除法）

```
def prime_factors(num):
    k = 2
    factors = []
    while k * k <= num:
        if num % k == 0:
            factors.append(k)
            while num % k == 0:
                num //= k
        else:
            k += 1
    if num > 1:
        factors.append(num)
    return factors
```

### 2. 由质因数求所有因子

```
s = set([1](@ref)
for p in prime_factors(n):
    mul = set()
    for x in s:
        mul.add(p * x)
    s.update(mul)
# 此时 s 包含 n 的所有正因子
```

### 3. 枚举有序对 (a, b) 满足 n % (a*b) == 0

```
ans = 0
for s1 in s:
    for s2 in s:
        if n % (s1 * s2) == 0:
            ans += 1
```

### 4. 质因子个数（不重复）

```
n = int(input())
ans = 0
k = 2
while k * k <= n:
    if n % k == 0:
        ans += 1
        while n % k == 0:
            n //= k
    else:
        k += 1
if n > 1:
    ans += 1
```

---

## 五、余数运算

- `(a + b) % m = (a % m + b % m) % m`
- `(a * b) % m = (a % m * b % m) % m`
- `(a ** b) % m = (a ** (b % m)) % m` （注意适用范围：当 m 为质数或特殊情况下可用，否则用快速幂取模）

---

## 六、中国剩余定理（模板）

**题型：** 给定一组除数 `dvs` 和对应余数 `rms`，求最小的正整数 `n` 满足所有条件。

```
import math

dvs = [...]   # 除数列表（两两互质？非必须，但需保证有解）
rms = [...]   # 余数列表
st = dvs[0] + rms[0]   # 初始值（保证第一个条件成立）
cd = dvs[0]            # 当前最小公倍数
for i in range(1, len(dvs)):
    while st % dvs[i] != rms[i]:
        st += cd
    cd = dvs[i] * cd // math.gcd(dvs[i], cd)
print(st)
```

精髓：每次满足一个新条件后，更新最小公倍数，后续步长变为新的公倍数，保证以前的条件不会被破坏。

---

## 七、差分数组（区间覆盖统计）

```
n = len(nums)
diff = [0] * (n + 2)   # 多开空间防止越界
for _ in range(q):
    l, r = map(int, input().split())
    diff[l] += 1
    diff[r + 1] -= 1
for i in range(1, n + 1):
    diff[i] += diff[i - 1]         # 此时 diff[i] 就是 i 点的覆盖次数
diff.sort(reverse=True)            # 从大到小排序
```

---

## 八、异或和

### 基础性质

- `a ^ a = 0`
- `a ^ 0 = a`
- 对特定位取反：`a ^ b`（b 中对应位为 1）
- 交换两数：`a ^= b; b ^= a; a ^= b`

### 1~n 连续整数异或和的规律

```
def xor_1_to_n(n):
    if n % 4 == 0: return n
    elif n % 4 == 1: return 1
    elif n % 4 == 2: return n + 1
    else: return 0
```

### 数组划分异或相等

若能将数组分成两部分使得各自异或和相等，则整个数组异或和为 0。

---

## 九、collections 模块

### 1. Counter

```
from collections import Counter

ct1 = Counter('adsa')
ct2 = Counter('aasa')
print(ct1 == ct2)       # False，判断是否是异位词

# 按词频从高到低排序，同频按字典序
words = ["apple", "banana", "apple", "orange", "banana", "apple"]
s = Counter(words)
result = sorted(s.items(), key=lambda x: (-x[1], x[0](@ref))
```

### 2. defaultdict

```
from collections import defaultdict
dis = defaultdict(int)               # 默认值 0
dis2 = defaultdict(lambda: -1)      # 自定义默认值 -1
```

### 3. deque（滑动窗口最大值）

```
from collections import deque

nums = [2,4,3,1,5,8,4,3]
k = 3

def max_sliding_window(nums, k):
    res = []
    dq = deque()
    for i, v in enumerate(nums):
        # 移除超出窗口的队首
        if dq and dq[0] <= i - k:
            dq.popleft()
        # 保持队列单调递减
        while dq and nums[dq[-1]] <= v:
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            res.append(nums[dq[0]])
    return res
```

### 4. 单调栈（找右侧第一个更大元素）

```
def next_greater(nums):
    stk = []
    res = [-1] * len(nums)
    for i, v in enumerate(nums):
        while stk and nums[stk[-1]] < v:
            idx = stk.pop()
            res[idx] = v
        stk.append(i)
    return res
```

单调栈/队列的思想很相似：新元素进来时，从尾部弹出不满足单调性的元素，然后加入新元素，保证容器内元素始终有序。这种动态维护的思路很适合处理“最近更大/更小”类问题。

---

## 十、维护两个最大值

```
a, b = 0, 0
for n in nums:
    if n > a:
        b = a
        a = n
    elif n > b:
        b = n
```

---

## 十一、二进制操作

- 左移 `<<`，右移 `>>`
- 获取二进制长度：`a.bit_length()`

---

## 十二、常见小技巧

- **判断回文：**`s == s[::-1]`
- **列表去重保持顺序：**`list(dict.fromkeys(lst))`
- **二维列表转置：**`list(zip(*matrix))`
- **前缀和：**`pre[i] = pre[i-1] + arr[i]`
- **快速幂取模：**

```
def qpow_mod(a, b, mod):
    res = 1
    while b:
        if b & 1:
            res = (res * a) % mod
        a = (a * a) % mod
        b >>= 1
    return res
```

---

© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议