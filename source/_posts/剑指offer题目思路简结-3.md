---
title: 剑指offer题目思路简结-3
tags: [leetcode]
categories: []
comments: true
date: 2020-06-25 09:36:29
updated: 2020-06-25 09:36:29
description: 剑指offer 51-75 题，思路简结
---
<!-- TOC -->

    1. [剑指 Offer 50. 第一个只出现一次的字符](#剑指-offer-50-第一个只出现一次的字符)
    2. [剑指 Offer 51. 数组中的逆序对](#剑指-offer-51-数组中的逆序对)
    3. [剑指 Offer 52. 两个链表的第一个公共节点](#剑指-offer-52-两个链表的第一个公共节点)
    4. [剑指 Offer 53 - I. 在排序数组中查找数字 I](#剑指-offer-53---i-在排序数组中查找数字-i)
    5. [指 Offer 53 - II. 0～n-1中缺失的数字](#指-offer-53---ii-0n-1中缺失的数字)
    6. [剑指 Offer 54. 二叉搜索树的第k大节点](#剑指-offer-54-二叉搜索树的第k大节点)
    7. [剑指 Offer 55 - I. 二叉树的深度](#剑指-offer-55---i-二叉树的深度)
    8. [剑指 Offer 55 - II. 平衡二叉树](#剑指-offer-55---ii-平衡二叉树)
    9. [剑指 Offer 56 - I. 数组中数字出现的次数](#剑指-offer-56---i-数组中数字出现的次数)
    10. [剑指 Offer 56 - II. 数组中数字出现的次数 II](#剑指-offer-56---ii-数组中数字出现的次数-ii)
    11. [剑指 Offer 57 - II. 和为s的连续正数序列](#剑指-offer-57---ii-和为s的连续正数序列)
    12. [剑指 Offer 58 - I. 翻转单词顺序](#剑指-offer-58---i-翻转单词顺序)
    13. [剑指 Offer 58 - II. 左旋转字符串](#剑指-offer-58---ii-左旋转字符串)
    14. [剑指 Offer 59 - I. 滑动窗口的最大值](#剑指-offer-59---i-滑动窗口的最大值)
    15. [剑指 Offer 59 - II. 队列的最大值](#剑指-offer-59---ii-队列的最大值)
    16. [剑指 Offer 60. n个骰子的点数](#剑指-offer-60-n个骰子的点数)
    17. [剑指 Offer 61. 扑克牌中的顺子](#剑指-offer-61-扑克牌中的顺子)
    18. [剑指 Offer 62. 圆圈中最后剩下的数字](#剑指-offer-62-圆圈中最后剩下的数字)
    19. [剑指 Offer 63. 股票的最大利润](#剑指-offer-63-股票的最大利润)
    20. [剑指 Offer 64. 求1+2+…+n](#剑指-offer-64-求12n)
    21. [剑指 Offer 65. 不用加减乘除做加法](#剑指-offer-65-不用加减乘除做加法)
    22. [剑指 Offer 66. 构建乘积数组](#剑指-offer-66-构建乘积数组)
    23. [剑指 Offer 67. 把字符串转换成整数](#剑指-offer-67-把字符串转换成整数)
    24. [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](#剑指-offer-68---i-二叉搜索树的最近公共祖先)
    25. [剑指 Offer 68 - II. 二叉树的最近公共祖先](#剑指-offer-68---ii-二叉树的最近公共祖先)
1. [完结散花 ~~~](#完结散花-)

<!-- /TOC -->

## 剑指 Offer 50. 第一个只出现一次的字符
遍历字符串，搞个map记录字符出现次数。再次遍历字符串，遇到出现次数为 1 的就返回。

## 剑指 Offer 51. 数组中的逆序对
这题值得hard难度。   
如果暴力解法，时间复杂度将是O(N^2)。比排序的O(NlogN)还大，那么可否先排序在比较，降低复杂度？   
比如 [7, 5, 6, 4]   

1. 先均分为两部分 [7, 5] 和 [6, 4]，分别排序得到，[5, 7] 和 [4, 6]。
2. 对于5，发现只比4大，说明只有一个[5, 4]，对于7，继续与6比较，而不用继续跟4比较，说明有[7, 4]、[7, 6] 两个。
3. 继续分别针对[7，5] 和 [6, 4] 重复 `均分->排序->比较` 这个过程。分别只有一个结果[7, 5]和[6, 4]。
4. 所以最终结果为 5 

```python
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        # 比较两个排好序的子数组
        def computePairs(arr1: List[int], arr2: List[int]) -> int:
            if len(arr1) == 0 or len(arr2) == 0:
                return 0
            l = 0
            r = 0
            res = 0
            while l<len(arr1) :
                while r < len(arr2) and arr1[l] > arr2[r]:
                    r += 1
                res += r
                l+=1
            return res

        if len(nums) <= 1:
            return 0

        # 划分 & 排序
        nums2 = nums.copy()
        left_arr = nums[:len(nums2)//2]
        right_arr = nums[len(nums2)//2:]
        left_arr.sort()
        right_arr.sort()

        # 计算本身结果，并递归子数组
        return computePairs(left_arr, right_arr) \
               + self.reversePairs(nums[:len(nums)//2]) \
               + self.reversePairs(nums[len(nums)//2:])
```

## 剑指 Offer 52. 两个链表的第一个公共节点
最简单的，先计算长度，然后比较两者的长度差，再利用快慢指针。   
一个巧妙的办法：

```python
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        c1 = headA
        c2 = headB

        # 1. 最终c1和c2走的长度是相等的，LA + LB
        # 2. 如果不存在相交，c1走到B的结尾时会被赋值为None，此时c2也恰好走到A的结尾被赋值为None，刚好两者相等，跳出循环
        while c1 != c2:
            c1 = c1.next if c1 else headB
            c2 = c2.next if c2 else headA

        return c1
```

## 剑指 Offer 53 - I. 在排序数组中查找数字 I
先二分查找位置，再左右扩展。复杂度最差 O(N)，平均 O(logN) + O(M)。M为结果个数。如果M==N，则平均复杂度退化为 O(logN) + O(N)    
再优化一下，可以先查左边界，再查右边界。就是 O(logN) 的复杂度，避免了最差O(N)的复杂度。   

都比暴力解法强吧 [doge][doge] 

## 指 Offer 53 - II. 0～n-1中缺失的数字
规律是：在缺失数左侧，每个数与其索引是相等的；在缺失数右侧，每个数 > 其索引。因此可利用二分查找缺失数的位置。

## 剑指 Offer 54. 二叉搜索树的第k大节点
按 `右子树 -> 根 -> 左子树` 的顺序遍历，使用全局变量记录还需遍历多少个节点。
```python
class Solution:
    def kthLargest(self, root: TreeNode, k: int) -> int:
        # 返回
        # 1. 是否找到
        # 2. 对应的值

        def dfs(r: TreeNode) -> (bool, int):

            if r is None:
                return False, 0
            # 遍历右子树
            found, val = dfs(r.right)
            if found:
                return found, val

            # 遍历根
            if self.nk == 1:
                return True, r.val
            self.nk -= 1

            # 遍历左子树
            return dfs(r.left)

        self.nk = k
        _, v = dfs(root)
        return v
```

## 剑指 Offer 55 - I. 二叉树的深度
DFS，`Depth(root) = 1 + max(Depth(root.left), Depth(root.right))`

## 剑指 Offer 55 - II. 平衡二叉树
1. 解法一：计算并检查每一个节点左右子树的深度，但是这样做会有很多的重复计算。
2. 解法二：自底向上，同时记录当前子树的深度，从而在计算父节点的深度时，避免重复计算。
```python
# -1 表示非平衡，直接返回
# >=0 表示树的深度
def valid(root: TreeNode) -> (int):
    if root is None:
        return 0
    left = valid(root.left)
    if left == -1:
        return -1
    right = valid(root.right)
    if right == -1:
        return -1

    if abs(left - right) > 1:
        return -1

    return max(left, right) + 1


class Solution:
    def isBalanced(self, root: TreeNode) -> bool:
        return valid(root) != -1
```

## 剑指 Offer 56 - I. 数组中数字出现的次数
1. 如果一个数组中，只有一个数字出现了一次，其他数字都出现了两次，那么可以通过对所有数字进行xor操作，最后得到的就是该数
2. 如果有两个数字a, b出现了一次，可以想办法将这两个数字划分到两个子数组，这两个子数组除下a, b外，其他都出现了两次，则可直接根据上述规律，xor遍历一遍得到a, b
```python
class Solution:
    def singleNumbers(self, nums: List[int]) -> List[int]:
        xor_res = 0
        for v in nums:
            xor_res = xor_res ^ v

        # 找xor_res为1的那一位
        # 也就是res1 和 res2 不同的那一位
        div = 1
        while (xor_res & div) == 0:
            div = div << 1

        xor1 = 0
        xor2 = 0
        for v in nums:
            if v & div == 0:
                xor1 = xor1 ^ v
            else:
                xor2 = xor2 ^ v

        return [xor1, xor2]
```

## 剑指 Offer 56 - II. 数组中数字出现的次数 II
1. 解法一：遍历统计每个bit 1 出现的次数，最后对 3 取模即可。
2. 解法二：还有个位运算的解，感觉没必要这么取巧。

## 剑指 Offer 57 - II. 和为s的连续正数序列
滑动窗口，`[i, j]`表示连续子数组的两端，临时sum <> s，j右移扩大窗口，否则 i 左移缩小窗口。

## 剑指 Offer 58 - I. 翻转单词顺序
split一下，然后倒序数组，最后拼接。

## 剑指 Offer 58 - II. 左旋转字符串
切片操作，不做赘述。

## 剑指 Offer 59 - I. 滑动窗口的最大值
这题标记为 easy 过分了。   
暴力解法就不说了，时间复杂度是 `O(k*N)`。   
可以优化一下，使用单调减的辅助队列（类似单调栈），来减少求每个窗口最大值时的遍历情况。复杂度为O(N)，因为辅助队列里的每个数，平均跟新进入队列的数，比较1次。（因为要么直接加入队列，要么，删掉队尾比它小元素，加入队列），被删掉的元素只会被比较 1 次。   
考虑极端情况，每个数都会加入辅助队列（原数组是递减的），则每次仍只需比较队尾元素和新加入元素 1 次。因此比较了 2N 次   
PS：单调栈和单调队列，是一个非常有帮助的思路。

## 剑指 Offer 59 - II. 队列的最大值
类似 [剑指 Offer 30. 包含min函数的栈](#剑指-offer-30-包含min函数的栈)，同样使用单调减的辅助队列。这题也可以称为”包含max函数的队列“。

## 剑指 Offer 60. n个骰子的点数
这个题目描述得实在不好理解。
> 你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

不过，写到这里，我发现用文字去描述一个算法思想，确实不太容易。   
至于这题，举个栗子，比如两个骰子，那么依次输出，抛出骰子的和为 2、3、4、5、6 ...的概率。

首先可知，有 n 个骰子，那么结果范围为 [n, 6*n]，那么利用动态规划思想：
1. 划分子问题：使用n个骰子抛出 x 的概率，等于使用一个骰子抛出 a 的概率 乘以 n-1 个骰子，抛出 x-a 的概率。一个骰子抛出各个值的概率自然是 1/6
2. 状态转移公式：dp(n, x) = 1/6 * dp(n, x-a), 其中 a in [1, 6]
3. 边界条件：dp(1, a) = 1/6，其中 a in [1, 6]

## 剑指 Offer 61. 扑克牌中的顺子
除下0之外，数组不能重复。同时计算数组的最小值、最大值。（除 0 以外）   
判断 `max_val - min_val` 是否 <= 4。 

## 剑指 Offer 62. 圆圈中最后剩下的数字
约瑟夫环，这特么竟然标记为**简单！！！** 说是`hard`真不算过分。   
因为输出是最后剩下的数字，也正好是其下标。   
约瑟夫环的关键在于递推公式:    
> `F(N, M) = (F(N-1, M) + M) % N`   

其中，`F(N,M)`表示 N 个人时，某未被删除的数字的下标。   
假设 `F(N,M) = y`，在经过一次删除操作后，y的下标变为了 `(y-M)%(N-1)`，即 F(N-1, M)，设为 x。
（因为 N 个数时，删除 M，会从第 M+1 处重新从 0 计算下标。被删除的 M 处于新数组的队尾，因此不会因为空洞之类的原因，影响新的下标计算。）   
根据   
> `(y-M)%(N-1)=x`   

可推得：
>  `y = (x + M) % N`   

其实也好理解，相当于删除的逆过程，x左移M个位置，然后对N取余。

已知，F(1, M) = 0，因为1个人的时候，剩下数字的下标就是 0，那么可以递推出 F(1, M)，F(2, M) 直到 F(N, M)。

```python   
class Solution:
    def lastRemaining(self, n: int, m: int) -> int:
        if n==1:
            return 0
        
        return (m + self.lastRemaining(n-1, m)) % n
```

## 剑指 Offer 63. 股票的最大利润
这个显然算 easy，却标记为 middle。不再啰嗦。

## 剑指 Offer 64. 求1+2+…+n
这个妙在，利用 and / or 操作的执行顺序。已知：
> A and B，如果 A 为false，则 B 不会再执行   
  A or B，如果 A 为true，则 B 不会再执行
```python
class Solution:
    def sumNums(self, n: int) -> int:
        self.res = 0

        def sum(n: int):
            _ = n != 1 and self.sumNums(n - 1)
            # _ = n == 1 or self.sumNums(n - 1)
            self.res += n
            return True

        sum(n)
        return self.res
```

## 剑指 Offer 65. 不用加减乘除做加法
这个特么又标为easy就离谱，里面的细节一点不少。   
两个数的相加，在二进制上可以表现为两部分：   
1. 直接相加，忽略进位：s1 = a ^ b，相同为0，不同为1 
2. 进位部分：s2 = (a & b) << 1，同为1的位，要向左进一位
3. 两部分相加：sum = s1 + s2

## 剑指 Offer 66. 构建乘积数组
使用两个辅助数组，分别计算从左到右，以及从右至左的乘积。   
再优化一下，可以只使用一个辅助数组。

## 剑指 Offer 67. 把字符串转换成整数
难点在于检查中间结果是否溢出的条件，两种情况：
1. `res > INT_MAX // 10`，此时 res * 10，肯定溢出了   
2. `res == INT_MAX // 10 and cur_num > 7`，因为INT_MAX = 2147483647，如果 res 是正数，显然越界；如果 res 是负数，INT_MIN = -2147483648，所以满足这个条件的，只有 INT_MIN 本身。

## 剑指 Offer 68 - I. 二叉搜索树的最近公共祖先
先分析下问题：
1. 如果 p == root 或 q == root，那么 root 本身就是最近公共祖先
1. 如果 p < root < q，那root一定是最近公共祖先
2. 如果不满足 1，那么p和q 一定同时在 root 的左子树或右子树   

上述root可能是某棵子树的根节点。

## 剑指 Offer 68 - II. 二叉树的最近公共祖先
先分析下问题：   
1. 这是一棵普通二叉树，非搜索二叉树，各节点是无序的
2. 最近公共祖先的定义不变，对于某节点来说，p/q 分别位于其左右子树；或其为p或q，并且q或p在其子树上；该节点就是最近公共祖先。

```python
class Solution:
    def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
        if root is None:
            return None

        # 你可能有疑问，不用确定另一个在不再它的子树里吗？无需确定
        # 假如root=p，若q在其子树里，root即为最近公共祖先；
        # 若q不在其子树里，那一定在别的子树里，此时会有left_res和right_res都不为None的时候，也就会返回对应的root
        if root.val == p.val or root.val == q.val:
            return root

        left_res = self.lowestCommonAncestor(root.left, p, q)
        right_res = self.lowestCommonAncestor(root.right, p, q)
        # p, q分别位于root的左右子树，root本身就是解
        if left_res is not None and right_res is not None:
            return root

        # 到这里，left_res 和 right_res 一个为None，一个不为None
        
        # 情况1：某子树root==p或q，因此返回了自己，此时最近公共祖先，就是该子树的root。
        # 就是left_res或right_res中不为空的那个。

        # 情况2：某棵子树上发现了最近公共祖先，将其传递至最上层
        return left_res if left_res else right_res
```

# 完结散花 ~~~
![abc](/images/jian_zhi_offer.jpg)












