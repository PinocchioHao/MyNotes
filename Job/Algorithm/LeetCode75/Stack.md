### [2390. 从字符串中移除星号](https://leetcode.cn/problems/removing-stars-from-a-string/)
[2390. Removing Stars From a String](https://leetcode.com/problems/removing-stars-from-a-string/)
中等
给你一个包含若干星号 `*` 的字符串 `s` 。
在一步操作中，你可以：
- 选中 `s` 中的一个星号。
- 移除星号 **左侧** 最近的那个 **非星号** 字符，并移除该星号自身。
返回移除 **所有** 星号之后的字符串**。**
**注意：**
- 生成的输入保证总是可以执行题面中描述的操作。
- 可以证明结果字符串是唯一的。

**Example 1:**
```
Input: s = "leet**cod*e"
Output: "lecoe"
```

解法：栈，正常压入元素，遇到`*`就弹出元素。可以利用`Deque，StringBuilder，char[]`都可以模拟栈，注意思想即可。StringBuilder遇到`*`可以`sb.deleteCharAt(sb.length() - 1);`，而`char[]`可以维护一天栈顶指针，遇到`*`则`top--`(注意top大于等于0)。

### [735. 小行星碰撞](https://leetcode.cn/problems/asteroid-collision/)
[735. Asteroid Collision](https://leetcode.com/problems/asteroid-collision/)
中等
给定一个整数数组 `asteroids`，表示在同一行的小行星。数组中小行星的索引表示它们在空间中的相对位置。
对于数组中的每一个元素，其绝对值表示小行星的大小，正负表示小行星的移动方向（正表示向右移动，负表示向左移动）。每一颗小行星以相同的速度移动。
找出碰撞后剩下的所有小行星。碰撞规则：两个小行星相互碰撞，较小的小行星会爆炸。如果两颗小行星大小相同，则两颗小行星都会爆炸。两颗移动方向相同的小行星，永远不会发生碰撞。

**Example 1:**
**Input:** asteroids = [5,10,-5]
**Output:** [5,10]
**Explanation:** The 10 and -5 collide resulting in 10. The 5 and 10 never collide.

思路：正数向右，负数向左，只有左正右负的情况会发生碰撞，发生碰撞时还需要判断碰撞是否会继续。

解法：栈。引入`isAlive`变量记录行星是否存活，使用Deque存入当前存活的行星，每遍历到一个元素则模拟一次碰撞，先默认它`isAlive`为真。然后while判断并处理碰撞，当当前元素小于0，栈非空且栈顶元素大于0才会发生碰撞，发生碰撞后判断行星大小，如果当前大则继续往左碰撞，即`deque.removeLast()`,如果当前更小则`isAlive = false`，否则双方相等则同时被消灭，循环最后判断`isAlive`来确定当前元素要不要被入栈。最后把deque转换成数组即所需结果。


### [394. 字符串解码](https://leetcode.cn/problems/decode-string/)
[394. Decode String](https://leetcode.com/problems/decode-string/)
中等
给定一个经过编码的字符串，返回它解码后的字符串。
编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。
你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k` ，例如不会出现像 `3a` 或 `2[4]` 的输入。
测试用例保证输出的长度不会超过 `105`。

**Example 2:**
**Input:** s = "3[a2[c]]"
**Output:** "accaccacc"

**Example 3:**
**Input:** s = "2[abc]3[cd]ef"
**Output:** "abcabccdcdcdef"

思路：不光要记录重复次数，以及还要注意顺序，还可能会产生嵌套，还要注意数字可能是多位数字。这里采用双栈，一个记录重复次数，一个记录正向的字符串数组。

