DFS 回溯
1、返回列表元素全排列 (无重复元素)核心思路通过一个 path 数组多次遍历取数，使用 mk 数组过滤已经使用的，当 path 代表全部元素后，结束递归，依次弹出 path 元素，并取消标记
python
运行
def pmt(nums):
	res = []
	mk = [False]*len(nums)
	def bck_tr(path):
		if len(path) == len(nums):
			res.append(path[:]) #后续还会修改path，即使是不同时刻添加的path，后续还会操作path,会修改值，所以这里必须使用拷贝
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
2、组合 (n 个选 K 个)核心思路，在递归函数中添加一个指示参数，表示当前 i 所到位置，用于剪枝，循环时保证当前选择的元素个数小于剩余元素个数 (使用 start 接受维护)，且不用再标记是否选过，因为它的便历就是向前走的，直到已经选择的长度足够就依次弹出，继续在上一层完成当前循环便历。
python
运行
def comb(nums,k):
	res = []
	def bac_tr(st,path):
		if len(path) == k:
			res.append(path[:])
			return
		lvsl=k-len(path) # 还需选的个数
		lvnm=n-st+1 # 还剩的个数
		# 因为lvsl <= lvnm => st<=n-(k-len(path))+2):
		rt = n-(k-len(path))+2
		lf = st
		for i in range(lf,rt):
			path.append(i)
			bac_tr(i+1,path)
			path.pop()
	bac_tr(1,[])
	return res
3、子集组合 (所有子集情况)
python
运行
def sbet(nums):
	res = []
	def bac_tr(st,path):
		res.append(path[:])
		for i in range(st,len(nums)):
			path.append(i)
			bac_tr(i+1,path)
			path.pop()
	bac_tr(0,[])
	return res
4、全排列有重复元素情况处理和正常全排列一样，但是要先排序，然后在循环中添加元素前要判断前一个元素是否相等，且没添加过，来判断是不是当前这段相等连续元素第一个，是的话就执行后面添加和递归语句，否则跳过当前循环。
python
运行
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
			# 连续相同元素，前一个没用过就直接跳过本轮，保证排列相等元素相对顺序
			if i>0 and nums[i]==nums[i-1] and not mk[i-1]:  
				continue
			mk[i] = True
			path.append(i)
			bac_tr(path)
			path.pop()
			mk[i] = False
	bac_tr([])
	return res
5、 迷宫问题，网格 dfs主要是递归边界判定，还有已经访问过的和地图障碍物判定
python
运行
def mzdf(x,y,gr,vit):
    '网格DFS模版'
    if x<0 or x>=len(gr) or y<0 or y>=len(gr[0])):
        return
    if vit[x][y] and gr[x][y] == 0:
        return
    vit[x][y] = True
    mzdf(x-1,y,gr,mz)
    mzdf(x+1,y,gr,mz)
    mzdf(x,y-1,gr,mz)
    mzdf(x,y+1,gr,mz)
    vit[x][y] = False # 根据题目要求是否需要回溯看是否保留这一步
6、 数字排列 (用 1-9 组成 3 个三位数，比值 1:2:3，列出所有可能)用全排列基本模版，取出所有数字组合，但是递归结束条件根据题目要求设定，去除对于每种 path 的元素，检测组成的三个数是否满足题目要求，即第二个数是第一个数的倍，第三个是第一个三倍，满足要求就加入结果组，不满足则 return, 在添加数字时，根据题意进行剪枝，通过 continue 跳过当轮循环，减少递归调用。
python
运行
def fd_nms():
    res= []
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
7、 生成给定数量对括号的不同排法使用一个 rt,lf 计数变量分别记录左括号，右括号数量，代替括号对数匹配的逻辑判断，关键思路保证右括号指量始终小于左括号抽象不理解
python
运行
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
无法理解，抽象
8、 N 皇后问题，方格两条对角线以及对应平行线只能使用依次，问排法还是使用 res []，递归循环，使用 path 存储并回溯的方法，但是使用三个集合存储当前 path 已经使用的情况，两类对角线分别满足 row+col 和 row-col 值相等，所以对应 row+col，row-col 只能使用一次，然后使用 col 记录已经使用的情况，递归函数同时传递一个 row 控制行数，最后每结束一次递归调用，弹出三个集合存户各自对应该行该位置的标记
python
运行
def solQue(n):
    res = []
    cls = set() # 存储使用了的纵坐标
    fmlw = set() # 存储左下到右上(都是45°线，对于每种情况直线上坐标横纵坐标差值相等,用差值量化不同情况)
    fmlh = set() # 存储左上到左下(都是135°,每种情况对应一个和)
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
9、 回文串分割 (将一个串分割成任意个由子回文串组合成的分割种数)主要还是 dfs 回溯，但是对于循环中子串是否是回文串进行添加的条件判断，可以提前动态规划使用一个二维数组存储 i-j 是否是回文串，这个动态规划的逻辑是，当前串首尾相等，且除去首尾，子串是回文那么当前串也是回文
python
运行
def par(s):
    res = []
    n = len(s)
    dp = [[False]*n for _ in range(n)] # 存储从i-j是否是回文串
    for i in range(n):
        dp[i][i] = True
    for i in range(n-1,-1,-1):
        for j in range(i+1,n):
            if s[i] == s[j]:
                # 对于当前子串s[i:j](根据前面判断，该子串已经满足首尾相等)
                if j == i+1 or dp[i+1][j-1]: # 如果当前长度为2或者内部子串是回文,那么当前串也是回文
                    dp[i][j] = True
    def bac_tr(st,path):
        # 这里使用st==n而不是len(path),因为它要的结果是保证最后path子串总长为n，而不是子串个数
        if st == n: # 当前path串已经组合还原，添加到结果集，回退
            res.append(path[:])
            return
        for ed in range(st,n):
            if dp[st][ed]:
                path.append(s[st:ed+1])
                bac_tr(ed+1,path)  # 注意这里传入的是ed+1，而不是st，因为st-ed已经取了
                path.pop()
    bac_tr(0,[])
    return res
print(par('abb'))
拓展 分割代价最小回文串分割 (动态规划)核心思路：还是先构建回文串判定二维数组，使用数组存储从长度为 0 到 n 切片对应最小划分，外层从 0 到 n 遍历，对应切片长加 1 的情况，如果当前切片直接满足回文串，那当前切片最小分割为 0, 否则对 j 从 0 到 i 依次遍历，如果当前 j+1 到 i 直接满足回文串，那么当前可选的最小分法可能为对应当前存储的 j 最小分法 (因为此 j<i，必然已经存储最小分割次数)+1 (分割从 j+1 到 i), 与对于当前存储的 i 已有的值进行比较，取小的那个，知道遍历完当前循环，所得即为当前 0-i 对应切片最小分割
python
运行
def par(s):
    res = []
    n = len(s)
    dp = [[False]*n for _ in range(n)] # 存储从i-j是否是回文串
    for i in range(n):
        dp[i][i] = True
    for i in range(n-1,-1,-1):
        for j in range(i+1,n):
            if s[i] == s[j]:
                # 对于当前子串s[i:j](根据前面判断，该子串已经满足首尾相等)
                if j == i+1 or dp[i+1][j-1]: # 如果当前长度为2或者内部子串是回文,那么当前串也是回文
                    dp[i][j] = True
    mnct = [float('inf')]*n
    for i in range(n):
        if dp[0][i]:
            mnct[i] = 0
        else:
            for j in range(i):
                if dp[j+1][i]: # 这里是j+1,因为(j<i),从0-j，已经存储最小分割，只需看j+1-i后段是否可分。
                    mnct[i] = min(mnct[i],mnct[j]+1)  # 可分就比较记录最小
    return mnct[n-1]
print(par('abbcb'))
练习：洛谷 P1406，要求输出给定阶数，和 n*n 个数字，构成的字典序最小对应阶幻方。思路：递归回溯模板，但是对于减枝使用一个 sur，suc 维护组成已选数字按序构成部分幻方的每行和以及每列和，当到达行尾 (最后一列) 或者列尾 (最下面一行)，计算对应和是否等于总数和除以阶数，不满足则还原记录和值，跳过当轮递归，当添加到左下元素时完成第一条对角线判定，最后一个元素完成第二条对角线和判定，最后找到第一个满足条件的数字组合就返回 True，快速结束递归，如果当轮没有，弹出后，不仅要取消标记，还要将对应行，列和记录值还原。这里还使用了一个 pre 变量进行去重，如果当前将添加元素的前一个元素是不满足条件值，那么如果当前元素等于那个元素也直接跳过，说明这个位置不可以添加这个元素，或者如果此轮 pre 是弹出元素，那么这轮也跳过，因为说明该位置是对应元素的情况在 pre 代表的上轮完成遍历，对于后面与之相等的元素可以直接跳过，避免重复递归，虽然去重在这里没啥用，因为是返回字典序最小幻方且数据几乎不咋重复，但是这个去重思想方法可以应用到寻找所有满足条件幻方或者不同幻方个数的变种题目中。
python
运行
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
    return res[0]
tms = istms()
print(smk)
lines = [' '.join(map(str,tms[i*n:(i+1)*n])) for i in range(n)]
print('\n'.join(lines))
还是蛮抽象的