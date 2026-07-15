必须先排序再处理
```java
// 按左边界升序排序（绝大多数情况，如哪些情况？）
Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

// 按右边界升序排序（贪心题专用，如435移除最少的区间数让剩下的无重叠，452最少数量的箭引爆气球）
Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));

// 不建议使用，因为如果是个很大的a[0]减去负数，会导致整数溢出
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
```
区间合并模板：
```java
public int[][] merge(int[][] intervals) {
    if (intervals.length <= 1) return intervals;
    // 1. 按左边界升序排序
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    List<int[]> res = new ArrayList<>();
    // 2. 初始化安全锚点：prev 代表当前正在维护（拉长）的那个区间
    int[] prev = intervals[0];
    res.add(prev); // Java 引用传递特性：放入 list 后，修改 prev 依然会同步修改 list 里的数据
    
    // 3. 开始遍历
    for (int i = 1; i < intervals.length; i++) {
        int[] curr = intervals[i];
        // 判断重叠：当前区间的起点 <= 锚点区间的终点
        if (curr[0] <= prev[1]) {
            // 【发生重叠】：合并，更新锚点区间的右边界为两者最大值
            prev[1] = Math.max(prev[1], curr[1]);
        } else {
            // 【无重叠】：断开连接，锚点区间已达最大形态
            // 将 curr 设置为新的锚点，并加入结果集
            prev = curr;
            res.add(prev);
        }
    }
    return res.toArray(new int[res.size()][]);
}
```
#### 🗺️ 区间问题分类

| **门派分类**                   | **核心目标**                                       | **排序策略**      | **核心维护变量**                              | **经典题目**                    |
| -------------------------- | ---------------------------------------------- | ------------- | --------------------------------------- | --------------------------- |
| **合并/覆盖类 (Merge)**         | 求连续覆盖的最大范围、合并重叠部分。                             | **按左边界升序**    | 维护当前合并区间的 `maxEnd`。                     | LC 56 (合并区间)、LC 57 (插入区间)   |
| **贪心/选择类 (Greedy)**        | 求最多能挑出几个互不重叠的区间。不关心区间长度，只关心区间结束值。结束越早，给后面空间越大。 | **按右边界升序**    | 维护上一个保留区间的 `lastEnd`。                   | LC 435 (无重叠区间)、LC 452 (射气球) |
| **交集/双指针类 (Intersection)** | 找两组区间的重叠部分。                                    | 通常已按**左边界**排好 | 双指针 `i, j`，取 `max(start)` 和 `min(end)`。 | LC 986 (区间列表的交集)            |

---

### [435. 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)
[435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)
给定一个区间的集合 `intervals` ，其中 `intervals[i] = [starti, endi]` 。返回 _需要移除区间的最小数量，使剩余区间互不重叠_ 。
**注意** 只在一点上接触的区间是 **不重叠的**。例如 `[1, 2]` 和 `[2, 3]` 是不重叠的。
**示例 1:**
```
输入: intervals = [[1,2],[2,3],[3,4],[1,3]]
输出: 1
解释: 移除 [1,3] 后，剩下的区间没有重叠。
```

题解：贪心。找出最多的不重叠区间n，那么最少移除数就等于区间总数-n。区间按照右边界排序，维护一个最小的右区间，以0号区间为基准，从1号区间开始遍历，若非重叠则立马计数，并更新最小的右边界为当前区间的右边界，若重叠则忽略，继续找下一个非重叠区间。这套逻辑保证按照最先结束的先找最后非重叠，所以没问题。

---

### [452. 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)
[452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)
有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。
一支弓箭可以沿着 x 轴从不同点 **完全垂直** 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `xstart`，`xend`， 且满足  `xstart ≤ x ≤ xend`，则该气球会被 **引爆** 。可以射出的弓箭的数量 **没有限制** 。 弓箭一旦被射出之后，可以无限地前进。
给你一个数组 `points` ，_返回引爆所有气球所必须射出的 **最小** 弓箭数_ 。

**示例 1：**
```
输入：points = [[10,16],[2,8],[1,6],[7,12]]
输出：2
解释：气球可以用2支箭来爆破:
-在x = 6处射出箭，击破气球[2,8]和[1,6]。
-在x = 11处发射箭，击破气球[10,16]和[7,12]。
```

题解：这个问题就是找最多的重叠区间。区间先按照右边界排序，以0号元素为基准，维护一个最小右边界和重叠区间数cnt（初始化1），如果重叠则不管，越过边界，则`cnt++`，并更新最小右边界，表示新的重叠区间，最后返回cnt