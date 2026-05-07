二维 DP
一、核心模板：
1、数字三角形 (路径权值最大和) p1216思路：将三角形用下三角矩阵进行存储，简单的递推，不过对于每一行，需要单独讨论第一个 (只来自于本列上一行) 和最后一个元素 (只来自于左上角)，中间元素 (取左上和上面中大的一个作为来源)。
python
运行
def mx_trg(trg):
    n = len(trg)
    dp = [[0]*(i+1) for i in range(n)] # 用对角矩阵存储
    dp[0][0] = trg[0][0]
    for i in range(1,n): # 排开第一行
        for j in range(i+1):  # 每行元素个数刚好是i+1
            if j == 0: # 第一个元素没有左上方元素
                dp[i][j] = dp[i-1][j]+trg[i][j]
            elif j == i: # 最后一个元素正上方没有元素
                dp[i][j] = dp[i-1][j-1]+trg[i][j]
            else:
                dp[i][j] = max(dp[i-1][j-1],dp[i-1][j]) + trg[i][j] # 取大的一个作为来源
    return max(dp[n-1]) # 最后一行取大的一个,就是路径最大值
2. 过河兵 (路径计数，障碍物) p1002思路：首先给每个位置设置可到达为 True, 然后单独标记障碍区为 True，从 0,0, 开始，每个位置路径数由左，上两个方向传来 (因为从前往后，每个位置必然只会遍历一次，也就是说它到达上面那个点和左边那个点的路径数已经确定，直接两个方向传递过来即可（如果不是边界的话)
python
运行
def hr_pth(n,m,hx,hy):
    dx = [-2,-2,-1,-1,0,1,1,2,2]
    dy = [1,-1,2,-2,0,2,-2,1,-1]
    bck = [[False]*(m+1) for _ in range(n+1)]
    for i in range(9):
        x,y = hx+dx[i],hy+dy[i]
        if 0<=x<=n and 0<=y<=m:
            bck[x][y] = True
    d = [[0]*(m+1) for _ in range(n+1)]
    d[0][0] = 1 if not bck[0][0] else 0
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
3. 最长公共子序列 (LCS) p2543核心思路：设置第一个二维 dp 组，下标组 i,j，表示分别截取两个串对应长时 (即 0-i 和 0-j) 所得的两个串对应最大公共子序列，对于每个使用两层循环遍历所有截取情况 (至少截取 1，即都从 1 开始)，对于遍历的每种情况，判断对应下标在各自串中的上一个元素 (取 s1 [i-1],s2 [j-1]) 是否相等，相等则说明对于 dp [i][j] 这种截取情况在各自串截取情况的下一个元素也相等，比如都是 2，表示公共序列多一个元素 2，所以对于 i,j 截取情况最大公公序列长加 1 即可，如果截取情况最后一对元素不相等，取各自截取的较大的那种情况的最大值，因为当前这种截取必然包含（i-1,j) 和 (i,j-1) 这两种情况，取大的那种情况即可。时间复杂度为 o (n**2)
python
运行
def LCS(s1,s2):
    n,m = len(s1),len(s2)
    dp = [[0]*(m+1) for _ in range(n+1)]
    for i in range(1,n+1):     # 注意取1，n+1 ，因为i,j表示的是截取到第n个元素的情况
        for j in range(1,m+1): # 注意取1，m+1
            if s1[i-1]==s2[j-1]:
                dp[i][j] = dp[i-1][j-1]+1
            else:
                dp[i][j] = max(dp[i-1][j],dp[i][j-1])
    return dp[n][m]
s1,s2 = input().split()
print(LCS(s1,s2))
改进：观察状态转换，对每种裁取情况相对 dp 组，只使用左上 (左列上行) 和上方 (本列上行), 左方 (左列本行), 考虑转换为一维 (形式转换，但是内核不变)，对于用 d 存储内层滚动结果，则 d 大小设置为内层循环次数 (假设 s2 为内层，则为 len (s2)+1), 然后对于上方元素即为本轮 d [j] 更新前结果 (上一行 i-1 行更新后结果), 左侧元素因为循环顺序执行自然已经更新，则对于当前情况的处理 (当前行、列)，就还差左上元素需要记录 (对应二维 d [i-1][j-1])，容易知道在上一列更新前提前记录，并在那一轮使用完 pr 后传给 pr 进行传递即可。
python
运行
def pl_sq(s1,s2):
    m,n = len(s1),len(s2)
    dp = [0]*(n+1)
    for i in range(1,m+1):
        pr = 0 # 第一列都没有左上元素，都看作0
        for j in range(1,n+1):
            tp = dp[j] # 提前保存未更新前当前列情况(表示上一行)
            if s1[i-1] == s2[j-1]:
                dp[j] = pr+1
            else:
                dp[j] = max(dp[j-1],dp[j]) # 当前行前一个(d[j-1]已经更新表示当前行)或者当前列上一个(d[j]还没更新表示本列上一行)
            pr = tp # 这里更新为相对下一列的上一行(还没更新前的d[j-1])
    return dp[n]
s1,s2 = input().split()
print(pl_sq(s1,s2))
变种：排列的最长公共序列 (LCS 转 LIS) p1439
python
运行
import bisect
n= int(input())
def ml_seq(seq):
    dp = []
    for i in seq:
        pos = bisect.bisect_left(dp,i)
        if pos ==len(dp):
            dp.append(i)
        else:
            dp[pos] = i
    return len(dp)
s1 = list(map(int,input().split()))
s2 = list(map(int,input().split()))
l = [0]*n
for i in range(n):
    l[s2[i]-1] = i
seq = [l[x-1] for x in s1]
print(ml_seq(seq))
4. 最少字符串操作 (最长公共子序列变种) p2758
python
运行
import sys
def ed_ch(s1,s2):
    m,n = len(s1),len(s2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1,m+1):  # 提前处理某一个串截取长度为0时情况，对另一个串的任意截取情况只做对应长度次删除操作
        dp[i][0] = i
    for j in range(1,n+1):
        dp[0][j] = j
    for i in range(1,m+1):
        for j in range(1,n+1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]   # 当前截取情况最后一个元素相等，直接等于都不看当前截取最后一个元素对应截取情况的记录值
            else:
                dp[i][j] = min(dp[i-1][j],dp[i][j-1],dp[i-1][j-1])+1 
    return dp[m][n]
s1 = input().strip()
s2 = input().strip()
print(ed_ch(s1,s2))
改进：类似于求公共序列，这里也可以使用滚动赋值转换为一维 dp, 关键还是上方，左方，左上三个值的处理，但是因为需要对边界单独赋值，对于 d [0][j] 的值好处理，直接在创建 d 时，给 d 每一个元素赋为 i 即可，对于每一列首元素即 dp [i][0]，在内层循环外单独处理即可，另外就是这里的 pr 始终指向的是左上元素，所以对于 d [i][1] 的左上即为 d [i-1][0], 所以不能设为 0，而应该是 i-1.
python
运行
import sys
def ed_ch(s1,s2):
    m,n = len(s1),len(s2)
    dp = [i for i in range(n+1)] # 要单独赋一下值，表示第一行所有列的初始值即d[0][j]
    for i in range(1,m+1):            
        pr = i-1  # 初始为左上角对应情况，即为d[i-1][0],由二维边界初始赋值可知，初始pr应为i-1
        dp[0] = i # 同上，对每行第一列初始值应为d[i][0],即i
        for j in range(1,n+1):
            tp = dp[j]
            if s1[i-1] == s2[j-1]:
                dp[j] = pr
            else:
                dp[j] = min(dp[j],dp[j-1],pr)+1
            pr = tp
    return dp[n]
s1 = input().strip()
s2 = input().strip()
print(ed_ch(s1,s2))
5. 二维背包问题 p1507
python
运行
def tw_dms_zrPck(V,W,n):
    dp = [[0]*(W+1) for _ in range(V+1)] # 外层是体积，内层是重量，注意顺序
    for k in range(n):
        v,w,e = map(int,input().split())
        for i in range(V,v-1,-1):
            for j in range(W,w-1,-1): # 第一项是外层，第二项列元素才是重量
                dp[i][j] = max(dp[i-v][j-w]+e,dp[i][j])
    return max([max(rl) for rl in dp]) # 不能一步返回二维dp的max，使用列表推广式求
变种：二维完全背包
python
运行
def tw_dms_zrPck(V,W,n):
    dp = [[0]*(W+1) for _ in range(V+1)] # 外层是体积(行元素)，内层是重量(列元素)，注意顺序
    for k in range(n):
        v,w,e = map(int,input().split())
        for i in range(v,V+1):
            for j in range(w,W+1): # 第一项是外层行元素，第二项列元素才是重量
                dp[i][j] = max(dp[i-v][j-w]+e,dp[i][j])
    return dp[V][W]
6. 矩阵取数 (三维 DP) p1004 方格取数
python
运行
n= int(input())
grd = [[0]*(n+1) for _ in range(n+1)] # 注意多开一层
def cnt_mx():
    dp = [[[0]*(n+1) for _ in range(n+1)] for _ in range(n*2+1)]
    for f in range(2,n*2+1):
        for i in range(1,n+1):
            y1 = f-i
            if 1<=y1<=n:
                tp = grd[i][y1]
            else:
                continue
            for j in range(1,n+1):
                y2 = f-j
                vl = tp
                if y2<1 or y2>n:
                    continue
                if i!=j:
                    vl+=grd[j][y2]
                dp[f][i][j] = max(
                        dp[f-1][i-1][j-1], # 都从左方来
                        dp[f-1][i-1][j], # 一左一上
                        dp[f-1][i][j-1],
                        dp[f-1][i][j] # 都从上方下来
                    ) + vl     
    return dp[2*n][n][n]      
while True:
    rl = input().strip()
    if not rl:
        break
    x,y,v = map(int,rl.split())
    if x==0 and y==0 and v==0:
        break
    grd[x][y] = v
print(cnt_mx())
拓展：实际上这里也可以转换成栈处理，每个栈元素是一个元组，类似于动态规划包含三个维度值 (步长，nx1 (第一个点),nx2 (第二个点)), 不过栈结构是对下面四种情况的处理 (可能多次到达这种情况，需要取最大值，所以需要使用三维数组存储值)，而动态规划是到达了当前情况才更新，只对每个情况进行一次更新 (递推过来)。
python
运行
from collections import deque
n = int(input())
grd = [[0]*(n) for _ in range(n)]
def sta_sqa():
    S = 2*n-2
    dp = [[[-1]*n for _ in range(n)] for _ in range(S+1)] # 不要赋值为0，否则第一个元素无法正常传递
    deq = deque([(0,0,0)]) # 三个维度，步长，坐标1，坐标2
    dp[0][0][0] = grd[0][0]
    while deq:
        cs,fx,sx = deq.popleft()
        cm = dp[cs][fx][sx]
        fy = cs-fx
        sy = cs-sx
        for i in [0,1]:
            nfx = i+fx
            nfy = fy+(1-i)
            if nfx>=n or nfy>=n:
                continue
            tp = grd[nfx][nfy]
            for j in [0,1]:
                nsx = j + sx
                nsy = sy+(1-j)
                if nsx >=n or nsy>=n:
                    continue
                vl = tp
                if nsx != nfx:
                    vl += grd[nsx][nsy]
                if cm+vl>dp[cs+1][nfx][nsx]:
                    dp[cs+1][nfx][nsx] = cm+vl
                    if cs==S-1:
                        continue
                    deq.append((cs+1,nfx,nsx))
    return dp[S][n-1][n-1]
while True:
    rl = input().strip()
    if not rl:
        break
    x,y,v = map(int,rl.split())
    if x==0 and y==0 and v==0:
        break
    grd[x-1][y-1] = v
print(sta_sqa())
7. 标准 young 表 (计数动规)
python
运行
# 2020(1~2020)个数放到2*1010矩阵，刚好放满(右大于左，下大于上)
def sol():
    n = 1010
    dp = [[0]*(n+1) for _ in range(n+1)]
    dp[0][0] = 1
    for i in range(n+1):
        for j in range(n+1):
            if i<j:
                continue
            if i+1<n+1:
                dp[i+1][j] = (dp[i+1][j] + dp[i][j]%2020)%2020
            if j+1<n+1 and j<i:
                dp[i][j+1] = (dp[i][j+1] + dp[i][j]%2020)%2020
    print(dp[n][n]%2020)
print(sol())
递归法
python
运行
def sol():
    @lru_cache(maxsize =2020)
    def dfs(i,j):
        if i<j:
            return 0
        else:
            if j==0:
                if i==0:
                    return 0
                else:
                    return 1
            else:
                if i==j:
                    return dfs(i,j-1)%2020
                else:
                    return (dfs(i-1,j)+dfs(i,j-1))%2020
    print(dfs(1010,1010))