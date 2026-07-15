BST是左侧严格小于右侧的二叉树，遍历时当前元素小于node.val则向左孩子继续遍历，大于则继续向右遍历。


---
### [700. 二叉搜索树中的搜索](https://leetcode.cn/problems/search-in-a-binary-search-tree/)
[700. Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/)
Easy
You are given the `root` of a binary search tree (BST) and an integer `val`.
Find the node in the BST that the node's value equals `val` and return the subtree rooted with that node. If such a node does not exist, return `null`.

题解：按照BSF定义来
```java
    public TreeNode searchBST(TreeNode root, int val) {
        if(root == null) return null;
        if(root.val == val) return root;
        if(val < root.val) return searchBST(root.left, val);
        else return searchBST(root.right, val);
    }
```

---

### [450. 删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/)
[450. Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)
Medium
Given a root node reference of a BST and a key, delete the node with the given key in the BST. Return _the **root node reference** (possibly updated) of the BST_.
Basically, the deletion can be divided into two stages:
1. Search for a node to remove.
2. If the node is found, delete the node.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/09/04/del_node_1.jpg)

**Input:** root = [5,3,6,2,4,null,7], key = 3
**Output:** [5,4,6,2,null,null,7]
**Explanation:** Given key to delete is 3. So we find the node with value 3 and delete it.
One valid answer is [5,4,6,2,null,null,7], shown in the above BST.
Please notice that another valid answer is [5,2,6,null,4,null,7] and it's also accepted.
![](https://assets.leetcode.com/uploads/2020/09/04/del_node_supp.jpg)

解法：dfs回溯。BST删除节点后会动态平衡，选右子树最小（或左子树最大）元素来补位。注意dfs带返回值，返回当前层整理好的root节点。dfs过程中如果目标在左右两侧，则去子树中删除节点，删完把子树接回来，如`root.left = deleteNode(root.left, key)`；如果找到了目标节点，则执行删除操作，这里又分情况讨论：如果左右孩子某一方为空，则把非空的一方返回给父节点接上；否则去右孩子里找最小值来替换掉自己，这里先用while往右孩子的最左边走，找到最小节点，然后把最小节点的值赋给当前节点，然后在右子树中把最小节点给删掉`root.right = deleteNode(root.right, minNode.val);`。
```java
// BFS删除节点。删除一个节点后BST会动态平衡，找右子树最小（或左子树最大）元素来补位，然后剩下的那一边进行动态平衡调整  
// 需要处理子树之间的关系，所以需要带返回值的dfs，注意父子节点的连接关系。  
// 平衡树那里采用递归实现  
public TreeNode deleteNode(TreeNode root, int key) {  
    // Base Case: 没找到要删除的节点，直接返回 null    if (root == null) return null;  
    // 第一阶段：寻找目标节点  
    if (key < root.val) {  
        // 目标在左边，去左子树删，删完把新左子树接回来  
        root.left = deleteNode(root.left, key);  
    } else if (key > root.val) {  
        // 目标在右边，去右子树删，删完把新右子树接回来  
        root.right = deleteNode(root.right, key);  
    } else {  
        // 第二阶段：找到了目标节点 (root.val == key)，开始执行删除  
  
        // 情况 1 & 2：左孩子为空，或者左右都为空  
        // 直接把右孩子返回给父节点（如果是叶子，root.right 就是 null，刚好符合）  
        if (root.left == null) return root.right;  
        // 情况 2：右孩子为空  
        // 直接把左孩子返回给父节点  
        if (root.right == null) return root.left;  
        // 情况 3：左右孩子都有  
        // 找到右子树的最小值（后继节点）来顶替自己  
        TreeNode rightMin = root.right;  
        while (rightMin != null && rightMin.left != null){  
            rightMin = rightMin.left;  
        }  
        // 替换数值  
        root.val = rightMin.val;  
        // 完美的闭环：去右子树里，把那个刚刚用来顶替的替换节点给删掉  
        root.right = deleteNode(root.right, rightMin.val);  
    }  
    // 回溯返回整理好之后的当前节点给上一层  
    return root;  
}
```

解法2：找到删除元素，且左右孩子都非空，需要将最小的右孩子来顶替的过程中，可以记录parent和孩子，分类讨论（注意这个方法边界情况太多，不如递归删除右侧最小节点巧妙）：
```java
// 情况 3：左右孩子都有  
// 找到右子树的最小值（后继节点）来顶替自己，然后删除掉右子树最小值  
// 这里使用迭代分类讨论法，需要记录parent，删除节点时注意要断开parent的连接  
TreeNode minParent = root;  
TreeNode rightMin = root.right;  
while (rightMin != null && rightMin.left != null){  
    minParent = rightMin;  
    rightMin = rightMin.left;  
}  
// 替换数值  
root.val = rightMin.val;  
  
// 删除rightMin子节点，由于rightMin一定没有左孩子，可能存在右孩子，需要处理它的右孩子  
if (minParent == root) {  
    // rightMin恰好就是root.right的情况，删除rightMin节点就是直接把rightMin的右子树接在root的又孩子上  
    minParent.right = rightMin.right;  
} else {  
    // rightMin在某个左分叉深处，删除rightMin节点就是把rightMin的右子树接在父节点的left上  
    minParent.left = rightMin.right;  
}
```
