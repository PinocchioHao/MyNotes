#### OA中进行打印
##### 1. 数组与二维数组（最易踩坑）

- **普通一维数组**：直冲 `Arrays.toString(nums)`。
    
- **二维数组（如本题的 `pairs`）**：千万别写 `Arrays.toString()`，那会打印出一堆内存地址。必须用 **`Arrays.deepToString(pairs)`**。
    

##### 2. 集合类（List, Set, Queue/PriorityQueue）

Java 的 `List`、`Set` 以及 `PriorityQueue` 都已经原生重写了 `toString()` 方法。

- 直接写 `System.out.println(heap);` 或 `System.out.println(list);` 就能完美输出 `[1, 2, 3]` 的精美格式。
    
- **注意**：`PriorityQueue` 的 `toString()` 打印出来的是它在内存里的物理二叉树顺序，并不是严格的从小到大排序，但用来观察里面的元素有没有丢，足够了。
    

##### 3. 双层映射（Map）

`Map` 也原生支持 `toString()`。如果你用了 `Map<String, Map<String, Double>>`（如 399 题），直接 `System.out.println(graph);` 就会输出形如 `{a={b=2.0}, b={c=3.0}}` 的嵌套结构。



#### switch的坑
- 当 `switch` 匹配到某个 `case` 之后，  
    **不会再去判断后面的 case 条件**。
    
- 而是从这个 `case` 开始，把后面的语句一股脑儿顺序执行下去。
    
- 除非遇到 `break`（跳出整个 `switch`）、`continue`（跳到下一轮循环）、`return`（直接结束方法）、或者抛出异常，才会停。

```java

// 输出 two  three  default
int x = 2;
switch (x) {
    case 1:
        System.out.println("one");
    case 2:
        System.out.println("two");
    case 3:
        System.out.println("three");
    default:
        System.out.println("default");
}
```




####  **Queue 队列**

常用来做 **BFS**。

**声明方式：**
```
Queue<TreeNode> q = new LinkedList<>();   // 最常见 
Queue<Integer> q2 = new ArrayDeque<>();   // 也常见，比 LinkedList 性能更好
```


**常用方法：**

- `q.add(x)` / `q.offer(x)` → 入队
    
- `q.poll()` → 出队（队空返回 `null`）
    
- `q.peek()` → 查看队头（不删除）
    
- `q.isEmpty()` / `q.size()`
    

---

#### **Deque 双端队列**

常用来做 **栈、单调队列、滑动窗口、双端 BFS**。

**声明方式：**

`Deque<Integer> dq = new LinkedList<>(); `
`Deque<Integer> dq2 = new ArrayDeque<>();`

**常用方法：**

- `addFirst(x)` / `addLast(x)`
    
- `pollFirst()` / `pollLast()`
    
- `peekFirst()` / `peekLast()`
    

👉 特别适合写 **单调队列、回文检查** 之类题。


相比于Stack，Java官方更推荐使用Deque，因为Stack继承自Vector被过度设计了，而且自带同步锁开销，性能不够高。

**而且Deque的场景能完全覆盖Stack和Queue的场景，既能当队列又能当栈使用。**


---

#### **Stack 栈**

常用来做 **DFS、括号匹配、单调栈**。
注意不用非得使用Stack来实现栈，如果数据允许的话可以使用Deque，数组（设计一个栈顶指针操控），甚至StringBuilder这种能够添加元素以及删除元素的数据结构作为”栈“来使用。


**声明方式：**

`Stack<Integer> st = new Stack<>();`

（也可以用 `Deque` 实现栈，推荐 `Deque<Integer> st = new ArrayDeque<>();`）

**常用方法：**

- `push(x)` → 入栈
    
- `pop()` → 出栈（返回栈顶元素）
    
- `peek()` → 查看栈顶元素（不删除）
    
- `isEmpty()` / `size()`
    

---

#### **Set 集合**

常用来做 **去重、快速查找**。

**声明方式：**

`Set<Integer> set = new HashSet<>();   // 最常见`
`Set<Integer> treeSet = new TreeSet<>(); // 自动排序`

**常用方法：**

- `set.add(x)`
    
- `set.remove(x)`
    
- `set.contains(x)`
    
- `set.isEmpty()` / `set.size()`
    
- 遍历：`for (int x : set) {...}`
    

---

#### **Map 映射**

常用来做 **计数、映射关系**。

**声明方式：**

`Map<Integer, Integer> map = new HashMap<>();`
`Map<Integer, Integer> treeMap = new TreeMap<>(); // 自动排序`

**常用方法：**

- `map.put(k, v)`
    
- `map.get(k)`（如果不存在，返回 `null`）
    
- `map.getOrDefault(k, defaultVal)`
    
- `map.containsKey(k)` / `map.containsValue(v)`
    
- `map.remove(k)`
    
- 遍历：
    
    `for (Map.Entry<Integer, Integer> e : map.entrySet()) {     int k = e.getKey(), v = e.getValue(); }`


`map.computeIfAbsent(u, k -> new ArrayList<>()).add(v);` 先去 map 里找 `u`，如果没找到，就执行后面的 Lambda 表达式 `new ArrayList<>()` 并且放进 map 里，**最后把这个 List 的引用返回**。常用于优雅建图。


``` java
// 合并并计数元素
Map<Integer, Integer> map = new HashMap();
for (int num : arr){
   map.put(c, map.getOrDefault(c, 0) + 1);
}

// 新特性优雅写法
map.merge(c, 1, Integer::sum); // 略微比上面那种方法慢

// stream 统计频率：将数组转为 Map<元素, 出现次数>  这种写法会造成不必要的开箱装箱操作，性能大幅下降，刷题不建议使用
Map<Integer, Long> counts = Arrays.stream(nums)
    .boxed() // int 转 Integer，因为 collect 不支持原始类型流
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()));


// 优雅建图
graph.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
// 等同于
if (!graph.containsKey(u)) {
    graph.put(u, new ArrayList<>());
}
graph.get(u).add(v);

```


#### TreeMap
排序map，底层红黑树，会**自动按照 Key 的大小顺序（自然顺序，或者你指定的 Comparator）在树上排好位置**。
- **代价：** 它的增删改查时间复杂度都是 $O(\log N)$，比 `HashMap` 慢。
- **收益：** 它不仅能告诉你“某个 Key 存不存在”，还能回答“**比这个 Key 稍微大一点的元素是谁？稍微小一点的元素是谁？**”

- **`floorKey(K key)`：向下取整查找（最常用！）**
    - **逻辑：** 找 $\le$ 目标值的**最大**的 Key。类似于《价格猜猜猜》里“最接近但不超过”的规则。
    - **举例：** `map.floorKey(25)` $\rightarrow$ 返回 `20`。`map.floorKey(30)` $\rightarrow$ 返回 `30`。
- **`ceilingKey(K key)`：向上取整查找**
    - **逻辑：** 找 $\ge$ 目标值的**最小**的 Key。
    - **举例：** `map.ceilingKey(25)` $\rightarrow$ 返回 `30`。
- **`lowerKey(K key)` 和 `higherKey(K key)`**
    - **逻辑：** 和上面类似，但它是**严格的大于/小于**（不包含等于）。
    - **举例：** `map.lowerKey(30)` $\rightarrow$ 返回 `20`。
        
此外，它还能像 PriorityQueue 一样操作首尾：

- **`firstKey()` / `lastKey()`：** 瞬间拿到当前树里最小或最大的 Key。
    
- **`pollFirstEntry()` / `pollLastEntry()`：** 弹出并删除最小或最大的元素。
    

### 3. 企业级实战与算法应用场景

既然它的核心能力是“**动态维持有序 + 模糊查找**”，它的应用场景就非常明确了：

#### A. 算法领域：动态区间合并 / 贪心匹配

这就是为什么昨天那道 SIG 的区间题最优解是 `TreeMap`。

当数据像流水一样一个个进来时，比如新来一个区间 `[5, 8]`。你可以直接用 `floorKey(8 + 1)` 瞬间在茫茫数据中揪出“左边界 $\le 9$”的那个区间，判断它们是否重合。不需要遍历，只需 $O(\log N)$。



#### Heap & Priority Queue
一般用于动态维护值，求第k大小的元素。
- **大顶堆（Max-Heap）**：堆顶永远是当前整棵树里**最大**的元素。
    
- **小顶堆（Min-Heap）**：堆顶永远是当前整棵树里**最小**的元素。

**求前 $K$ 个最大的元素（或第 $K$ 大） $\rightarrow$ 用小顶堆（剔除小的，留下大的）** **求前 $K$ 个最小的元素（或第 $K$ 小） $\rightarrow$ 用大顶堆（剔除大的，留下小的）**

在 Java 中，`Priority Queue`（优先队列）底层就是用二叉堆实现的。它的核心操作极其高效：
- 插入一个元素：$O(\log N)$
    
- 偷看一眼堆顶：$O(1)$
    
- 把堆顶扔掉（弹出）：$O(\log N)$


```java
// Java 的 PriorityQueue 默认就是【小顶堆】 
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// 通过 Collections.reverseOrder() 将优先队列强行改装为【大顶堆】 
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

heap.offer(num); //元素入堆
heap.poll(); // 弹出堆顶元素


```
