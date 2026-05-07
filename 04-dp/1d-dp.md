# 一维DP

---

## 一、核心模板

### 1、线性递推类（斐波那契式递推）

**核心思路：**

- 利用后面情况值完全由前面已经确定值递推确定
- i层之前的情况数必然已经推出，i层的走法由i-1，i-2，i-3，...，i-k（或者0）层推得，是这些情况的和

```
MOD = 100003

def clb(n,k):
    dp = [0]*(n+1)
    dp[0] = 1 # 保证当站在第0层的时候也算一种情况，不然无法推着走
    for i in range(n+1):
        mxk = min(i,k) # 可能当前所站层数小于k(从1开始)，不用跨k步就能到
        for j in range(1,mxk+1): # 从1开始
            dp[i] = (dp[i]+dp[i-j])%MOD
    return dp[n]

n,k = map(int,input().split())
print(clb(n,k))
```

关键这个dp[0]=1需要记一下。

---

### 2、最大子段和（Kadane算法）

**核心思路：**

- 以当前数字结尾的前面部分数字的最大段和必然等于前一个数字记录最大段和加上当前数字与当前数字进行比较取大的那个结果
- 之所以和还不一定最大，是因为前面那个数记录的最大段和可能为负，所以需要判断
- 用一个变量动态更新最大段和记录

```
def mxSubArray(nms):
    dp = [0]*len(nms)
    dp[0] = nms
    mx_su = dp
    for i in range(1,len(nms)):
        dp[i] = max(nms[i],dp[i-1]+nms[i])
        mx_su = max(mx_su,dp[i])
    return mx_su
```

因为只用到了dp[i-1]，所以可以把dp[i-1]用一个中间变量cur表示：

```
def mxSubArray(nms):
    cr = nms
    mx_su = nms
    for i in range(1,len(nms)):
        cr = max(nms[i],cr+nms[i])
        mx_su = max(cr,mx_su)
    return mx_su
```

这里关键就是找到状态转移方程，还是挺抽象的，构建了以某一个数结尾的最大段和作为记录量，难得想到。

---

### 3、最长上升子序列（LIS）

**核心思路：**

- 和最大子段和类似，都是记录每个数字所在位置对应数组前面部分包含自身在内最大序列长
- 后面的位置某个数要求以它为最大值的前面最大序列长(小)就等于他前面所有小于它本身所有数字所在位置记录的序列最长记录的一个最大值
- 使用一个变量mx_su动态更新维护每个位置数字记录的它前面部分小于等于它本身的最长序列记录值的最大值

```
def mx_sul(nms):
    dp = [1]*len(nms) # 每个位置记录的自身前面部分小于等于自己的序列包含自身
    mx_su = 0
    for i in range(len(nms)):
        for j in range(i):
            if nms[i] > nms[j]:
                dp[i] = max(dp[i],dp[j]+1)
        mx_su = max(dp[i],mx_su)
    return mx_su

nms = list(map(int,input().split()))
print(mx_sul(nms))
```

关键还是需要找到用什么状态量化题意（直接对每个位置存储题目要求结果的值，还是可以记录别的最后简单处理），然后是用这种量化状态的方式能不能用程序语句表示对应状态转移方式。

#### 拓展：贪心+二分优化（O(n log n)）

**核心思路：**

- 使用一个数组d存储遍历数组过程中当前值能放到最大位置
- 如果取出数大于d数组每个值，就说明当前遍历数之前部分至少已经满足最长序列可以加1，则在数组d添加一个当前元素
- 否则找到当前遍历数在d数组中对应第一个大于等于遍历数的位置，并将d数组中这个位置的数替换为这个新的相对较小的遍历数
- 替换是保证当前组尽可能的可以保证继续添加新的元素

```
import bisect

def mx_sl(nms):
    d = []
    for x in nms:
        pos = bisect.bisect_left(d,x)
        if pos == len(d):
            d.append(x)
        else:
            d[pos] = x
    return len(d)
```

关键就是事先设定目标上升序列的记录组d，遍历原数组过程中，为了尽可能保证序列最长，目标记录组每个位置的数就应该尽可能小（虽然影响序列长的是添加元素步骤，只看最后一个数，但是前面数可能也可以有更好的选择，而使得后面出现完全不一样的序列，比如这组数34123它突然来一个1，并且后面三个数明显可以组成更好的情况），所以需要把遍历新得到的数放到合适位置，从而保证当前结果序列最后一个数可以调换的下限更小，从而达到尽可能增长d数组长度的目的。开始还以为他这个结果得到的d也是原数组对应顺序挑取的一个序列，实际并不是，比如2 3 1 4 5，最后得到的实际是1 3 4 5而不是实际的2 3 4 5，它这个1换到这里也好，换到第三个位置也好（如果可以的话）只是为了一步步服务于最后一个数（当前序列最大数）尽可能缩小，如果这里换成一个小的数比如3，那么最后一个5可选的调换数就可以是4了，这样再来一个5就直接追加到d数组尾部，数组长度加一，从而最长上升序列长度加一（因为实际只关心最终d数组长度和是否大于d最后一个数以及d最后一个数的更新维护）。 抽象至极尼玛。

#### 还原上升子序列（回溯前驱）

**核心思路：**

- 定义一个ix数组，专门动态记录当前d对应每个数在原数组位置
- 针对原数组每一个数，设置一个pre用于标记它的前一个数，默认-1，用于回溯
- d最后一个元素一定就是该最长上升序列的末尾数，可以作为回溯的起点
- 根据pre数组，使用cr遍历回溯即可

```
import bisect

def lis_seq(nms):
    d,ix = [],[]
    pr = [-1]*len(nms)
    for i,x in enumerate(nms):
        ps = bisect.bisect_left(d,x)
        if ps == len(d):
            d.append(x)
            ix.append(i)
        else:
            d[ps] = x
            ix[ps] = i
        if ps>0:
            pr[i] = ix[ps-1]
    rs = []
    cr = ix[-1]
    while cr != -1:
        rs.append(nms[cr])
        cr = pr[cr]
    return rs[::-1]

nms = list(map(int,input().split()))
print(*lis_seq(nms))
```

也是抽象。

#### 变种：p8776（替换连续一段为相同数，问最长不下降序列）

**核心思路：**使用idx列表标记最后不在序列中的下标，并标记为1，最后计算这些下标中连续的最长段即可。

```
import bisect
import sys

n,k = map(int,input().split())
ns = list(map(int,sys.stdin.readline().split()))
p = [0]*n
dp = []
idx = []
for i in range(n):
    ps = bisect.bisect_right(dp,ns[i]) # 找第一个大于x的下标
    if ps == len(dp):
        dp.append(ns[i])
        idx.append(i)
    else:
        dp[ps] = ns[i]
        p[idx[ps]] = 1 #标记不在序列中的i
        idx[ps] = i

su = [0]*(n+1)
rs = 0
for i in range(1,n+1):
    su[i] = su[i-1]+1 if p[i-1] != 0 else 0
    rs = max(su[i],rs)
rs = k if rs>=k else rs
print(rs+len(dp))
```

这个idx的使用还是挺抽象的。

---

## 二、背包问题

### 1、01背包（一维优化）

**概念：**对于物件列表每个物件，不能多次选，即选与不选的问题。

**核心思路：**

- 当前背包容量最大价值V，对于当前物件的选取，记录每一个加上该物件容量w[i]后背包当前容量d[i]对应价值
- 维护选取该物件后容量的对应价值，此时容量i最少为w[i]，最多为容量V
- 从后往前逆序遍历，对每一个容量i，对应价值更新为不选该物件之前背包容量对应价值(d[v-w[i]])加上选的该物件价值v[i]的和与自身已存价值d[i]的比较维护
- 最后每一个d记录的值，就是对应能选取的最大价值情况

```
def zer_pck(n,V,w,v):
    dp = [0]*(V+1)
    for i in range(n):
        cw = w[i]
        cv = v[i]
        for j in range(V,cw-1,-1):
            dp[j] = max(dp[j],dp[j-cw]+cv)
    return dp[V]
```

关键就是对于每一轮选取，前面的dp记录必然已经是那个容量在已经完成轮次选取的对应最大价值，感觉相当于倒着装，不正向思考剩的还能装多少对应价值变为多少，而是倒着看装了某物件后这个容量对应价值是否变大（所以必须逆序遍历，如果顺序遍历就是前面已经选了该物件，然后后面又用那个选了的最大价值更新，就错了，因为只能选一次）。还是挺神奇的。

---

### 2、完全背包（一维）

**概念：**相对于01背包，每一个物件可以多次选。

**核心思路：**

- 对于每一件物品，每一个当前可能容量状态对应价值等于不选该物件前容量状态记录价值加上当前物件价值
- 不同于01背包，在遍历容量状态时采用顺序遍历的方式
- 从前面所述01背包反面看，从后往前遍历保证用的是不选该物件前容量状态对应价值，对每一个状态更新从计算顺序上保证了选取的一次性
- 那么反过来顺着遍历，就是在计算时间上的自然延后，用于当前容量状态维护的不选该物件时对应容量对应价值就已经是完成选取与否判断后结果（即允许多次选取）

```
def cmpt_pck(n,V,w,v):
    dp = [0]*(V+1)
    for i in range(n):
        cw = w[i]
        cv = v[i]
        for j in range(cw,V+1):
            dp[j] = max(dp[j],dp[j-cw]+cv)
    return dp[V]
```

#### 优化（乘积分解法）——个人探索

如果背包容量过大，就比较费空间。对于所有物件重量如果存在一个非1最大公因数，可以对dp长度进行优化。自己又想了用所有物件乘积进行优化的方法，具体逻辑如下（经过大量随机测试验证正确）：

```
import math

def user_method(V, items):
    # 找出性价比最高的物品
    mx_cho = items
    max_ratio_num = mx_cho
    max_ratio_den = mx_cho
    for w, v in items[1:]:
        if v * max_ratio_den > max_ratio_num * w:
            mx_cho = (w, v)
            max_ratio_num = v
            max_ratio_den = w
        elif v * max_ratio_den == max_ratio_num * w and w < mx_cho[0]:
            mx_cho = (w, v)
    # 计算所有物品重量乘积，若超过V则pdt=0
    pdt = 1
    for w, _ in items:
        pdt *= w
        if pdt > V:
            pdt = 0
            break
    dw = items
    for w, _ in items[1:]:
        dw = math.gcd(dw, w)
    if dw == 0: dw = 1
    bas = 0
    if pdt != 0:
        lv = V - pdt
        bas = (lv // mx_cho[0](@ref) * mx_cho
        V = (lv % mx_cho[0](@ref) + pdt
    mdl = V // dw
    d = [0] * (mdl + 1)
    scaled_items = [(w // dw, v) for w, v in items]
    for cw, cv in scaled_items:
        for j in range(cw, mdl + 1):
            d[j] = max(d[j], d[j - cw] + cv)
    return d[mdl] + bas
```

让程序自己跑了几个小时，结果都是一致，可以深入验证一下，还是挺有意思的。就是如果他这个物件重量本身也很大，即使物件数量小，乘积也可能超过了背包容量，而无法起到简化空间作用，但是对于不算太多个小重量物件还是好用的（速度很快只取决于物件乘积），因为它物件乘积不会太大（或许也可以换成最小公倍数，但改变不大），用于计算最优的存储用情况的dp长度不大。不过缺乏数学证明，虽然经过很大数据验证了。

#### 恰好装满问题

对于背包问题，不管是01还是完全背包，如果要求恰好装满，且问最大价值，初始化dp[0]=0，除了的都初始化为负无穷，最后取dp[V]。如果有值则恰好装满最大价值，否则无解。

---

## 三、练习与变种

### 1、01背包计数问题（洛谷1164 小A的菜）

**核心思路：**一样的处理方法，但要单独给d[0]赋值为1。转移方式为当前加上不选这个物品对应拥有的选法。因为这里是刚好用完m，所以最后返回花费m钱对应的d[m]，而不是max(d)。

```
def ct_zrpck(m,cps):
    d = [0]*(m+1)
    d[0] = 1
    for cp in cps:
        for i in range(m,cp-1,-1):
            d[i] += d[i-cp]
    return d[m]

m = int(input())
cps = list(map(int,input().split()))
print(ct_zrpck(m,cps))
```

需要多次加这个d是因为这里d类似于01背包容量价值问题，对应于这个容量的，选到当前物件的最大价值的计算，不过这里是前面累计选法数量的结果。个人理解是因为这里是把选择新物件（未更新）之前的当前遍历价钱看作当前钱还没选新物件时对应选法量作为基底，对于加入的每个新物件，对于当前倒序遍历的钱又加入了一件新物品，结果还是这个钱，说明有一种新的来源，当前钱选法自然等于新来源那个钱记录的选法，这是第一层意思；另外如果那个来源前面已经加过一次，但是因为当前物件又是新的了，虽然价格相同，整个最终方案又不同了，所以需要再加一遍，这是第二层意思。简而言之：抽象。

#### 完全背包计数变种：

```
def cn_cmtPck(M,cps):
    d = [0]*(M+1)
    d[0] = 1
    for cp in cps:
        for i in range(cp,M+1):
            d[i] += d[i-cp]
    return d[M]
```

---

### 2、01背包特殊情况（物件价值等于重量）p1049

**核心思路：**多种解法，这里给出三种典型方法。

#### 解法1：标准01背包求最大体积

```
V = int(input())
n = int(input())
def pck_lf():
    dp = [0]*(V+1)
    for i in range(n):
        v = int(input())
        for j in range(V,v-1,-1):
            dp[j] = max(dp[j],dp[j-v]+v)
    return V-dp[V]
print(pck_lf())
```

#### 解法2：布尔类型状态传递（恰好装满）

```
V = int(input())
n = int(input())
def pck_zr():
    dp = [False]*(V+1)
    dp[0] = True
    for i in range(n):
        v = int(input())
        for j in range(V,v-1,-1):
            dp[j] = dp[j-v] or dp[j]
    for i in range(V,-1,-1):
        if dp[i]:
            return V-i
print(pck_zr())
```

#### 解法3：位运算优化

```
V = int(input())
n = int(input())
def dit_rcd():
    bt = 1
    msk = (1<<(V+1))-1
    for _ in range(n):
        i = int(input())
        bt = bt | (bt << i)
        bt = msk & bt
    return bt.bit_length()-1
print(V - dit_rcd())
```

#### 解法4：DFS搜索+剪枝（适合n较小）

```
V = int(input())
n = int(input())
vs = [0]*n
suf = [0]*(n+1)
for i in range(n):
    vs[i] = int(input())
vs.sort()
for i in range(n-1,-1,-1):
    suf[i] = suf[i+1] + vs[i]
rs = 0
def bac_src(si,cr):
    nonlocal rs
    if cr + suf[si] <= V:
        rs = max(rs,cr+suf[si])
        return
    if cr > rs:
        rs = cr
    pr = -1
    for i in range(si,n):
        if vs[i] == pr:
            continue
        if cr + vs[i] > V:
            return
        bac_src(i+1, cr+vs[i])
        pr = vs[i]
bac_src(0,0)
print(V-rs)
```

总结：这里比较关键的是这个递归代码结构的构造，其次为了防止pr对不同层次的影响，在循环之前恢复为-1，避免修改不同层pr造成影响。并且这里还提前建立了一个后缀和哈希组，而不是在递归体中多次计算，也是比较巧妙。但是这里因为还是使用的递归结构，多次调用栈，当n很大时（大于30），还是耗时。

---

### 3、排数问题（保留特定子序列）p12377

**核心思路：**递归形式的状态传递。对每一个数位从高位起使用循环递归遍历，对每一层递归，在循环外定义ct，初始化为0，累加上当前层当前位置各个数字，及确定当前位数后，之后所有数字位对应情况的和（递归调用下一个位置对应的不同状态）。循环结束后返回这个ct给上一层，达到计数的效果。

```
from functools import lru_cache

def nx_staus(st,cd):
    if st==0:
        return 1 if cd ==2 else 0
    elif st==1:
        return 2 if cd == 0 else 1
    elif st==2:
        return 3 if cd == 2 else 2
    elif st==3:
        return 4 if cd == 3 else 3
    else:
        return 4

def solve():
    lf,rt = input().split()
    lgt = len(lf)
    @lru_cache(maxsize=None)
    def dfs(stu,ps,lsu,hsu):
        if stu==4:
            return 0
        if ps==lgt:
            return 1
        lo = int(lf[ps]) if lsu else 0
        hi = int(rt[ps]) if hsu else 9
        tol = 0
        for cd in range(lo,hi+1):
            nx_stu = nx_staus(stu,cd)
            nx_lsu = lsu and (cd==lo)
            nx_hsu = hsu and (cd==hi)
            tol += dfs(nx_stu,ps+1,nx_lsu,nx_hsu)
        return tol
    ans = dfs(0,0,True,True)
    return ans

print(solve())
```

总结：主要就是这个递归内部使用cnt计数嵌套循环代码结构感觉还是挺重要的，之前看到过几次都不太理解，然后就是这个状态传递的定义，用到这个题上刚好合适。对于递归的不同参数组合，对应一种状态，四个参数的更新传递，完美展示动归状态转换的递归形式，但是因为状态数不算太多，便又可以使用递归缓存已经遍历情况，妙。

---
© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议