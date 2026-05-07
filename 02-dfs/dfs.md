# 二、DFS回溯

---

## 1、返回列表元素全排列（无重复元素）

**核心思路：**

- 通过一个path数组多次遍历取数，使用mk数组过滤已经使用的元素
- 当path代表全部元素后，结束递归
- 依次弹出path元素，并取消标记
- 注意：添加结果时必须用拷贝（path[:]），否则后续修改会污染结果

```
def pmt(nums):
    res = []
    mk = [False]*len(nums)
    def bck_tr(path):
        if len(path) == len(nums):
            res.append(path[:])
            return
        for i in range(len(nums)):
            if mk[i]:
                continue
            mk[i] = True
            path.append(nums[i])
            bac_tr(path)
            path.pop()
            mk[i] = False
    bac_tr([])
    return res
```

---

## 2、组合（n个选K个）

**核心思路：**

- 在递归函数中添加一个指示参数start，表示当前i所到位置，用于剪枝
- 循环时保证当前选择的元素个数小于剩余元素个数（使用start维护）
- 不用再标记是否选过，因为遍历就是向前走的
- 直到已经选择的长度足够就依次弹出，继续在上一层完成当前循环遍历
- 剪枝公式：还需选的个数 ≤ 还剩的个数 → st ≤ n - (k - len(path)) + 2

```
def comb(nums,k):
    res = []
    def bac_tr(st,path):
        if len(path) == k:
            res.append(path[:])
            return
        lvsl = k - len(path)
        lvnm = n - st + 1
        rt = n - (k - len(path)) + 2
        lf = st
        for i in range(lf, rt):
            path.append(i)
            bac_tr(i+1, path)
            path.pop()
    bac_tr(1, [])
    return res
```

---

## 3、子集组合（所有子集情况）

**核心思路：**

- 每次进入递归都先记录当前路径（所有子集包括空集）
- 从start开始循环，避免重复组合
- 递归后回溯（pop）

```
def sbet(nums):
    res = []
    def bac_tr(st, path):
        res.append(path[:])
        for i in range(st, len(nums)):
            path.append(i)
            bac_tr(i+1, path)
            path.pop()
    bac_tr(0, [])
    return res
```

---

## 4、全排列有重复元素情况处理

**核心思路：**

- 和正常全排列一样，但是要先排序
- 在循环中添加元素前要判断前一个元素是否相等，且没添加过
- 如果是当前这段相等连续元素的第一个，就执行后面添加和递归语句，否则跳过当前循环
- 保证重复元素的相对顺序固定（去重关键）

```
def pemt_uq(nums):
    nums.sort()
    res = []
    mk = [False]*len(nums)
    def bac_tr(path):
        if len(path) == len(nums):
            res.append(path[:])
            return
        for i in range(len(nums)):
            if mk[i]:
                continue
            if i>0 and nums[i]==nums[i-1] and not mk[i-1]:
                continue
            mk[i] = True
            path.append(i)
            bac_tr(path)
            path.pop()
            mk[i] = False
    bac_tr([])
    return res
```

---

## 5、迷宫问题，网格DFS

**核心思路：**

- 递归边界判定：坐标越界或遇到障碍物或已访问
- 根据题目要求决定是否需要回溯（恢复标记）
- 四个方向递归

```
def mzdf(x,y,gr,vit):
    if x<0 or x>=len(gr) or y<0 or y>=len(gr:
        return
    if vit[x][y] and gr[x][y] == 0:
        return
    vit[x][y] = True
    mzdf(x-1,y,gr,vit)
    mzdf(x+1,y,gr,vit)
    mzdf(x,y-1,gr,vit)
    mzdf(x,y+1,gr,vit)
    vit[x][y] = False
```

---

## 6、数字排列（用1-9组成3个三位数，比值1:2:3）

**核心思路：**

- 用全排列基本模版，取出所有数字组合
- 递归结束条件：len(path)==9，然后检查三个数是否满足1:2:3
- 在添加数字时根据题意进行剪枝
- 通过continue跳过当轮循环，减少递归调用
- 注意维护cns列表，在回溯时弹出

```
def fd_nms():
    res = []
    mk = [False]*10
    cns = []
    def bac_tr(path):
        if len(path)==9:
            if cns[1] == cns[0]*2 and cns[2]==cns[0]*3:
                res.append(path[:])
            return
        for i in range(1,10):
            if mk[i]:
                continue
            if len(path)==2:
                cn = path[0]*100+path[1]*10+i
                cns.append(cn)
                if cn<123 or cn>333:
                    continue
            if len(path)==3:
                if i==1 or i > 6:
                    continue
            if len(path)==5:
                cn = path[3]*100+path[4]*10+i
                if cn != cns[0]*2:
                    continue
                cns.append(cn)
            if len(path)==6 and i<3:
                continue
            if len(path)==8:
                cn = path[6]*100+path[7]*10+i
                if cn != cns[0]*3:
                    continue
                cns.append(cn)
            mk[i] = True
            path.append(i)
            bac_tr(path)
            mk[i] = False
            path.pop()
            if len(path) == 8 or len(path) == 5 or len(path) == 2:
                cns.pop()
    bac_tr([])
    return res
print(fd_nms())
```

---

## 7、生成给定数量对括号的不同排法

**核心思路：**

- 使用两个计数变量lf（左括号数量）、rt（右括号数量）
- 关键：保证右括号数量始终小于左括号数量（rt < lf）
- 当左括号数小于n时可以加左括号，当右括号数小于左括号数时可以加右括号
- 结束条件：path长度等于2n

抽象不理解，背模板即可。

```
def gtbck(n):
    res = []
    def bac_tr(lf,rt,path):
        if len(path) == 2*n:
            res.append(''.join(path))
            return
        if lf<n:
            path.append('(')
            bac_tr(lf+1,rt,path)
            path.pop()
        if rt<lf:
            path.append(')')
            bac_tr(lf,rt+1,path)
            path.pop()
    bac_tr(0,0,[])
    return res
print(gtbck(3))
```

---

## 8、N皇后问题

**核心思路：**

- 使用三个集合：cls存储已用的列，fmlw存储左下到右上对角线（差值row-col），fmlh存储左上到右下对角线（和row+col）
- 递归函数传递行号rw，逐行尝试放置皇后
- 如果当前位置在该列、两条对角线上都没有冲突，则放置并标记，然后递归下一行
- 回溯时从三个集合中移除标记
- 当rw==n时，生成棋盘表示并加入结果

```
def solQue(n):
    res = []
    cls = set()
    fmlw = set()
    fmlh = set()
    def bac_tr(rw,path):
        if rw == n:
            bod = []
            for cl in path:
                bod.append('.'*cl+'Q'+'.'*(n-cl-1))
            res.append(bod)
        for cl in range(n):
            if cl in cls or cl-rw in fmlw or cl+rw in fmlh:
                continue
            cls.add(cl)
            fmlw.add(cl-rw)
            fmlh.add(cl+rw)
            path.append(cl)
            bac_tr(rw+1,path)
            path.pop()
            cls.remove(cl)
            fmlw.remove(cl-rw)
            fmlh.remove(cl+rw)
    bac_tr(0,[])
    return res

result = solQue(6)
for re in result:
    for i in re:
        print(' '.join(i))
    print('************************************')
```

---

## 9、回文串分割

**核心思路：**

- 先用动态规划构建二维dp数组，dp[i][j]表示s[i:j+1]是否是回文串
- 转移方程：如果s[i]==s[j]且（j==i+1或dp[i+1][j-1]为真），则dp[i][j]=True
- 然后DFS回溯：从start开始遍历end，如果dp[start][end]为真，则加入子串，递归处理剩余部分（start=end+1）
- 结束条件：start==n，此时所有子串已分割完毕，记录结果
- 注意递归参数是end+1而不是start+1

```
def par(s):
    res = []
    n = len(s)
    dp = [[False]*n for _ in range(n)]
    for i in range(n):
        dp[i][i] = True
    for i in range(n-1,-1,-1):
        for j in range(i+1,n):
            if s[i] == s[j]:
                if j == i+1 or dp[i+1][j-1]:
                    dp[i][j] = True
    def bac_tr(st,path):
        if st == n:
            res.append(path[:])
            return
        for ed in range(st,n):
            if dp[st][ed]:
                path.append(s[st:ed+1])
                bac_tr(ed+1,path)
                path.pop()
    bac_tr(0,[])
    return res
print(par('abb'))
```

### 拓展：最小回文串分割次数（动态规划）

**核心思路：**

- 先构建回文串判定二维数组（同上）
- 使用一维数组mnct存储从0到i的最小分割次数
- 遍历i从0到n-1：如果dp[i]为真，则mnct[i]=0；否则枚举j，如果dp[j+1][i]为真，则尝试mnct[j]+1，取最小值
- 最终返回mnct[n-1]

```
def par(s):
    n = len(s)
    dp = [[False]*n for _ in range(n)]
    for i in range(n):
        dp[i][i] = True
    for i in range(n-1,-1,-1):
        for j in range(i+1,n):
            if s[i] == s[j]:
                if j == i+1 or dp[i+1][j-1]:
                    dp[i][j] = True
    mnct = [float('inf')]*n
    for i in range(n):
        if dp[i]:
            mnct[i] = 0
        else:
            for j in range(i):
                if dp[j+1][i]:
                    mnct[i] = min(mnct[i], mnct[j]+1)
    return mnct[n-1]
print(par('abbcb'))
```

---

## 10、幻方（洛谷P1406）

**核心思路：**

- 递归回溯模板，但剪枝复杂：使用sur（行和）、suc（列和）数组维护当前部分和
- 每当到达行尾（最后一列）时，检查当前行和是否等于总和/阶数；到达列尾（最后一行）时检查列和
- 添加左下角元素时检查主对角线，添加最后一个元素时检查副对角线
- 使用pre变量去重：如果当前元素等于上一次尝试失败的元素（pre），则跳过，避免重复无效搜索
- 找到第一个满足条件的组合就返回True，快速结束递归
- 回溯时不仅要取消标记，还要还原行和、列和以及pre

还是蛮抽象的。

```
n = int(input())
nms = list(map(int,input().split()))
smk = sum(nms)//n
nms.sort()

def istms():
    res = []
    mk = [False]*n*n
    suc = [0]*n
    sur = [0]*n
    def bac_tr(path=[],pre=None):
        l = len(path)
        for ci in range(n*n):
            if nms[ci] == pre:
                continue
            if mk[ci]:
                continue
            tc = l%n
            tr = l//n
            sur[tr] += nms[ci]
            if l == (tr+1)*n-1:
                if sur[tr]!=smk:
                    sur[tr] -= nms[ci]
                    pre = nms[ci]
                    continue
                if tr == n-1:
                    sm = 0
                    for i in range(tr):
                        sm+=path[(n+1)*i]
                    if sm+nms[ci] != smk:
                        sur[tr] -= nms[ci]
                        pre = nms[ci]
                        continue
            suc[tc] += nms[ci]
            if l==(n-1)*n+tc:
                if suc[tc] != smk:
                    sur[tr] -= nms[ci]
                    suc[tc] -= nms[ci]
                    pre = nms[ci]
                    continue
                if tc==0:
                    sm=0
                    for i in range(tr):
                        sm+=path[(i+1)*(n-1)]
                    if sm+nms[ci] != smk:
                        sur[tr] -= nms[ci]
                        suc[tc] -= nms[ci]
                        pre = nms[ci]
                        continue
            pre = None
            mk[ci] = True
            path.append(nms[ci])
            if l == n*n-1:
                res.append(path[:])
                return True
            if bac_tr(path,pre):
                return True
            path.pop()
            mk[ci] = False
            suc[tc] -= nms[ci]
            sur[tr] -= nms[ci]
            pre = nms[ci]
        return False
    bac_tr()
    return res

tms = istms()
print(smk)
lines = [' '.join(map(str,tms[i*n:(i+1)*n])) for i in range(n)]
print('\n'.join(lines))
```

---

© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议