## 643 子数组的最大平均数I
[643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/)
You are given an integer array `nums` consisting of `n` elements, and an integer `k`.
Find a contiguous subarray whose **length is equal to** `k` that has the maximum average value and return _this value_. Any answer with a calculation error less than `10-5` will be accepted.
**Example 1:**
**Input:** nums = [1,12,-5,-6,50,3], k = 4
**Output:** 12.75000
**Explanation:** Maximum average is (12 - 5 - 6 + 50) / 4 = 51 / 4 = 12.75

解法1：定长滑窗。遍历数组，维护一个长度为k的滑窗的和值，运行到下一步就加后面元素减前面元素，求最大即可。

解法2：前缀和。用一个数组存储当前位置前所有元素的和，遍历到`i>=k`的位置后,`prefixSum[i] - prefixSum[i-k]`就是这段长度为k的滑窗的和值，找到最大的即可。


### 1456 定长子串中元音的最大数目
[1456. Maximum Number of Vowels in a Substring of Given Length](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)
Given a string `s` and an integer `k`, return _the maximum number of vowel letters in any substring of_ `s` _with length_ `k`.
**Vowel letters** in English are `'a'`, `'e'`, `'i'`, `'o'`, and `'u'`.

**Example 1:**
**Input:** s = "abciiidef", k = 3
**Output:** 3
**Explanation:** The substring "iii" contains 3 vowel letters.

解法1：长度为k的固定滑窗，遍历数组维护滑窗中元音个数与最大个数，思路跟643一样。

解法2：前缀和。用一个数组存储前i个子串中的元音个数，那么`prefixSum[i] - prefixSum[i-k]`就是这段长度为k的子串的元音字母的个数，思路也跟643一样。

### 1004 最大连续1的个数 III
[1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)
Given a binary array `nums` and an integer `k`, return _the maximum number of consecutive_ `1`_'s in the array if you can flip at most_ `k` `0`'s.

**Example 1:**
**Input:** nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2
**Output:** 6
**Explanation:** [1,1,1,0,0, **1** ,1,1,1,1, **1** ]
Bolded numbers were flipped from 0 to 1. The longest subarray is underlined.

解法：滑窗。右指针每轮往右遍历一位，先判断当前位置是否为0，是则将zeroCount++累计0的数量，然后再检测zeroCount是否大于k，如果是，则需要收缩窗口左边界，收缩的时候再判断是否左指针值为0，如果是则代表要将这个0舍弃，那么zeroCount--。for循环每一轮比较`r - l + 1`和原最大值进行更新。


### [1493. 删掉一个元素以后全为 1 的最长子数组](https://leetcode.cn/problems/longest-subarray-of-1s-after-deleting-one-element/)
[1493. Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/)
给你一个二进制数组 `nums` ，你需要从中删掉一个元素。
请你在删掉元素的结果数组中，返回最长的且只包含 1 的非空子数组的长度。
如果不存在这样的子数组，请返回 0 。

**Example 1:**
**Input:** nums = [1,1,0,1]
**Output:** 3
**Explanation:** After deleting the number in position 2, [1,1,1] contains 3 numbers with value of 1's.

解法1：滑窗。即k为1的1004题，利用类似思路。

解法2：动态规划。p1记录全为1的子数组长度，遇到0时重置为0（表示移除这个0，从这一点再开始计数），p2记录只有一个0的子数组长度，遇到0时重置为p1。每次循环则比较p2和前最大值更新最大值。注意是假定遇到0可以将0抹除重新记录1的个数，如果遇到全为1的数组，必须需要**强行移除**一个，所以当maxLen为数组长时，需要返回maxLen-1。