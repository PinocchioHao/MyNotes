一般用于找下一个更大或更小的数。
模板：
遍历数组，拿当前数跟stack顶比较（while条件），如果符合某种大小关系，则一股脑pop出数据，并且做相关操作，否则则正常将当前遍历到的数据入栈。
以下为找第一个大于栈顶元素的位置：
```java
public int[] monotonicStackTemplate(int[] nums) {
    int n = nums.length;
    int[] res = new int[n];
    
    // 通常栈内存放的是数组的【下标】（索引），因为下标既能查到值，又能算距离
    Deque<Integer> stack = new ArrayDeque<>();
    
    for (int i = 0; i < n; i++) {
        // 【核心结算逻辑】：当栈不为空，且当前元素打破了单调性。以下为找第一个大于栈顶元素的值。
        while (!stack.isEmpty() && nums[stack.peekLast()] < nums[i]) {
            // 弹出被打破单调性的栈顶元素
            int prevIdx = stack.removeLast();
            
            // 结算历史记录的答案
            // 例如：计算距离就是 i - prevIdx；记录具体数值就是 nums[i]
            res[prevIdx] = i - prevIdx; 
        }
        
        // 当前元素进栈，成为新的“等待者”
        stack.addLast(i);
    }
    return res;
}
```

---

### [739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)
[739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)
给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。
**示例 1:**
**输入:** temperatures = [73,74,75,71,69,72,76,73]
**输出:** [1,1,4,2,1,1,0,0]

题解：单调栈。套用单调栈模板即可，栈里记录元素下标，遍历元素时比较当前值与栈顶元素，将栈中小于当前元素的全部弹出，弹出的同时更新rlt结果集为当前下标-弹出元素下标，即表示第一个大于该天的温度出现的天数。栈操作完毕的while循环外push当前节点入栈。

---

### [901. 股票价格跨度](https://leetcode.cn/problems/online-stock-span/)
[901. Online Stock Span](https://leetcode.com/problems/online-stock-span/)

设计一个算法收集某些股票的每日报价，并返回该股票当日价格的 **跨度** 。
当日股票价格的 **跨度** 被定义为股票价格小于或等于今天价格的最大连续日数（从今天开始往回数，包括今天）。
- 例如，如果未来 7 天股票的价格是 `[100,80,60,70,60,75,85]`，那么股票跨度将是 `[1,1,1,2,1,4,6]` 。
实现 `StockSpanner` 类：
- `StockSpanner()` 初始化类对象。
- `int next(int price)` 给出今天的股价 `price` ，返回该股票当日价格的 **跨度** 。
**示例：**
```
输入：
["StockSpanner", "next", "next", "next", "next", "next", "next", "next"]
[[], [100], [80], [60], [70], [60], [75], [85]]
输出：
[null, 1, 1, 1, 2, 1, 4, 6]

解释：
StockSpanner stockSpanner = new StockSpanner();
stockSpanner.next(100); // 返回 1
stockSpanner.next(80);  // 返回 1
stockSpanner.next(60);  // 返回 1
stockSpanner.next(70);  // 返回 2
stockSpanner.next(60);  // 返回 1
stockSpanner.next(75);  // 返回 4 ，因为截至今天的最后 4 个股价 (包括今天的股价 75) 都小于或等于今天的股价。
stockSpanner.next(85);  // 返回 6
```

题解：使用两个栈记录股价及其跨度（或使用Deque<int[]>）。初始跨度为1，比栈顶元素是否小于当前元素，如果是则弹出并将之前的跨度也累积上。弹出操作完毕后将当前元素的价格和跨度都入栈。
```java
public int next(int price) {
	int span = 1;
	while (!stack.isEmpty() && price >= stack.peekLast()[0]){
		int preSpan = stack.removeLast()[1];
		span += preSpan;
	}
	stack.addLast(new int[] {price, span});
	return span;
}
```