Java Stream 绝对是 Java 8 以来最伟大的“魔法”之一。在 LeetCode 中，它能帮你把原本 10 行的循环缩减成 1 行，而且代码读起来就像英语句子一样顺滑。

你可以把 Stream 想象成一条**自动化流水线**：原料（集合/数组）进去，经过一道道工序（过滤、转换、排序），最后产出成品（新集合/结果）。

---

### 1. 流水线核心三部曲

无论多复杂的 Stream，都逃不出这三个步骤：

1. **创建流** (`stream()` / `Arrays.stream()`)
    
2. **中间操作** (工序：`filter`, `map`, `sorted`) —— **注意：** 它们是延迟执行的。
    
3. **终端操作** (出货：`collect`, `forEach`, `sum`) —— 执行这一步，流水线才真正动起来。
    

---

### 2. LeetCode 最常用模板

#### A. 基础变换：过滤、转换、去重

当你需要对一个列表进行初步加工时：

```Java
List<String> result = list.stream()
    .filter(s -> s.length() > 3)      // 1. 过滤：只要长度大于3的
    .distinct()                       // 2. 去重：去掉重复项
    .map(String::toUpperCase)         // 3. 转换：全转大写
    .limit(5)                         // 4. 截取：只要前5个
    .collect(Collectors.toList());    // 5. 收货：打包成 List
```

#### B. 数字处理：求和、最大、最小值 (LeetCode 常用)

处理 `int[]` 数组时，`Arrays.stream` 配合 `IntStream` 非常强大：


```Java
int[] nums = {1, 2, 3, 4, 5};

int sum = Arrays.stream(nums).sum();                       // 求和
int max = Arrays.stream(nums).max().getAsInt();             // 最大值
int min = Arrays.stream(nums).min().getAsInt();             // 最小值
double avg = Arrays.stream(nums).average().getAsDouble();  // 平均值
long count = Arrays.stream(nums).filter(n -> n > 2).count(); // 计数
```


```java
//Map元素转数组
int[] countArr1 = map1.values().stream().mapToInt(i -> i).sorted().toArray();

//int[] 转 List<Integer>
List<Integer> rowList = Arrays.stream(row).boxed().collect(Collectors.toList());
```



#### C. 数据整理：转为 Map 或分组

这是在做“频率统计”类题目（比如统计字符出现次数）时的神技：


```Java
// 1. 统计频率：将数组转为 Map<元素, 出现次数>
Map<Integer, Long> counts = Arrays.stream(nums)
    .boxed() // int 转 Integer，因为 collect 不支持原始类型流
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()));

// 2. 根据属性分组：比如按字符串长度分组
Map<Integer, List<String>> groups = list.stream()
    .collect(Collectors.groupingBy(String::length));
```




---

### 3. Stream 常用算子对比表

|**分类**|**算子**|**功能描述**|**备注**|
|---|---|---|---|
|**中间**|`filter(p)`|过滤掉不符合断言 `p` 的元素|常用|
|**中间**|`map(f)`|把元素 A 变成 B (映射)|**核心**|
|**中间**|`flatMap(f)`|把“嵌套集合”打平 (如 `List<List>` 转 `List`)|高阶|
|**中间**|`sorted()`|排序|可加 Comparator|
|**终端**|`collect(c)`|收集成 List, Set, Map 等|**最常用终止**|
|**终端**|`anyMatch(p)`|只要有一个符合就返回 true|判断类|
|**终端**|`allMatch(p)`|全部符合才返回 true|判断类|
|**终端**|`reduce(v, f)`|归约操作 (如累加、累乘)|较底层|

---

### 4. 避坑指南（重点！）

1. **性能成本**：Stream 虽美，但它有额外的对象创建开销。在 LeetCode 极严苛的时间限制下，如果 $N$ 达到 $10^7$ 以上，传统的 `for` 循环通常比 Stream 快 3-5 倍。
    
2. **原始类型坑**：`int[]` 数组流是 `IntStream`，它没有 `collect(toList)`。你需要先调用 `.boxed()` 把它变成 `Stream<Integer>`。
    
3. **只能消费一次**：一个流一旦执行了终端操作（如 `collect`），它就“死”了。不能再次使用，否则会报 `IllegalStateException`。
    

---

### 5. 实战小练习

如果你要把一个 `int[]` 数组中所有的**偶数**挑选出来，**平方**后按**降序**排列，最后转成 `List<Integer>`，Stream 会怎么写？

> **提示：**
> 
> 1. `Arrays.stream(arr)`
>     
> 2. `.filter(n -> n % 2 == 0)`
>     
> 3. `.map(n -> n * n)`
>     
> 4. `.boxed().sorted(Collections.reverseOrder())`
>     
> 5. `.collect(Collectors.toList())`
>     

是不是感觉这种链式调用比写一堆 `if-else` 加临时 `List` 爽多了？

关于 Stream，你还有哪些具体的操作（比如 `toMap` 的冲突处理）想深入了解的吗？或者我们可以继续刷下一道题？
