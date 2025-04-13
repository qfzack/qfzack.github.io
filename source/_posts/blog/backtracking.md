---
title: "Python-回溯算法"
date: 2019-10-11T05:00:00Z
categories: ["Algorithm"]
author: "Qingfeng Zhang"
---

## 一、算法概述

回溯法的思想就是把问题的解空间转化为图或者树的结构表示，然后使用深度优先搜索策略进行遍历，在遍历的过程中记录和寻找所有可行解或者最优解：

* 其思想类同于**图的深度优先搜索**、**二叉树的后序遍历**
* 回溯法的实现：递归和递推
* 经典问题：八皇后问题和迷宫问题

## 二、算法实例

### NO.22 括号生成

* 问题描述  
给出一个数n表示可以使用的括号的对数，返回n对括号可以组成的所有有效括号的组合；

* 思路分析  
最简单的想法就是，对于每一种组合，先从第一个括号开始放，直到放完2n个括号，期间要保证括号序列是合法的，也就是说，每放一个括号的前后都要保证右括号数量不能多于左括号的数量；  
对于当前的合法序列，如果左右括号数量相等，那么下一步就只能放左括号；如果左括号的数量多于右括号，那么下一步就有两种情况：放左括号和放右括号；  
第一步只能放一个左括号，对于n=3的情况如下图所示：

![](images/backtracking/bt1.jpg)

在两种情况下一个节点没有两个分支：左括号数量等于n、左右括号数量相等

* 求解代码

```python
class Solution:  
    def generateParenthesis(self, n: int) -> List[str]:  
        ans = []  #存放每种括号组合的列表  
        def backtrack(s = '', left = 0, right = 0):  
            if len(s) == n*2:  
                ans.append(s)  #当括号序列的长度为2n时，该序列就是一种组合  
                return   #不返回任何值，进代表函数运行结束  
            if left < n:  #表示还有左括号可以用  
                backtrack(s+'(', left+1, right)  #left表示使用了左括号的个数  
            if right < left:  #表示此时可以使用右括号  
                backtrack(s+')', left, right+1)  #right表示使用了右括号的个数  
        backtrack()  
        return ans  
```
  
代码的运行过程如下：

![](images/backtracking/bt2.jpg)

backtrack函数由三个if语句组成，后两个if表示能否在当前的序列中加上左括号或右括号；  
代码的运行过程与第一张图的**树的后序遍历**
过程相似，先执行上图的函数1，函数内调用自身，执行函数2，函数而内调用自身，执行函数3和函数4.但是函数4是等函数3执行完（深度遍历结束进行回溯）再执行，函数3内的函数5和函数6同理，这个过程也可以看做**图的深度优先遍历**。

### NO.39 组合总数

* 问题描述  
给出一个没有重复元素的数组candidates，一个目标数target，找出canditates中任意个元素的组合，该组合的元素和为target，元素可以重复使用

* 思路分析  
如果使用传统的方法一层一层的循环去找非常的耗费时间，因此使用回溯法，假设candidates的元素个数为n，则此问题的解空间就是一个n叉树，对此n叉树进行后序遍历

* 求解代码    
```python
class Solution:  
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:  
        ans = []  
        n = len(candidates)  
        candidates.sort()  #排序是为了大于target之后不用再遍历下去  
        def trackback(i, tmp_sum, tmp):  
            if tmp_sum > target or i == n:  #如果总和大于target或者遍历canditates结束  
                return   #仅仅起到结束函数的作用  
            if tmp_sum == target:  
                ans.append(tmp)  
                return   
            trackback(i, tmp_sum+candidates[i], tmp+[candidates[i]])  #用于深度遍历  
            trackback(i+1, tmp_sum, tmp)  #用于回溯  
        trackback(0,0,[])  
        return ans  
```

代码的运行过程如下：  

![](images/backtracking/bt3.jpg)

当一个分支满足终止条件（总和大于target或者遍历candidates结束）时，将其结束（return）；总和等于target时，将组合存放在ans中；  
每一个节点都可以进行分支（没有条件），函数的参数中，i表示candidates中的下标，tmp_sum表示当前找到的序列的元素和，tmp表示当前找到的序列，最终符合要求的序列由tmp得到，这些参数都是由函数的参数表示

### NO.40 组合总数II

* 问题描述  
与39题非常相似，区别在于candidates中的元素在每个组合中不能重复使用

* 思路分析  
依旧是根据问题的解空间进行求解，对于当前的结点有两种选择，一种选择是保留当前结点的值，另种是不保留，这两种选择各对应一种递归（感觉递归的写法大同小异，需要思考的是每次递归的退出和符合的判断条件），递归函数有两个参数：candidates的坐标和组合的列表，因此，当组合列表值的和等于target时将其加入ans并停止当前函数的递归，或者坐标超出范围或是组合列表的和大于target也停止函数的递归

* 求解代码
```python    
class Solution:  
    def combinationSum2(self, candidates: List[int], target: int) -> List[List[int]]:  
        ans = []  
        n = len(candidates)  
        candidates.sort()  
        def trackback(i,l):  
            if sum(l)==target:  
                if l not in ans:  
                    ans.append(l)  
                return   
            if i>n-1 or sum(l)>target:  
                return   
            trackback(i+1,l+[candidates[i]])  
            trackback(i+1,l)  
        trackback(0,[])  
        return ans  
```

### NO.77 列表的组合

* 问题描述  
给定一个整数n，对应一个从1到n整数列表L，再给定一个组合长度k，从L中选出k个不同的元素进行组合（组合的有序结果不能相同），返回所有的组合结果

* 思路分析  
依然是简单的回溯问题，问题的解空间可以看成一棵树，按照之前的”套路”，定义一个用于递归的函数，其参数一个是下标，一个是需要的组合，这个问题固定了树的深度为k+1，即每个组合的长度为k；  
为了防止每种组合的有序结果不重复，可以对于当前的结点L[i]，其子节点从L[i+1]开始往后找；  
问题的解空间如下图所示：

![](images/backtracking/bt4.jpg)

* 求解代码
```python
class Solution:  
    def combine(self, n: int, k: int) -> List[List[int]]:  
        L = list(range(1,n+1))  
        ans = []  
        def backtrack(i, l):  
            if len(l) == k and l not in ans:  
                ans.append(l)  
                return   
            if i == n:  
                return   
            backtrack(i+1, l+[L[i]])  
            backtrack(i+1, l)  
        backtrack(0, [])  
        return ans  
```

### NO.78 列表的子集

* 问题描述  
给定一组不含重复元素数的列表，返回该列表所有可能的子集

* 思路分析  
依然是使用回溯法进行求解，问题的解空间可以用一颗二叉树表示，对于当前的结点有两个分支，一是加上下一个值作为列表，二是加上一个空列表（即不变）；  
函数的参数只有两个：列表的下标i和所需的子集序列l；  
问题的解空间如下图所示：

![](images/backtracking/bt5.jpg)

* 求解代码
```python
class Solution:  
    def subsets(self, nums: List[int]) -> List[List[int]]:  
        ans = []  
        def backtrack(i, l):  
            if l not in ans:  
                ans.append(l)  
            if i == len(nums):  
                return   
            backtrack(i+1, l+[nums[i]])  
            backtrack(i+1, l)  
        backtrack(0, [])  
        return ans  
```

### N0.90 子集II

* 问题描述
给定一个整数数组nums，其中可能含有重复元素，要得到该列表的所有可能的子集（不能重复）

* 思路分析  
又是一个非常典型的使用回溯法求解的问题，数组的子集中会包含空列表和自身，因此在解空间中，对于树的每一个节点有两个分支，一是往当前的列表l中加入nums的第i个元素，二是不对l添加元素，因此，递归的终止条件就是遍历nums结束  
为了防止子集中因为顺序不同导致重复的情况，如[1,1,2]和[1,2,1]，先对nums进行排序，对于终止的每一个l，如果不在ans中，就将其添加到ans中  
以[1,2,2]为例，解空间的树结构为：

![](images/backtracking/bt6.jpg)

* 求解代码

```python
class Solution:  
    def subsetsWithDup(self, nums: List[int]) -> List[List[int]]:  
        ans = []  
        nums.sort()  
        def trackback(i,l):  
            if i>len(nums)-1:  
                if l not in ans:  
                    ans.append(l)  
                return   
            trackback(i+1,l)  
            trackback(i+1,l+[nums[i]])  
        trackback(0,[])  
        return ans  
```

### NO.131 分割回文串

* 问题描述  
给定一个字符串s，将其分割为多个字符串，要求每个分割后的每个字符串都是回文串，满足要求的一种分割作为一种组合，返回所有可能的组合

* 思路分析
这道题依然是使用回溯法，但是这道题的解空间不想之前的题那样直观，如果用一棵树来表示，那么每一种组合就是树的一层节点的组合；  
递归的结束条件依然是遍历s结束，递归函数的参数为传入的字符串s和当前的回文串的列表L，这两个参数的元素的和就是s；  
函数内先判断是否满足退出条件，如果满足的话，L就是s的回文串分割组合，将其加入ans；  
再对传入的字符串进行遍历，如果前i个元素组成回文串，就把剩余子串和前i个元素加入列表传入函数进行递归；

* 求解代码

```python    
class Solution:  
    def partition(self, s: str) -> List[List[str]]:  
        ans = []  
        def trackback(s, L):  
            if not s:  
                ans.append(L)  
                return   
            for i in range(1,len(s)+1):  
                if s[:i] == s[:i][::-1]:  #判断是否是回文串  
                    trackback(s[i:], L+[s[:i]])  
        trackback(s, [])  
        return ans  
```
  
函数的调用过程如下图所指示：

![](images/backtracking/bt7.jpg)

### NO.216 组合总数III

* 问题描述  
给定一个数n和k，找出为n的k个数的所有组合，其中，每个数都在[1,9]之间

* 问题分析  
任然是一道用回溯法求解的问题，依旧可以用一棵树来表示问题的解空间，但是这棵树的深度由k决定，并且元素的值在[1,9]之间；  
因此，递归的结束条件是当当前元素的值大于9（进入递归加一变为10），或是组合中的元素个数等于k；  
一开始还是搞不懂要怎么递归下去，但是当我把解空间的树画出来，想到对于当前节点，下一个对象要么是其子节点，要么是其右兄弟节点；  
因此，参数为(1,[])的函数内调用自身的参数为(2,[])，表示查看其右兄弟节点，或是(2,[1])，表示查看其第一个子节点，依次递归下去，一步步地就写出来了。

* 求解代码
```python
class Solution:  
    def combinationSum3(self, k: int, n: int) -> List[List[int]]:  
        ans = []  
        def trackback(i, l):  
            if i > 10:  
                return   
            if len(l) == k:  
                if sum(l) == n and l not in ans:  
                    ans.append(l)  
                return  
            trackback(i+1, l+[i])  
            trackback(i+1, l)  
        trackback(1, [])  
        return ans  
```

## 三、另一种解题代码

在Java刷题遇到回溯问题时，看了一下题解，发现了一个新的解题框架，就在这边也记录一下，就拿上面的`NO.39组合总数`为例，这道题也可以用下面的方法求解：

```python    
def combinationSum(candidates, target):  
    ans = []  
    n = len(candidates)  
    candidates.sort()  
    def backtrack(i, tmp_sum, tmp):  
        if(tmp_sum>target or i==n):  
            return  
        if(tmp_sum==target):  
            ans.append(tmp)  
            return   
        for start in range(i,n):  
            if(tmp_sum>target):  
                break  
            backtrack(start,tmp_sum+candidates[start],tmp+[candidates[start]])  
    backtrack(0,0,[])  
    return ans  
ans = combinationSum([2,3,6,7],9)  
```
这也是深度优先遍历，但是这种方法更好，因为上面的方法会遍历到重复的节点，因此耗时多一点，这种方法就不会，只要在backtrack函数里输出当前的tmp变量值就可以看到。
