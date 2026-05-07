# 二分查找算法详解

本文档总结了二分查找的多种变体及其应用，包括精确查找、边界查找、旋转数组搜索、峰值查找、降序查找、二分答案、开方运算和方程求根。每种方法均包含核心思想、边界条件、代码模板及注意事项。

---

## 1. 基础模板

- **通用结束条件**：`left = right + 1`（即 `while lf <= rt`）
- **中间值计算**：`mid = lf + (rt - lf) // 2`（防止整数溢出）

---

## 2. 精确查找（查找特定元素）

### 2.1 目标

在有序数组中查找等于目标值 `tar` 的下标，不存在则返回 -1。

### 2.2 流程

- 初始左右指针：`lf = 0`，`rt = n-1`
- 循环 `lf <= rt`：
    - 若 `ans[mid] == tar`，直接返回 `mid`
    - 若 `ans[mid] < tar`，目标在右半部分，`lf = mid + 1`
    - 否则 `rt = mid - 1`
- 循环结束返回 -1

### 2.3 代码

```
def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] == tar:
            return mid
        elif ans[mid] < tar:
            lf = mid+1
        else:
            rt = mid-1
    return -1

def main():
    ans = [5,9,13,14,15,20]
    tar = 13
    print(bdS_exa(ans,tar))

main()
```

---

## 3. 查找左边界（第一个 ≥ 目标值的位置）

### 3.1 目标

返回第一个满足 `ans[i] >= tar` 的下标（下界）。若所有元素均小于 `tar`，则返回 `len(ans)`（最大下标+1）。

### 3.2 流程

- 循环 `lf <= rt`：
    - 若 `ans[mid] < tar`，向右缩小：`lf = mid + 1`
    - 否则（`ans[mid] >= tar`），向左缩小：`rt = mid - 1`
- 循环结束返回 `lf`

### 3.3 代码

```
def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] < tar:
            lf = mid+1
        else:
            rt = mid-1
    return lf

def main():
    ans = [5,13,13,14,15,20]
    tar = 12
    print(bdS_exa(ans,tar))

main()
```

---

## 4. 查找右边界（最后一个 ≤ 目标值的位置）

### 4.1 目标

返回最后一个满足 `ans[i] <= tar` 的下标（上界）。若所有元素均大于 `tar`，则返回 -1。

### 4.2 流程

- 循环 `lf <= rt`：
    - 若 `ans[mid] <= tar`，向右缩小：`lf = mid + 1`
    - 否则（`ans[mid] > tar`），向左缩小：`rt = mid - 1`
- 循环结束返回 `rt`

### 4.3 代码

```
def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] <= tar:
            lf = mid+1
        else:
            rt = mid-1
    return rt

def main():
    ans = [5,13,13,14,15,20]
    tar = 13
    print(bdS_exa(ans,tar))

main()
```

### 4.4 边界总结

- **左边界**：相等情况归入左半部分，返回 `lf`
- **右边界**：相等情况归入右半部分，返回 `rt`
- **精确查找**：相等时直接返回 `mid`，否则 -1

---

## 5. 旋转排序数组的搜索

### 5.1 问题描述

一个升序数组在某个未知点旋转（例如 `[4,5,6,7,0,1,2]`），搜索目标值。数组由两个有序段组成，前一段全部大于后一段。

### 5.2 核心思想

利用自相似性：每次划分后至少有一半是有序的，据此缩小搜索范围。

### 5.3 流程

1. 计算 `mid`
2. 若 `ans[mid] == tar`，直接返回
3. 若 `ans[lf] <= ans[mid]`（左半部分有序）：
    - 若 `tar` 在 `[ans[lf], ans[mid])` 内，则 `rt = mid - 1`
    - 否则 `lf = mid + 1`
4. 否则（右半部分有序）：
    - 若 `tar` 在 `(ans[mid], ans[rt]]` 内，则 `lf = mid + 1`
    - 否则 `rt = mid - 1`
5. 循环结束返回 -1

### 5.4 代码

```
def bd_rot(ans,tar):
    lf = 0
    rt = len(ans)-1
    while lf <= rt:
        mid = (lf+rt)//2
        if ans[mid] == tar:
            return mid
        if ans[lf] <= ans[mid]:
            if ans[lf] <= tar < ans[mid]:
                rt = mid - 1
            else:
                lf = mid + 1
        else:
            if ans[mid] < tar <= ans[rt]:
                lf = mid + 1
            else:
                rt = mid - 1
    return -1

ans = [4,5,6,1,2,3]
tar = 2
print(bd_rot(ans,2))
```

**个人理解**：实际是划成3部分处理，一部分是mid到对应上段左边界，一部分是mid到对应下段右边界，然后中间是一块包含断点的部分，这部分对于上段，用lf+1维护，对于下段用rt-1维护。

---

## 6. 峰值元素查找（局部最大值）

### 6.1 问题描述

在一个任意数组（相邻元素不相等）中找到一个峰值（大于左右邻居）。假设 `ans[-1] = -∞`，`ans[n] = -∞`。

### 6.2 核心思想

二分查找，比较 `ans[mid]` 与 `ans[mid+1]` 的大小。

### 6.3 流程

- 循环 `lf < rt`（防止死循环）
- 若 `ans[mid] > ans[mid+1]`，峰值在左半部分（含 mid），`rt = mid`
- 否则，峰值在右半部分，`lf = mid + 1`
- 循环结束返回 `lf`

### 6.4 代码

```
def peak_ele(ans):
    lf = 0
    rt = len(ans)-1
    while lf < rt:  # 不要用等于，最多就等于峰值处，用等于就让lf跑过去了
        mid = lf+(rt-lf)//2
        if ans[mid] > ans[mid+1]:
            rt = mid
        else:
            lf = mid + 1
    return lf

ans = [1,3,8,10,5,3,0,-10]
print(peak_ele(ans))
```

---

## 7. 降序数组的二分查找

### 7.1 问题描述

在严格递减的数组中查找目标值。

### 7.2 流程

- 循环 `lf <= rt`
- 若 `ans[mid] == tar`，返回 `mid`
- 若 `ans[mid] > tar`（递减序列中较大的值在左边），目标在右边（更小），所以 `lf = mid + 1`
- 否则 `rt = mid - 1`
- 循环结束返回 -1

### 7.3 代码

```
def dec_bdS(ans,tar):
    lf = 0
    rt = len(ans)-1
    while lf <= rt:
        mid = lf+(rt-lf)//2
        if ans[mid] == tar:
            return mid
        if ans[mid] > tar:
            lf = mid+1
        else:
            rt = mid-1
    return -1

ans = [57,51,43,37,21,1]
print(dec_bdS(ans,51))
```

---

## 8. 二分答案（最大值最小化 / 最小值最大化）

### 8.1 应用场景

常见于将数组分成 k 个子数组，求子数组和的最大值最小（或最小值最大）。

### 8.2 模板步骤

1. 确定答案范围：下界通常为 `max(arr)`，上界为 `sum(arr)`
2. 在范围上二分，每次取 `mid` 作为候选答案
3. 编写判断函数 `can(mid)`，检验是否能在约束下满足条件（如分段数 ≤ k）
4. 根据 `can(mid)` 的返回值调整边界
5. 最终返回 `lf` 或 `rt`（取决于具体实现）

### 8.3 示例：将数组分成最多 k 段，使每段和的最大值最小

```
def can_split(nums, k, mid):
    seg = 1
    cur_sum = 0
    for num in nums:
        if cur_sum + num > mid:
            seg += 1
            cur_sum = num
            if seg > k:
                return False
        else:
            cur_sum += num
    return True

def minimize_max_sum(nums, k):
    lf = max(nums)
    rt = sum(nums)
    while lf < rt:
        mid = (lf + rt) // 2
        if can_split(nums, k, mid):
            rt = mid
        else:
            lf = mid + 1
    return lf
```

---

## 9. 浮点数二分：求任意次方根

### 9.1 问题描述

求 `num` 的 `k` 次方根（以立方根为例），精度为 `p`。

### 9.2 初始区间

- 若 `num > 1`：`[1, num]`
- 若 `0 < num < 1`：`[0, 1]`
- 若 `num = 0`：直接返回 0

### 9.3 流程

- 循环 `lf < r`（注意使用 `<` 防止死循环）
- 计算 `mid = (lf + r) / 2`
- 计算 `mid^k`，若与 `num` 的差值小于 `p`，返回 `mid`
- 若 `mid^k < num`，则 `lf = mid`，否则 `r = mid`
- 循环结束时返回 `lf`

### 9.4 代码（立方根示例）

```
def sqrtbinary(num, p):
    if num < 0:
        return None
    elif num == 0:
        l = 0
        r = 0
    elif num > 1:
        l = 1
        r = num
    else:
        l = 0
        r = 1
    while l < r:
        mid = (l + r) / 2
        curnum = mid ** 3
        if abs(curnum - num) < p:
            return mid
        elif curnum < num:
            l = mid
        else:
            r = mid
    return l

num = 8
print(sqrtbinary(num, 1e-07))
```

---

## 10. 二分法找零点（方程求根）

### 10.1 问题描述

求一元三次方程 `f(x)=ax³+bx²+cx+d=0` 在区间 `[-100, 100]` 内的实根。假设两根之间的距离 ≥ 1。

### 10.2 方法

以整数步长遍历区间（每隔 1），对每个子区间 `[i, i+1]`：

- 检查端点函数值是否异号（`f(i) * f(i+1) ≤ 0`），若同号则跳过
- 若异号，在区间内二分，直到区间长度小于精度 `1e-7`
- 记录中点作为近似根（注意去重，可用 `set`）

### 10.3 代码

```
a,b,c,d = list(map(float,input().split()))
res = set()

def cr_seg(i):
    lf = i
    rt = i+1
    frt = a*pow(rt,3)+b*pow(rt,2)+c*rt+d
    flf = a*pow(lf,3)+b*pow(lf,2)+c*lf+d
    while rt-lf > 1e-7:
        mid = (lf+rt)/2
        fm = a*pow(mid,3)+b*pow(mid,2)+c*mid+d
        if flf*frt > 0:
            return
        if flf*fm > 0:
            lf = mid
            flf = fm
        else:
            rt = mid
            frt = fm
    res.add(round(mid,2))

for i in range(-100,100):
    cr_seg(i)

res = list(res)
res.sort()
print(' '.join(f'{i:.2f}' for i in res if i<101))
```

### 10.4 测试用例

- 输入：`1 -5 -4 20` → 输出：`-2.00 2.00 5.00`
- 输入：`1 -4.65 2.25 1.4` → 输出：`-0.35 1.00 4.00`

---

## 11. 总结表

| 场景 | 循环条件 | 相等时归属 | 返回值 |
| --- | --- | --- | --- |
| 精确查找 | lf <= rt | 直接返回 mid | mid 或 -1 |
| 左边界（≥） | lf <= rt | 左半部分（rt = mid-1） | lf |
| 右边界（≤） | lf <= rt | 右半部分（lf = mid+1） | rt |
| 旋转搜索 | lf <= rt | 直接返回 mid | mid 或 -1 |
| 峰值查找 | lf < rt | 无相等情况 | lf |
| 降序查找 | lf <= rt | 直接返回 mid | mid 或 -1 |
| 二分答案 | lf <= rt | 根据判断函数调整 | 视题目而定 |
| 浮点数开方/求根 | lf < r | 精度满足时返回 | 近似根 |

**关键提示**：

- 浮点数二分常用 `lf < r` 或 `rt - lf > eps` 防止死循环。
- 整数二分中，当 `lf = mid` 时容易死循环，需注意 `mid` 向上或向下取整。
- 边界返回值 `lf` 或 `rt` 需结合循环结束时的状态理解（`lf` 是第一个满足条件的，`rt` 是最后一个满足条件的）。


---

© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议