# 二维DP

---

## 一、核心模板

### 1、数字三角形（路径权值最大和） P1216

**思路：**

- 将三角形用下三角矩阵存储，简单的递推
- 对于每一行，需要单独讨论第一个（只来自于本列上一行）和最后一个元素（只来自于左上角），中间元素取左上和上面中较大的一个作为来源

```
def mx_trg(trg):
    n = len(trg)
    dp = [[0]*(i+1) for i in range(n)]   # 用对角矩阵存储
    dp[0] = trg
    for i in range(1,n):                 # 排开第一行
        for j in range(i+1):             # 每行元素个数刚好是 i+1
            if j == 0:                   # 第一个元素没有左上方元素
                dp[i][j] = dp[i-1][j] + trg[i][j]
            elif j == i:                 # 最后一个元素正上方没有元素
                dp[i][j] = dp[i-1][j-1] + trg[i][j]
            else:
                dp[i][j] = max(dp[i-1][j-1], dp[i-1][j]) + trg[i][j]
    return max(dp[n-1])                  # 最后一行取最大的一个，就是路径最大值
```

---

### 2、过河兵（路径计数，障碍物） P1002

**思路：**

- 给每个位置设置可到达为True，然后单独标记障碍区为True
- 从(0,0)开始，每个位置路径数由左、上两个方向传来
- 因为从前往后，每个位置必然只会遍历一次，到达上面点和左边点的路径数已经确定，直接两个方向传递过来即可（如果不是边界的话）

```
def hr_pth(n,m,hx,hy):
    dx = [-2,-2,-1,-1,0,1,1,2,2]
    dy = [1,-1,2,-2,0,2,-2,1,-1]
    bck = [[False]*(m+1) for _ in range(n+1)]
    for i in range(9):
        x,y = hx+dx[i], hy+dy[i]
        if 0<=x<=n and 0<=y<=m:
            bck[x][y] = True
    d = [[0]*(m+1) for _ in range(n+1)]
    d[0] = 1 if not bck[0] else 0
    for i in range(n+1):
        for j in range(m+1):
            if bck[i][j]:
                continue
            if i>0:
                d[i][j] += d[i-1][j]
            if j>0:
                d[i][j] += d[i][j-1]
    return d[n][m]

n,m,hx,hy = map(int,input().split())
print(hr_pth(n,m,hx,hy))
```

这里对于边界的处理还是巧妙，不是直接一步加左右两个方向。因为是计算路径数，没有先后顺序，可以拆成两个方向，单独判断是否是边界，再加对应方向路径情况。

---

### 3、最长公共子序列（LCS） P2543

**核心思路：**

- 设置二维dp数组，下标(i,j)表示分别截取两个串对应长度时（即0-i和0-j）所得的最大公共子序列长度
- 两层循环遍历所有截取情况（至少截取1，即都从1开始）
- 如果 s1[i-1]==s2[j-1]，则 dp[i][j]=dp[i-1][j-1]+1
- 否则 dp[i][j]=max(dp[i-1][j], dp[i][j-1])

```
def LCS(s1,s2):
    n,m = len(s1), len(s2)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(1,n+1):
        for j in range(1,m+1):
            if s1[i-1]==s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[n][m]

s1,s2 = input().split()
print(LCS(s1,s2))
```

因为数据范围较大，dp占用空间可能超限，需要优化为一维滚动数组。

#### 改进：一维滚动数组（空间优化）

**思路：**观察状态转移，只用到左、上、左上三个值。将内层循环的结果用一维数组存储，使用变量 `pr` 记录左上角的值。

```
def pl_sq(s1,s2):
    m,n = len(s1), len(s2)
    dp = [0]*(n+1)
    for i in range(1,m+1):
        pr = 0                # 第一列左上角为0
        for j in range(1,n+1):
            tp = dp[j]        # 保存未更新前的当前列（上一行）
            if s1[i-1] == s2[j-1]:
                dp[j] = pr + 1
            else:
                dp[j] = max(dp[j-1], dp[j])   # 左方（已更新）和上方（未更新）
            pr = tp            # 更新左上角为下一列的上一行
    return dp[n]

s1,s2 = input().split()
print(pl_sq(s1,s2))
```

关键：使用一个变量代替了大量无用的二维存储，巧妙利用更新前后值的不同含义简化存储。

#### 变种：排列的最长公共序列（LCS转LIS） P1439

**核心思路：**

- 因为两个序列都是排列，数字集相同且无重复，可以建立一个映射：`l[s2[i]] = i`
- 将 s1 中的每个数字翻译成它在 s2 中的位置，得到一个位置序列
- 求这个位置序列的最长上升子序列（LIS）即可得到原问题的 LCS 长度

```
import bisect

n = int(input())
s1 = list(map(int,input().split()))
s2 = list(map(int,input().split()))
pos = [0]*n
for i in range(n):
    pos[s2[i]-1] = i
seq = [pos[x-1] for x in s1]

def lis_len(seq):
    d = []
    for x in seq:
        idx = bisect.bisect_left(d, x)
        if idx == len(d):
            d.append(x)
        else:
            d[idx] = x
    return len(d)

print(lis_len(seq))
```

巧妙+抽象！从架构层面将二维问题降维成一维最长上升问题。条件是两个序列数字集相等且互不相同（排列）。即使不是排列，只要映射关系存在，都可以转化为 LIS。

---

### 4、最少字符串操作（编辑距离） P2758

**思路：**

- 与 LCS 类似，但初始值需要调整（因为可以删除）
- `dp[i][0] = i`（删除 i 次），`dp[j] = j`
- 状态转移：如果 s1[i-1]==s2[j-1]，则 `dp[i][j]=dp[i-1][j-1]`；否则取三种操作的最小值再加1

```
def ed_ch(s1,s2):
    m,n = len(s1), len(s2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1,m+1):
        dp[i][0] = i
    for j in range(1,n+1):
        dp[j] = j
    for i in range(1,m+1):
        for j in range(1,n+1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1
    return dp[m][n]

s1 = input().strip()
s2 = input().strip()
print(ed_ch(s1,s2))
```

抽象！主要是如何表示三种操作对状态转换的影响。看到两个串，就要想到状态转换和公共序列操作。

#### 改进：一维滚动数组

```
def ed_ch_1d(s1,s2):
    m,n = len(s1), len(s2)
    dp = [i for i in range(n+1)]   # 第一行初始值
    for i in range(1,m+1):
        pr = i-1                    # 左上角初始为 dp[i-1]
        dp[0] = i                   # 每行第一列
        for j in range(1,n+1):
            tp = dp[j]
            if s1[i-1] == s2[j-1]:
                dp[j] = pr
            else:
                dp[j] = min(dp[j], dp[j-1], pr) + 1
            pr = tp
    return dp[n]

print(ed_ch_1d(s1,s2))
```

---

### 5、二维背包问题 P1507

**核心：**类似于一维01背包，多一个维度，对应多一层循环。注意循环顺序与下标对应。

```
def tw_dms_zrPck(V,W,n):
    dp = [[0]*(W+1) for _ in range(V+1)]   # 外层是体积（行），内层是重量（列）
    for k in range(n):
        v,w,e = map(int,input().split())
        for i in range(V, v-1, -1):
            for j in range(W, w-1, -1):
                dp[i][j] = max(dp[i-v][j-w] + e, dp[i][j])
    return max(max(row) for row in dp)

# 变种：二维完全背包（内层循环改为正序）
def tw_dms_cmpt(V,W,n):
    dp = [[0]*(W+1) for _ in range(V+1)]
    for k in range(n):
        v,w,e = map(int,input().split())
        for i in range(v, V+1):
            for j in range(w, W+1):
                dp[i][j] = max(dp[i-v][j-w] + e, dp[i][j])
    return dp[V][W]
```

---

### 6、矩阵取数（方格取数） P1004

**思路：**从左上走到右下走两次，可以看成两个人同时从左上出发，速度相同，用同一步数。用三维dp：步数k，第一个人的横坐标i，第二个人的横坐标j（纵坐标可由步数-横坐标得到）。转移时考虑四种前驱状态，并累加当前两个位置的值（如果重合只加一次）。

```
n = int(input())
grd = [[0]*(n+1) for _ in range(n+1)]   # 多开一层避免边界处理

def cnt_mx():
    # dp[k][i][j] ：步数k，第一个人横坐标i，第二个人横坐标j
    dp = [[[0]*(n+1) for _ in range(n+1)] for _ in range(2*n+1)]
    for f in range(2, 2*n+1):            # 步数从2开始（因为第一步已经走）
        for i in range(1, n+1):
            y1 = f - i
            if y1 < 1 or y1 > n:
                continue
            for j in range(1, n+1):
                y2 = f - j
                if y2 < 1 or y2 > n:
                    continue
                val = grd[i][y1]
                if i != j:
                    val += grd[j][y2]
                dp[f][i][j] = max(
                    dp[f-1][i-1][j-1],   # 两人都从左来
                    dp[f-1][i-1][j],     # 左、上
                    dp[f-1][i][j-1],     # 上、左
                    dp[f-1][i][j]        # 都从上
                ) + val
    return dp[2*n][n][n]

# 读取输入
while True:
    line = input().strip()
    if not line:
        break
    x,y,v = map(int,line.split())
    if x==0 and y==0 and v==0:
        break
    grd[x][y] = v

print(cnt_mx())
```

关键：用相同的步长表示两次移动，将先后两次走的问题转换成同时两个人同样速度走，公用同一个步长，很巧妙。

#### 另外的栈（BFS）实现

```
from collections import deque

def sta_sqa():
    S = 2*n-2
    dp = [[[-1]*n for _ in range(n)] for _ in range(S+1)]
    q = deque([(0,0,0)])     # (步数, 第一个横坐标, 第二个横坐标)
    dp[0] = grd
    while q:
        cs,fx,sx = q.popleft()
        fy = cs - fx
        sy = cs - sx
        cm = dp[cs][fx][sx]
        # 两种移动：向右(0) 或 向下(1)
        for i in [0,1]:
            nfx = fx + i
            nfy = fy + (1-i)
            if nfx>=n or nfy>=n: continue
            val = grd[nfx][nfy]
            for j in [0,1]:
                nsx = sx + j
                nsy = sy + (1-j)
                if nsx>=n or nsy>=n: continue
                if nfx != nsx or nfy != nsy:
                    val += grd[nsx][nsy]
                if cm + val > dp[cs+1][nfx][nsx]:
                    dp[cs+1][nfx][nsx] = cm + val
                    if cs+1 == S: continue
                    q.append((cs+1, nfx, nsx))
    return dp[S][n-1][n-1]
```

---

### 7、标准Young表（计数动规）

**问题：**将1~2020个数放到2×1010的矩阵中，满足右大于左、下大于上，求放法数。

**思路：**将选数问题转化为按序选数放入合适位置。用dp[i][j]表示第一行放了i个数，第二行放了j个数的方案数。要求i≥j（保证第二行可以放），转移：放上面行时dp[i+1][j] += dp[i][j]；放下面行时（i>j）则dp[i][j+1] += dp[i][j]。

```
def sol():
    n = 1010
    dp = [[0]*(n+1) for _ in range(n+1)]
    dp[0] = 1
    for i in range(n+1):
        for j in range(n+1):
            if i < j: continue
            if i+1 <= n:
                dp[i+1][j] = (dp[i+1][j] + dp[i][j]) % 2020
            if j+1 <= n and i > j:
                dp[i][j+1] = (dp[i][j+1] + dp[i][j]) % 2020
    print(dp[n][n] % 2020)
```

关键还是将随机选数填表，根据题目限制，巧妙转换为按序取数从左到右放入表，而不需要考虑当前取得数，只需要关心当前位置来数后对应状态方法的继承，有意思。

---



© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议