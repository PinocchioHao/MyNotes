### 1768 交替合并字符串
[1768. Merge Strings Alternately](https://leetcode.com/problems/merge-strings-alternately/)
两个字符串，按顺序穿插合并成一个新字符串
**Input:** word1 = "abc", word2 = "pqr"
**Output:** "apbqcr"

解法：一个StringBuilder，分别遍历两个字符串，遍历一个输出一个；有两种遍历方法，一个是`i < word1.length() || j < word2.length()`里面用if处理在哪段字符串；另一种是先用与操作`&&`处理完公共部分，然后针对剩下部分给组合进去


### [1071. 字符串的最大公因子](https://leetcode.cn/problems/greatest-common-divisor-of-strings/)
[1071. Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/)
对于字符串 `s` 和 `t`，只有在 `s = t + t + t + ... + t + t`（`t` 自身连接 1 次或多次）时，我们才认定 “`t` 能除尽 `s`”。
给定两个字符串 `str1` 和 `str2` 。返回 _最长字符串 `x`，要求满足 `x` 能除尽 `str1` 且 `x` 能除尽 `str2`_ 。
**示例 2：**
**输入：**str1 = "ABABAB", str2 = "ABAB"
**输出：**"AB"

解法1：暴力法。先定义一个计算sub是否是target的因子的函数，target.length()/sub.length()应该为整数，然后判断str2循环这么多次后是否等于str1。然后外层暴力从str1和str2中更短的一方开始，依次往短遍历，看是否是另一方的因子。（这里依次遍历可以再优化下，子串长度如果是因子才继续判断）

解法2：gcd算法。如果两个字符串有公因子，那么`str1+str2==str2+str1`，基于这个条件再用gcd算法算出两个子串长度的最大公因子进行截取即可。
```java
   // 核心思想就是把原来的除数给 a，把余数给 b，直到余数为 0    
   // 例如：a = 48, b = 18  
    // 1. 48 % 18 = 12, a = 18, b = 12    
    // 2. 18 % 12 = 6, a = 12, b = 6    
    // 3. 12 % 6 = 0, a = 6, b = 0    
    // 当 b 变成 0 时，当前的 a 就是最后一个能整除的数，即最大公约数  
    public static int gcd(int a, int b) {  
        // 迭代写法：  
        // 只要余数 b 不为 0，就一直算下去  
        while (b != 0) {  
            int remainder = a % b; // 1. 先求出余数  
            a = b;                 // 2. 把除数挪给 a，作为下一轮的被除数  
            b = remainder;         // 3. 把余数挪给 b，作为下一轮的除数  
        }  
        // 除到最后，当 b 变成 0 时，当前的 a 就是最后一个能整除的数，即最大公约数  
        return a;  
  
        // 递归写法：  
//        if (b == 0) return a;  
//        return gcd(b, a % b);  
  
    }
```


### [1431. 拥有最多糖果的孩子](https://leetcode.cn/problems/kids-with-the-greatest-number-of-candies/)
[1431. Kids With the Greatest Number of Candies](https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/)
有 `n` 个有糖果的孩子。给你一个数组 `candies`，其中 `candies[i]` 代表第 `i` 个孩子拥有的糖果数目，和一个整数 `extraCandies` 表示你所有的额外糖果的数量。

返回一个长度为 `n` 的布尔数组 `result`，如果把所有的 `extraCandies` 给第 `i` 个孩子之后，他会拥有所有孩子中 **最多** 的糖果，那么 `result[i]` 为 `true`，否则为 `false`。

注意，允许有多个孩子同时拥有 **最多** 的糖果数目。

解法1：暴力法O(n2)。双层循环，每遍历到一个元素的位置，就再遍历一遍数组，判断剩下每一个人本身的candies跟当前人candies+extraCandies的值。
解法2：贪心，分两次遍历数组O(n)。第一次找到最大值maxCandies，第二次再遍历看当前位置的candies+extraCandies跟maxCandies比较，大于即可。

### 605 
[605. Can Place Flowers](https://leetcode.com/problems/can-place-flowers/)
You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.
Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` _if_ `n` _new flowers can be planted in the_ `flowerbed` _without violating the no-adjacent-flowers rule and_ `false` _otherwise_.
**Example 1:**
**Input:** flowerbed = [1,0,0,0,1], n = 1
**Output:** true

**Example 2:**
**Input:** flowerbed = [1,0,0,0,1], n = 2
**Output:** false

解法：贪心。当前位置能种花的前提是d[i-1], d[i], d[i+1]都为0（注意这里要单独讨论首尾处），求得传入的flowerbed[]数组最多能种的花max，比较max跟n的值即可。


### 345 反转字符串中的元音
[345. Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/)

Given a string `s`, reverse only all the vowels in the string and return it.
The vowels are `'a'`, `'e'`, `'i'`, `'o'`, and `'u'`, and they can appear in both lower and upper cases, more than once.
**Example 1:**
**Input:** s = "IceCreAm"
**Output:** "AceCreIm"
**Explanation:**
The vowels in `s` are `['I', 'e', 'e', 'A']`. On reversing the vowels, s becomes `"AceCreIm"`.

解法1：栈，分两次遍历。第一次遍历遇到元音就入栈，第二次遍历组成字符串。

解法2：双指针。左右指针分别从字符串开头和结尾往中间收缩，找到元音就交换元素的位置，否则继续遍历。


### 151 反转一句话中的单词
[151. Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)
Given an input string `s`, reverse the order of the **words**.
A **word** is defined as a sequence of non-space characters. The **words** in `s` will be separated by at least one space.
Return _a string of the words in reverse order concatenated by a single space._
**Note** that `s` may contain leading or trailing spaces or multiple spaces between two words. The returned string should only have a single space separating the words. Do not include any extra spaces.

**Example 1:**
**Input:** s = "the sky is blue"
**Output:** "blue is sky the"

解法1：Java API。利用正则匹配`\\s+`把s拆分成单词数组，然后把数组给反序，最后join加上空格再trim一下去除多余空格。
```java
List<String> list = new ArrayList();  
// split方法的参数是一个正则表达式，\\s+表示匹配一个或多个空格，这样可以处理多个连续的空格情况。  
list.addAll(Arrays.asList(s.split("\\s+")));  
Collections.reverse(list);  
return String.join(" ", list).trim();
```

解法2：双指针。双指针同侧出发，一个找单词头部，找到头部后另一个指针找尾部，然后char[left, right-left]就是单词。如果双指针正向遍历的话可以把每个单词加入到Deque最后输出就是逆序的；如果双指针从尾部往前遍历则可以直接用string builder输出。


### 238数组除自己以外的乘积
[238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)
Given an integer array `nums`, return _an array_ `answer` _such that_ `answer[i]` _is equal to the product of all the elements of_ `nums` _except_ `nums[i]`.
The product of any prefix or suffix of `nums` is **guaranteed** to fit in a **32-bit** integer.
You must write an algorithm that runs in `O(n)` time and without using the division operation.

**Example 1:**
**Input:** nums = [1,2,3,4]
**Output:** [24,12,8,6]

解法：除自己以外的乘积=当前位置左侧乘积 x 当前位置右侧乘积。所以可以从左右两边分别遍历一次求该位置左侧和右侧乘积，然后再遍历一次计算乘积。经过改进可以在第二次遍历的时候同时把结果给算出来，数组也可以进行复用来降低开销。


### 334 递增三元子序列
[334. Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/)

解法1：从左往右和从右往左遍历两次数组，时间开销O(2n), 空间开销O(n)。分别记录当前位置左边最小元素在数组中，当前位置右边最大元素，遍历第二次的时候判断当前位置的元素是否大于左边最小右边最大。遍历完没找到则返回false。注意边界用例数组小于3。

解法2：维护两个变量min和second，顺序遍历，利用if-else以及遍历过程中下标不断增大的特性。时间开销O(n)，空间开销O(1)
```java
if (nums[i] <= min) {
	min = nums[i];
} else if (nums[i] <= second){
	second = nums[i];
}
else {
	// 当前元素比second还大就fanh
	return true;
}
```

### 443 压缩字符串
Given an array of characters `chars`, compress it using the following algorithm:
Begin with an empty string `s`. For each group of **consecutive repeating characters** in `chars`:
- If the group's length is `1`, append the character to `s`.
- Otherwise, append the character followed by the group's length.
The compressed string `s` **should not be returned separately**, but instead, be stored **in the input character array `chars`**. Note that group lengths that are `10` or longer will be split into multiple characters in `chars`.
After you are done **modifying the input array,** return _the new length of the array_.
You must write an algorithm that uses only **constant extra space.**
**Note:** The characters in the array beyond the returned length do not matter and should be ignored.

**Example 1:**
**Input:** chars = ["a","a","b","b","c","c","c"]
**Output:** 6
**Explanation:** The groups are "aa", "bb", and "ccc". This compresses to "a2b2c3".

思路1：遍历数组判断相邻元素并计数，组成新字符串。但是这种方法需要额外O(n)的空间开销，不符合题意。

解法2：快慢指针（读写指针）。读指针用于遍历数组，计数，当读指针遍历完当前位置后，则写指针把读指针之前读的字母以及遍历完字母的计数写入当前位置，注意需要把count给转为char，以及write指针的移动细节。