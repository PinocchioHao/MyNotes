
### 二分查找
**一定要注意边界！！！** 
并且二分查找只适用于排好序的数组，操作前**先排序**！！！

全闭区间交叉法(“左神模板”)：
##### 精确匹配
```java
// 搜索区间：[left, right] （左闭右闭）
int left = 0;
int right = nums.length - 1; // 因为是闭区间，right 必须是真实的最后一个元素索引

while (left <= right) { // 关键：必须是 <=。因为 left == right 时，那个唯一的元素还没被检查！
    // int mid = left + ((right - left) >> 1); 
    // 或者采用这种移位操作写法，注意括号要再包两层，如果left + (right - left) >> 1 这种写法相当于没括号，等同于(left + right - left) >> 1，最后变成了 right >> 1
    int mid = left + (right - left) / 2;
    
    if (nums[mid] == target) {
        return mid; // 找到了，直接收工
    } else if (nums[mid] < target) {
        left = mid + 1; // mid 太小了，往更大的半边收缩
    } else {
        right = mid - 1; // mid 太大了，往更小的半边收缩
    }
}
return -1; // 找遍了也没有
```

##### 寻找左边界（第一个>=target的位置）
```java
int left = 0, right = arr.length - 1;
int ans = -1; // 或者初始化为arr.length都可以，某些场景这个值可便于操作
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (arr[mid] >= target) {
	    // mid所在元素只要>=target就记录进ans，随后右边界收缩
	    // 最后ans记录的一定是第一个>=target的元素 
        ans = mid;      // 当前位置符合，先记下来！
        right = mid - 1; // 继续往左逼近，看有没有更靠左的符合条件的位置
    } else {
        left = mid + 1;  // 不达标，往右找
    }
    
    // 最终左右边界都会收缩一致，成为left == right，此时为第一个>=target的位置
    // 注意最最最后这一步执行完后退出循环，是left>right (left = right + 1)
    // 常规情况下，如果找得到这个元素，那么最终结果ans = left = right + 1，但是不能无脑返回left，因为ans还起了一个保护作用，如果找不到，应该返回-1，而left则不能表现
}
return ans; 
// 循环结束，ans 里记录的就是最后一个被记下来的达标者，也就是最左边的那个！
```


##### 寻找右边界（最后一个<=target的位置）
```java
int left = 0, right = arr.length - 1;
int ans = -1; // 记录
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (arr[mid] <= target) {
        ans = mid;      // 当前位置符合，先记下来！
        left = mid + 1;  // 继续往右逼近，看有没有更靠右的符合的位置
    } else {
        right = mid - 1; // 不达标，往左找
    }
}
return ans; 
// 循环结束，ans 里记录的就是最后一个被记下来的达标者，也就是最右边的那个！
```

其实寻找右边界还可以通过左边界函数反推，左边界的左边元素其实就是最后一个<target的元素；如果我们要用左边界模板找最后一个<=target的元素，还可以找第一个>=target+1的元素，然后这个元素左边的位置就是最后一个<=target的元素（前提是整数数组）。


---

### [374. 猜数字大小](https://leetcode.cn/problems/guess-number-higher-or-lower/)
[374. Guess Number Higher or Lower](https://leetcode.com/problems/guess-number-higher-or-lower/)

我们正在玩猜数字游戏。猜数字游戏的规则如下：
我会从 `1` 到 `n` 随机选择一个数字。 请你猜选出的是哪个数字。（我选的数字在整个游戏中保持不变）。
如果你猜错了，我会告诉你，我选出的数字比你猜测的数字大了还是小了。
你可以通过调用一个预先定义好的接口 `int guess(int num)` 来获取猜测结果，返回值一共有三种可能的情况：
- `-1`：你猜的数字比我选出的数字大 （即 `num > pick`）。
- `1`：你猜的数字比我选出的数字小 （即 `num < pick`）。
- `0`：你猜的数字与我选出的数字相等。（即 `num == pick`）。

返回我选出的数字。

**示例 1：**
输入：n = 10, pick = 6
输出：6

题解：用基础的二分搜索从1到n之间猜数字即可。

---
### [2300. 咒语和药水的成功对数](https://leetcode.cn/problems/successful-pairs-of-spells-and-potions/)
[2300. Successful Pairs of Spells and Potions](https://leetcode.com/problems/successful-pairs-of-spells-and-potions/)

给你两个正整数数组 `spells` 和 `potions` ，长度分别为 `n` 和 `m` ，其中 `spells[i]` 表示第 `i` 个咒语的能量强度，`potions[j]` 表示第 `j` 瓶药水的能量强度。
同时给你一个整数 `success` 。一个咒语和药水的能量强度 **相乘** 如果 **大于等于** `success` ，那么它们视为一对 **成功** 的组合。
请你返回一个长度为 `n` 的整数数组 `pairs`，其中 `pairs[i]` 是能跟第 `i` 个咒语成功组合的 **药水** 数目。

**Example 1:**
```
Input: spells = [5,1,3], potions = [1,2,3,4,5], success = 7
Output: [4,0,3]
Explanation:
- 0th spell: 5 * [1,2,3,4,5] = [5,10,15,20,25]. 4 pairs are successful.
- 1st spell: 1 * [1,2,3,4,5] = [1,2,3,4,5]. 0 pairs are successful.
- 2nd spell: 3 * [1,2,3,4,5] = [3,6,9,12,15]. 3 pairs are successful.
Thus, [4,0,3] is returned.
```

题解：二分查找找左边界。排序potions数组便于二分操作。然后遍历speels数组堆每一组值进行计算，left取0，right取potions.length-1。计算乘积的时候注意要**转long**。`long prd = (long)spell * potions[mid];`（前后都要加上long，后面不加long还是int计算）

---

### [162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)
峰值元素是指其值严格大于左右相邻值的元素。
给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 **任何一个峰值** 所在位置即可。你可以假设 `nums[-1] = nums[n] = -∞` 。对于所有有效的 `i` 都有 `nums[i] != nums[i + 1]`。
你必须实现时间复杂度为 `O(log n)` 的算法来解决此问题。

**示例 1：**
```
输入：nums = [1,2,3,1]
输出：2
解释：3 是峰值元素，你的函数应该返回其索引 2。
```

思路：遍历所有元素比较连续三个元素需要的代价为O(n)，可以采用二分搜索。由于 `nums[-1] = nums[n] = -∞` ，所以在数组中一定能找到峰值，遇到爬坡的，那一定能遇到山顶，往爬坡的一方收缩。如果nums[i]>nums[i+1]，则往自己及左半边找，如果nums[i]<nums[i+1]，则往i+1及其右半边找。（边界情况：一直下坡，则0号元素为峰值；一直上坡，则最后一个元素为峰值）
二分核心代码（特殊情况中仅考虑一直上坡最后一个元素为峰值的情况即可，一路下坡的情况能直接进条件中判断到，而一路上坡需要额外判断下是否到最后一个位置）：
```java
while (left <= right) {  
    int mid = left + (right - left) / 2;  
    // 【核心逻辑】：只要是上坡，mid 必不是峰顶，直接往右走  
    // mid < nums.length - 1 防越界，以及确保不是最后一个元素的情况下往右
    if (mid < nums.length - 1 && nums[mid] < nums[mid + 1]) {  
        left = mid + 1;  
    } else {  
        // 否则（下坡，或者走到了最右边悬崖），mid 有可能是峰顶！  
        res = mid;         // 记下候选人  
        right = mid - 1;   // 往左逼近  
    }

	// 或者以下逻辑：
	//【正向思维】：什么情况下 mid 有可能是峰顶？
	// 1. 走到了最右边的悬崖 (mid == nums.length - 1)
	// 2. 明确的下坡 (nums[mid] > nums[mid + 1])
	// 注意：必须把 mid == nums.length - 1 写在前面，利用 || 的短路特性防止越界！
	if (mid == nums.length - 1 || nums[mid] > nums[mid + 1]) {
		ans = mid;         // 当前位置符合“下坡/悬崖”，记下这个可能的峰顶！
		right = mid - 1;   // 往左边逼近，看有没有更靠左的峰
	} else {
		// 否则（明确的上坡，且没到悬崖）
		left = mid + 1;    // 放心往右爬
	}

}
```


### [875. 爱吃香蕉的珂珂](https://leetcode.cn/problems/koko-eating-bananas/)
[875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)
珂珂喜欢吃香蕉。这里有 `n` 堆香蕉，第 `i` 堆中有 `piles[i]` 根香蕉。警卫已经离开了，将在 `h` 小时后回来。
珂珂可以决定她吃香蕉的速度 `k` （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 `k` 根。如果这堆香蕉少于 `k` 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉。  
珂珂喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。
返回她可以在 `h` 小时内吃掉所有香蕉的最小速度 `k`（`k` 为整数）。

**Example 1:**
**Input:** piles = [3,6,7,11], h = 8
**Output:** 4

解法：二分查找在速度[1, piles.max]里面找一个能够吃完的最小速度。
判断能否吃完的工具函数，注意边界以及除法取整技巧：
```java
boolean canFinish(int[] piles, int h, int speed){  
	// 一定注意用long，防止累加过程中整形溢出  
	long finishHours = 0;  
	for (int pile : piles){  
		// int除法整除后不需要向上取整的特殊操作  
		int time = (pile - 1) / speed + 1;  
		finishHours += time;  
	}  
	return finishHours <= h;  
}
```