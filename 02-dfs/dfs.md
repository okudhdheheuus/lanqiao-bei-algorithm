# DFS 回溯算法笔记
## 1. 返回列表元素全排列 (无重复元素)
核心思路通过一个 path 数组多次遍历取数，使用 mk 数组过滤已经使用的。当 path 代表全部元素后，结束递归，依次弹出 path 元素，并取消标记。

python
def pmt(nums):
    res = []
    mk = [False] * len(nums)
    
    def bck_tr(path):
        if len(path) == len(nums):
            # 必须使用拷贝，因为后续还会修改path
            res.append(path[:])
            return
        
        for i in range(len(nums)):
            if mk[i]:
                continue
            mk[i] = True
            path.append(nums[i])
            bck_tr(path)
            path.pop()
            mk[i] = False
    
    bck_tr([])
    return res
## 2. 组合 (n 个选 K 个)
核心思路：在递归函数中添加一个指示参数 start 表示当前 i 所到位置，用于剪枝。循环时保证当前选择的元素个数不超过剩余元素个数，不用再标记是否选过。

python
def comb(nums, k):
    res = []
    def bac_tr(st, path):
        if len(path) == k:
            res.append(path[:])
            return
        
        # 剪枝优化：如果剩下能选的数不够填满剩余位置，直接结束
        lvsl = k - len(path)
        lvnm = len(nums) - st + 1
        if lvsl > lvnm:
            return
        
        for i in range(st, len(nums)):
            path.append(i)  # 这里注意：原代码存的索引，按题目也可能存 nums[i]
            bac_tr(i+1, path)
            path.pop()
            
    bac_tr(0, [])
    return res
## 3. 子集组合 (所有子集情况)
python
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
## 4. 全排列有重复元素情况
要排序，然后在循环前判断前一个元素是否相等且没有被使用过。

python
def pemt_uq(nums):
    nums.sort()
    res = []
    mk = [False] * len(nums)
    
    def bac_tr(path):
        if len(path) == len(nums):
            res.append(path[:])
            return
        
        for i in range(len(nums)):
            if mk[i]:
                continue
            # 连续相同元素，前一个没用过就直接跳过本轮
            if i > 0 and nums[i] == nums[i-1] and not mk[i-1]:
                continue
            mk[i] = True
            path.append(i)
            bac_tr(path)
            path.pop()
            mk[i] = False
    bac_tr([])
    return res
## 5. 迷宫问题，网格 DFS
主要是递归边界判定，以及访问过/障碍物判定。

python
def mzdfs(x, y, gr, vit):
    """网格DFS模版"""
    if x < 0 or x >= len(gr) or y < 0 or y >= len(gr[0]):
        return
    if vit[x][y] or gr[x][y] == 0:  # 0代表障碍物
        return
    
    vit[x][y] = True
    mzdfs(x-1, y, gr, vit)
    mzdfs(x+1, y, gr, vit)
    mzdfs(x, y-1, gr, vit)
    mzdfs(x, y+1, gr, vit)
    vit[x][y] = False  # 根据是否保留路径决定是否回溯
## 6. 数字排列 (1-9 组成 3 个三位数，比值 1:2:3)
python
def fd_nms():
    res = []
    mk = [False] * 10
    cns = []
    
    def bac_tr(path):
        if len(path) == 9:
            if cns[1] == cns[0] * 2 and cns[2] == cns[0] * 3:
                res.append(path[:])
            return
        
        for i in range(1, 10):
            if mk[i]:
                continue
            
            # 各种剪枝条件（略复杂，保持原样）
            if len(path) == 2:
                cn = path[0]*100 + path[1]*10 + i
                cns.append(cn)
                if cn < 123 or cn > 333:
                    continue
            # ... （此处省略其他具体剪枝逻辑，保留你的原逻辑）
            
            mk[i] = True
            path.append(i)
            bac_tr(path)
            mk[i] = False
            path.pop()
            if len(path) in [8, 5, 2]:
                cns.pop()
                
    bac_tr([])
    return res
## 7. 生成给定数量对括号
使用 lf 和 rt 计数变量记录左右括号数量，核心是保证右括号数量始终小于等于左括号。

python
def gtbck(n):
    res = []
    def bac_tr(lf, rt, path):
        if len(path) == 2 * n:
            res.append(''.join(path))
            return
        if lf < n:
            path.append('(')
            bac_tr(lf+1, rt, path)
            path.pop()
        if rt < lf:
            path.append(')')
            bac_tr(lf, rt+1, path)
            path.pop()
    bac_tr(0, 0, [])
    return res
## 8. N 皇后问题
使用 row+col 和 row-col 来标记两条对角线。

python
def solQue(n):
    res = []
    cls = set()
    fmlw = set()
    fmlh = set()  # 存储 row+col 的值
    
    def bac_tr(rw, path):
        if rw == n:
            bod = []
            for cl in path:
                bod.append('.'*cl + 'Q' + '.'*(n-cl-1))
            res.append(bod)
            return
        
        for cl in range(n):
            if cl in cls or (cl - rw) in fmlw or (cl + rw) in fmlh:
                continue
            cls.add(cl)
            fmlw.add(cl - rw)
            fmlh.add(cl + rw)
            path.append(cl)
            bac_tr(rw+1, path)
            path.pop()
            cls.remove(cl)
            fmlw.remove(cl - rw)
            fmlh.remove(cl + rw)
            
    bac_tr(0, [])
    return res
## 9. 回文串分割 (动态规划 + DFS)
python
def par(s):
    res = []
    n = len(s)
    dp = [[False]*n for _ in range(n)]
    for i in range(n):
        dp[i][i] = True
    for i in range(n-1, -1, -1):
        for j in range(i+1, n):
            if s[i] == s[j]:
                if j == i+1 or dp[i+1][j-1]:
                    dp[i][j] = True
                    
    def bac_tr(st, path):
        if st == n:
            res.append(path[:])
            return
        for ed in range(st, n):
            if dp[st][ed]:
                path.append(s[st:ed+1])
                bac_tr(ed+1, path)
                path.pop()
    bac_tr(0, [])
    return res