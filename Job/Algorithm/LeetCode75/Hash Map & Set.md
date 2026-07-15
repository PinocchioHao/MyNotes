### [2215. 找出两数组的不同](https://leetcode.cn/problems/find-the-difference-of-two-arrays/)
[2215. Find the Difference of Two Arrays](https://leetcode.com/problems/find-the-difference-of-two-arrays/)
给你两个下标从 `0` 开始的整数数组 `nums1` 和 `nums2` ，请你返回一个长度为 `2` 的列表 `answer` ，其中：
- `answer[0]` 是 `nums1` 中所有 **不** 存在于 `nums2` 中的 **不同** 整数组成的列表。
- `answer[1]` 是 `nums2` 中所有 **不** 存在于 `nums1` 中的 **不同** 整数组成的列表。
**注意：**列表中的整数可以按 **任意** 顺序返回。
```
**Example 2:**
**Input:** nums1 = [1,2,3,3], nums2 = [1,1,2,2]
**Output:** [[3],[]]
**Explanation:**
For nums1, nums1[2] and nums1[3] are not present in nums2. Since nums1[2] == nums1[3], their value is only included once and answer[0] = [3].
Every integer in nums2 is present in nums1. Therefore, answer[1] = [].
```

解法：把两个数组的元素放到Set中，使用`Set.contains()`判断元素是否在另一个set中出现过。本题可以用stream几行搞定，但是效率不如手搓for高。

### [1207. 独一无二的出现次数](https://leetcode.cn/problems/unique-number-of-occurrences/)
[1207. Unique Number of Occurrences](https://leetcode.com/problems/unique-number-of-occurrences/)
Given an array of integers `arr`, return `true` _if the number of occurrences of each value in the array is **unique** or_ `false` _otherwise_.

**Example 1:**
**Input:** arr = [1,2,2,1,1,3]
**Output:** true
**Explanation:** The value 1 has 3 occurrences, 2 has 2 and 3 has 1. No two values have the same number of occurrences.

解法1：HashMap+Set。遍历第一次将数和出现次数记录进HashMap中，第二次遍历HashMap的values集合，将其放入Set中，每次放入前检查Set中是否已存在，存在相同次数则直接返回false，遍历完则返回true

解法2：使用数组模拟统计次数，由于题目给定了条件`-1000 <= arr[i] <= 1000`，所以可以使用一个`int[2001]`来存下所有数的出现次数，之后用set检查相同次数即可，注意初始化这些元素有很多0，需要排除。


### [1657. 确定两个字符串是否接近](https://leetcode.cn/problems/determine-if-two-strings-are-close/)
[1657. Determine if Two Strings Are Close](https://leetcode.com/problems/determine-if-two-strings-are-close/)
如果可以使用以下操作从一个字符串得到另一个字符串，则认为两个字符串 **接近** ：
- 操作 1：交换任意两个 **现有** 字符。
    - 例如，`abcde -> aecdb`
- 操作 2：将一个 **现有** 字符的每次出现转换为另一个 **现有** 字符，并对另一个字符执行相同的操作。
    - 例如，`aacabb -> bbcbaa`（所有 `a` 转化为 `b` ，而所有的 `b` 转换为 `a` ）
你可以根据需要对任意一个字符串多次使用这两种操作。
给你两个字符串，`word1` 和 `word2` 。如果 `word1` 和 `word2` **接近** ，就返回 `true` ；否则，返回 `false` 。
**Example 1:**
**Input:** word1 = "cabbba", word2 = "abbccc"
**Output:** true
**Explanation:** You can attain word2 from word1 in 3 operations.
Apply Operation 1: "cabbba" -> "caabbb"
Apply Operation 2: "caabbb" -> "baaccc"
Apply Operation 2: "baaccc" -> "abbccc"


思路：如果两个字符串的字符种类一样，字符出现的总体频率一样，则这两个字符串相近，因为字符串总能通过操作变换成另一个。

解法：两个Map分别存储两个字符串的字符和出现频率，检查这两个map的key和value是否相等即可，检查key可以用`map1.keySet().equeals(map2.keySet())`检查value可以转换成数组排序后用`Arrays.equals()`检查。优化：由于元素只有26个字母，所以可以用`int[26]`模拟Map，`for (char c : word1.toCharArray()) count1[c - 'a']++`添加元素，之后排序后判断两数组相等即可。



### [2352. 相等行列对](https://leetcode.cn/problems/equal-row-and-column-pairs/)
[2352. Equal Row and Column Pairs](https://leetcode.com/problems/equal-row-and-column-pairs/)
给你一个下标从 **0** 开始、大小为 `n x n` 的整数矩阵 `grid` ，返回满足 `Ri` 行和 `Cj` 列相等的行列对 `(Ri, Cj)` 的数目_。_
如果行和列以相同的顺序包含相同的元素（即相等的数组），则认为二者是相等的。

**示例 1：**
**Input:** grid = [[3,1,2,2],[1,4,4,5],[2,4,2,2],[2,4,2,2]]
**Output:** 3
**Explanation:** There are 3 equal row and column pairs:
- (Row 0, Column 0): [3,1,2,2]
- (Row 2, Column 2): [2,4,2,2]
- (Row 3, Column 2): [2,4,2,2]

解法：分别按行和列遍历，将行组成的数组和列组成的数组以及其出现次数分别存入Map中，找出相同的行列数组并计数`count += rowMap.getOrDefault(colList, 0);` 第二次遍历的时候可以边遍历边检查，省下一个Map以及一次遍历。