# BFS广搜
## 一、核心模板
1、二维网格BFS(迷宫问题):
核心思路，创建一个队列，用于存每个第一次读入的节点，然后使用vis数组标记已经访问的节点，如果已经访问则不用再加入队列，然后依次弹出队列节点，对每个节点根据其可到达的四个方向坐标进行遍历，没访问过的，加入队列，并标记访问，其路径长就为当前节点路径加1，直到某个节点的等于目标点，就返回其路径。
```python
from collections import deque
def bfs_grid(grd,st,tg):
    if not grd or grd[0]:
        return -1
    rws = len(grd)
    cls = len(grd[0])
    dircs = [(1,0),(0,1),(-1,0),(0,-1)]
    deq = deque([st])
    vis = set([st])
    stps = {st:0}
    while deq:
        x,y = deque.popleft()
        if (x,y) == tg:
            return stps[(x,y)]
        for dx,dy in dircs:
            nx,ny = x+dx,y+dy
            if 0<=nx<rws and 0<=y<cls and grid[nx][ny] == 0
                if (nx,ny) not in vis:
                    vis.add((nx,ny))
                    deq.append((nx,ny))
                    stps[(nx,ny)] = stps[(x,y)]+1
    return -1
2、层序遍历 BFS，计算层数，最短路径核心思路：主要还是用列表存储第一次遍历的节点，然后依次取出，依次将其子节点加入到列表 (如果没有遍历过的话)，主要区别是一个层次路径长，计算方法是在循环体开头计算当前队列元素个数，然后根据这个长度 (当前层元素个数)，如果遍历完当前层所有元素 (即依次完成对当前层各个节点子节点的添加)，就将 stp 加 1 (当前层所有子节点路径长相等)，最后如果某个弹出元素等于目标节点就返回 stp
python
运行
from collections import deque
def bfs_lev(st,tg):
    if st==tg:
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
除了这种统一计算每一层所有元素路径，还可以向网格遍历那样使用 stps 字典对每个节点路径进行记录，如下：
python
运行
stps = {st:0}
while deq:
	node = deq.popleft()
	for neb in get_nebighbors(node):
		if neb not in vis:
			vis.add(neb)
			deq.append(neb)
			stps[neb] = stps[node] + 1
二、常见例题
1、迷宫最短路径 (返回 st 到 ed 最短路径，不能穿过障碍物)：类似于模板 2 解法，但是 node 节点换成二维坐标，初始化一个二维数组用于标记访问情况，使用层次遍历计算路径
python
运行
from collections import deque
def min_dis(grd,st,ed):
    if st==ed:
        return False
    rws = len(grd)
    cls = len(grd[0])
    dirc = [(1,0),(-1,0),(0,1),(0,-1)]
    vis = [[False]*cls]*rws
    vis[st[0]][st[1]] = True
    stp = 0
    deq = deque([st])
    while deq:
        size = len(deq)
        for _ in size:
            x,y = deq.popleft()
            if (x,y) == ed:
                return stp
            for dx,dy in dirc:
                nx,ny = x+dx,y+dy
                if nx<=x<rws and ny<=0<cls and not vis[nx][ny] and grd[nx][ny]==0:
                    vis[nx][ny] = True
                    deq.append((nx,ny))
        stp += 1
    return -1
2、 多源 BFS (多个起点同时扩散) 返回每个位置到最近起点距离。思路和基本模板一致，但是对于多个源头，在初始化队列时，先将每一个源头依次加入队列，并且将对应路径设置为 0，还有就是因为多源 BFS，对每一个坐标专门使用 dist 数组存放对应路径，所以可以和标记访问数组合并为一个效果，即初始化每个坐标对应 dist 为 - 1，既表示还没有访问过，也存储路径。对于每个到达的节点，判断时直接使用 dist 是否等于 - 1 判断是否到达过，没到达过然后将其路径从 - 1 更新为到达它的坐标的路径长 + 1。
python
运行
from collections import deque
def msr_bfs(grd,srcs):
    rws = len(grd)
    cls = len(grd[0])
    deq = deque()
    dis = [[-1]*cls]*rws
    dirc = [(1,0),(-1,0),(0.-1),(0,1)]
    for x,y in srcs:
        deq.append((x,y))
        dis[x][y] = 0
    while deq:
        x,y = deq.popleft()
        for dx,dy in dirc:
            nx,ny = x+dx,y+dy
            if 0<=nx<rws and 0<=ny<cls and dis[nx][ny] == -1 and grd[nx][ny] == 0:
                dis[nx][ny] = dis[x][y] + 1
                deq.append((x,y))
    return dis
3、 双向 bfs (从起点和终点同时开始搜索，相遇结束，优化空间)核心就是维护两个队列，两个距离存储数组，对于每一层遍历，分为从前往后，从后往前两个方向的遍历，首先根据从前往后搜索的队列长度，计算出当前层待启动节点数，然后对每个节点，依次遍历其子节点，如果子节点已经在从后往前的距离存储字典中，说明相遇，可以直接返回从前往后到当前点对应路径 + 存储的从后往前到当前节点的距离值。如果不在，说明还没相遇，按常规模板处理当前节点 (不在从前往后记录中，就记录，并将该邻居节点加入到从前往后队列)，同理对从后往前搜索进行类似操作。
python
运行
from collections import deque
def bdrc_bfs(st,tg,get_nbs):
    """
    双向BFS模板
    从起点和终点分别开始搜索，相遇时结束
    """
    if st == tg:
        return 0
    deq_st = deque([st])
    deq_tg = deque([tg])
    vis_st = {st:0}  # 两个方向同时记录距离
    vis_tg = {tg:0}
    while deq_st and deq_tg:
        # 从开始方向
        st_sz = len(deq_st)
        for _ in range(st_ze):
            node = deq_st.popleft()
            for neb in get_nebs(node):
                if neb in vis_tg:
                    return vis_st[node]+1+vis_tg[neb]
                if neb not in vis_st:
                    vis_st[neb] = vis_st[node]+1
                    deq_st.append(neb)
        tg_sz = len(deq_tg)
        for _ in range(tg_sz):
            node = deq_tg.popleft()
            for neb in get_nebs(node):
                if neb in vis_st:
                    return vis_tg[node]+1+vis_st[neb]
                if neb not in vis_tg:
                    vis_tg[neb] = vis_tg[node]+1
                    deq_tg.append(neb)
    return -1 # 不连通
关键就是整两个队列，两个距离记录，然后使用邻居来判断是否相遇，相遇就加上从前往后从后往前到该节点各自距离和
4、 回溯 bfs (使用 prv 记录前驱，求最短路径)第十届 javaB 组 迷宫关键，使用 prv 数组记录到达每一个点的走法，因为每一步权值为 1, 第一次到达某个点时必然就是到达该点的最短路径，使用 prv 记录从上一个点，到达该点对应选择的方向，最后当弹出点时出口时，即可根据 prv 数组回溯得到最短路径，并结束 bfs 搜索。
python
运行
import sys
from collections import deque
def main():
    gd = []
    n,m=30,50
    for _ in range(n):
        rl = sys.stdin.readline().strip()
        gd.append(rl)
    if gd[0][0] == '1' or gd[n-1][m-1] == '1':
        print("")
        return
    # 方向按字典序排列（D < L < R < U）
    dirs = [('D', 1, 0), ('L', 0, -1), ('R', 0, 1), ('U', -1, 0)]
    mk = [[False] * m for _ in range(n)]
    q = deque()
    q.append((0, 0))
    mk[0][0] = True
    prv = [[None]*m for _ in range(n)]
    def bac():
        i,j = n-1,m-1
        rs = []
        while (i,j) != (0,0):
            pd = prv[i][j]
            rs.append(pd)
            for dr,di,dj in dirs:
                if dr==pd:
                    i,j = i-di,j-dj
                    break
        print(''.join(reversed(rs)))
        return 
    def chk_ps(i,j):
        if not (0<=i<n and 0<=j<m) or gd[i][j]=='1' or mk[i][j]:
            return True
        return False
    while q:
        ci,cj = q.popleft()
        if ci==n-1 and cj==m-1:
            bac()
            return 
        for ch,di,dj in dirs:
            ni,nj = ci+di,cj+dj
            if chk_ps(ni,nj):
                continue
            mk[ni][nj] = True
            prv[ni][nj] = ch
            q.append((ni,nj))
    # 无路径
    print("")
if __name__ == "__main__":
    main()
这里还要求路径字典序最小，使用的处理方式就是根据方向字母调整遍历顺序，保证当前选择方向始终是当前步数下当点出发的字典序最小的，从而第一次到达点是，路径字典序最小。还是有点意思。
三、练习题
1、填涂颜色 (寻找方格数字被 1 围起来的所有 0)法 1：排除与外面连通的 0思路：初始化队列，并添加所有边界 0，看成一个多源 BFS 问题，然后依次弹出元素，找四个方向没有添加过 (标记) 的 0，将其加入队列，并标记，这样当队列为空时，所有与外界想通的 0 都被标记，剩下的没有被标记的 0 就是封闭 0.
python
运行
from collections import deque
import sys
grd = []
n = int(input())
for _ in range(n):
    row = sys.stdin.readline().strip().split()
    grd.append(row)
def fd_cls():
    rws,cls = n,n
    vis = [[False]*cls for _ in range(rws)]
    deq = deque()
    dirc = [(0,1),(0,-1),(1,0),(-1,0)]
    for i in range(rws):  # 添加第一列和最后一列的0
        for j in [0,cls-1]:
            if grd[i][j]== '0':
                deq.append((i,j))
                vis[i][j] = True # 这里标记主要是防止四角重复添加
    for j in range(cls):   # 添加第一行和最后一行的0
        for i in [0,rws-1]:
            if grd[i][j] == '0' and not vis[i][j]:
                deq.append((i,j))
                vis[i][j] = True
    while deq:
        i,j = deq.popleft()
        for di,dj in dirc:
            ci,cj = i+di,j+dj 
            if 0<=ci<rws and 0<=cj<cls and grd[ci][cj]=='0' and not vis[ci][cj]: 
                deq.append((ci,cj))
                vis[ci][cj] = True  # 标记所有与外界连通0
    for i in range(rws):
        for j in range(cls):
            if grd[i][j]=='0' and not vis[i][j]:
                grd[i][j]='2'
fd_cls()
for i in range(n):
    print(*grd[i])
法 2：寻找可能 0，保留封闭 0这里我想的思路是先在输入时处理数据，定义记录每一行 (记录列最大范围) 和每一列 (记录行最大范围) 的被 1 包围范围的两个数组，然后在输入方格时就简单维护这两个记录数组，则所有封闭 0 坐标必然满足改行对应封闭列范围和该列对应封闭行范围 (但是这些范围对应 0 并不总是满足封闭，因为可能出现连通外界情况)，所以需要把这些 0 中能与外界联通的部分筛除。具体筛除方法，就是首先用 dit 字典标记这些 0 坐标，然后将这些坐标添加到列表，复制这个标记字典到 dits 字典，然后对于每一个坐标，将其作为队列头部，进行 bfs 搜素依次找它四个方向，如果元素在是初步筛选范围中 0，就将用 dit 字典标记为已访问，然后添加继续到队列，寻找这块连通 0，或者如果元素不是初步筛选中的 0，说明这块与外界联通，使用 isbad 标记当前块不满足，最后当前块搜索结束后，在 dit 字典中将所有当前块中点删除 (本轮 bfs 中所有 dit 值为 True 的点)，并且如果该块标记了 isbad，则说明该块所有 0 与外界连通，应当在范围内 0 集中筛除，然后继续根据复制的 dits 字典选取新的坐标开始新的轮次 bfs，如果这个新坐标已经不在 dit 字典中，说明当前坐标对应块已经是验证完了的，直接跳过，而不必重复遍历所在块，以达到控制对不同块的 bfs。主要是剔除部分还是挺抽象的 (复制了一个字典做母版，并删除子版字典元素，对两个字典进行比对，来控制块的不重复访问)，想了好久。
python
运行
import sys
from collections import deque
n = int(input())
grd = []
cls = [[-1,-1,-1] for _ in range(n)]
rws = [[-1,-1,-1] for _ in range(n)]
scrs = []
for i in range(n):
    row = sys.stdin.readline().strip().split()
    grd.append(row)
    for j in range(n):
        if row[j] != '0': # 当前位置为1，则当前位置为纵向起点
            if cls[j][0] == -1:
                cls[j][0] = i
            cls[j][2] = i
            if rws[i][0] == -1:
                rws[i][0] = j
            rws[i][2] = j
for i in range(n):
    cls[i][1] = cls[i][2]
    rws[i][1] = rws[i][2]
dit = {}
for i in range(n):
    for j in range(n):
        if grd[i][j]=='0' and cls[j][0]<i<cls[j][1] and rws[i][0]<j<rws[i][1]:
            grd[i][j] = '2'
            dit[(i,j)] = False
dits = dit.copy()       
dirc = [(1,0),(-1,0),(0,1),(0,-1)]
for i,j in dits:
    if (i,j) not in dit.keys():
        continue
    deq = deque()
    isbad = False
    deq.append((i,j))
    dit[(i,j)] = True
    while deq:
        ri,ci = deq.popleft()
        for dx,dy in dirc:
            nr,nc = ri+dx,ci+dy
            if (nr,nc) in dit and not dit[(nr,nc)]:
                deq.append((nr,nc))
                dit[(nr,nc)] = True
            if grd[nr][nc] == '0':
                isbad = True
    for k in dits:
        x,y = k
        if k in dit and dit[k]:
            if isbad:
                grd[x][y]='0'
            dit.pop(k)
for i in range(n):
    print(*grd[i])
拓展法 3：染色问题 (dfs 搜索，转换思想）思路：假定方格最外层填了一圈 0，将扩圈后方格左上角 0 作为起点，并标记当前点为 False, 对四个方向判定，如果下一个为 0 且没有越界，则递归执行上述标记判定流程，以达到标记所有外界联通 0 的情况。这里很巧妙的是填充一圈 0，以保证可以只选择一个起点作为递归入口，便能通过内层递归将原方格所有边界点带入递归。
python
运行
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
dirc = [(1,0),(-1,0),(0,1),(0,-1)]
def dfs_ot(cod):
    i,j = cod
    mk[i][j] = True
    for di,dj in dirc:
        ci,cj=i+di,j+dj
        if 0<=ci<n+2 and 0<=cj<n+2 and grd[ci][cj]=='0' and not mk[ci][cj]:
            dfs_ot((ci,cj))
dfs_ot((0,0))
for i in range(n+2):
    for j in range(n+2):
        if grd[i][j] == '0' and not mk[i][j]:
            grd[i][j] = '2'
for i in range(1,n+1):
    print(*grd[i][1:n+1])
还是挺神奇的
2、棋盘 (洛谷 P3956)因为这题它有一个魔术过程，且魔法是设定当前位置为任意一种颜色，且不能连续使用魔法，所以可以初始给地图颜色标记组 (grd 存储) 做一个处理，将可以施魔法的区域先标记为 2，这些可以施魔法的区域和有颜色区域总的就是能到达的点，对于此，可以先定义一个字典存储所有这些可到达点的最小花费 (用于后续判断是否添加新的路径，如果某个点当前路径到达该点后花费小于这个记录的最小值，才添加包含（该点，这条新路经到达该点对应价格，和这个点对应颜色) 的队列元素。然后初始传入开始节点对应信息，注意初始距离设置为 0，其他值在字典定义存储时默认设定为无穷大。然后弹出队列元素，并提取当前点坐标信息，这个队列元素代表路径到当前点对应价格，当前点设定颜色。对于弹出每个点，首先遍历它的四个 next 点，如果某个 nx 点不在可到达范围内，则不用继续添加直接 continue，或者下个点是可选择颜色点并且当前所站位置点也是可选颜色点，也跳过，因为不能连续使用颜色，对于每个空白点，因为它颜色的选择如果要满足不会因为到达它时设定颜色不同而对后续价格维护产生影响，所以队列元素设计时需要多加两个存量 (点颜色和路径到该选定颜色变色点最小价格，而不是直接根据字典取存储点价格 (可能这个最小价格对应的不是该颜色) 进行维护更新)。如果 nx 点颜色可选则移动代价为 2，因为整个搜索过程价格不管哪条路径，代价最小始终是从前往后推，所以选择 nx 点颜色与父节点相同，只增加变色代价 (也因此如前述需要单独区分不同颜色对应到该点的代价)，如果当前点初始有色且与父点颜色相同则移动代价为 0, 有色不相等则移动代价为 1，最后判断当前路线该 nx 点的合计代价是否小于该 nx 点记录的最小代价，小于则将该点加入队列。这里使用的是普通队列，没有使用优先队列，但是因为缩减了大量范围，且传递每条路径距离详细信息，所以并没有因为可变色点对应不同弹出顺序导致可能的错误结果
python
运行
from collections import deque
m,n = map(int,input().split())
grd = [[-1]*m for _ in range(m)]
pcs = []
derc = [(0,1),(0,-1),(1,0),(-1,0)]
mnys = {}
cnt = 0
for i in range(n):
    pi,pj,pc = map(int,input().split())
    grd[pi-1][pj-1] = pc
    pcs.append((pi-1,pj-1))
    mnys[(pi-1,pj-1)] = 1000
for i in range(n):
    ci,cj = pcs[i]
    for di,dj in derc:
        if not (0<=ci+di<m and 0<=cj+dj<m):
            continue
        if grd[ci+di][cj+dj]==-1:
            grd[ci+di][cj+dj]=2
            mnys[(ci+di,cj+dj)] = 1000
mnys[(0,0)] = 0
def main():
    ans = []
    if (m-1,m-1) not in mnys:
        print(-1)
        return
    deq = deque([(0,0,grd[0][0],0)])
    while deq:
        ci,cj,cl,cmy = deq.popleft()
        if cmy < mnys[(ni,nj)]:
            continue
        for di,dj in derc:
            ni,nj=ci+di,cj+dj
            nmy = cmy
            if ni<0 or ni>=m or nj<0 or nj>=m:
                continue
            ncl = grd[ni][nj]                
            if ncl==-1 or (ncl==2 and grd[ci][cj] == 2):
                continue
            if ncl==2:
                nmy += 2 
                ncl = cl
            else:
                nmy += 0 if ncl==cl else 1
            if nmy < mnys[(ni,nj)]:
                mnys[(ni,nj)] = nmy
                deq.append((ni,nj,ncl,nmy))
    if mnys[(m-1,m-1)] == 1000:
        print(-1)
    else:
        print(mnys[(m-1,m-1)])
main()
解法 2、基于 Dijkstra 思想，使用优先队列算法思想理解：使用优先队列可以保证每次弹出的点永远是当前队列中基于 ct 值最小的节点，这样当这个点第一次弹出时 dis 所存的该点代价一定是最小的，但是对于邻居的处理是怎样的呐，如果当前点已经得到它的最小代价，它到邻居代价一定也是最小的嘛，不一定，如果这个点与它邻居的边权远大于第二个代价的点到这个邻居点的权值，即代价和更小，这个邻居点的最小代价就是由那个点决定的，只是说因为那个点代价值大于当前点，所以暂时没有弹出来。这样这一轮次它对邻居节点的更新可能也不是到达邻居实际的最小代价（即使加入了队列），等到这个点处理完邻居点，那个第二代价的点发现这个邻居还可以更新，就修改字典它对应的代价值，并加入到队列，这样如果这次加入的确实是到达该点最小的情况了，等到这个新加入的情况优先弹出（因为代价更小）时，此时这个点对应的代价存储字典的值就是最小的，而前面先加入的不是当下不是最小的情况就可以在弹出时 (必然已经确定该点最小代价值了) 通过判断是否小于字典存储的最小值进行过滤
python
运行
import sys
import heapq
def slv():
    input = sys.stdin.readline
    m,n = map(int,input().split())
    grd = [[-1]*(m+2) for _ in range(m+2)]
    ps = []
    for i in range(n):
        ci,cj,cl = map(int,input().split())
        ps.append([ci,cj,cl])
        grd[ci][cj] = cl
    derc = [(1,0),(-1,0),(0,1),(0,-1)]
    for ci,cj,cl in ps:
        for di,dj in derc:
            if grd[ci+di][cj+dj] == -1:
                grd[ci+di][cj+dj] = 2
    if grd[m][m] == -1:
        print(-1)
        return
    INF = 10 ** 9
    dis = [[[INF,INF] for _ in range(m+2)] for _ in range(m+2)]
    pq = [(0,1,1,grd[1][1])]
    dis[1][1][grd[ci][cj]] = 0
    while pq:
        ct,ci,cj,cl = heapq.heappop(pq)
        if ci == m and cj == m:
            break
        if ct > dis[ci][cj][cl]:
            continue
        for di,dj in derc:
            ni,nj = ci+di,cj+dj
            nct,ncl = ct,grd[ni][nj]
            if ncl == -1:
                continue
            if ncl == 2:
                if grd[ci][cj]==2:
                    continue
                nct += 2
            else:
                nct += 0 if cl == ncl else 1
            if nct < dis[ni][nj][cl]:
                dis[ni][nj][cl] = nct
                heapq.heappush(pq,(nct,ni,nj,ncl))
    ans = min(dis[m][m])
    print(ans if ans<INF else -1)
if __name__ == '__main__':
    slv()
解法 3、标准 spfa 解法 (使用三维数组存储每个点对应不同颜色最低代价)：核心思路：结合 dijkstra 算法代码形式，但是还是使用普通队列，核心点和法一差不多
python
运行
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
        pi,pj,pc = map(int,input().split())
        grd[pi-1][pj-1] = pc
        pcs.append((pi-1,pj-1))
    for i in range(n):
        ci,cj = pcs[i]
        for di,dj in dirs:
            if not (0<=ci+di<m and 0<=cj+dj<m): continue
            if grd[ci+di][cj+dj]==-1:
                grd[ci+di][cj+dj]=2
    if grd[m-1][m-1] == -1:
        return -1
    dist = [[[INF, INF] for _ in range(m)] for _ in range(m)] # 这里用列表存储同一个点不同颜色对应最小值
    start_color = grd[0][0]
    dist[0][0][start_color] = 0 # 初始值设为0
    deq = deque([(0, 0, start_color)]) # 相比dijkstra优先队列算法和法1，这里除了传点，只传入颜色，这个颜色是对应点的设定颜色(可变色点)或者本来颜色(固定色)
    while deq:
        x, y, col = deq.popleft()
        cost = dist[x][y][col]
        for dx, dy in dirs:
            nx, ny = x+dx, y+dyS
            if not (0<=nx<m and 0<=ny<m): continue
            nxt = grd[nx][ny]
            if nxt == -1: continue
            if grd[x][y] == 2 and nxt == 2: continue
            if nxt == 2:
                new_cost = cost + 2
                new_col = col  # 邻居是可变色，就继承父亲颜色
            else:
                new_col = nxt
                new_cost = cost + (0 if col == new_col else 1)
            if new_cost < dist[nx][ny][new_col]:  # 只和对应颜色最小值进行判断，代码逻辑清晰简洁，队列元素传递参数和该点最小值下标一一对应，保证结果准确性
                dist[nx][ny][new_col] = new_cost
                deq.append((nx, ny, new_col)) 
    ans = min(dist[m-1][m-1][0], dist[m-1][m-1][1])
    return ans if ans!=INF else -1
print(solve_spfa())
解法 4 （大规模数据不太稳定，受代价分配比例影响）思想：和法三类似，因为固定色点只有一种最小值所以就思考每个点只存储最小值和最小值对应颜色，而不单独存储不同颜色对应的该点最小值，所以单独处理可变色点进入和取出的逻辑，如果当前进入可变色，可变色设定颜色和可变色点记录最小值对应颜色不等，但是两种颜色当前算出来又都可以作最小值，就按着原来的逻辑设定其颜色为 2，这样当它被弹出来的时候它的颜色就不固定，存入时需要单独标记处理。
python
运行
from collections import deque
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
        if not (0<=ci+di<m and 0<=cj+dj<m):
            continue
        if grd[ci+di][cj+dj]==-1:
            grd[ci+di][cj+dj]=2
            mnys[(ci+di,cj+dj)] = [INF,2]
mnys[(0,0)] = [0,grd[0][0]]
def main():
    if (m-1,m-1) not in mnys:
        return -1
    deq = deque([(0,0)])
    while deq:
        ci,cj = deq.popleft()
        cmy = mnys[(ci,cj)][0]
        cl = mnys[(ci,cj)][1]
        for di,dj in derc:
            ni,nj=ci+di,cj+dj
            if ni<0 or ni>=m or nj<0 or nj>=m or grd[ni][nj] == -1:
                continue
            nmy = cmy
            ncl = mnys[(ni,nj)][1]
            if grd[ci][cj] == 2 and grd[ni][nj] == 2:
                continue
            if cl == 2:
                nmy += 0
            elif grd[ni][nj]!=2:
                nmy += 0 if cl==ncl else 1
            else:
                nmy += 2
                if nmy == mnys[(ni,nj)][0] and cl != ncl and ncl != 2:
                    mnys[(ni,nj)] = [nmy,2]
                    deq.append((ni,nj))
                    continue
                ncl = cl
            if nmy <  mnys[(ni,nj)][0]:
                mnys[(ni,nj)] = [nmy,ncl]
                deq.append((ni,nj))
    ans = mnys[(m-1,m-1)][0]
    return ans if ans<INF else -1
print(main())
使用大规模数据 (300*300 表格，30000 变色点) 是有点问题 (改变权值分别为 1,3,4 和标准结果差 1)，暂时没想到怎么解决测试代码
python
运行
import sys
from collections import deque
import heapq
import time
import random
import io
# ---------- 随机棋盘生成器（保证有合法路径，禁止连续魔法） ----------
def generate_valid_board(m, n, seed=None, max_attempts=100):
    """
    生成一个随机棋盘，保证存在从(1,1)到(m,m)的路径，
    且路径中不允许连续两个魔法格（2）。
    """
    if seed is not None:
        random.seed(seed)
    dirs = [(-1,0),(1,0),(0,-1),(0,1)]
    def has_valid_path(board):
        """检查是否存在合法路径（禁止连续魔法）"""
        if board[1][1] == -1 or board[m][m] == -1:
            return False
        visited = [[False]*(m+1) for _ in range(m+1)]
        q = deque()
        q.append((1,1))
        visited[1][1] = True
        while q:
            x,y = q.popleft()
            if x==m and y==m:
                return True
            for dx,dy in dirs:
                nx,ny = x+dx, y+dy
                if 1<=nx<=m and 1<=ny<=m and not visited[nx][ny] and board[nx][ny] != -1:
                    # 禁止连续魔法
                    if board[x][y] == 2 and board[nx][ny] == 2:
                        continue
                    visited[nx][ny] = True
                    q.append((nx,ny))
        return False
    for attempt in range(max_attempts):
        # 起点终点必须有颜色
        points = [(1,1,0), (m,m,1)]
        # 随机添加其他彩色点
        while len(points) < n:
            x = random.randint(1, m)
            y = random.randint(1, m)
            c = random.randint(0, 1)
            if (x,y) not in [(p[0],p[1]) for p in points]:
                points.append((x,y,c))
        # 构建棋盘
        board = [[-1]*(m+1) for _ in range(m+1)]
        for x,y,c in points:
            board[x][y] = c
        # 在彩色点四邻域生成可涂色格子 (2)
        for x,y,_ in points:
            for dx,dy in dirs:
                nx, ny = x+dx, y+dy
                if 1<=nx<=m and 1<=ny<=m and board[nx][ny]==-1:
                    board[nx][ny] = 2
        if has_valid_path(board):
            return board, points
    return None, None
# ---------- 你的 SPFA 算法（禁止连续魔法） ----------
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
            if not (0<=ci+di<m and 0<=cj+dj<m):
                continue
            if grd[ci+di][cj+dj]==-1:
                grd[ci+di][cj+dj]=2
                mnys[(ci+di,cj+dj)] = [INF,2]
    mnys[(0,0)] = [0,grd[0][0]]
    if (m-1,m-1) not in mnys:
        return -1
    deq = deque([(0,0)])
    while deq:
        ci,cj = deq.popleft()
        cmy = mnys[(ci,cj)][0]
        cl = mnys[(ci,cj)][1]
        for di,dj in derc:
            ni,nj=ci+di,cj+dj
            if ni<0 or ni>=m or nj<0 or nj>=m or grd[ni][nj] == -1:
                continue
            nmy = cmy
            ncl = mnys[(ni,nj)][1]
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
    ans = mnys[(m-1,m-1)][0]
    return ans if ans<INF else -1
# ---------- 优先队列 Dijkstra（同样禁止连续魔法） ----------
def solve_dijkstra_forbidden(data):
    sys.stdin = io.StringIO(data)
    m, n = map(int, input().split())
    board = [[-1]*(m+1) for _ in range(m+1)]
    color_points = []
    for _ in range(n):
        x, y, c = map(int, input().split())
        board[x][y] = c
        color_points.append((x, y))
    dirs = [(-1,0),(1,0),(0,-1),(0,1)]
    for x, y in color_points:
        for dx, dy in dirs:
            nx, ny = x+dx, y+dy
            if 1<=nx<=m and 1<=ny<=m and board[nx][ny]==-1:
                board[nx][ny] = 2
    if board[1][1]==-1 or board[m][m]==-1:
        return -1
    INF = 10**9
    dist = [[[INF, INF] for _ in range(m+1)] for _ in range(m+1)]
    start_color = board[1][1]
    dist[1][1][start_color] = 0
    pq = [(0, 1, 1, start_color)]
    while pq:
        cost, x, y, col = heapq.heappop(pq)
        if cost > dist[x][y][col]:
            continue
        for dx, dy in dirs:
            nx, ny = x+dx, y+dy
            if not (1<=nx<=m and 1<=ny<=m): continue
            nxt = board[nx][ny]
            if nxt == -1: continue
            # 禁止连续魔法
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
    ans = min(dist[m][m][0], dist[m][m][1])
    return ans if ans < INF else -1
# ---------- 主程序 ----------
if __name__ == "__main__":
    m = 300         # 棋盘大小
    n = 30000        # 彩色点个数（含起点终点）
    seed = 20240213 # 随机种子
    max_attempts = 50  # 最大尝试次数
    print(f"正在生成 {m}×{m} 棋盘，彩色点数 {n}，种子 {seed}，最大尝试次数 {max_attempts}")
    board, points = generate_valid_board(m, n, seed=seed, max_attempts=max_attempts)
    if board is None:
        print("生成失败：无法找到有解的棋盘，请尝试增大 n 或减少 m。")
        sys.exit(1)
    # 转换为输入字符串
    lines = [f"{m} {len(points)}"]
    for x,y,c in points:
        lines.append(f"{x} {y} {c}")
    data = "\n".join(lines)
    # 运行 Dijkstra (禁止连续魔法)
    start = time.perf_counter()
    ans_dij = solve_dijkstra_forbidden(data)
    time_dij = time.perf_counter() - start
    
    # 运行 SPFA
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
        print("请检查算法实现差异。")
总体而言，djkistra 优先队列解法更稳定，且时间更快，快 1 到 10 倍。
3、多轮 bfs 的访问标记的巧妙应用 (p1902) 个人思想精华思路：这道题主要使用一个 flhs 字典，用于存储每轮 bfs 在当前 cn 限制下刚好遇到的边界，即搜索时遇到 grd [ni][nj]>cn, 表示这条路径到这里因为 cn 的限制断了，就把这个点存入这个 flhs 字典 (当线性搜索传入的 cn 变大时，就可以从这些点作为起点)，否则标记并加入 deque 队列，继续搜索，直到把所有 (在当前这个 cn 限制下) 能搜索到的点都访问一遍。然后如果新的 cn 来了，就把这个新 cn 在 flhs 字典中存储的点集作为该轮 bfs 起点，重复，知道某个 cn 刚好访问到了出口，则返回 True，对应 cn 就是最小值。
python
运行
import sys
import heapq
from collections import deque
from collections import defaultdict
input = sys.stdin.readline
n,m = map(int,input().strip().split())
grd = [[0]*m]
rmg = 0
rng = 10**9
input()
for i in range(n-2):
    rl = list(map(int,input().strip().split()))
    rmg = max(rmg,max(rl))
    rng = min(rng,min(rl))
    grd.append(rl)
input()
grd.append([0]*m)
dirs = [(1,0),(-1,0),(0,1),(0,-1)]
msk = [[False]*m for _ in range(n)]
flhs = [[] for _ in range(1001)]
deq = deque()
for i in range(m):
    deq.append((0,i))
    msk[0][i] = True
def vrf_rd(cn):
    cps = flhs[cn]
    while cps:
        cx,cy = cps.pop()
        deq.append((cx,cy))
        msk[cx][cy] = True
    while deq:
       ci,cj = deq.pop()
       for di,dj in dirs:
           ni,nj = ci+di,cj+dj
           if ni < 0 or ni>=n or nj<0 or nj>=m:
               continue
           if ni == n-1:
               return True
           if msk[ni][nj]:
               continue
           if grd[ni][nj]<=cn:
                deq.append((ni,nj))
                msk[ni][nj] = True
                continue
           msk[ni][nj] = True
           flhs[grd[ni][nj]].append((ni,nj)) #记录当前最大值，在多条路径下的对应点，作为下一轮开始起点
    return False
def src():
    lf = rng
    hg = rmg+1
    while lf<hg:
        if vrf_rd(lf):
            return lf
        else:
            lf += 1
print(src())