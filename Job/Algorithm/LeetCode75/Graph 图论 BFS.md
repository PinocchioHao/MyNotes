解题模板类似二叉树BFS，注意节点必须入队时就标记访问，否则会出现OOM！
```java
public void bfs(int start, Map<Integer, List<Integer>> graph) {
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();

    // 1. 起点入队，并立刻标记已访问
    queue.offer(start);
    visited.add(start);

    int step = 0; // 如果题目求最短路径，用 step 记录层数

    while (!queue.isEmpty()) {
        int size = queue.size(); // 固定当前层的节点数
        
        for (int i = 0; i < size; i++) {
            int curr = queue.poll();
            
            // 2. 遍历当前节点的所有邻居
            for (int neighbor : graph.getOrDefault(curr, new ArrayList<>())) {
                if (!visited.contains(neighbor)) {
                    // 3. 邻居入队，并【立刻标记已访问】
                    visited.add(neighbor);
                    queue.offer(neighbor);
                }
            }
        }
        // 一层扩散完毕，距离 + 1
        // 注意这里可能需要加某些条件，如994题腐烂橘子题中需要下一轮有新鲜橘子才累加
        step++; 
    }
}
```

Tips:
1. 方向可以用`int[][] dirs = new int[][]{{1,0},{-1,0},{0,1},{0,-1}};`遍历4个方向写在for循环里
2. 注意边界，需要在上下左右有效范围内

---

### [1926. 迷宫中离入口最近的出口](https://leetcode.cn/problems/nearest-exit-from-entrance-in-maze/)
[1926. Nearest Exit from Entrance in Maze](https://leetcode.com/problems/nearest-exit-from-entrance-in-maze/)
给你一个 `m x n` 的迷宫矩阵 `maze` （**下标从 0 开始**），矩阵中有空格子（用 `'.'` 表示）和墙（用 `'+'` 表示）。同时给你迷宫的入口 `entrance` ，用 `entrance = [entrancerow, entrancecol]` 表示你一开始所在格子的行和列。
每一步操作，你可以往 **上**，**下**，**左** 或者 **右** 移动一个格子。你不能进入墙所在的格子，你也不能离开迷宫。你的目标是找到离 `entrance` **最近** 的出口。**出口** 的含义是 `maze` **边界** 上的 **空格子**。`entrance` 格子 **不算** 出口。
请你返回从 `entrance` 到最近出口的最短路径的 **步数** ，如果不存在这样的路径，请你返回 `-1` 。
**示例 1：**

![](https://assets.leetcode.com/uploads/2021/06/04/nearest1-grid.jpg)

```
输入：maze = [["+","+",".","+"],[".",".",".","+"],["+","+","+","."]], entrance = [1,2]
输出：1
解释：总共有 3 个出口，分别位于 (1,0)，(0,2) 和 (2,3) 。
一开始，你在入口格子 (1,2) 处。
- 你可以往左移动 2 步到达 (1,0) 。
- 你可以往上移动 1 步到达 (0,2) 。
从入口处没法到达 (2,3) 。
所以，最近的出口是 (0,2) ，距离为 1 步。
```

题解：把整个矩阵看作图，上下左右4格为邻居，BFS遍历，求遍历层数。套BFS模板即可，入队后可以将元素置为墙。判断如果元素走到边界上即可退出返回step，以及进入下一层时要判断邻居在矩阵范围内，以及为空格才进行。while循环里面处理完for当前层数的逻辑后，最后一步层数累加。


---

### [994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)
[994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：
- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。
每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。
返回 _直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`_ 。

**示例 1：**

**![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2019/02/16/oranges.png)**
```
输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

题解：依然是BFS扩散问题。将所有腐烂橘子入队，并记录新鲜橘子数量。注意while循环条件要加上新鲜橘子大于0的判定。每扩散到邻居就`fresh--`，每扩散完一层就`minutes++`。

```java
public int orangesRotting(int[][] grid) {
	int[][] dirs = new int[][]{{0,1}, {0,-1}, {1,0}, {-1,0}};
	int fresh = 0;
	Queue<int[]> queue = new LinkedList();
	for(int i = 0; i < grid.length; i++){
		for(int j = 0; j < grid[0].length; j++){
			if(grid[i][j] == 2) {
				// 腐烂橘子入队
				queue.offer(new int[]{i,j});
			}
			else if(grid[i][j] == 1) {
				fresh++;
			}
		}
	}
	
	int minutes = 0;
	// 只有当图里还有好橘子(fresh > 0)，且队列里还有可以继续扩散的传染源时才继续
	// 因为fresh--是在当前“透支”下一层的，所以必须是加上这个条件
	while(fresh >0 && !queue.isEmpty()){
		int size = queue.size();
		for(int i = 0; i < size; i++){
			int[] curr = queue.poll();
			// 向邻居扩散
			for(int[] dir : dirs) {
				int[] next = new int[]{curr[0]+dir[0], curr[1]+dir[1]};
				if(next[0] >= 0 && next[0] <= grid.length - 1 && next[1] >= 0 && next[1] <= grid[0].length - 1 && grid[next[0]][next[1]] == 1){
					queue.offer(next);
					grid[next[0]][next[1]] = 2;
					fresh--;
				}
			}

		}
		minutes++;
	}
	// 遍历完如果有橘子没被感染输出-1，全被感染输出分钟数
	return fresh > 0 ? -1 :minutes;
}
```