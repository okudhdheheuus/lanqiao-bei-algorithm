# BFS广搜

---

## 一、核心模板

### 1、二维网格BFS（迷宫问题）

**核心思路**：

- 创建一个队列，用于存每个第一次读入的节点
- 使用 vis 数组标记已经访问的节点，已访问则不用再加入队列
- 依次弹出队列节点，对每个节点根据其可到达的四个方向坐标进行遍历
- 没访问过的加入队列并标记访问，其路径长就为当前节点路径加1
- 直到某个节点等于目标点，就返回其路径

```
from collections import deque

def bfs_grid(grd, st, tg):
    if not grd or grd[0]:
        return -1
    rws = len(grd)
    cls = len(grd[0](@ref)
    dircs = [(1,0), (0,1), (-1,0), (0,-1)]
    deq = deque([st])
    vis = set([st])
    stps = {st: 0}
    while deq:
        x, y = deq.popleft()
        if (x, y) == tg:
            return stps[(x, y)]
        for dx, dy in dircs:
            nx, ny = x + dx, y + dy
            if 0 <= nx < rws and 0 <= ny < cls and grid[nx][ny] == 0:
                if (nx, ny) not in vis:
                    vis.add((nx, ny))
                    deq.append((nx, ny))
                    stps[(nx, ny)] = stps[(x, y)] + 1
    return -1
```

### 2、层序遍历BFS（计算层数、最短路径）

**核心思路**：

- 用列表存储第一次遍历的节点，然后依次取出，将其子节点加入到列表（如果没有遍历过的话）
- 主要区别是一个层次路径长，计算方法是在循环体开头计算当前队列元素个数
- 根据这个长度（当前层元素个数），如果遍历完当前层所有元素，就将 stp 加1
- 当前层所有子节点路径长相等，最后如果某个弹出元素等于目标节点就返回 stp

```
from collections import deque

def bfs_lev(st, tg):
    if st == tg:
        return 0
    deq = deque([st])
    vis = set([st])
    stp = 0
    while deq:
        l = len(deq)
        for _ in range(l):
            node = deq.popleft()
            if node == tg:
                return stp
            for neb in get_neighbors(node):
                if neb not in vis:
                    vis.add(neb)
                    deq.append(neb)
        stp += 1
    return -1
```

除了这种统一计算每一层所有元素路径，还可以向网格遍历那样使用 stps 字典对每个节点路径进行记录：

```
stps = {st: 0}
while deq:
    node = deq.popleft()
    for neb in get_neighbors(node):
        if neb not in vis:
            vis.add(neb)
            deq.append(neb)
            stps[neb] = stps[node] + 1
```

---

## 二、常见例题

### 1、迷宫最短路径（返回st到ed最短路径，不能穿过障碍物）

类似于模板2解法，但是 node 节点换成二维坐标，初始化一个二维数组用于标记访问情况，使用层次遍历计算路径。

```
from collections import deque

def min_dis(grd, st, ed):
    if st == ed:
        return False
    rws = len(grd)
    cls = len(grd[0](@ref)
    dirc = [(1,0), (-1,0), (0,1), (0,-1)]
    vis = [[False]*cls for _ in range(rws)]
    vis[st[0]][st[1]] = True
    stp = 0
    deq = deque([st])
    while deq:
        size = len(deq)
        for _ in range(size):
            x, y = deq.popleft()
            if (x, y) == ed:
                return stp
            for dx, dy in dirc:
                nx, ny = x + dx, y + dy
                if 0 <= nx < rws and 0 <= ny < cls and not vis[nx][ny] and grd[nx][ny] == 0:
                    vis[nx][ny] = True
                    deq.append((nx, ny))
        stp += 1
    return -1
```

### 2、多源BFS（多个起点同时扩散）返回每个位置到最近起点距离

**核心思路**：

- 和基本模板一致，但是对于多个源头，在初始化队列时，先将每一个源头依次加入队列，并将对应路径设置为0
- 对每一个坐标专门使用 dist 数组存放对应路径，可以和标记访问数组合并为一个效果
- 初始化每个坐标对应 dist 为 -1，既表示还没有访问过，也存储路径
- 对于每个到达的节点，判断时直接使用 dist 是否等于 -1 判断是否到达过
- 没到达过则将其路径从 -1 更新为到达它的坐标的路径长+1

```
from collections import deque

def msr_bfs(grd, srcs):
    rws = len(grd)
    cls = len(grd[0](@ref)
    deq = deque()
    dis = [[-1]*cls for _ in range(rws)]
    dirc = [(1,0), (-1,0), (0,-1), (0,1)]
    for x, y in srcs:
        deq.append((x, y))
        dis[x][y] = 0
    while deq:
        x, y = deq.popleft()
        for dx, dy in dirc:
            nx, ny = x + dx, y + dy
            if 0 <= nx < rws and 0 <= ny < cls and dis[nx][ny] == -1 and grd[nx][ny] == 0:
                dis[nx][ny] = dis[x][y] + 1
                deq.append((nx, ny))
    return dis
```

### 3、双向BFS（从起点和终点同时开始搜索，相遇结束，优化空间）

**核心思路**：

- 维护两个队列，两个距离存储数组
- 对于每一层遍历，分为从前往后、从后往前两个方向的遍历
- 首先根据从前往后搜索的队列长度，计算出当前层待启动节点数
- 对每个节点依次遍历其子节点，如果子节点已经在从后往前的距离存储字典中，说明相遇
- 相遇则直接返回从前往后到当前点对应路径 + 存储的从后往前到当前节点的距离值
- 如果不在，说明还没相遇，按常规模板处理当前节点
- 同理对从后往前搜索进行类似操作

```
from collections import deque

def bdrc_bfs(st, tg, get_nbs):
    if st == tg:
        return 0
    deq_st = deque([st])
    deq_tg = deque([tg])
    vis_st = {st: 0}
    vis_tg = {tg: 0}
    while deq_st and deq_tg:
        # 从开始方向
        st_sz = len(deq_st)
        for _ in range(st_sz):
            node = deq_st.popleft()
            for neb in get_nbs(node):
                if neb in vis_tg:
                    return vis_st[node] + 1 + vis_tg[neb]
                if neb not in vis_st:
                    vis_st[neb] = vis_st[node] + 1
                    deq_st.append(neb)
        # 从目标方向
        tg_sz = len(deq_tg)
        for _ in range(tg_sz):
            node = deq_tg.popleft()
            for neb in get_nbs(node):
                if neb in vis_st:
                    return vis_tg[node] + 1 + vis_st[neb]
                if neb not in vis_tg:
                    vis_tg[neb] = vis_tg[node] + 1
                    deq_tg.append(neb)
    return -1
```

关键就是整两个队列，两个距离记录，然后使用邻居来判断是否相遇，相遇就加上从前往后、从后往前到该节点各自距离和。

### 4、回溯BFS（使用 prv 记录前驱，求最短路径）

第十届javaB组 迷宫。

**核心思路**：

- 使用 prv 数组记录到达每一个点的走法
- 因为每一步权值为1，第一次到达某个点时必然就是到达该点的最短路径
- 使用 prv 记录从上一个点到达该点对应选择的方向
- 最后当弹出点是出口时，即可根据 prv 数组回溯得到最短路径，并结束 BFS 搜索

```
import sys
from collections import deque

def main():
    gd = []
    n, m = 30, 50
    for _ in range(n):
        rl = sys.stdin.readline().strip()
        gd.append(rl)
    if gd[0] == '1' or gd[n-1][m-1] == '1':
        print("")
        return
    # 方向按字典序排列（D < L < R < U）
    dirs = [('D', 1, 0), ('L', 0, -1), ('R', 0, 1), ('U', -1, 0)]
    mk = [[False] * m for _ in range(n)]
    q = deque()
    q.append((0, 0))
    mk[0] = True
    prv = [[None]*m for _ in range(n)]

    def bac():
        i, j = n-1, m-1
        rs = []
        while (i, j) != (0, 0):
            pd = prv[i][j]
            rs.append(pd)
            for dr, di, dj in dirs:
                if dr == pd:
                    i, j = i - di, j - dj
                    break
        print(''.join(reversed(rs)))
        return

    def chk_ps(i, j):
        if not (0 <= i < n and 0 <= j < m) or gd[i][j] == '1' or mk[i][j]:
            return True
        return False

    while q:
        ci, cj = q.popleft()
        if ci == n-1 and cj == m-1:
            bac()
            return
        for ch, di, dj in dirs:
            ni, nj = ci + di, cj + dj
            if chk_ps(ni, nj):
                continue
            mk[ni][nj] = True
            prv[ni][nj] = ch
            q.append((ni, nj))
    print("")

if __name__ == "__main__":
    main()
```

这里还要求路径字典序最小，使用的处理方式就是根据方向字母调整遍历顺序，保证当前选择方向始终是当前步数下当点出发的字典序最小的，从而第一次到达点时，路径字典序最小。还是有点意思。

---

## 三、练习题

### 1、填涂颜色（寻找方格数字被1围起来的所有0）

#### 法1：排除与外面连通的0

**思路**：

- 初始化队列，并添加所有边界0，看成一个多源BFS问题
- 依次弹出元素，找四个方向没有添加过（标记）的0，将其加入队列并标记
- 当队列为空时，所有与外界联通的0都被标记
- 剩下的没有被标记的0就是封闭0

```
from collections import deque
import sys

grd = []
n = int(input())
for _ in range(n):
    row = sys.stdin.readline().strip().split()
    grd.append(row)

def fd_cls():
    rws, cls = n, n
    vis = [[False]*cls for _ in range(rws)]
    deq = deque()
    dirc = [(0,1), (0,-1), (1,0), (-1,0)]

    for i in range(rws):
        for j in [0, cls-1]:
            if grd[i][j] == '0':
                deq.append((i, j))
                vis[i][j] = True
    for j in range(cls):
        for i in [0, rws-1]:
            if grd[i][j] == '0' and not vis[i][j]:
                deq.append((i, j))
                vis[i][j] = True

    while deq:
        i, j = deq.popleft()
        for di, dj in dirc:
            ci, cj = i+di, j+dj
            if 0 <= ci < rws and 0 <= cj < cls and grd[ci][cj] == '0' and not vis[ci][cj]:
                deq.append((ci, cj))
                vis[ci][cj] = True

    for i in range(rws):
        for j in range(cls):
            if grd[i][j] == '0' and not vis[i][j]:
                grd[i][j] = '2'

fd_cls()
for i in range(n):
    print(*grd[i])
```

#### 法2：寻找可能0，保留封闭0（个人思路）

**思路**：

- 在输入时处理数据，定义记录每一行和每一列的被1包围范围的两个数组
- 所有封闭0坐标必然满足该行对应封闭列范围和该列对应封闭行范围
- 但是这些范围对应0并不总是满足封闭，因为可能出现连通外界情况
- 所以需要把这些0中能与外界联通的部分筛除
- 首先用 dit 字典标记这些0坐标，复制这个标记字典到 dits 字典
- 对于每一个坐标，将其作为队列头部进行 bfs 搜索
- 如果元素在初步筛选范围中0，就用 dit 字典标记为已访问，继续添加到队列
- 如果元素不是初步筛选中的0，说明这块与外界联通，使用 isbad 标记当前块不满足
- 当前块搜索结束后，在 dit 字典中将所有当前块中点删除
- 如果该块标记了 isbad，则说明该块所有0与外界连通，应当在范围内0集中筛除

主要是剔除部分还是挺抽象的（复制了一个字典做母版，并删除子版字典元素，对两个字典进行比对，来控制块的不重复访问），想了好久。

```
import sys
from collections import deque

n = int(input())
grd = []
cls_arr = [[-1,-1,-1] for _ in range(n)]
rws_arr = [[-1,-1,-1] for _ in range(n)]

for i in range(n):
    row = sys.stdin.readline().strip().split()
    grd.append(row)
    for j in range(n):
        if row[j] != '0':
            if cls_arr[j][0] == -1:
                cls_arr[j][0] = i
            cls_arr[j][2] = i
            if rws_arr[i][0] == -1:
                rws_arr[i][0] = j
            rws_arr[i][2] = j

for i in range(n):
    cls_arr[i][1] = cls_arr[i]
    rws_arr[i][1] = rws_arr[i]

dit = {}
for i in range(n):
    for j in range(n):
        if grd[i][j] == '0' and cls_arr[j][0] < i < cls_arr[j][1] and rws_arr[i][0] < j < rws_arr[i][1]:
            grd[i][j] = '2'
            dit[(i,j)] = False

dits = dit.copy()
dirc = [(1,0), (-1,0), (0,1), (0,-1)]

for i, j in dits:
    if (i, j) not in dit.keys():
        continue
    deq = deque()
    isbad = False
    deq.append((i, j))
    dit[(i, j)] = True
    while deq:
        ri, ci = deq.popleft()
        for dx, dy in dirc:
            nr, nc = ri+dx, ci+dy
            if (nr, nc) in dit and not dit[(nr, nc)]:
                deq.append((nr, nc))
                dit[(nr, nc)] = True
            if 0 <= nr < n and 0 <= nc < n and grd[nr][nc] == '0':
                isbad = True
    for k in list(dit.keys()):
        if dit[k]:
            if isbad:
                x, y = k
                grd[x][y] = '0'
            dit.pop(k)

for i in range(n):
    print(*grd[i])
```

#### 法3：染色问题（DFS搜索，转换思想）

**思路**：

- 假定方格最外层填了一圈0
- 将扩圈后方格左上角0作为起点，并标记当前点为 True
- 对四个方向判定，如果下一个为0且没有越界，则递归执行上述标记判定流程
- 以达到标记所有外界联通0的情况
- 这里很巧妙的是填充一圈0，以保证可以只选择一个起点作为递归入口
- 便能通过内层递归将原方格所有边界点带入递归

```
from collections import deque
import sys

n = int(input())
row = ['0']*(n+2)
grd = []
grd.append(row)
for _ in range(n):
    tp = ['0']*(n+2)
    li = sys.stdin.readline().strip().split()
    for i in range(n):
        tp[i+1] = li[i]
    grd.append(tp)
grd.append(['0']*(n+2))

mk = [[False]*(n+2) for _ in range(n+2)]
dirc = [(1,0), (-1,0), (0,1), (0,-1)]

def dfs_ot(cod):
    i, j = cod
    mk[i][j] = True
    for di, dj in dirc:
        ci, cj = i+di, j+dj
        if 0 <= ci < n+2 and 0 <= cj < n+2 and grd[ci][cj] == '0' and not mk[ci][cj]:
            dfs_ot((ci, cj))

dfs_ot((0,0))

for i in range(n+2):
    for j in range(n+2):
        if grd[i][j] == '0' and not mk[i][j]:
            grd[i][j] = '2'

for i in range(1, n+1):
    print(*grd[i][1:n+1])
```

还是挺神奇的。

---

### 2、棋盘（洛谷P3956）

这题有一个魔术过程，且魔法是设定当前位置为任意一种颜色，且不能连续使用魔法。

**核心思路**：

- 初始给地图颜色标记组做一个处理，将可以施魔法的区域先标记为2
- 定义一个字典存储所有可到达点的最小花费
- 初始传入开始节点对应信息，初始距离设置为0
- 弹出队列元素，提取当前点坐标信息
- 遍历四个next点，如果不在可到达范围内则continue
- 如果下个点是可选颜色点且当前点也是可选颜色点，跳过（不能连续使用颜色）
- 队列元素设计时需要多加两个存量：点颜色和路径到该选定颜色变色点最小价格
- 如果nx点颜色可选则移动代价为2，选择nx点颜色与父节点相同
- 如果当前点初始有色且与父点颜色相同则移动代价为0，有色不相等则移动代价为1
- 判断当前路线该nx点的合计代价是否小于该nx点记录的最小代价，小于则加入队列

**示例数据：**

 3 4

 1 1 0

 1 2 1

 2 1 0

 2 3 1

 这组数据，如果不单独对每个点传递实际路径长情况，比如假定前面已由1,1→2,1，更新(2,2)为最小值为2，对应的2,3代价为3，然后1,1→1,2这条路又来看到它的记录值已经小于自己，就直接将这个最小值当做自己最优策略下对应自己颜色的最小值，结果就是2,2变为1色，且2,2记录路径还是2，则2,3路径就变成2这个错误答案。

```
from collections import deque

m, n = map(int, input().split())
grd = [[-1]*m for _ in range(m)]
pcs = []
derc = [(0,1),(0,-1),(1,0),(-1,0)]
mnys = {}

for i in range(n):
    pi, pj, pc = map(int, input().split())
    grd[pi-1][pj-1] = pc
    pcs.append((pi-1, pj-1))
    mnys[(pi-1, pj-1)] = 1000

for i in range(n):
    ci, cj = pcs[i]
    for di, dj in derc:
        if not (0 <= ci+di < m and 0 <= cj+dj < m):
            continue
        if grd[ci+di][cj+dj] == -1:
            grd[ci+di][cj+dj] = 2
            mnys[(ci+di, cj+dj)] = 1000

mnys[(0,0)] = 0

def main():
    if (m-1, m-1) not in mnys:
        print(-1)
        return
    deq = deque([(0, 0, grd[0], 0)])
    while deq:
        ci, cj, cl, cmy = deq.popleft()
        for di, dj in derc:
            ni, nj = ci+di, cj+dj
            nmy = cmy
            if ni < 0 or ni >= m or nj < 0 or nj >= m:
                continue
            ncl = grd[ni][nj]
            if ncl == -1 or (ncl == 2 and grd[ci][cj] == 2):
                continue
            if ncl == 2:
                nmy += 2
                ncl = cl
            else:
                nmy += 0 if ncl == cl else 1
            if nmy < mnys[(ni, nj)]:
                mnys[(ni, nj)] = nmy
                deq.append((ni, nj, ncl, nmy))
    if mnys[(m-1, m-1)] == 1000:
        print(-1)
    else:
        print(mnys[(m-1, m-1)])

main()
```

#### 解法2：基于Dijkstra思想，使用优先队列

**算法思想理解**：

- 使用优先队列可以保证每次弹出的点永远是当前队列中基于 ct 值最小的节点
- 当这个点第一次弹出时 dis 所存的该点代价一定是最小的
- 但是对于邻居的处理：如果当前点已经得到它的最小代价，它到邻居代价不一定也是最小的
- 如果这个点与它邻居的边权远大于第二个代价的点到这个邻居点的权值，即代价和更小
- 这个邻居点的最小代价就是由那个点决定的，只是说因为那个点代价值大于当前点，所以暂时没有弹出来
- 等到那个第二代价的点发现这个邻居还可以更新，就修改字典它对应的代价值，并加入到队列
- 如果这次加入的确实是到达该点最小的情况，等到这个新加入的情况优先弹出时
- 此时这个点对应的代价存储字典的值就是最小的
- 前面先加入的不是最小的情况就可以在弹出时通过判断是否小于字典存储的最小值进行过滤

具体方法：方格预处理大致和上述方法一致，代价存储字典多定义一个颜色维度用于控制是否入队，每个入队节点元素构造为 (ct, ci, cj, cl) 的形式，其中 cl 表示当前点颜色，如果当前点是可变色，则表示进入它的父亲点颜色。移动代价的更新也和前面思路差不多。

写到这里，感觉第一种思路确实不够好，它能实现的原因是因为这个边权实际简化后只有0，1两种形式，可能通过条件也可以进行简单过滤，如果拓展点变大可能确实耗时（因为可能重复搜索，最终也能收敛），本质上少了一个过滤的区别。

```
import sys
import heapq

def slv():
    input = sys.stdin.readline
    m, n = map(int, input().split())
    grd = [[-1]*(m+2) for _ in range(m+2)]
    ps = []
    for i in range(n):
        ci, cj, cl = map(int, input().split())
        ps.append([ci, cj, cl])
        grd[ci][cj] = cl

    derc = [(1,0), (-1,0), (0,1), (0,-1)]
    for ci, cj, cl in ps:
        for di, dj in derc:
            if grd[ci+di][cj+dj] == -1:
                grd[ci+di][cj+dj] = 2
    if grd[m][m] == -1:
        print(-1)
        return

    INF = 10 ** 9
    dis = [[[INF, INF] for _ in range(m+2)] for _ in range(m+2)]
    pq = [(0, 1, 1, grd[1](@ref)]
    dis[grd[1]] = 0
    while pq:
        ct, ci, cj, cl = heapq.heappop(pq)
        if ci == m and cj == m:
            break
        if ct > dis[ci][cj][cl]:
            continue
        for di, dj in derc:
            ni, nj = ci+di, cj+dj
            nct, ncl = ct, grd[ni][nj]
            if ncl == -1:
                continue
            if ncl == 2:
                if grd[ci][cj] == 2:
                    continue
                nct += 2
            else:
                nct += 0 if cl == ncl else 1
            if nct < dis[ni][nj][ncl]:
                dis[ni][nj][ncl] = nct
                heapq.heappush(pq, (nct, ni, nj, ncl))
    ans = min(dis[m][m])
    print(ans if ans < INF else -1)

if __name__ == '__main__':
    slv()
```

#### 解法3：标准SPFA解法（使用三维数组存储每个点对应不同颜色最低代价）

**核心思路**：结合dijkstra算法代码形式，但是还是使用普通队列，核心点和法一差不多。

```
import sys
from collections import deque

def solve_spfa():
    input = sys.stdin.readline
    m, n = map(int, input().split())
    grd = [[-1]*m for _ in range(m)]
    pcs = []
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    INF = 10**9
    for i in range(n):
        pi, pj, pc = map(int, input().split())
        grd[pi-1][pj-1] = pc
        pcs.append((pi-1, pj-1))
    for i in range(n):
        ci, cj = pcs[i]
        for di, dj in dirs:
            if not (0 <= ci+di < m and 0 <= cj+dj < m):
                continue
            if grd[ci+di][cj+dj] == -1:
                grd[ci+di][cj+dj] = 2
    if grd[m-1][m-1] == -1:
        return -1
    dist = [[[INF, INF] for _ in range(m)] for _ in range(m)]
    start_color = grd
    dist[start_color] = 0
    deq = deque([(0, 0, start_color)])
    while deq:
        x, y, col = deq.popleft()
        cost = dist[x][y][col]
        for dx, dy in dirs:
            nx, ny = x+dx, y+dy
            if not (0 <= nx < m and 0 <= ny < m):
                continue
            nxt = grd[nx][ny]
            if nxt == -1:
                continue
            if grd[x][y] == 2 and nxt == 2:
                continue
            if nxt == 2:
                new_cost = cost + 2
                new_col = col
            else:
                new_col = nxt
                new_cost = cost + (0 if col == new_col else 1)
            if new_cost < dist[nx][ny][new_col]:
                dist[nx][ny][new_col] = new_cost
                deq.append((nx, ny, new_col))
    ans = min(dist[m-1][m-1][0], dist[m-1][m-1][1](@ref)
    return ans if ans != INF else -1

print(solve_spfa())
```

#### 解法4（大规模数据不太稳定，受代价分配比例影响）

**思想**：和法三类似，因为固定色点只有一种最小值所以就思考每个点只存储最小值和最小值对应颜色，而不单独存储不同颜色对应的该点最小值。单独处理可变色点进入和取出的逻辑，如果当前进入可变色，可变色设定颜色和可变色点记录最小值对应颜色不等，但是两种颜色当前算出来又都可以作最小值，就按着原来的逻辑设定其颜色为2，这样当它被弹出来的时候它的颜色就不固定，存入时需要单独标记处理。

```
from collections import deque

m, n = map(int, input().split())
grd = [[-1]*m for _ in range(m)]
pcs = []
derc = [(0,1),(0,-1),(1,0),(-1,0)]
mnys = {}
INF = 10**9

for i in range(n):
    pi, pj, pc = map(int, input().split())
    grd[pi-1][pj-1] = pc
    pcs.append((pi-1, pj-1))
    mnys[(pi-1, pj-1)] = [INF, pc]

for i in range(n):
    ci, cj = pcs[i]
    for di, dj in derc:
        if not (0 <= ci+di < m and 0 <= cj+dj < m):
            continue
        if grd[ci+di][cj+dj] == -1:
            grd[ci+di][cj+dj] = 2
            mnys[(ci+di, cj+dj)] = [INF, 2]

mnys[(0,0)] = [0, grd[0]]

def main():
    if (m-1, m-1) not in mnys:
        return -1
    deq = deque([(0, 0)])
    while deq:
        ci, cj = deq.popleft()
        cmy = mnys[(ci, cj)]
        cl = mnys[(ci, cj)]
        for di, dj in derc:
            ni, nj = ci+di, cj+dj
            if ni < 0 or ni >= m or nj < 0 or nj >= m or grd[ni][nj] == -1:
                continue
            nmy = cmy
            ncl = mnys[(ni, nj)]
            if grd[ci][cj] == 2 and grd[ni][nj] == 2:
                continue
            if cl == 2:
                nmy += 0
            elif grd[ni][nj] != 2:
                nmy += 0 if cl == ncl else 1
            else:
                nmy += 2
                if nmy == mnys[(ni, nj)][0] and cl != ncl and ncl != 2:
                    mnys[(ni, nj)] = [nmy, 2]
                    deq.append((ni, nj))
                    continue
                ncl = cl
            if nmy < mnys[(ni, nj)][0]:
                mnys[(ni, nj)] = [nmy, ncl]
                deq.append((ni, nj))
    ans = mnys[(m-1, m-1)]
    return ans if ans < INF else -1

print(main())
```

使用大规模数据（300*300表格，30000变色点）是有点问题（改变权值分别为1,3,4和标准结果差1），暂时没想到怎么解决。

#### 测试代码（随机棋盘生成与两种算法对比）

以下为随机棋盘生成器与 SPFA / Dijkstra 的对比测试代码，用于验证算法的正确性和性能。

```
import sys
from collections import deque
import heapq
import time
import random
import io

def generate_valid_board(m, n, seed=None, max_attempts=100):
    if seed is not None:
        random.seed(seed)

    dirs = [(-1,0),(1,0),(0,-1),(0,1)]

    def has_valid_path(board):
        if board[1] == -1 or board[m][m] == -1:
            return False
        visited = [[False]*(m+1) for _ in range(m+1)]
        q = deque()
        q.append((1,1))
        visited[1] = True
        while q:
            x,y = q.popleft()
            if x==m and y==m:
                return True
            for dx,dy in dirs:
                nx,ny = x+dx, y+dy
                if 1<=nx<=m and 1<=ny<=m and not visited[nx][ny] and board[nx][ny] != -1:
                    if board[x][y] == 2 and board[nx][ny] == 2:
                        continue
                    visited[nx][ny] = True
                    q.append((nx,ny))
        return False

    for attempt in range(max_attempts):
        points = [(1,1,0), (m,m,1)]
        while len(points) < n:
            x = random.randint(1, m)
            y = random.randint(1, m)
            c = random.randint(0, 1)
            if (x,y) not in [(p[0],p[1](@ref) for p in points]:
                points.append((x,y,c))

        board = [[-1]*(m+1) for _ in range(m+1)]
        for x,y,c in points:
            board[x][y] = c

        for x,y,_ in points:
            for dx,dy in dirs:
                nx, ny = x+dx, y+dy
                if 1<=nx<=m and 1<=ny<=m and board[nx][ny]==-1:
                    board[nx][ny] = 2

        if has_valid_path(board):
            return board, points
    return None, None

def solve_spfa(data):
    sys.stdin = io.StringIO(data)
    m,n = map(int,input().split())
    grd = [[-1]*m for _ in range(m)]
    pcs = []
    derc = [(0,1),(0,-1),(1,0),(-1,0)]
    mnys = {}
    INF = 10**9
    for i in range(n):
        pi,pj,pc = map(int,input().split())
        grd[pi-1][pj-1] = pc
        pcs.append((pi-1,pj-1))
        mnys[(pi-1,pj-1)] = [INF,pc]
    for i in range(n):
        ci,cj = pcs[i]
        for di,dj in derc:
            if not (0<=ci+di=m or nj<0 or nj>=m or grd[ni][nj] == -1:
                continue
            nmy = cmy
            ncl = mnys[(ni,nj)]
            if grd[ci][cj] == 2 and grd[ni][nj] == 2:
                continue
            if cl == 2:
                nmy += 4
            elif grd[ni][nj]!=2:
                nmy += 1 if cl==ncl else 3
            else:
                nmy += 4
                if nmy == mnys[(ni,nj)][0] and cl != ncl:
                    mnys[(ni,nj)] = [nmy,2]
                    deq.append((ni,nj))
                    continue
                ncl = cl
            if nmy <  mnys[(ni,nj)][0]:
                mnys[(ni,nj)] = [nmy,ncl]
                deq.append((ni,nj))
    ans = mnys[(m-1,m-1)]
    return ans if ans dist[x][y][col]:
            continue
        for dx, dy in dirs:
            nx, ny = x+dx, y+dy
            if not (1<=nx<=m and 1<=ny<=m): continue
            nxt = board[nx][ny]
            if nxt == -1: continue
            if board[x][y] == 2 and nxt == 2: continue
            if nxt == 2:
                new_cost = cost + 4
                new_col = col
            else:
                new_col = nxt
                new_cost = cost + (1 if col == new_col else 3)
            if new_cost < dist[nx][ny][new_col]:
                dist[nx][ny][new_col] = new_cost
                heapq.heappush(pq, (new_cost, nx, ny, new_col))

    ans = min(dist[m][m][0], dist[m][m][1](@ref)
    return ans if ans < INF else -1

if __name__ == "__main__":
    m = 300
    n = 30000
    seed = 20240213
    max_attempts = 50

    print(f"正在生成 {m}×{m} 棋盘，彩色点数 {n}，种子 {seed}")
    board, points = generate_valid_board(m, n, seed=seed, max_attempts=max_attempts)
    if board is None:
        print("生成失败")
        sys.exit(1)
    lines = [f"{m} {len(points)}"]
    for x,y,c in points:
        lines.append(f"{x} {y} {c}")
    data = "\n".join(lines)

    start = time.perf_counter()
    ans_dij = solve_dijkstra_forbidden(data)
    time_dij = time.perf_counter() - start

    start = time.perf_counter()
    ans_spfa = solve_spfa(data)
    time_spfa = time.perf_counter() - start

    print(f"\nSPFA 答案: {ans_spfa}  | 耗时 {time_spfa:.4f} 秒")
    print(f"Dijkstra 答案: {ans_dij} | 耗时 {time_dij:.4f} 秒")

    if ans_spfa == ans_dij:
        print(f"\n✅ 答案一致，均为 {ans_spfa}")
        print(f"性能比 (SPFA/Dijkstra): {time_spfa/time_dij:.2f}")
    else:
        print(f"\n❌ 答案不一致！SPFA={ans_spfa}, Dijkstra={ans_dij}")
```

总体而言，dijkstra优先队列解法更稳定，且时间更快，快1到10倍。

---

### 3、多轮BFS的访问标记的巧妙应用（P1902）

个人思想精华

**思路**：

- 使用一个 flhs 字典，用于存储每轮 BFS 在当前 cn 限制下刚好遇到的边界
- 搜索时遇到 grd[ni][nj] > cn，表示这条路径到这里因为 cn 的限制断了
- 就把这个点存入这个 flhs 字典（当线性搜索传入的 cn 变大时，就可以从这些点作为起点）
- 否则标记并加入 deque 队列，继续搜索，直到把所有在当前 cn 限制下能搜索到的点都访问一遍
- 如果新的 cn 来了，就把这个新 cn 在 flhs 字典中存储的点集作为该轮 bfs 起点
- 重复，直到某个 cn 刚好访问到了出口，则返回 True，对应 cn 就是最小值

```
import sys
from collections import deque

input = sys.stdin.readline
n, m = map(int, input().strip().split())
grd = [[0]*m]
rmg = 0
rng = 10**9
input()
for i in range(n-2):
    rl = list(map(int, input().strip().split()))
    rmg = max(rmg, max(rl))
    rng = min(rng, min(rl))
    grd.append(rl)
input()
grd.append([0]*m)

dirs = [(1,0), (-1,0), (0,1), (0,-1)]
msk = [[False]*m for _ in range(n)]
flhs = [[] for _ in range(1001)]
deq = deque()

for i in range(m):
    deq.append((0, i))
    msk[i] = True

def vrf_rd(cn):
    cps = flhs[cn]
    while cps:
        cx, cy = cps.pop()
        deq.append((cx, cy))
        msk[cx][cy] = True
    while deq:
        ci, cj = deq.pop()
        for di, dj in dirs:
            ni, nj = ci+di, cj+dj
            if ni < 0 or ni >= n or nj < 0 or nj >= m:
                continue
            if ni == n-1:
                return True
            if msk[ni][nj]:
                continue
            if grd[ni][nj] <= cn:
                deq.append((ni, nj))
                msk[ni][nj] = True
                continue
            msk[ni][nj] = True
            flhs[grd[ni][nj]].append((ni, nj))
    return False

def src():
    lf = rng
    hg = rmg + 1
    while lf < hg:
        if vrf_rd(lf):
            return lf
        else:
            lf += 1

print(src())
```
---

© 2026 个人笔记 · 转载需注明出处

 本作品采用

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

 许可协议