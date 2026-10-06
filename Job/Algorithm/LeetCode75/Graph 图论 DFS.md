### 图的结构：
一般的图的结构可以是`Map<起点, List<终点>>`，带权重的可以是`Map<起点, Map<终点, 权重>>`，一些题目直接给的一些二维数组，比如岛屿数量、省份数量、房间钥匙数、腐烂橘子等；


解题模板：
```java
class GraphTemplate {
    // 记录访问节点，防止死循环的“备忘录”
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
    // 这种方法属于“门外验完再放行”（前瞻式 / 提前剪枝）
    private void dfs(int curr, Map<Integer, List<Integer>> graph) {
        // 1. 进场：标记当前点已被访问
        // 既然进来了，说明门外已经验过了，直接标记
        visited.add(curr); 

        // 2. 扩散：遍历当前点的所有邻居
        if (graph.containsKey(curr)) {
            for (int neighbor : graph.get(curr)) {
                // 3. 剪枝：去过的地方绝不再去（门外验证身份）
                if (!visited.contains(neighbor)) {
	                // 合法的才会进入下一次递归
                    dfs(neighbor, graph);
                }
            }
        }
    }

	// 另一种“进门再验身份”（防御式 / 兜底拦截）
	private void dfs(int curr, ...) {
	    if (!visited[curr]) { // 进门第一件事：验明正身
	        visited[curr] = true;
	        for (...) {
	            dfs(neighbor, ...); // 不管三七二十一，先无脑调用再说
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
1. 使用`map.computeIfAbsent()`进行节点连接，`graph.computeIfAbsent(u, k -> new ArrayList<>()).add(v);`
2. 有些图需要把正向逆向都构建出来，可以设置不同的权重


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

解法2：也可以用BFS，套用标准BFS模板，初始化把0号房入队列，拿到当前房的钥匙出队，然后根据遍历这些钥匙对应的房间，如果没被访问过，则将其对应的visited置为访问过，把当前位置的房间钥匙入队之后再开。也可以维护遍历数`cnt`，遍历完比较`cnt`跟`rooms.size`是否相等。
```java
public boolean canVisitAllRooms(List<List<Integer>> rooms) {
	boolean[] visited = new boolean[rooms.size()];
	Queue<Integer> queue = new LinkedList();
	queue.offer(0);
	visited[0] = true;
	while(!queue.isEmpty()){
		int size = queue.size();
		for(int i = 0; i < size; i++){
			int curr = queue.poll();
			List<Integer> neighbors = rooms.get(curr);
			for (int neighbor : neighbors){
				if (!visited[neighbor]){
					queue.offer(neighbor);
					visited[neighbor] = true;
				}
			}
		}
	}
	
	for (boolean v : visited){
		if (!v) return false;
	}
	return true;
}
```



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

思路：根据路径建立正向和逆向的图，分别设置不同的权重来标记。dfs从0号城市遍历图，记录所有的正向路径，即需要反转的路径数。

```java
// 构建权重图，记正向的权重为1，逆向的权重为0  
// 从0号城市出发，dfs遍历图，累加权重即可得到题目要求需要逆向的边数  
public int minReorder2(int n, int[][] connections) {  
    // int[] 0号位置为邻接点，1号位置为权重  
    Map<Integer, List<int[]>> graph = new HashMap();  
  
    // 建图  
    for(int[] edge : connections){  
        int from = edge[0];  
        int to = edge[1];  
        // 正向的权重记1，反向的也记录权重为0  
        graph.computeIfAbsent(from, k -> new ArrayList()).add(new int[]{to, 1});  
        graph.computeIfAbsent(to, k -> new ArrayList()).add(new int[]{from, 0});  
    }  
    boolean[] visited = new boolean[n];  
    dfs(0, graph, visited);  
    return reorderCnt;  
  
}  
  
public void dfs(int curr, Map<Integer, List<int[]>> graph, boolean[] visited){  
    // 访问即置为true  
    visited[curr] = true;  
    List<int[]> neighbors = graph.get(curr);  
    for(int[] neighbor : neighbors) {  
        // 如果邻居没去过就访问  
        if(!visited[neighbor[0]]){  
            // 需要逆向的边即从0出发正向的权重  
            reorderCnt += neighbor[1];  
            dfs(neighbor[0], graph, visited);  
        }  
    }  
}
```



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

**注意审题这里面每个变量名字独立，不存在"结合律"，比如bc/cd不等于b/d，表示独立全新的变量。**

思路：这题就是带权重路径乘积。建图将`a/b`和`b/a`的值存入图中。然后求`queries`数组中的值即可转换为求图中是否存在一条相应的路径，以及计算这个路径的权重累乘。注意可能不存在路径，以及需要判断结束。
```java
    // 将除法转为图，A/B转为A->B和B->A，以及它们的权重，针对queries里面的每一组做图遍历  
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {  
        // 1. 建图：Map<起点, Map<终点, 权重>>  
        Map<String, Map<String, Double>> graph = new HashMap<>();  
        for (int i = 0; i < values.length; i++) {  
            String u = equations.get(i).get(0);  
            String v = equations.get(i).get(1);  
            double val = values[i];  
            // 先找graph里有没有以u为key的，如果有则直接拿出来返回，没有则执行lambda表达式现场新建HashMap往里面.put(v,val)  
            graph.computeIfAbsent(u, k -> new HashMap<>()).put(v, val);  
            graph.computeIfAbsent(v, k -> new HashMap<>()).put(u, 1.0 / val);  
  
            // 也可用以下传统方法建图，注意没有就先new一个，然后put进去  
//            // 拿到起点 u，看看外层 Map 里有没有为它建立过内层 Map//            if (!graph.containsKey(u)) {  
//                // 如果没有，手动 new 一个内层 Map，并塞进外层 Map//                graph.put(u, new HashMap<>());  
//            }  
//            // 此时，u 对应的内层 Map 绝对百分百存在了  
//            // 直接把它 get 出来，然后把邻居 v 和权重放进去  
//            graph.get(u).put(v, val);  
  
            // 或者以下  
//            String start = equations.get(i).get(0);  
//            String end = equations.get(i).get(1);  
//            Map<String, Double> val1 = graph.getOrDefault(start, new HashMap());  
//            val1.put(end, values[i]);  
//            graph.put(start, val1);  
//            Map<String, Double> val2 = graph.getOrDefault(end, new HashMap());  
//            val2.put(start, 1/values[i]);  
//            graph.put(end, val2);  
  
        }  
  
        // 2. 准备结果数组  
        double[] results = new double[queries.size()];  
  
        // 3. 针对每一个 query，做一次独立的 DFS 寻路  
        for (int i = 0; i < queries.size(); i++) {  
            String start = queries.get(i).get(0);  
            String end = queries.get(i).get(1);  
  
            // 特殊情况：如果图里压根没有这个变量，直接判死刑，返回 -1.0            if (!graph.containsKey(start) || !graph.containsKey(end)) {  
                results[i] = -1.0;  
            } else if (start.equals(end)) {  
                // 自己除以自己，结果必为 1.0，不加这个分支，放到dfs里面也没啥影响  
                results[i] = 1.0;  
            } else {  
                // 正常的图寻路，每次询问都用一个干净的 visited 集合  
                Set<String> visited = new HashSet<>();  
                results[i] = dfs(start, end, 1, graph, visited);  
            }  
        }  
  
        return results;  
    }  
  
    // 自顶向下 DFS 寻路函数    
     private double dfs(String curr, String target, double currentProduct, Map<String, Map<String, Double>> graph, Set<String> visited) {  
        // 1. 进场打标签，防止死循环  
        visited.add(curr);  
  
        // 2. 基准情况（找到了！）：如果当前节点就是终点  
        // 直接把这一路走来、已经算好的 currentProduct 吐出去  
        if (curr.equals(target)) {  
            return currentProduct;  
        }  
  
        // 3. 拿到当前节点的所有邻居开始遍历  
        Map<String, Double> neighbors = graph.get(curr);  
        for (String neighbor : neighbors.keySet()) {  
  
            // 如果邻居没去过，继续往前走  
            if (!visited.contains(neighbor)) {  
  
                // 【核心变化点】：自顶向下  
                // 迈向邻居这一步时，直接用“当前的积”乘上“到邻居的权重”  
                // 算出一个全新的累乘值，当成“历史包袱”直接丢给下一层  
                double nextProduct = currentProduct * neighbors.get(neighbor);  
  
                double result = dfs(neighbor, target, nextProduct, graph, visited);  
  
                // 关键传递：如果子搜索返回的不是 -1.0，说明它在更深的地方摸到了终点  
                // 因为结果在终点处已经完全算好了，所以这里不需要做任何加工，直接原样往上传  
                // 如果不加这层判断，直接 return dfs(...)，会怎么样？会不会有问题？（会的！）为什么？（因为如果子搜索没找到终点，它会返回 -1.0，这个 -1.0 也会被原样往上传，导致父搜索以为自己找到了终点了，结果就错了！）  
                if (result != -1.0) {  
                    return result;  
                }  
            }  
        }  
  
        // 4. 走投无路，返回 -1.0        
        return -1.0;  
    }
```

