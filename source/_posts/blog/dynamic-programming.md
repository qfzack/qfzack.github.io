---
title: "Python-动态规划算法"

date: 2019-10-11T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Algorithm"]
author: "Qingfeng Zhang"
---


## 一、方法概述

动态规划是把复杂问题分解为简单的问题进行求解，适用于有重叠子问题和最优子结构性质（问题的最优解包含其子问题的最优解）的问题；  
先描述最优解的结构，递归定义最优解的值，再按照自底向上的方式计算最优解的值，最后由计算的结果构造最优解；

  * 例子：  
看一个问题：给定一个数组，可以任意取不相邻的元素，求取出的元素和的最大值；  
对于每一个数都有两种选择：选或不选这个数；  
假设是数组从左往右选，并且选元素i的最大和为opt()，则opt(i)=max(opt(i-2)+i,opt(i-1));  
按照这个规则，从左往右计算数组的opt值，则最后一个元素的opt就是最终的结果;  
dp的第i个位置表示在数组L中从L[0]找到L[i]的时候可以得到的最大和  
求解代码：

```python
L = [1,2,4,1,7,8,3]  
  
def rec_opt(L, i):  
    if i == 0:  
        return L[0]  
    elif i == 1:  
        return max(L[0],L[1])  
    else:  
        A = rec_opt(L, i-2)+L[i]  
        B = rec_opt(L, i-1)  
        return max(A,B)  
  
def dp_opt(L):  
    dp = [0 for i in L]  
    dp[0] = L[0]  
    dp[1] = max(L[0],L[1])  
    for i in range(2,len(L)):  
        A = dp[i-2]+L[i]  
        B = dp[i-1]  
        dp[i] = max(A,B)  
    return dp[-1]  
```
  

## 二、方法实例

### NO.62 不同路径

  * 问题描述  
给定一个数组，从数组的左上角出发走到数组的右下角，求一共有多少中不同的路径；

  * 思路分析  
到数组中一个位置[i,j]的路径数应该等于到位置[i-1,j]的路径数加上到[i,j-1]的路径数，因此这就是一个动态规划的问题；  
先用1初始化一个相同大小的dp数组，因为dp的到第一行和第一列的每个位置都只有一种路径，因此这些位置的dp值都为1，从dp[i,j]开始按行的顺序计算每个位置的值`dp[i][j]
= dp[i][j-1]+dp[i-1][j]`；  
最终的结果就是dp[-1][-1]位置的值

  * 求解代码
    
```python
class Solution:  
    def uniquePaths(self, m: int, n: int) -> int:  
        dp = [[1]*m for i in range(n)]  
        for i in range(1,n):  
            for j in range(1,m):  
                dp[i][j] = dp[i][j-1]+dp[i-1][j]  
        return dp[-1][-1]  
```
  

### NO.64 最小路径和

  * 问题描述  
对于一个[m,n]大小的网格，每个位置的元素值表示到达该位置的路径长度，要求找出从[0,0]位置到[-1,-1]（右下角元素）的最短路径

  * 思路分析  
这是使用动态规划的一道基础题  
以0初始化一个数组dp，dp是一个和grid大小相同的数组，dp中每个位置[i,j]的元素表示从grid[0,0]到grid[i,j]的最短路径  
对于dp的第一行和第一列元素，只有唯一的选择，因此这些位置的值是确定的，因此先对dp第一行和第一列的值进行计算  
对于dp其它位置的值，需要根据该位置上面和右边的dp元素值取最小，来计算得到当前位置的dp值  
当遍历完grid的所有元素，到达每个位置的最短路径也就确定了，因此到达最后一个位置的最短路径是dp[-1,-1]

  * 求解代码

```python
class Solution:  
    def minPathSum(self, grid: List[List[int]]) -> int:  
        row = len(grid)  
        col = len(grid[0])  
        dp = [[0]*col for i in range(row)]  
        dp[0][0] = grid[0][0]  
        for i in range(1,row):  
            dp[i][0] = dp[i-1][0]+grid[i][0]  #得到dp的第一列，表示到[i][0]的路径和  
        for j in range(1,col):  
            dp[0][j] = dp[0][j-1]+grid[0][j]  #得到dp的第一行，表示到[0][i]的路径和  
        #因为到grid第一行和第一列的每个位置都只有唯一选择，因此路径和固定  
        for i in range(1,row):  
            for j in range(1,col):  
                dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j]  
        return dp[-1][-1]  
```

### NO.198 打家劫舍

  * 问题描述  
给定一个数组，每个元素表示可以在这里打劫获得的金钱，但是不能打劫相邻的两家，求最后可以得到的最多金钱

  * 思路分析  
动态规划的一道简单题，但是思路很好，简单来说就是在一个数组中，找一些不相邻的数使得这些数的和最大；  
使用dp数组，dp[i]表示从0到i可以选的数和的最大值，先初始化dp[0]=nums[0],dp[1]=max(nums[0],nums[1])；  
从第三个元素开始遍历nums，也就是对于当前的这个数，有两种选择：

1.选这个数，那么选前前一个数和这个数，将它们的和作为当前的dp值（表示打劫这一家和前前一家）；  
2.不选这个数，那么选前一个数，将前一个数做为当前的dp值（表示不打劫这一家，因此得到的钱不变）；  
选取上面两种情况中最大的作为当前的dp值，遍历nums结束即得到结果

  * 求解代码
    
```python
class Solution:  
    def rob(self, nums):  
        if len(nums)==0:  
            return 0  
        if len(nums)<=2:  
            return max(nums)  
        dp = [0]*len(nums)  
        dp[0],dp[1] = nums[0],max(nums[0],nums[1])  
        for i in range(2,len(nums)):  
            dp[i] = max(dp[i-1],dp[i-2]+nums[i])  
        return dp[-1]  
```

### NO.279 完全平方数

  * 问题描述  
给定一个数字n，要求将数字n分解成几个平方数的和，并要求平方数的个数最少；

  * 思路分析  
应用动态规划的思想，可以将原问题进行分解得到重叠子问题，以n=13为例，设opt(13)表示和为13的平方数的最小个数，则：  
opt(13)=min(opt(13),opt(12)+1,opt(9)+1,opt(4)+1)，即13减掉1*1,2*2,3*3，再与opt(13)取最小（我的理解是需要求最小值，但是需要一个初始值opt(13)进行比较）；  
对于opt(12),opt(9),opt(4)就可以看成opt(13)的重叠子问题进行求解；  
因此，遍历数组[1:n]，求解对应dp数组的opt值，即可得到结果，过程如下：  
![](images/dynamic-programming/dp1.jpg)

  * 求解代码

```python
class Solution:  
    def numSquares(self, n: int) -> int:  
        dp = [n for i in range(n+1)]  
        dp[0] = 0  
        for i in range(1,n+1):  
            j = 1  
            while i-j*j >= 0:  
                dp[i] = min(dp[i],dp[i-j*j]+1)  
                j += 1  
        return dp[-1]  
```

### NO.393 最佳买股票时机含冷冻期

  * 问题描述  
给定一个数组，每个数表示一天股票的价格，每天可以买入或者卖出股票，但是前一天卖出不能在下一天买入，求最后可以获得的最大利润；

  * 思路分析  
又是一个股票的问题，与基本股票问题的区别在于包含冷冻期，即卖掉之后不能立即买入；  
每一天的状态有两种：持有股票和不持有股票；  
如果某天持有股票，可能是前一天就持有股票，也可能是前一天不持有，当天买入股票（并且前一天不能卖出，如果前一天有卖出，那么就看前前一天不持有股票的利润，如果前一天没卖出，那么其实利润和前前一天是一样的，因此，直接看前前一天不持有股票的利润就行）；  
因此，设置两个与prices等长的数组hold和nothold，分别记录每一天持有和不持有股票时的总利润；  
如果某天不持有股票，可能是前一天就不持有股票，也可能是前一天持有，当天卖掉股票；  
遍历一遍prices，最后的结果就是最后一天不持有股票的利润；

  * 求解代码
    
```python
class Solution:  
    def maxProfit(self, prices: List[int]) -> int:  
        if len(prices)==0:  
            return 0  
        hold = [0]*len(prices)  
        nothold = hold.copy()  
        hold[0] = -prices[0]  
        for i in range(1,len(prices)):  
            hold[i] = max(hold[i-1],nothold[i-2]-prices[i])  
            nothold[i] = max(nothold[i-1],hold[i-1]+prices[i])  
        return nothold[-1]  
```

### NO.712 两个字符串的最小ASCII删除和

  * 问题描述  
给定两个字符串，删去两个字符串的一些元素可以使得这两个字符串相等，求删去的这些字符的ASCII值的和；

  * 思路分析  
一开始以为只要找出两个字符串之间相同的字符，再将两个字符串相同元素之外的元素进行ASCII求和就可以得到结果，但是最后才发现删除之后两个字符串的元素排列也要相同（不只是元素相同）；  
正确的解法还是动态规划；以s1 = ‘seerda’，s2 = ‘eafert’为例：  
先用0初始化一个数组，数组的[i,j]表示要使s1[i-1:]和s2[j-1:]相等所要删去的字符的ASCII值的和；  
可以先求得dp最后一行的值，此时s2[len(s2)-1:]为空；  
再求dp最后一列的值，此使s1[len(s1)-1:]为空；  
对于dp其他位置的值，同时遍历s1和s2，如果当前遍历的两个值相同，则dp[i][j]与dp[i+1][j+1]相等，否则取dp[i+1][j]+ord(s1[i])和dp[i][j+1]+ord(s2[j])的最小值；  
当i,j循环到[5,1]的时候，dp[5][1]表示s1[5:] (‘a)和s2[1:]
(‘afert’)达到相等时需要删去的字符的ASCII值的和，结果与使’’和’fert’相等的

  * 求解代码

```python
class Solution:  
    def minimumDeleteSum(self, s1: str, s2: str) -> int:  
        dp = [[0]*(len(s2)+1) for i in range(len(s1)+1)]  
          
        for i in range(len(s1)-1,-1,-1):  
            dp[i][len(s2)] = dp[i+1][len(s2)]+ord(s1[i])  
        for j in range(len(s2)-1,-1,-1):  
            dp[len(s1)][j] = dp[len(s1)][j+1]+ord(s2[j])  
              
        for i in range(len(s1)-1,-1,-1):  
            for j in range(len(s2)-1,-1,-1):  
                if s1[i]==s2[j]:  
                    dp[i][j] = dp[i+1][j+1]  
                else:  
                    dp[i][j] = min(dp[i+1][j]+ord(s1[i]),dp[i][j+1]+ord(s2[j]))  
  
        return dp[0][0]  
```

### NO.714 买股票的最佳时机含手续费

  * 问题描述  
给定一个数组，表示股票价格的变化，可以在某一时刻买入股票，另一时刻卖出（会有手续费），求可以获得的最大利润；

  * 思路分析  
关于股票的问题Leetcode上有一个非常好的总结：<https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/solution/yi-ge-fang-fa-tuan-mie-6-dao-gu-piao-wen-ti-by-l-2>  
之前只知道动态规划是可以把问题分解为子问题，然后求解子问题得到最终的解，这道题中，可以遍历prices列表；  
对于每个值都可以看成一个子问题，即求在当前值可获得的最大利润，但是每个值可以是买入（前提是已卖出），可以是卖出（前提是已买入），也可以是无操作；  
对于每个子问题，可以通过两个变量来记录其结果：  
一个变量是记录在当前状态，如果持有股票的情况下其最大利润，这种情况可能是上一个状态就持有股票，也可能是上一个状态不持有，当前状态买入的，求最大值：max(dp[i-1][0],
dp[i-1][1]+prices[i]-fee)  
另一个变量是记录在当前状态，如果不持有股票的情况下其最大利润，这种情况可能是上一个状态就不持有股票，也可能是上一个状态持有股票，当前状态卖出的，求最大值：max(dp[i-1][1],
dp[i-1][0]-prices[i])  
由此可以对每个子问题求解，遍历prices直到最后一个值，其不持有股票的变量值就是最终的结果；

  * 求解代码

```python
class Solution:  
    def maxProfit(self, prices: List[int], fee: int) -> int:  
        dp = [[0,0] for i in range(len(prices))]  
        dp[0][0] = 0  #第一步就不持有，总利润为0  
        dp[0][1] = -prices[0]  #第一步就买入，总费用减去第一个股票价格  
        for i in range(1,len(prices)):  
            dp[i][0] = max(dp[i-1][0], dp[i-1][1]+prices[i]-fee)  #当前不持有可能是上一步不持有，或是上一步持有，在当前一步卖掉  
            dp[i][1] = max(dp[i-1][1], dp[i-1][0]-prices[i])  #当前持有可能是上一步持有，或是上一步不持有，在当前一步买入  
        return dp[-1][0]  
```

### NO.718 最长重复子数组

  * 问题描述  
给定两个数组，求最长的公共子数组；

  * 思路分析  
最简单的方法就是暴力循环，但是这样做的时间和空间复杂度较高；  
比较合适的方法是使用动态规划，很久没做动态规划了，最重要的是找出状态转移方程，然后用状态转移方程去更新dp数组，最后的答案就在dp数组里；  
这里的状态转移方程需要从后往前遍历，即A[i:]和B[j:]的最长公共数组是由A[i+1:]和B[j+1:]决定的，分两种情况：

1.如果A[i]!=B[j]，那么A[i:],B[j:]的最长公共数组和A[i+1:],B[j+1:]的一样；  
2.如果A[i]==B[j]，那么A[i:],B[j:]的最长公共数组要在A[i+1:],B[j+1:]的基础上加1；  
对于dp数组，将其初始化为0，并且dp[:][-1]和dp[-1][:]的值就是为0，不用更新；  
问题的求解过程如下图所示：  
![](images/dynamic-programming/dp2.jpg)  
一般动态规划的问题需要找到当前状态最优值与前一状态最优值之间的关系，由此得到状态转移方程，并且不同问题的dp数组形式由有些不同，一般为二维数组，也有多个一维数组的，比如股票问题；

  * 求解代码
    
```python
class Solution:  
    def findLength(self, A: List[int], B: List[int]) -> int:  
        dp = [[0]*(len(A)+1) for i in range(len(B)+1)]  
        for i in range(len(A)-1,-1,-1):  
            for j in range(len(B)-1,-1,-1):  
                if A[i]==B[j]:  
                    dp[i][j] = dp[i+1][j+1]+1  
        return max([max(l) for l in dp])  
```

### NO.740 删除获得的点数

  * 问题描述  
给定一个数组，其中会有重复的元素，可以选定其中一个数i，得到点数i，删除这个值，然后删除所有的i+1和i-1，直到所有的数都删完，可以得到的最大点数是多少

  * 思路分析  
用动态规划进行求解，与NO.198 打家劫舍非常相似，但是打家劫舍是位置相邻，这里是数值相邻；  
先对nums去重排序得到l，用C记录其中每个元素出现的次数，如果选中一个3，删除2和4，那么剩下的所有3都会被选；  
建立dp数组，每个值记录当前可以获得的最大点数；  
对于l，每次需要判断是否与前一个数字相差1，是的话就和打家劫舍的思路一致，否则直接加上当前的数值乘以其个数；

  * 求解代码
    
```python
class Solution:  
    def deleteAndEarn(self, nums: List[int]) -> int:  
        l = [0,0] + sorted(list(set(nums)))  
        C = [nums.count(i) for i in l]  
        if len(l)==0:  
            return 0  
        dp = [0]*len(l)  
        for i in range(2,len(l)):  
            if l[i]-1==l[i-1]:  
                dp[i] = max(dp[i-1],dp[i-2]+l[i]*C[i])  
            else:  
                dp[i] = dp[i-1]+l[i]*C[i]  
        return dp[-1]  
```

### NO.931 下降路径的最小和

  * 问题描述  
给定一个二维数组，可以从第一行任意一个位置开始走到最后一行，每次下降一行，但是不能一次跨过两列，求走到最后一行时经过所有位置数和的最小值；

  * 思路分析  
先初始化一个dp数组，这里可以直接复制A，因为计算dp的元素值的时候也会用到A的元素值，dp的第一行等于A的第一行，因此，从dp的第二行开始计算：  
dp每一个位置的值等于其自身加上其上方，左上方（如果存在），右上方（如果存在）的最小值，依次计算dp每个位置的值，最后返回dp最后一行的最小值就是结果；

  * 求解代码
    
```python
class Solution:  
    def minFallingPathSum(self, A: List[List[int]]) -> int:  
        dp = A.copy()  
        for i in range(1,len(A)):  
            for j in range(len(A[i])):  
                if j > 0 and j+1 < len(A[i]):  
                    dp[i][j] += min(A[i-1][j-1],A[i-1][j],A[i-1][j+1])  
                elif j > 0:  
                    dp[i][j] += min(A[i-1][j-1],A[i-1][j])  
                elif j+1 < len(A[i]):  
                    dp[i][j] += min(A[i-1][j],A[i-1][j+1])  
        return min(A[-1])
```

### NO.983 最低票价

  * 问题描述  
给定一个升序排列的数组，其中每个数字表示1-365天中的一天，现在又三种票：日票，月票，周票，要买票使得在数组中的日期中都持有票，求最少的花费

  * 问题分析  
典型的动态规划问题，先定义一个dp数组，dp[i]表示**从days[1]开始到days[i]所要花费的票**
,i=0,1,2,……,len(days)，因此dp[0]=0;  
每一天都可以买日票，周票，月票，但是有时候买周票或月票不划算，有时候没必要买日票，因此取三种情况的最小值；  
因为周票可以用7天，月票可以用30天，因此在遍历的时候看是不是已经过了7天或30天；  
如果已经过了7天，那么用7天前买的票就可以了，如果过了30天，那么用30天前买的票就可以了，这里用week记录最接近7天的日期，用month记录最接近30天的日期；  
动态规划的计算公式为：  
`dp[i+1] = min(dp[i]+costs[0],dp[week]+costs[1],dp[month]+costs[2])`  
因此可以将原问题分解为子问题，转换为对子问题的求解

  * 求解代码
    
```python
class Solution:  
    def mincostTickets(self, days: List[int], costs: List[int]) -> int:  
        dp = [0]*(len(days)+1)  
        week, month = 0, 0  
        for i in range(len(days)):  
            while days[week]+6 < days[i]:  
                week += 1  
            while days[month]+29 < days[i]:  
                month += 1  
            dp[i+1] = min(dp[i]+costs[0],dp[week]+costs[1],dp[month]+costs[2])  
        return dp[-1]  
```
