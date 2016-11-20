---
title: LeetCode 377. Combination Sum IV
date: 2016-10-07 15:49:41
tags: ['LeetCode','动态规划']
---

# 题目:

> Given an integer array with all positive numbers and no duplicates, find the number of possible combinations that add up to a positive integer target.

`Example:`
```python
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.

Therefore the output is 7.
```
<!-- more -->
`Follow up:`
- What if negative numbers are allowed in the given array?
- How does it change the problem?
- What limitation we need to add to the question to allow negative numbers?

## 思路:Dynamic Programming

其实对于动态规划的题目还是很好解决的,动态规划的题目基本上很多思路都是想通的,对于动态规划的题目,解决问题是一个循序渐进的过程,首先需要求出特殊位置的值(开始位置/结束位置/第一行或第一列/最后一行或最后一列),然后再挨个推导,直到求出最后的结果。

首先创建一个数组,数组的大小为目标值加1,数组中每个位置的值dp[i]的含义为用题目中所给的数组中的数字组成i的组合数,然后我们从1开始,一直计算到target,dp[target]即为所求。

接下来我们应该考虑怎么来计算dp[i]的值,应该怎么计算呢?

对于数组中的每个值num,如果num与i相等,说明num自己就可以组成i;如果num大于i,则不可能组成i;如果num小于i,说明它需要和别的数字组合才能组成i,那么和谁组合呢?当然是i-num,这时候就要看i-num有多少个组合个数,此时dp[i]的值就需要加上dp[i-num]的值。对于每个位置i,都需要遍历一遍数组中的元素,当i等于target的时候,dp[target]即为所求。
```
1.dp[0]=1

2.dp[i]+=dp[i-num]
```
```python
class Solution(object):
    def combinationSum4(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        dp = [0] * (target+1)
        dp[0] = 1
        nums.sort()

        for i in xrange(1, target+1):
            for j in xrange(len(nums)):
                if nums[j] <= i:
                    dp[i] += dp[i - nums[j]]
                else:
                    break

        return dp[target]

if __name__ == "__main__":
    nums, target = [1,3,5],5
    result = Solution().combinationSum4(nums, target)
    print result
```

