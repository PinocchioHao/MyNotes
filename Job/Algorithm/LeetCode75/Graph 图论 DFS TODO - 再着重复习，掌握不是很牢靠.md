解题模板：
```java
class GraphTemplate {
    // 核心装备：防止死循环的“备忘录”
    Set<Integer> visited = new HashSet<>(); 
    // 或者用 boolean[] visited = new boolean[N]; (性能更好)

    public void solve(int n, int[][] edges) {
        // 【第一步：建图】 将题目给的二维数组，转化为邻接表 Map<点, List<点>>
        Map<Integer, List<Integer>> graph = new HashMap<>();
        for (int[] edge : edges) {
            graph.computeIfAbsent(edge[0], k -> new ArrayList<>()).add(edge[1]);
            // 如果是无向图，反向也要加：
            // graph.computeIfAbsent(edge[1], k -> new ArrayList<>()).add(edge[0]);
        }

        // 【第二步：根据题目要求调用 DFS】
        // 场景 A：从单点出发
        dfs(0, graph); 
        
        // 场景 B：图可能不连通，需要“点名机制”（每个点都扫一遍）
        for (int i = 0; i < n; i++) {
            if (!visited.contains(i)) {
                dfs(i, graph);
                // 这里通常可以统计“连通分量”的个数
            }
        }
    }

    // 【第三步：标准的 DFS 函数】
    private void dfs(int curr, Map<Integer, List<Integer>> graph) {
        // 1. 进场：标记当前点已被访问
        visited.add(curr);

        // 2. 扩散：遍历当前点的所有邻居
        if (graph.containsKey(curr)) {
            for (int neighbor : graph.get(curr)) {
                // 3. 剪枝：去过的地方绝不再去
                if (!visited.contains(neighbor)) {
                    dfs(neighbor, graph);
                }
            }
        }
    }
}
```

解题套路：
1. 建图（有可能图中会给）；
2. 带防重机制的递归；
3. 梳理节点关系或者边之间的关系


Tips:
1. 使用`computeIfAbsent`



---

### [841. 钥匙和房间](https://leetcode.cn/problems/keys-and-rooms/)
[841. Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/)
有 `n` 个房间，房间按从 `0` 到 `n - 1` 编号。最初，除 `0` 号房间外的其余所有房间都被锁住。你的目标是进入所有的房间。然而，你不能在没有获得钥匙的时候进入锁住的房间。
当你进入一个房间，你可能会在里面找到一套 **不同的钥匙**，每把钥匙上都有对应的房间号，即表示钥匙可以打开的房间。你可以拿上所有钥匙去解锁其他房间。
给你一个数组 `rooms` 其中 `rooms[i]` 是你进入 `i` 号房间可以获得的钥匙集合。如果能进入 **所有** 房间返回 `true`，否则返回 `false`。
```
Example 2:

Input: rooms = [[1,3],[3,0,1],[2],[0]]
Output: false
Explanation: We can not enter room number 2 since the only key that unlocks it is in that room.
```

解法1：这道题是求单源连通性。rooms即题中已给出的图的关系，从0号下标dfs遍历图，维护visited数组，遍历完图后遍历visited数组，看是否全被遍历。

解法2：也可以用BFS，套用标准BFS模板，维护一个`cnt`表示遍历的房间数，初始化把0号房入队列，拿到当前房的钥匙出队，然后根据遍历这些钥匙对应的房间，如果没被访问过，则将其对应的visited置为访问过，`cnt++`，把当前位置的房间钥匙入队之后再开，最后比较`cnt`跟`rooms.size`是否相等。

---

### [547. 省份数量](https://leetcode.cn/problems/number-of-provinces/)
[547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/)
有 `n` 个城市，其中一些彼此相连，另一些没有相连。如果城市 `a` 与城市 `b` 直接相连，且城市 `b` 与城市 `c` 直接相连，那么城市 `a` 与城市 `c` 间接相连。
**省份** 是一组直接或间接相连的城市，组内不含其他没有相连的城市。
给你一个 `n x n` 的矩阵 `isConnected` ，其中 `isConnected[i][j] = 1` 表示第 `i` 个城市和第 `j` 个城市直接相连，而 `isConnected[i][j] = 0` 表示二者不直接相连。
返回矩阵中 **省份** 的数量。

```
示例 1：
输入：isConnected = [[1,1,0],[1,1,0],[0,0,1]]
输出：2
```


解法：这道题是求连通分量个数。注意dfs里面遍历下一个节点的判断`if (isConnected[currentCity][neighbor] == 1 && !visited[neighbor])`。以及在调用处遍历所有城市，判断为如果未被访问，则`cnt++`。

---

### [1466. 重新规划路线](https://leetcode.cn/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/)
[1466. Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/)
`n` 座城市，从 `0` 到 `n-1` 编号，其间共有 `n-1` 条路线。因此，要想在两座不同城市之间旅行只有唯一一条路线可供选择（路线网形成一颗树）。去年，交通运输部决定重新规划路线，以改变交通拥堵的状况。
路线用 `connections` 表示，其中 `connections[i] = [a, b]` 表示从城市 `a` 到 `b` 的一条有向路线。
今年，城市 0 将会举办一场大型比赛，很多游客都想前往城市 0 。
请你帮助重新规划路线方向，使每个城市都可以访问城市 0 。返回需要变更方向的最小路线数。
题目数据 **保证** 每个城市在重新规划路线方向后都能到达城市 0 。
**示例 1：**

**![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2020/05/30/sample_1_1819.png)**

```
输入：n = 6, connections = [[0,1],[1,3],[2,3],[4,0],[4,5]]
输出：3
解释：更改以红色显示的路线的方向，使每个城市都可以到达城市 0 。
```

思路：有向图改无向图+状态记录


TODO 这里改下代码，图用List<List<>>不太直观，重新理解下这道题





---

### [399. 除法求值](https://leetcode.cn/problems/evaluate-division/)

给你一个变量对数组 `equations` 和一个实数值数组 `values` 作为已知条件，其中 `equations[i] = [Ai, Bi]` 和 `values[i]` 共同表示等式 `Ai / Bi = values[i]` 。每个 `Ai` 或 `Bi` 是一个表示单个变量的字符串。
另有一些以数组 `queries` 表示的问题，其中 `queries[j] = [Cj, Dj]` 表示第 `j` 个问题，请你根据已知条件找出 `Cj / Dj = ?` 的结果作为答案。
返回 **所有问题的答案** 。如果存在某个无法确定的答案，则用 `-1.0` 替代这个答案。如果问题中出现了给定的已知条件中没有出现的字符串，也需要用 `-1.0` 替代这个答案。
**注意：输入总是有效的。你可以假设除法运算中不会出现除数为 0 的情况，且不存在任何矛盾的结果。
注意：未在等式列表中出现的变量是未定义的，因此无法确定它们的答案。**

**示例 1：**
```
输入：equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
输出：[6.00000,0.50000,-1.00000,1.00000,-1.00000]
解释：
条件：a / b = 2.0, b / c = 3.0
问题：a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ?
结果：[6.0, 0.5, -1.0, 1.0, -1.0 ]
注意：x 是未定义的 => -1.0
```


注意审题这里面每个变量名字独立，不存在"结合律"，比如bc/cd不等于b/d，表示独立全新的变量。

思路：这题就是带权重路径乘积。

