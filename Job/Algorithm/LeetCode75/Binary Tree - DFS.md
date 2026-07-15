Tips:
何时dfs传参处理，合适采用dfs返回值：
传参适合自顶向下处理
采用dfs返回值适合自底向上处理，，节点本身需要向父节点汇报




### [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
Easy
Given the `root` of a binary tree, return _its maximum depth_.
A binary tree's **maximum depth** is the number of nodes along the longest path from the root node down to the farthest leaf node.

解法1：自底向上
```java
// 自底向上，自己的高度取决于子孩子的最大高度+1，先跑到最底层，最后解递归时depth+1  
public int maxDepth(TreeNode root) {  
    if (root == null) return 0;  
    int leftMaxDepth = maxDepth(root.left);  
    int rightMaxDepth = maxDepth(root.right);  
    return Math.max(leftMaxDepth, rightMaxDepth) + 1;  
}
```
解法2：自顶向下
```java
public int maxDepth1(TreeNode root) {  
    return dfs(root,0);  
}  
  
// 自顶向下，带着状态递归，每向下一层就把depth+1传给孩子，找到叶子就把depth报出来  
public int dfs(TreeNode root, int depth){  
    if (root==null) return depth;  
    return Math.max(dfs(root.left,depth + 1), dfs(root.right, depth +1));  
}
```

---

### [872. 叶子相似的树](https://leetcode.cn/problems/leaf-similar-trees/)
[872. Leaf-Similar Trees](https://leetcode.com/problems/leaf-similar-trees/)
Easy
请考虑一棵二叉树上所有的叶子，这些叶子的值按从左到右的顺序排列形成一个 叶值序列 。
举个例子，如上图所示，给定一棵叶值序列为 (6, 7, 4, 9, 8) 的树。
如果有两棵二叉树的叶值序列是相同，那么我们就认为它们是 叶相似 的。
如果给定的两个根结点分别为 root1 和 root2 的树是叶相似的，则返回 true；否则返回 false 。

解法：dfs遍历root1和root2，判断如果是叶子节点就把叶子节点的值存入数组，最后判断两个数组是否相等

---
### [1448. 统计二叉树中好节点的数目](https://leetcode.cn/problems/count-good-nodes-in-binary-tree/)
中等
给你一棵根为 `root` 的二叉树，请你返回二叉树中好节点的数目。
「好节点」X 定义为：从根到该节点 X 所经过的节点中，没有任何节点的值大于 X 的值。
**示例 1：**

**![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2020/05/16/test_sample_1.png)**

**Input:** root = [3,1,4,3,null,1,5]
**Output:** 4
**Explanation:** Nodes in blue are **good**.
Root Node (3) is always a good node.
Node 4 -> (3,4) is the maximum value in the path starting from the root.
Node 5 -> (3,4,5) is the maximum value in the path
Node 3 -> (3,1,3) is the maximum value in the path.

解法1：自顶向下dfs，定义全局变量cnt，dfs返回void入参`TreeNode`和`maxVal`，如果节点空则直接返回，否则`if(root.val >= maxVal)`则cnt计数，并更新maxVal，然后dfs左右子树，每往下一层则会判断并cnt计数。（注意这里的`maxVal`是值传递，下一层改变成什么值并不会影响上一层的值，因此每次都是使用的当前层的最大值）

解法2：自底向上dfs，不用定义全局变量而是使用函数返回值在回溯的过程中产生值。若当前节点是好节点，则更新最大值且`currCnt = 1`,返回当前层数一共有的好节点数`currCnt + dfs(node.left, maxVal) + dfs(node.right, maxVal);`

---
### [112. Path Sum](https://leetcode.com/problems/path-sum/)
Easy
Given the `root` of a binary tree and an integer `targetSum`, return `true` if the tree has a **root-to-leaf** path such that adding up all the values along the path equals `targetSum`.
A **leaf** is a node with no children.
**Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/18/pathsum1.jpg)

**Input:** root = [5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22
**Output:** true
**Explanation:** The root-to-leaf path with the target sum is shown.

解法：自顶向下dfs，注意要把`targetSum - root.val`传到下一层，每到一层先判断当前节点非空则直接返回false，如果是叶子节点则判断`targetSum - root.val` 是否为0，如果为0则说明这条路径就是给定总和的路径，则返回true。否则递归判断左右孩子，取或值。

---
### [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/)
Medium
Given the `root` of a binary tree and an integer `targetSum`, return _all **root-to-leaf** paths where the sum of the node values in the path equals_ `targetSum`_. Each path should be returned as a list of the node **values**, not node references_.
A **root-to-leaf** path is a path starting from the root and ending at any leaf node. A **leaf** is a node with no children.

```
**Input:** root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
**Output:** [[5,4,11,2],[5,8,4,5]]
**Explanation:** There are two paths whose sum equals targetSum:
5 + 4 + 11 + 2 = 22
5 + 8 + 4 + 5 = 22
```

思路：这个问题跟112类似，只是dfs过程中使用List记录遍历元素即可，这里要注意List新增元素需要重新new一个以及回溯时需要移除当前的节点的细节。

---
### [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)
[437. Path Sum III](https://leetcode.com/problems/path-sum-iii/)
中等 -- 偏难
给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。
**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。
![](https://assets.leetcode.com/uploads/2021/04/09/pathsum3-1-tree.jpg)

**Input:** root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
**Output:** 3
**Explanation:** The paths that sum to 8 are shown.

**Constraints:**
- The number of nodes in the tree is in the range `[0, 1000]`.
- `-109 <= Node.val <= 109`
- `-1000 <= targetSum <= 1000`

思路：前缀和+dfs+回溯+HashMap。记录每个节点的前缀和，若存在两个前缀和`sum2-sum1 == targetSum`,则说明有一条子路径的和值为targetSum。因此HashMap可以记录前缀和，遍历到当前位置时查找HashMap中是否存有`currSum - targetSum`的值即可，注意如果`currSum == targetSum`,则是一条直达路径，此时需要初始化HashMap存一个`<0,1>`的值。

解法1：暴力嵌套dfs。思路类似112题，只不过这次可以是中间节点。dfs遍历树的每一个节点，基于这个节点以它为根又遍历找到累加为target的子节点，累加全局cnt。
```java
// 暴力法，嵌套dfs  
// 遍历所有树节点，以遍历的当前节点为root，向下找到累加为target的子节点  
public int pathSum(TreeNode root, int targetSum) {  
    if (root != null) {  
        dfs(root, targetSum);  
        pathSum(root.left, targetSum);  
        pathSum(root.right, targetSum);  
    }  
    return globalCnt;  
}  
  
// 递归找路径和，类似112题思路，只不过这里可以是遍历过程中任意节点，再注意targetSum用long防止int超界  
void dfs(TreeNode node, long targetSum) {  
    if (node == null) return;  
    // 路径中任意一节点都可以判断是否累加到targetSum  
    if (node.val == targetSum) globalCnt++;  
    // 持续向左右子树递归  
    dfs(node.left, targetSum - node.val);  
    dfs(node.right, targetSum - node.val);  
}
```

解法2：前缀和+dfs+回溯+HashMap。注意题目的坑有整数溢出需要long类型；以及初始化sumMap需要加入`<0,1>`表示累加值跟targetSum相等的场景，需要查得`<0,1>`来进行处理。
```java
    int cnt = 0;
    public int pathSum(TreeNode root, int targetSum) {
        Map<Long, Integer> prefixSumMap = new HashMap<>();
        // 【重要基准】：前缀和为 0 默认出现 1 次。
        // 理由：如果有一条直接路径，当前 currSum 正好等于 targetSum，那么 currSum - targetSum = 0
        // 此时我们需要从 map 里能查到这个 0，并且值为1，表示能够直达targetSum的这种情况。
        prefixSumMap.put(0L, 1);
        dfs(root, targetSum, 0, prefixSumMap);
        return cnt;
    }

    public void dfs(TreeNode node, int targetSum, long curSum, Map<Long, Integer> sumMap){
        if(node == null) return;
        // 当前节点新和值
        long newSum = node.val + curSum;
        // 凑targetSum的差值
        long pathValidSub = newSum - targetSum;
        // 先查找和处理（看历史记录里有没有能凑成 target 的），如果 currSum 是 18，target 是 8，我们找历史里有没有 10
        cnt += sumMap.getOrDefault(pathValidSub, 0);
        // 然后再将当前节点前缀和记录，供子节点使用
        sumMap.merge(newSum, 1, Integer::sum);
        // 用新数据递归进入子节点
        dfs(node.left, targetSum, newSum, sumMap);
        dfs(node.right, targetSum, newSum, sumMap);
        // 回溯把map中key为当前的前缀和的值-1，相当于删除了这条路径，避免返回父节点遍历另一分支造成污染
        sumMap.put(newSum, sumMap.get(newSum) - 1);
    }
```


---

### [1372. 二叉树中的最长交错路径](https://leetcode.cn/problems/longest-zigzag-path-in-a-binary-tree/)
[1372. Longest ZigZag Path in a Binary Tree](https://leetcode.com/problems/longest-zigzag-path-in-a-binary-tree/)
中等
给你一棵以 `root` 为根的二叉树，二叉树中的交错路径定义如下：
- 选择二叉树中 **任意** 节点和一个方向（左或者右）。
- 如果前进方向为右，那么移动到当前节点的的右子节点，否则移动到它的左子节点。
- 改变前进方向：左变右或者右变左。
- 重复第二步和第三步，直到你在树中无法继续移动。
交错路径的长度定义为：**访问过的节点数目 - 1**（单个节点的路径长度为 0 ）。
请你返回给定树中最长 **交错路径** 的长度。
**示例 1：**

**![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2020/03/07/sample_1_1702.png)**

**输入：**root = [1,null,1,1,1,null,null,1,1,null,1,null,null,null,1,null,1]
**输出：**3
**解释：**蓝色节点为树中最长交错路径（右 -> 左 -> 右）。

思路：1. dfs暴力法，遍历每个节点，从当前节点向左右子树分别while循环zigzag向下走，统计最长路径；2. 一遍dfs，传入上一步是往哪个方向走的，如果下一步往相同方向走则把path更新为1，如果往反方向zigzag走则累加path长度

解法1：定义全局变量maxPath，dfs遍历树，每遍历到当前节点则判断null，调用工具方法找左右最长zigzag路径长，更新maxPath，然后继续dfs遍历左右子树；工具方法传参node和方向，cnt记录遍历节点数，while节点非空遍历当前节点往下走，cnt++并反转方向，最后返回cnt-1即为path长。

解法2：定义全局变量maxPath:
```java
// 状态标记法，只dfs遍历一遍树  
public int longestZigZag1(TreeNode root){  
    // 树为空返回0  
    if(root == null) return 0;  
    // dfs遍历树，由于是根节点出发向左右孩子，此时路径长初始为1  
    dfs(root.left, true, 1);  
    dfs(root.right, false, 1);  
    return maxPath1;  
}  
  
public void dfs(TreeNode node, boolean goLeft, int len){  
    if(node == null) return;  
    // 非空更新最长路径  
    maxPath1 = Math.max(maxPath1, len);  
    // 递归zigzag遍历  
    if (goLeft) {  
        // 如果进入这个节点是向左走的，那么递归向右走能延续这个长度，len+1  
        dfs(node.right, false, len + 1);  
        // 此时向左右则不符合原来zigzag规律，自己开辟一条新路径计算长度  
        dfs(node.left, true, 1);  
    } else {  
        // 进入这个节点的是向右走的，此时向左右能延续zigzag，向右走则新开路径  
        dfs(node.left, true, len + 1);  
        dfs(node.right, false, 1);  
    }
```



---

### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
[236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
[百度百科](https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

**示例 1：**

![](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

```
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
Output: 3
Explanation: The LCA of nodes 5 and 1 is 3.
```

思路：dfs，回溯过程中进行判断（此时父节点已经能拿到子节点的数据），若左右子树中找到目标节点，则返回当前节点（目标节点分散在两侧）；若左子树中找到，右子树中没找到，那么则返回左孩子，右边找到则返回右孩子（目标节点分布在一侧，共同祖先是某一个目标节点）

题解：
```java
// dfs回溯。递归在递的过程中寻找p和q，找到一方则立马返回，回溯时已经进行完寻找过程，判断自己的左右孩子是否找到元素即可。  
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {  
    // 向下寻找子节点，寻找到立马返回该节点，返回的是找到的最上层的这个节点  
    if (root == null || root.val == p.val || root.val == q.val) {  
        return root;  
    }  
    // dfs遍历  
    TreeNode left = lowestCommonAncestor(root.left, p, q);  
    TreeNode right = lowestCommonAncestor(root.right, p, q);  
    // 回溯过程，此时左右节点已遍历完，当前是作为左右节点的父亲来判断。  
    // 若p,q位于子树左右异侧，则当前节点是它们的祖先  
    if (left != null && right != null) {  
        return root;  
    }  
    // p,q位于子树同侧，此时找到了一个节点就不会往下找了，另一侧就会返回null  
    return left != null ? left : right;  
}  
  
// 另外也可以使用HashMap记录父亲节点来实现，类似两条链表找交叉，这样需要额外空间开销。
```