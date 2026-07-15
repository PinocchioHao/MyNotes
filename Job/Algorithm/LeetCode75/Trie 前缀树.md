### 一、 物理结构与底层逻辑

字典树是一种典型的“空间换时间”的多叉树数据结构。一般用于工业场景的搜索引擎。

1. **节点（Node）不存字符：** 节点仅作为状态机，主要存储两个信息：
    
    - `isEnd`（布尔值）：标记从根节点走到当前节点，是否构成了一个完整的单词。
        
    - `children`（数组或哈希表）：指向下一层子节点的指针集合。
        
2. **边（Edge）代表字符：** 字符隐式地存在于父节点指向子节点的“路径/索引”上。
    
3. **根节点（Root）：** 代表空字符串 `""`，是所有单词的绝对公共前缀起点。
    

如果采用节点存值，那么查某个字符需要遍历所有children，确定这个children的值之后再进行下一步，开销极大，所以采取路径存值，节点只做状态记录，要走哪条路通过HashMap走；

### 二、 核心特征与性能对比

#### 1. 核心优势

- **极致的查询速度：** 无论树中有多少个单词，查询一个长度为 $L$ 的单词，时间复杂度永远是严格的 $O(L)$。
    
- **前缀共享：** 拥有相同前缀的单词会共享同一条祖先路径，物理上极大地压缩了重复前缀占用的空间。
    
- **有序性：** 如果按照字母顺序遍历（如 DFS 遍历 `children` 数组），字典树天然能以字典序输出所有单词。
    

#### 2. 致命劣势

- **内存消耗巨大：** 如果单词之间毫无公共前缀，或者树极其稀疏，每个节点都要开辟长度为 26 的数组或哈希表，指针占用的内存会远超字符本身。

结构：
![[Pasted image 20260706190126.png]]

模板：
```java
// 类自身包装成一个节点，孩子用Map来存，一般树的通解  
class Trie {  
    // 此时 Trie 类本身就是一个节点！  
    boolean isEnd;
    // 如果只存单词的话可以考虑Trie[26] children;
    Map<Character, Trie> children;  
  
    // 初始化节点  
    public Trie() {  
        isEnd = false;  
        children = new HashMap<>();  
    }  
  
    // ==========================================  
    // 插入单词  
    // ==========================================  
    public void insert(String word) {  
        Trie node = this; // this 就是当前节点（最开始调用时就是根节点）  
  
        for (char c : word.toCharArray()) {  
            // 【极其优雅的一行代码】：如果 map 里没有这个字符，就 new 一个新的 Trie 节点放进去  
            node.children.putIfAbsent(c, new Trie());  
  
            // 指针往下跳  
            node = node.children.get(c);  
        }  
        node.isEnd = true; // 插入完后到达的节点打上结束标记  
    }  
  
    // ==========================================  
    // 搜索完整单词  
    // ==========================================  
    public boolean search(String word) {  
        Trie node = this;  
  
        for (char c : word.toCharArray()) {  
            // 如果 map 里找不着这个字符，说明路断了  
            node = node.children.get(c);  
            if (node == null) {  
                return false;  
            }  
        }  
        // 走完了，看看有没有结束标记  
        return node.isEnd;  
    }  
  
    // ==========================================  
    // 搜索前缀  
    // ==========================================  
    public boolean startsWith(String prefix) {  
        Trie node = this;  
  
        for (char c : prefix.toCharArray()) {  
            node = node.children.get(c);  
            if (node == null) {  
                return false; // 前缀断了，直接返回 false            }  
        }  
        return true; // 能顺利走完，说明前缀存在  
    }  
}
```


---

### [208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)
[208. Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)


题解：参考标准前缀树模板。


---

### [1268. 搜索推荐系统](https://leetcode.cn/problems/search-suggestions-system/)
[1268. Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/)
给你一个产品数组 `products` 和一个字符串 `searchWord` ，`products`  数组中每个产品都是一个字符串。
请你设计一个推荐系统，在依次输入单词 `searchWord` 的每一个字母后，推荐 `products` 数组中前缀与 `searchWord` 相同的最多三个产品。如果前缀相同的可推荐产品超过三个，请按字典序返回最小的三个。
请你以二维列表的形式，返回在输入 `searchWord` 每个字母后相应的推荐产品的列表。
**示例 1：**
```
输入：products = ["mobile","mouse","moneypot","monitor","mousepad"], searchWord = "mouse"
输出：[
["mobile","moneypot","monitor"],
["mobile","moneypot","monitor"],
["mouse","mousepad"],
["mouse","mousepad"],
["mouse","mousepad"]
]
解释：按字典序排序后的产品列表是 ["mobile","moneypot","monitor","mouse","mousepad"]
输入 m 和 mo，由于所有产品的前缀都相同，所以系统返回字典序最小的三个产品 ["mobile","moneypot","monitor"]
输入 mou， mous 和 mouse 后系统都返回 ["mouse","mousepad"]
```

题解1：暴力搜索。先排序满足题目要求，找到searchWord每一个前缀在products里面出现的第一个位置（从前往后遍历第一个满足`startsWith`的即可，找不到就返回`products.length`方便外层处理；注意这里可以用二分查找优化性能，注意比较的时候用`if (products[mid].compareTo(prefix) >= 0)`而不能使用`startsWith`，因为可能落在中间部分），然后再向后找最多3个相同前缀的字符串。

题解2：双指针。先排序满足题目要求。left和right指针分别代表products数组中的左右。遍历每层前序，在内层while循环操作双指针的元素，它们的第i位应该也相同，如果不匹配则指针收缩，注意指针不回头，收缩完后即满足条件的候选者，从这里面选left靠后的3个，注意加约束条件，也注意不要额外操纵指针。
```java
// 左右指针必须放外层，因为要记忆每一步的操作，不能回头，放内层会只判断当前位置的字符，忘记前面收缩的空间，出问题  
int left = 0;  
int right = products.length - 1;  
for (int i = 0; i < searchWord.length(); i++) {  
    // 跑到左边第一个找到searchWork[i]的位置，如果长度比当前searchWord的遍历长度短或者不匹配则直接跳过  
    while (left <= right && (products[left].length() < i + 1 || products[left].charAt(i) != searchWord.charAt(i))) {  
        left++;  
    }  
    // 从右边找同理  
    while (left <= right && (products[right].length() < i + 1 || products[right].charAt(i) != searchWord.charAt(i))) {  
        right--;  
    }  
  
    // 经历过这两层while后，[left,right]区间就是符合的字符，选前3个打印  
    List<String> suggest = new ArrayList<>();  
    for (int j = 0; j < 3; j++) {  
        // 这段代码有问题，不能操作left和right指针，因为要记忆到下一步，所以这一步的区间不能动  
        // if (left <= right){  
        //     suggest.add(products[left]);        //     left++;        // }        // 反而用j和left+j来表示区间内的数  
        if (left + j <= right) {  
            suggest.add(products[left + j]);  
        }  
  
    }  
    res.add(suggest);  
}
```


题解3：字典树。构造字典树，额外加一个`List<String> top3`变量用于在节点维护遍历到当前路径的前三候选词。先排序以满足题目要求。然后将products数组所有词插入前缀树中，对每个节点如果它的top3数组size不足3，则把products当前词添加进去。最后根据searchWord逐步遍历前缀树，取出每个节点的top3数组的值进结果集即可。再最后手动补齐没找到的空数组。
```java
public List<List<String>> suggestedProducts1(String[] products, String searchWord) {  
    // 先按字典序排序  
    Arrays.sort(products);  
    TrieNode root = new TrieNode();  
    // 存数进前缀树  
    for (String product : products) {  
        TrieNode node = root;  
        for (char c : product.toCharArray()) {  
            int index = c - 'a';  
            if (node.children[index] == null) {  
                node.children[index] = new TrieNode();  
            }  
            node = node.children[index];  
  
            // 每个node会有3个产品推荐，存最先遍历到node代表的字符的3个product，而product已经按照字典排序了，所以符合要求  
            if (node.top3.size() < 3) {  
                node.top3.add(product);  
            }  
        }  
    }  
  
    List<List<String>> res = new ArrayList<>();  
    // 遍历前缀树取值，取到前缀树节点的top3数组即可  
    for (char c : searchWord.toCharArray()) {  
        int index = c - 'a';  
        root = root.children[index];  
        if (root != null) {  
            res.add(new ArrayList<>(root.top3));  
        } else {  
            break;  
        }  
    }  
  
    // 手动补齐没找到的空数组  
    for (int i = res.size(); i < searchWord.length(); i++) {  
        res.add(new ArrayList<>());  
    }  
  
    return res;  
  
}
```