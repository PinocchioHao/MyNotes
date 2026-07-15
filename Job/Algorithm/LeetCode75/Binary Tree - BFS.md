BFS模板：
```java
Queue<TreeNode> queue = new LinkedList<>();
// 先把根节点入队
queue.offer(root);
while(!queue.isEmpty()){
	// 注意这里要把size快照拿出来，如果直接i<queue.size()，由于queue是在动态变化，所以会出问题
	int levelNums = queue.size();
	for(int i = 0; i < levelNums; i++){
		TreeNode node = queue.poll();
		// 这里可以写需要对元素进行的操作逻辑
		
		//孩子入队
		if(node.left != null) queue.offer(node.left);
		if(node.right != null) queue.offer(node.right);
	}
}
```



---

### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)
[199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

Given the `root` of a binary tree, imagine yourself standing on the **right side** of it, return _the values of the nodes you can see ordered from top to bottom_.

**Example 1:**
**Input:** root = [1,2,3,null,5,null,4]
**Output:** [1,3,4]
**Explanation:**

![](https://assets.leetcode.com/uploads/2024/11/24/tmpd5jn43fs-1.png)


题解：BFS按每层元素遍历树，如果是当前层的最后一个元素即加入结果集中

---

### [1161. 最大层内元素和](https://leetcode.cn/problems/maximum-level-sum-of-a-binary-tree/)
[1161. Maximum Level Sum of a Binary Tree](https://leetcode.com/problems/maximum-level-sum-of-a-binary-tree/)
Given the `root` of a binary tree, the level of its root is `1`, the level of its children is `2`, and so on.
Return the **smallest** level `x` such that the sum of all the values of nodes at level `x` is **maximal**.
**Example 1:**

![](https://assets.leetcode.com/uploads/2019/05/03/capture.JPG)

**Input:** root = [1,7,0,7,-8,null,null]
**Output:** 2
**Explanation:** 
Level 1 sum = 1.
Level 2 sum = 7 + 0 = 7.
Level 3 sum = 7 + -8 = -1.
So we return the level with the maximum sum which is level 2.

题解：BFS按层遍历树，每层元素求合，并记录更新每层和值最大值。


