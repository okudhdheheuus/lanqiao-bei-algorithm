二分

1. 查找特定元素，初始设置左右指针分别为0,n-1，使用循环依次取中间mid元素，与目标值大小关系，看目标元素在哪半边，在右边则lf=mid+1,左边则rt=mid-1，直到lf>rt,如果没有mid=tar的情况说明没找到。
def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] == tar:
            return mid
        elif ans[mid] < tar:
            lf = mid+1 # 在右边
        else:
            rt = mid-1
    return -1
def main():
    ans = [5,9,13,14,15,20]
    tar = 13
    print(bdS_exa(ans,tar))
main()
2. 查找第一个大于等于元素(左边界),方法和找特定元素一样，不用特判相等元素，对于在左半部分，条件拓展为mid元素大于等于目标值,不用内置返回条件，结束循环返回lf,当所有元素都小于目标值，返回最大下标+1

def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] < tar:
            lf =mid+1   # 目标值在右半，最后为lf或者lf+1(也是前面某个mid值)
        else:           # 第一个满足目标值在左半部分
            rt =mid-1
    return lf 
def main():
    ans = [5,13,13,14,15,20]
    tar = 12
    print(bdS_exa(ans,tar))
main()
3. 查找最后一个小于等于元素(右边界),方法和查找左边界相反,但是相等判断归类到右半部分,最后相应返回rt(可能为-1，即所有元素大于目标值)

def bdS_exa(ans,tar):
    rt = len(ans)-1
    lf = 0
    while lf <= rt:
        mid = lf + (rt-lf)//2
        if ans[mid] <= tar: # 右边界在右半部分
            lf = mid+1     
        else:           # 目标值在左半,因为先判断在减一，最后为rt,或者可能为当前rt+1(也是前面某次mid)
            rt = mid-1
    return rt
def main():
    ans = [5,13,13,14,15,20]
    tar = 13
    print(bdS_exa(ans,tar))
main()



总结：
结束条件统一为left = right+1
看一开始在哪边吧，一开始右边满足条件就返回lf,反之返回rt，比如找左边界，一开始右边边界就满足条件，所以最后返回lf
找左边界，返回lf，对应相等情况算到左半部分
找右边界，返回rt，对应相等情况算到右半部分
找谁返回谁
找特定值，返回mid或者-1

拓展
4、旋转排序数组的搜索
旋转数组：一个列表由两个有序序列组成，存在两个断点，即第一个序列右侧最大，与第二序列左侧最小的突然跳跃
# 利用自相似思想，只关注划分后有序部分，如果当前ans[lf]<=ans[mid]说明左半部分是有序的
# 根据这一点，可以对左半部分进行缩小范围处理，否则，右半边有序，就对右半部分进行范围缩小处理
# 对于两种情况，如果目标值不在有序范围段，即另一种子情况，则将其对应指针调整为mid+1(在上段mid右半块和下段mid左半块)
# 从而不断通过有序部分的调整，最终确定目标位置
# 抽象题，拓展了我对递归子问题思想的1理解
def bd_rot(ans,tar):    
    lf = 0
    rt = len(ans)-1
    while lf<=rt:
        mid = (lf+rt)//2
        if ans[mid] == tar:
            return mid
        if ans[lf]<=ans[mid]:
            if ans[lf] <= tar < ans[mid]:
                rt =mid - 1
            else:
                lf =mid+1
        else:
            if ans[mid]<tar<=ans[rt]:
                lf=mid+1
            else:
                rt=mid-1
    return -1
ans = [4,5,6,1,2,3]
tar = 2
print(bd_rot(ans,2))

个人理解：
也就是说实际是划成3部分处理，一部分是mid到对应上段左边界,一部分是mid到对应下段右边界，然后中间是一块包含断点的部分，这部分对于上段，用lf+1维护，对于下段用rt-1维护，OK，抽象

5、 找峰值
峰值，即找开口向下抛物线对称轴，根据中值下标处单调性判断
def peak_ele(ans):
    lf = 0
    rt = len(ans)-1
    while lf < rt:  # 这里不要用等于，最多就等于峰值处，用等于就让lf跑过去了
        mid = lf+(rt-lf)//2
        if ans[mid]>ans[mid+1]:
            rt = mid
        else:
            lf = mid + 1 # +1防止最后死循环
    return lf
ans = [1,3,8,10,5,3,0,-10]
print(peak_ele(ans))


6. 降序查找
def dec_bdS(ans,tar):       # 递减数组
    lf = 0
    rt = len(ans)-1
    while lf<=rt:
        mid = lf+(rt-lf)//2
        if ans[mid] == tar:
            return mid
        if ans[mid]>tar: # 因为递减，所以如果当前mid对应值大于目标值，在右半部分找，所以lf=mid+1 —mid—tg->
            lf = mid+1
        else:
            rt = mid-1
    return -1
ans = [57,51,43,37,21,1]
print(dec_bdS(ans,51))

7、 二分答案搜索(最大值的最小，最小值的最大)
类似于峰值查找，不过右边界缩小判断条件根据题意进行自己更改，封装为一个返回boll类型的函数
二分模板如下：
def mnx_sp(ns,k):
    lf = max(ns)
    rt = sum(ns)
    while lf<rt:  # 跟峰值一样，这里是小于
        mid = lf+(rt-lf)//2
        if check(mid,k,ns):
            rt = mid  # rt始终满足条件，最后只需要调整lf即可
        else:
            lf = mid + 1
    return lf
假设给定数组，求划分数不大于k的每个划分和最小值，这里就可以根据划分和的两个最值，数组最小值(对应划分n次),数组和(对应划分1次)，根据定义的判断当前某个值checK函数是否满足，使用二分法不断缩小范围，查找最小满足的情况。
def check(cmx_su,k,ns):
    seg = 0
    cur_su = 0
    for cn in ns:
		if cur_su+cn>cmx_su:
            seg += 1
            cur_su = cn
            if seg>k:
                return False
        else:
            cur_su += cn
    return True



8、 二分法求开方
根据被开方数与1的大小关系，初步设置大小搜索范围。然后逐步用得到mid乘方对应次数，直到乘方数所的数与num相减精或者l=r

def sqrtbinary(num, p):
    if num < 0:
        return None
    elif num==0:
        l=0
        r=0
    elif num > 1:
        l = 1
        r = num
    else:
        l = 0
        r = 1  # 小于1，初始右边界设置为1
    while l < r:  # 这里是小于，防止死循环
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

9、 使用二分法找零点(方程求根问题	)
方程求根，根据区间左端点与右端点异号，假定方程两个根距离大于等于1，则以整数间隔依次查找，即找0-1,1-2,2-3....范围内零点，依次类推。循环结束条件为两个边界值差值小于某个精度，或者如果左边界，右边界值同号则舍去。
a,b,c,d = list(map(float,input().split()))
res = set()  # 如果零点落在划分整数边界上可能导致重复添加(前后两个划分分别计入一次)，所以需要使用集合去重
def cr_seg(i):
    lf = i
    rt = i+1
    frt = a*pow(rt,3)+b*pow(rt,2)+c*rt+d
    flf = a*pow(lf,3)+b*pow(lf,2)+c*lf+d
    while rt-lf>1e-7:  # 最后两个边界差值小于这个范围
        mid = (lf+rt)/2
        fm = a*pow(mid,3)+b*pow(mid,2)+c*mid+d
        if flf*frt>0:
            return
        if flf*fm>0:
            lf = mid
            flf = fm
        else:
            rt = mid
            frt = fm
    res.add(round(mid,2))  # 这里返回的是mid，如果零点落在划分整数边界上可能导致重复添加(前后两个划分分别计入一次)

for i in range(-100,100):
    cr_seg(i)
res = list(res)
res.sort()
print(' '.join(f'{i:.2f}' for i in res if i<101))

测试值 ：1 -5 -4 20   -2.00 2.00 5.00
1 -4.65 2.25 1.4   -0.35 1.00 4.00



