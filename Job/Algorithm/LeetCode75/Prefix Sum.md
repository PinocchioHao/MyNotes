### [1732. 找到最高海拔](https://leetcode.cn/problems/find-the-highest-altitude/)
[1732. Find the Highest Altitude](https://leetcode.com/problems/find-the-highest-altitude/)

有一个自行车手打算进行一场公路骑行，这条路线总共由 `n + 1` 个不同海拔的点组成。自行车手从海拔为 `0` 的点 `0` 开始骑行。

给你一个长度为 `n` 的整数数组 `gain` ，其中 `gain[i]` 是点 `i` 和点 `i + 1` 的 **净海拔高度差**（`0 <= i < n`）。请你返回 **最高点的海拔** 。

**Example 1:**
**Input:** gain = [-5,1,5,0,-7]
**Output:** 1
**Explanation:** The altitudes are [0,-5,-4,1,1,-6]. The highest is 1.

解法： 直接累加算前缀和。累加的前缀和即为当前位置海拔，更新最大海拔即可。


### [724. 寻找数组的中心下标](https://leetcode.cn/problems/find-pivot-index/)
给你一个整数数组 `nums` ，请计算数组的 **中心下标** 。
数组 **中心下标** 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。
如果中心下标位于数组最左端，那么左侧数之和视为 `0` ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。
如果数组有多个中心下标，应该返回 **最靠近左边** 的那一个。如果数组不存在中心下标，返回 `-1` 。
**Example 1:**
**Input:** nums = [1,7,3,6,5,6]
**Output:** 3
**Explanation:**
The pivot index is 3.
Left sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11
Right sum = nums[4] + nums[5] = 5 + 6 = 11

解法：两个数组分别记录前缀和后缀和，先反向再正向遍历两次数组，在维护前缀和的时候也检查前缀和后缀和是否相等，相等则返回下标，遍历完没找到就返回-1。