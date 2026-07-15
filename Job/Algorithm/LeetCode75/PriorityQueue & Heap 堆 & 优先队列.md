堆专题考察的则是“动态维护极值”的能力。当面对海量数据，不需要全盘排序，而只需要前 K 个最大、最小、或者频率最高的元素时，堆就是无可替代的最优解。

堆的哲学是：动态打擂台。
	小顶堆（Min-Heap）：**堆顶**永远是当前整棵树里最小的元素。 `PriorityQueue<Integer> minHeap = new PriorityQueue<>();` 创建默认小顶堆。	
	大顶堆（Max-Heap）：堆顶永远是当前整棵树里最大的元素。`PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());` 反序将队列强行改为大顶堆

在 Java 中，Priority Queue（优先队列）底层就是用二叉堆实现的。它的核心操作极其高效：
	插入一个元素：$O(\log N)$
	偷看一眼堆顶：$O(1)$
	把堆顶扔掉（弹出）：$O(\log N)$

在做“Top K”类题目时，初学者最容易把大顶堆和小顶堆用反。记住下面这个反直觉的无敌口诀：

> **求前 K 个最大的元素（或第 K 大） → 用小顶堆（剔除小的，留下大的）** 
> **求前 K 个最小的元素（或第 K 小） → 用大顶堆（剔除大的，留下小的）**


---

### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
[215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
Given an integer array `nums` and an integer `k`, return _the_ `kth` _largest element in the array_.
Note that it is the `kth` largest element in the sorted order, not the `kth` distinct element.
Can you solve it without sorting?
**Example 1:**
**Input:** nums = [3,2,1,5,6,4], k = 2
**Output:** 5

解法1：小顶堆，维护一个size=k的小顶堆，将所有值都入堆，i>=k时弹出堆顶，最后peek堆顶元素就是第k大的元素。

解法2：大顶堆，将所有元素压入大顶堆后，弹出k-1个元素，此时堆顶元素就是第k大的元素。


---

### [2336. 无限集中的最小数字](https://leetcode.cn/problems/smallest-number-in-infinite-set/)
[2336. Smallest Number in Infinite Set](https://leetcode.com/problems/smallest-number-in-infinite-set/)

现有一个包含所有正整数的集合 `[1, 2, 3, 4, 5, ...]` 。
实现 `SmallestInfiniteSet` 类：
- `SmallestInfiniteSet()` 初始化 **SmallestInfiniteSet** 对象以包含 **所有** 正整数。
- `int popSmallest()` **移除** 并返回该无限集中的最小整数。
- `void addBack(int num)` 如果正整数 `num` **不** 存在于无限集中，则将一个 `num` **添加** 到该无限集中。
**示例：**
```
输入
["SmallestInfiniteSet", "addBack", "popSmallest", "popSmallest", "popSmallest", "addBack", "popSmallest", "popSmallest", "popSmallest"]
[[], [2], [], [], [], [1], [], [], []]
输出
[null, null, 1, 2, 3, null, 1, 4, 5]
```
**解释**
SmallestInfiniteSet smallestInfiniteSet = new SmallestInfiniteSet();
smallestInfiniteSet.addBack(2);    // 2 已经在集合中，所以不做任何变更。
smallestInfiniteSet.popSmallest(); // 返回 1 ，因为 1 是最小的整数，并将其从集合中移除。
smallestInfiniteSet.popSmallest(); // 返回 2 ，并将其从集合中移除。
smallestInfiniteSet.popSmallest(); // 返回 3 ，并将其从集合中移除。
smallestInfiniteSet.addBack(1);    // 将 1 添加到该集合中。
smallestInfiniteSet.popSmallest(); // 返回 1 ，因为 1 在上一步中被添加到集合中， 且 1 是最小的整数，并将其从集合中移除。
smallestInfiniteSet.popSmallest(); // 返回 4 ，并将其从集合中移除。
smallestInfiniteSet.popSmallest(); // 返回 5 ，并将其从集合中移除。

题解：成员变量k用于累加表示正整数，minHeap用来记录比k小的后来add进来的数。popSmallest方法中判断如果minHeap非空，则返回弹出的堆顶，否则则返回k++。addBack方法中如果num<k，并且minHeap中不含num，则num入堆。（也可引入一个set用于去重，空间换时间）

---

### [2542. 最大子序列的分数](https://leetcode.cn/problems/maximum-subsequence-score/)
[2542. Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/)
You are given two **0-indexed** integer arrays `nums1` and `nums2` of equal length `n` and a positive integer `k`. You must choose a **subsequence** of indices from `nums1` of length `k`.
For chosen indices `i0`, `i1`, ..., `ik - 1`, your **score** is defined as:
- The sum of the selected elements from `nums1` multiplied with the **minimum** of the selected elements from `nums2`.
- It can defined simply as: `(nums1[i0] + nums1[i1] +...+ nums1[ik - 1]) * min(nums2[i0] , nums2[i1], ... ,nums2[ik - 1])`.
Return _the **maximum** possible score._
A **subsequence** of indices of an array is a set that can be derived from the set `{0, 1, ..., n-1}` by deleting some or no elements.

**Example 1:**
**Input:** nums1 = [1,3,3,2], nums2 = [2,1,3,4], k = 3
**Output:** 12
**Explanation:** 
The four possible subsequence scores are:
- We choose the indices 0, 1, and 2 with score = (1+3+3) * min(2,1,3) = 7.
- We choose the indices 0, 1, and 3 with score = (1+3+2) * min(2,1,4) = 6. 
- We choose the indices 0, 2, and 3 with score = (1+3+2) * min(2,3,4) = 12. 
- We choose the indices 1, 2, and 3 with score = (3+3+2) * min(1,3,4) = 8.
Therefore, we return the max score, which is 12.

解法：贪心+小顶堆。nums1和nums2绑定成一个二维数组，根据nums2的值排序。这样处理之后遍历起来能保证第i个nums2的元素是最大值，因此只用关心nums1中的子序列和。然后维护一个大小为k的小顶堆，遍历所有数，当i>=k-1时每一轮都会计算最大值，并且进行堆的更新与淘汰。

---

### [2462. 雇佣 K 位工人的总代价](https://leetcode.cn/problems/total-cost-to-hire-k-workers/)
[2462. Total Cost to Hire K Workers](https://leetcode.com/problems/total-cost-to-hire-k-workers/)
给你一个下标从 **0** 开始的整数数组 `costs` ，其中 `costs[i]` 是雇佣第 `i` 位工人的代价。

同时给你两个整数 `k` 和 `candidates` 。我们想根据以下规则恰好雇佣 `k` 位工人：

- 总共进行 `k` 轮雇佣，且每一轮恰好雇佣一位工人。
- 在每一轮雇佣中，从最前面 `candidates` 和最后面 `candidates` 人中选出代价最小的一位工人，如果有多位代价相同且最小的工人，选择下标更小的一位工人。
    - 比方说，`costs = [3,2,7,7,1,2]` 且 `candidates = 2` ，第一轮雇佣中，我们选择第 `4` 位工人，因为他的代价最小 `[_3,2_,7,7,_**1**,2_]` 。
    - 第二轮雇佣，我们选择第 `1` 位工人，因为他们的代价与第 `4` 位工人一样都是最小代价，而且下标更小，`[_3,**2**_,7,_7,2_]` 。注意每一轮雇佣后，剩余工人的下标可能会发生变化。
- 如果剩余员工数目不足 `candidates` 人，那么下一轮雇佣他们中代价最小的一人，如果有多位代价相同且最小的工人，选择下标更小的一位工人。
- 一位工人只能被选择一次。

返回雇佣恰好 `k` 位工人的总代价。

**Example 1:**
**Input:** costs = [17,12,10,2,7,2,11,20,8], k = 3, candidates = 4
**Output:** 11
**Explanation:** We hire 3 workers in total. The total cost is initially 0.
- In the first hiring round we choose the worker from [17,12,10,2,7,2,11,20,8]. The lowest cost is 2, and we break the tie by the smallest index, which is 3. The total cost = 0 + 2 = 2.
- In the second hiring round we choose the worker from [17,12,10,7,2,11,20,8]. The lowest cost is 2 (index 4). The total cost = 2 + 2 = 4.
- In the third hiring round we choose the worker from [17,12,10,7,11,20,8]. The lowest cost is 7 (index 3). The total cost = 4 + 7 = 11. Notice that the worker with index 3 was common in the first and last four workers.
The total hiring cost is 11.

题解：双指针+小顶堆。维护左右指针以及左右两个小顶堆，小顶堆大小为candidates。每一轮peek左右堆顶元素，取最小的一个弹出（相同取左堆），完成一个员工招聘。注意边界条件，left<=right（小于会少元素），以及维护堆元素的条件为size<candidates（小于等于会多存一个数，因为条件里面还会入堆一次）。
