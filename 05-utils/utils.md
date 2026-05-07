蓝桥杯临考函数知识点
datetime 模块
下面是一个从初日期到某个末日期计算特定日的简单例子
python
运行
import datetime # 导入模块
std = datetime.date(2000,1,1) # 日期对象定义
ed = datetime.date(2025,1,1)
dy = datetime.timedelta(days=1) # 时间间隔定义
ans = 0
while st<=ed:
    if st.day == 1 and st.isoweekday()==1: # xx.isoweekday()返回星期几,xx.day,该日期对应日
        ans+=1
    st+=dy # 日期增量为前面定义的间隔对象
print(ans)
这里 st.isoweekday () 也可以换成 st.weekday () 不过前者对应星期范围是 1-7, 后者是 0-6
取年，月，日，
python
运行
print(st.month)
print(st.year)
print(st.day)
计算间隔天数
python
运行
std = datetime.date(2000,1,1) # 日期对象定义
ed = datetime.date(2025,1,1)
delta = ed-st # 这里得到一个间隔对象
print(delta.days) # 调用delta.days获取间隔天数 包含最后一天
格式化与解析日期字符串
python
运行
d = ed.strftime('%Y-%m-%d') #将日期转换为字符串"2020-10-01"
d2 = datetime.datetime.strptime("2020-10-01","%Y-%m-%d").date() # 字符串转日期
Fraction 模块 分数处理
导入模块
python
运行
from fractions import Fraction
a = Fraction(1,3)   # 分别传入分子,分母构建分数
b = Fraction('1/3') # 字符串构建分数
不过用分数很容易超时，通过最简化使用元组表示比较好，如下
python
运行
def reduce_frac(num, den):
    """将分数化为最简，分母为正"""
    if den < 0:
        num = -num
        den = -den
    g = math.gcd(abs(num), den)
    return (num // g, den // g)
杨辉三角
每个位置对应数即为 C (n,m) 其中 n,m 分别为行数，列数每行最大值为 C (n,n//2)
分解因数题
使用试除法分解
python
运行
while pow(k,2) <= num: # 分解质因子
    if num%k==0:
        num//=k
        ks.append(k)
    else:
        k+=1
if num>1:
    ks.append(num)
根据质因数两两乘积组合得到所有目标数所有因数
python
运行
s = set() # 去重
s.add(1)
for i in ks:
    mul = set()
    for j in s: # 这里是从s取保证取到每一个因子
        mul.add(i*j) #相当于提前获得两个因子乘积组合
    for k in mul: 
        s.add(k)
枚举有序对 (遍历两次)
python
运行
ans = 0
for s1 in s:
    for s2 in s:
        if n%(s1*s2)==0:
            ans+=1
类似的寻找质约数个数也可以用试除法
python
运行
n = int(input())
ans = 0
k = 2 # 从2开始避免死循环，所以最后可以根据需要加上因数1
while k*k<=n:
	if n%k==0:
		ans+=1
		while n%k==0: # 避免重复添加该质因数
			n//=k
	else:
		k+=1
if n>1:
	ans+=1 # 最后n大于1，说明此时的n也是一个质数
余数：
python
运行
(a+b)%m=(a%m+b%m)%m
(a*b)%m = (a%m*b%m)%m
(a**b)%m = (a**(b%m))%m
中国剩余定理 p12364
易知 n≡n+(k×a)% a k 为任意正整数，即 n + 多个除数的最小公倍数的任意倍数去除以这些除数，各自对应余数等式恒成立 被这些除数整除所得的余数保持不变，即满足题目的整除余数结果，根据这个条件可以遍历 (1,50), 从最小的除数开始，依次对当前 n 加上当前除数集最小公倍数，如果当前 n 得到下一条余数记录，就根据新加入的除数更新这个已遍历除数集最小公倍数。
python
运行
cd = dvs[0]*dvs[1]
for i in range(2,48):
    while st%dvs[i]!=rms[i]:
        st+=cd
    cd = dvs[i]*cd//math.gcd(dvs[i],cd)
差分数组
使用标记统计区间覆盖次数
python
运行
ns = list(map(int,input().split()))
n = len(ns)
dif = [0]*(n+2) #注意这里是n+2,因为原下标从1,n，又需要用到r+1，故这里标记开(n+2)个
for i in range(q):
	l,r = map(int,input().split())
	dif[l]+=1
	dif[r+1]-=1
for i in range(1,n+1):
	dif[i] += dif[i-1]
dif.sort(reverse = True) #排序后dif即为每个下标覆盖次数排序最大到最小
异或和
任何数异或本身结果 0：a^a = 0利用这个规律，如果两个数组划分各自异或和结果相等，则说明这个数组所有数异或和为 0
任何位对应数与 0 异或等于本身，与 1 异或取反，使用异或对特定位取反:a = 1010001, 令 b=0011000, 则 a^b 刚好为对应 3，4 位取反结果
不使用临时变量交换 a,b 值：
python
运行
a= a^b  # 基底，利用与本身异或结果为0
b = a^b
a = a^b
对于 1-n 的所有连续整数 1，2，3...n 异或和 s 满足
python
运行
if n%4==0: s=n
elif n%4==1: s=1
elif n%4 == 2:s=n+1
else: s= 0
collections 模块
Counter
判断两个字符串 是否是异位串c ['a'] 获取 a 出现次数，不存在返回 0c.most_common (2) 返回频率最高前两项c.elements () 按计数展开为迭代器
python
运行
ct1 = Counter('adsa')
ct2 = Counter('aasa')
# 如果ct1==ct2 说明是打乱顺序的异位串
按词频从高到低排序，同频按字典序
python
运行
words = ["apple", "banana", "apple", "orange", "banana", "apple"]
s = Counter(words)
rs = sorted(s.items(),key=lambda x:(-x[1],x[0]))  # x[1]是频率，x[0]是统计字符
defaultdict
返回拥有默认值的字典
python
运行
dis = defaultdict(int) # 默认0
defaultdict(lambda:-1) # 自定义默认-1
deque
python
运行
dq = deque(maxlen=3) # 这里maxlen可以用与控制最大长度，使用append添加元素，如果满了，则剔除左端元素，用appendleft反之  
deque 实现滑动窗口 (每个窗口最大值):
python
运行
ns = [2,4,3,1,5,8,4,3]
k = 3
def sol():
    res  =[]
    dq = deque()
    for i,v in enumerate(ns):
        if dq and dq[0]<=i-k: # 队首过期,比如当前下标3,则前面访问3个元素(0,1,2),当前窗口最小下标是0,如果这个0还在队列中就需要剔除
            dq.popleft()
        while dq and ns[dq[-1]]<=v: # 从后往前删除，保证队列元素单调递减,即新添加的放到最后，且最小
            dq.pop()
        dq.append(i)
        if i>=k-1:
            res.append(ns[dq[0]])
    return res
相对应的栈顶动态维护返回每个元素右侧最近的比它大的值
python
运行
def sol():
	stk = []
	res = [-1]*len(ns)
	for i,v in enumerate(ns):
		while stk and ns[stk[-1]]<v:
			ix = stk.pop()
			rs[ix] = v
		stk.append(i)
	return rs
上述两个处理，本质是通过限定条件和加入的新元素，类似于动态化学方程，不关注过程，只关注结果和条件限制
维护两个最大值，使用两个变量即可维护 a,b
python
运行
if n>a:
	b =a
	a = n
elif n>b:
	b=n
二进制
<<左移，>> 右移a.bit_length () 获得 a 的二进制长度
