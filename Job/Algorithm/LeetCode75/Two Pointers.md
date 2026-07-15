### 283 移动零
[283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)
Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.
**Note** that you must do this in-place **without making a copy of the array**.

**Example 1:**
**Input:** nums = [0,1,0,3,12]
**Output:** [1,3,12,0,0]

解法1：双指针交换0与非0元素。快指针找非0元素，慢指针维护字符数组，如果快指针非0，则交换快慢指针位置，如果快指针为0，则继续往后找非0元素，这种相当于把数组中的0一个一个排到最后面。（这里还可以加个条件当读写指针不一样时，才进行交换操作，减少操作次数）

解法2：双指针覆盖法。读指针往后找非0元素，找到了写指针就在当前位置写入读指针元素，最后再将写指针之后的元素全部置为0。这样减少了很多交换操作，效率更高。


### 392 判断子序列
[392. Is Subsequence](https://leetcode.com/problems/is-subsequence/)
Given two strings `s` and `t`, return `true` _if_ `s` _is a **subsequence** of_ `t`_, or_ `false` _otherwise_.
A **subsequence** of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., `"ace"` is a subsequence of `"abcde"` while `"aec"` is not).

**Example 1:**
**Input:** s = "abc", t = "ahbgdc"
**Output:** true

解法1：双指针。指针i遍历s，指针j遍历t，移动指针j在t中找指针i所指元素，找到则双方都往后走一位，如果`i==s.length`，则说明找完了，返回true，否则最后返回false。

解法2：Java API，底层优化好。遍历s中的字符，在t的子串中利用indexOf找到字符在子串中出现的第一个位置，然后遍历到s中的下一个字符，在t的剩下的子串中利用indexOf再继续找。
```java
int index = -1;  
for (char c : s.toCharArray()) {  
    // 注意这里要从上一个找到的字符位置之后开始找下一个字符，如果找不到，直接返回 false    index = t.indexOf(c, index + 1);  
    if (index == -1) {  
        return false;  
    }  
}  
return true;
```


### 11 盛最多水的容器
[11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `ith` line are `(i, 0)` and `(i, height[i])`.
Find two lines that together with the x-axis form a container, such that the container contains the most water.
Return _the maximum amount of water a container can store_.
**Notice** that you may not slant the container.


**Input:** height = [1,8,6,2,5,4,8,3,7]
**Output:** 49
**Explanation:** The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.

解法1：暴力法两层遍历，第一层遍历到一个下标后，第二层从它往后依次遍历到结尾，求最大面积。

解法2：贪心+双指针。从两边往中间遍历，由于宽度在缩减，而面积又受限于矮边，所以想要增加容量，必须挪动矮边，才有可能导致面积增长。遍历一次即可搞定。

贪心策略论证：
$$Area = \min(h[l], h[r]) \times (r - l)$$
当我们处于左右指针 $(l, r)$ 时，假设 $h[l] < h[r]$。如果我们移动**高边** $r$：
- **宽度** $(r - l)$ 肯定减小了。
- **高度** $\min(h[l], h_{new\_r})$ 绝不会超过 $h[l]$（因为高度受限于矮边）。
- **结论**：移动高边，面积**一定**会变小。所以，我们**必须**移动矮边，面积才**有可能变大**。(大概率是外面的面积更大，移动过程中面积变大的条件比较苛刻，但仍有可能)
当h[l] == h[r]时，随便移动哪一个，甚至两个一起移动，结果都是正确的，根本不需要判断h[l+1]和h[r-1]的大小，因为面积受限于矮边，中间的不管有多高，它的面积都是被矮边限制死了。


### 1679 和为K的数对的最大个数
You are given an integer array `nums` and an integer `k`.
In one operation, you can pick two numbers from the array whose sum equals `k` and remove them from the array.
Return _the maximum number of operations you can perform on the array_.

**Example 1:**
**Input:** nums = [1,2,3,4], k = 5
**Output:** 2
**Explanation:** Starting with nums = [1,2,3,4]:
- Remove numbers 1 and 4, then nums = [2,3]
- Remove numbers 2 and 3, then nums = []
There are no more pairs that sum up to 5, hence a total of 2 operations.

解法1：排序+双指针。排好序后可从两边往中间遍历，比较`nums[l] + nums[r]`和`k`的关系，等于则双方同时往中间收缩，小于k则`l++`，大于k则`r--`。

解法2：HashMap。HashMap用于存遍历过程遇到的数以及出现的次数。遍历的时候先检查HashMap是否存了k-nums[i]这个数，如果存了则`cnt++`，并且存的k-nums[i]的次数-1，否则将当前数存到HashMap中。