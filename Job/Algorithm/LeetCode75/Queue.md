### [933. 最近的请求次数](https://leetcode.cn/problems/number-of-recent-calls/)
[933. Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/)
简单
写一个 `RecentCounter` 类来计算特定时间范围内最近的请求。
请你实现 `RecentCounter` 类：
- `RecentCounter()` 初始化计数器，请求数为 0 。
- `int ping(int t)` 在时间 `t` 添加一个新请求，其中 `t` 表示以毫秒为单位的某个时间，并返回过去 `3000` 毫秒内发生的所有请求数（包括新请求）。确切地说，返回在 `[t-3000, t]` 内发生的请求数。
**保证** 每次对 `ping` 的调用都使用比之前更大的 `t` 值。

```
**Input**
["RecentCounter", "ping", "ping", "ping", "ping"]
[[], [1], [100], [3001], [3002]]
**Output**
[null, 1, 2, 3, 3]
```

解法：维护一个队列，记录近3000ms的请求数，返回queue.size()即可。每次收到ping的时候，将请求入队尾，并且while检查队首，如果`queue.peekFirst() < t-3000`则持续`removeFirst()`



### [649. Dota2 参议院](https://leetcode.cn/problems/dota2-senate/)
[649. Dota2 Senate](https://leetcode.com/problems/dota2-senate/)
中等
Dota2 的世界里有两个阵营：`Radiant`（天辉）和 `Dire`（夜魇）
Dota2 参议院由来自两派的参议员组成。现在参议院希望对一个 Dota2 游戏里的改变作出决定。他们以一个基于轮为过程的投票进行。在每一轮中，每一位参议员都可以行使两项权利中的 **一** 项：
- **剥夺一名参议员的权利**：一名参议员可以使另一名参议员在本轮及所有后续轮次中失去所有权利。
- **宣布胜利**：如果参议员发现有权利投票的参议员都是 **同一个阵营的** ，他可以宣布胜利并决定在游戏中的有关变化。
给你一个字符串 `senate` 代表每个参议员的阵营。字母 `'R'` 和 `'D'`分别代表了 `Radiant`（天辉）和 `Dire`（夜魇）。然后，如果有 `n` 个参议员，给定字符串的大小将是 `n`。
以轮为基础的过程从给定顺序的第一个参议员开始到最后一个参议员结束。这一过程将持续到投票结束。所有失去权利的参议员将在过程中被跳过。
假设每一位参议员都足够聪明，会为自己的政党做出最好的策略，你需要预测哪一方最终会宣布胜利并在 Dota2 游戏中决定改变。输出应该是 `"Radiant"` 或 `"Dire"` 。

**Example 2:**
**Input:** senate = "RDD"
**Output:** "Dire"
**Explanation:** 
The first senator comes from Radiant and he can just ban the next senator's right in round 1. 
And the second senator can't exercise any rights anymore since his right has been banned. 
And the third senator comes from Dire and he can ban the first senator's right in round 1. 
And in round 2, the third senator can just announce the victory since he is the only guy in the senate who can vote.

思路：顺序靠前的能干掉顺序靠后的异党，然后进入下一轮，可以利用两个队列来模拟两党竞争。

解法：队列。利用两个队列dq和rq分别存d党和r当参议员的位置下标，每次取队首元素比较，下标较大的一方则干掉下标较小的一方，较小的一方从队列中移除，较大的一方`idx+=size`排到队尾去参加下一轮竞争。这样模拟最后有一个队列为空，则被干掉，可以返回哪党赢了。