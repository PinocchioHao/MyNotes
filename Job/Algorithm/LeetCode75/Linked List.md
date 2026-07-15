技巧：`dummy.next` 存的是一个“地址（坐标）”，而不是变量名 `even`。所以`dummy.next`来标记表头是很稳妥的一个方法

### 删除链表中间节点
[2095. Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/)
Medium
You are given the `head` of a linked list. **Delete** the **middle node**, and return _the_ `head` _of the modified linked list_.
The **middle node** of a linked list of size `n` is the `⌊n / 2⌋th` node from the **start** using **0-based indexing**, where `⌊x⌋` denotes the largest integer less than or equal to `x`.
- For `n` = `1`, `2`, `3`, `4`, and `5`, the middle nodes are `0`, `1`, `1`, `2`, and `2`, respectively.

**Input:** head = [1,3,4,7,1,2,6]
**Output:** [1,3,4,1,2,6]
**Explanation:**
The above figure represents the given linked list. The indices of the nodes are written below.
Since n = 7, node 3 with value 7 is the middle node, which is marked in red.
We return the new list after removing this node.

解法：快慢指针。快指针每次走两步，慢指针每次走一步，快指针走到结尾处慢指针刚好到中间，维护一个前一元素的指针，将`pre.next = slow.next`，最后返回head即可。



### [328. 奇偶链表](https://leetcode.cn/problems/odd-even-linked-list/)
中等
[328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/)
给定单链表的头节点 `head` ，将所有索引为奇数的节点和索引为偶数的节点分别分组，保持它们原有的相对顺序，然后把偶数索引节点分组连接到奇数索引节点分组之后，返回重新排序的链表。
**第一个**节点的索引被认为是 **奇数** ， **第二个**节点的索引为 **偶数** ，以此类推。
请注意，偶数组和奇数组内部的相对顺序应该与输入时保持一致。
你必须在 `O(1)` 的额外空间复杂度和 `O(n)` 的时间复杂度下解决这个问题。

**Input:** head = [1,2,3,4,5]
**Output:** [1,3,5,2,4]

解法：维护odd和even两个链表，遍历整个链表，odd和even分别穿插下标为奇数和偶数的元素，最后将odd.next指向even即可（此前需要设置dummy.next = even来记录even的头结点）


### [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
Easy
Given the `head` of a singly linked list, reverse the list, and return _the reversed list_.
**Input:** head = [1,2,3,4,5]
**Output:** [5,4,3,2,1]

解法：初始化一个pre为null，遍历链表，记录next，cur，pre，交换指针即可。


### [2130. 链表最大孪生和](https://leetcode.cn/problems/maximum-twin-sum-of-a-linked-list/)
[2130. Maximum Twin Sum of a Linked List](https://leetcode.com/problems/maximum-twin-sum-of-a-linked-list/)
中等
在一个大小为 `n` 且 `n` 为 **偶数** 的链表中，对于 `0 <= i <= (n / 2) - 1` 的 `i` ，第 `i` 个节点（下标从 **0** 开始）的孪生节点为第 `(n-1-i)` 个节点 。
- 比方说，`n = 4` 那么节点 `0` 是节点 `3` 的孪生节点，节点 `1` 是节点 `2` 的孪生节点。这是长度为 `n = 4` 的链表中所有的孪生节点。
**孪生和** 定义为一个节点和它孪生节点两者值之和。
给你一个长度为偶数的链表的头节点 `head` ，请你返回链表的 **最大孪生和** 。

**Input:** head = [4,2,2,3]
**Output:** 7
**Explanation:**
The nodes with twins present in this linked list are:
- Node 0 is the twin of node 3 having a twin sum of 4 + 3 = 7.
- Node 1 is the twin of node 2 having a twin sum of 2 + 2 = 4.
Thus, the maximum twin sum of the linked list is max(7, 4) = 7.

解法1：快慢指针找中点，前半部分值可以压入栈，后半段按顺序遍历跟栈中的值弹出相加找最大值即可，这样会额外产生O(n/2)空间开销

解法2：依旧快慢指针，slow在遍历前半部分链表时可以顺手把链表反转，fast走完之后slow恰好位于下半部分的头指针，pre恰好位于反转后的前半部分的头指针，再遍历一半链表pre和slow相加找最大值即可