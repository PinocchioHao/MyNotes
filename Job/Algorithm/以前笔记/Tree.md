##### 100 相同的树
解法1：递归，若全空返回true，若一方为空返回false，**若值不等返回false（注意这里不能是值相等返回true，因为这样会直接返回true，而不会进入子树判断；这一步也相当于剪枝操作）**，否则继续递归`return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);`

解法2：把树递归遍历取到list里面，比较list

##### 101 对称的树
解法1：dfs递归，注意写个helper，传参为left和right.

解法2：bfs迭代，queue先进入左右孩子，while循环里弹出两个进行比较（第一步判断两个为null时需要continue，一方空或不等输出false），最后进队列的时候控制顺序：左左右右，左右右左进入。遍历完输出true


##### 102 分别输出树每层的元素
解法1：按层遍历的bfs

解法2：dfs：dfs入参为树节点，层数，结果集，注意在第一次访问该层的时候结果集要添加当前层级的数组，每次访问到该节点则往当前层的结果集里添加当前节点值，最后往新的一层dfs。

##### 103 之字形遍历树
解法1：类似102题bfs按层遍历，双数层的list反转（可用Collections.reverse(); 或者使用Deque来实现）


##### 104 找树最大高度
解法1：dfs，找深度模板
```java
public static int maxDepth(TreeNode root) {  
    if (root == null) return 0;  
    int leftDepth = maxDepth(root.left);  
    int rightDepth = maxDepth(root.right);  
    return Math.max(leftDepth, rightDepth) + 1;  
}
```

解法2：bfs，遍历完每层高度+1；



##### 110 判断树是否是平衡二叉树
解法1：递归看左右子树高度差是否小于1；求左右子树的高度，若相差大于1则返回false，否则递归`return isBalanced(root.left) && isBalanced(root.right);`
注意这种解法套了双层递归，复杂度O(n方)

解法2：在递归求树的高度的时候可以加入平衡判定，正常返回树高，如果不平衡可以返回-1


##### 111 找树最小高度
解法1：dfs，用104类似求深度模板，但是要**单独讨论一边为null另一边非null情况**

解法2：bfs，找到第一个叶子节点所在位置就是最小高度

解法3：dfs+剪枝，声明全局变量minDepth，新建递归helper函数，传TreeNode和depth，每找到叶子节点就更新树高，剪枝判断当前depth大于等于minDepth就直接return，递归遍历左右子树