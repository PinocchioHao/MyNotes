所有回溯问题，都可以抽象为一棵**N 叉树**：
- **树的深度（纵向）：** 决定了递归的层数（往往对应你要找的组合长度 `k`，或者字符串长度）。
- **树的宽度（横向）：** 决定了 `for` 循环的次数（当前层有多少个选项可以选）。
    
**万能模板：**

```java
// 绝大多数回溯函数返回类型为 void，靠操作传入的全局变量 res 和 path 来记录结果，还需要传整体数据集以及当前位置
public void backtrack(参数) { 
    // 1. 终点拦截（到达树的叶子节点）
    if (终止条件（如 path.size() == k 或 index == str.length） ) {
        // 【天坑注意】：必须 new 一个新对象存入结果集！
        res.add(new ArrayList<>(path)); 
        return;
    }

    // 2. 遍历当前层的所有选择
    for (选择 : 本层集合中元素) {
        // 剪枝操作（可选）：如果发现当前选择肯定没戏，直接 continue 或 break
        if (不合法或没必要) continue; 

        // 3. 做选择（记录路径、扣减目标值、更新状态）
        path.add(选择);

        // 4. 递归下探（进入下一层）
        // 注意参数的变化：比如 startIdx + 1，或者 target - nums[i]
        backtrack(路径，选择列表); 

        // 5. 撤销选择（回溯核心：恢复现场！）
        // 刚才加进去的元素已经穷尽了它的所有可能，必须踢掉，腾出位置给下一个兄弟节点
        path.remove(path.size() - 1); 
    }
}
```

相关题型：排列问题、组合问题、子集问题。

|**门派题型**|**核心特征**|**for 循环起点控制**|**结果收集时机**|
|---|---|---|---|
|**组合 (Combinations)**|顺序无关。`[1, 2]` 等同于 `[2, 1]`。|传入 `startIdx`。下一层从 `i + 1` 开始，坚决不回头。|到达叶子节点（满足指定长度或目标和）。|
|**子集 (Subsets)**|收集所有节点。可以看作是无限制条件的组合。|传入 `startIdx`。下一层从 `i + 1` 开始。|树的**每一个节点**都要收集（通常在函数第一行直接 add）。|
|**排列 (Permutations)**|顺序相关。`[1, 2]` 不同于 `[2, 1]`。|每次都从 `0` 开始！需要 `boolean[] used` 数组跳过已选元素。|到达叶子节点（路径长度等于原数组长度）。|

---

### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)
[17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。
给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![](https://pic.leetcode.cn/1752723054-mfIHZs-image.png)

**示例 1：**
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]


题解1：自底向上递归。当前位置的结果是前i-1个组合串和第i个字符的组合体。conbine函数负责将之前组成的字符串数组跟剥离出来的这位字符进行笛卡尔式组合。
```java
public List<String> dfs(String digits) {  
    // 递归终止条件：字符串已经被切完了，返回一个空集合作为初始的“地基”  
    if (digits.length() == 0){  
        return new ArrayList<>();  
    }  
    int len = digits.length();  
    // 剥离出当前最后一个字符  
    char ch = digits.charAt(len - 1);  
    // 核心递归：先去求解前面部分的组合结果，等前面算完了，再和当前字符 ch 进行组合  
    return combine(dfs(digits.substring(0, len - 1)), ch);  
}
```


题解2：回溯。回溯树每层为输入的数字对应的字符，深度为输入的数字的个数，path可采用StringBuilder或者String，当path长度为输入的数字数时停止，深拷贝记录path到结果集中return掉。遍历每层时将当前层元素添加到path中，执行backtrack函数进下一层（index+1），执行完后`path.deleteCharAt(path.length() - 1);`进行回溯。

---
### [216. 组合总和 III](https://leetcode.cn/problems/combination-sum-iii/)
[216. Combination Sum III](https://leetcode.com/problems/combination-sum-iii/)
找出所有相加之和为 `n` 的 `k` 个数的组合，且满足下列条件：
- 只使用数字1到9
- 每个数字 **最多使用一次** 
返回 _所有可能的有效组合的列表_ 。该列表不能包含相同的组合两次，组合可以以任何顺序返回。

**示例 2:**
```
输入: k = 3, n = 9
输出: [[1,2,6], [1,3,5], [2,3,4]]
解释:
1 + 2 + 6 = 9
1 + 3 + 5 = 9
2 + 3 + 4 = 9
没有其他符合的组合了。
```


题解：回溯法。回溯函数参数当前元素curr， 层数k，目标数tar，路径path和结果集rlt。当纵向遍历完决策树即`path.size==k`时判断tar是否为0，是则说明这条路径相加恰好为n。决策树每层元素为curr到9，for循环开始时可以进行剪枝操作，然后将每层元素将i加入到path中，调用回溯函数进入下一层，此时传参curr传`i+1`，tar传`tar - i`，回溯后`path.removeLast()`。