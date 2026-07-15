### Iterator  
An iterator (which is also a design pattern) is a "lightweight" object with low creation overhead that can traverse and select objects within a sequence. This allows developers to navigate through a collection without needing to understand its underlying structure.

In Java, the `Iterator` functionality is relatively simple and only allows one-way movement:

- The `iterator()` method requests the container to return an `Iterator`. The first call to the `next()` method of the `Iterator` returns the first element in the sequence. Note: the `iterator()` method is part of the `java.lang.Iterable` interface, which is inherited by the `Collection` interface.
- The `next()` method retrieves the next element in the sequence.
- The `hasNext()` method checks if there are more elements in the sequence.
- The `remove()` method removes the element that was last returned by the iterator.

#### Pitfalls:(陷阱)

- **Removing Elements During Iteration:**  
    If you use the `remove()` method on the data set while iterating, the structure of the data set changes, which can throw an exception and terminate the iterator.
    
- **Iterator State After Removal:**  
    After calling the iterator's `remove()` method, the iterator does not immediately point to the next element. You need to call `next()` again.
    
    Example:  
    Given a data set `{1, 2, 3, 4}`, when the iterator reaches `2` and you call `remove()` twice, it will not delete both `2` and `3`. Instead, it throws an `IllegalStateException`.
    
- **Thread Safety Risk:**  
    The introduction of an iterator increases the risk of thread safety issues. Even if the container is thread-safe, using an iterator can still introduce concurrency problems (e.g., if the data set and iterator are modified simultaneously, a `ConcurrentModificationException` might be thrown).
    
    - For `ArrayList` and `LinkedList`, modifying the list while iterating might throw a `ConcurrentModificationException`. To avoid this, you can use `synchronizedList()` and lock the list during iteration.-------------存疑！！！！
    - For `Vector`, the collection is synchronized, but the iterator is still not thread-safe, which can also cause errors.
    - However, with `CopyOnWriteArrayList`, the iterator retrieves a snapshot of the elements when `iterator()` is called. This iterator is thread-safe and doesn't require locking.---最推荐


### ConcurrentHashMap -- todo--hash collision
Reference : [ConcurrentHashMap（JDK8）-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1873182)

`ConcurrentHashMap` is a thread-safe variant（变体） of `HashMap` that used a segmented lock design before JDK 7. It can be thought of as an array of `Segment` instances, where each `Segment` is a segment of `HashMap`.
#### Data Structure
![[Pasted image 20240817114214.png]]

#### Changes in JDK 8:

1. **Red-Black Trees:** In JDK 8, if a linked list in a bucket exceeds a length of 8, it is converted into a red-black tree. This helps improve performance for large lists.
    
2. **Insertion Method:** JDK 7 used a head insertion method, while JDK 8 uses a tail insertion method.
    
3. **Segmented Locks:** JDK 7 employed segmented locks to handle concurrency, while JDK 8 removed this approach and no longer uses segment locks.
    
4. **Locking Mechanism:** JDK 7 used `ReentrantLock` for synchronization within segments. JDK 8 has moved to using `Synchronized` blocks for thread safety.
    
5. **Resizing:** In JDK 7, resizing was done internally within each segment without affecting others. In JDK 8, resizing is similar to `HashMap`, but it supports concurrent resizing and ensures thread safety.

These changes in JDK8 aimed to improve the efficiency, scalability, and simplicity of `ConcurrentHashMap`, making it more suitable for high-concurrency scenarios while reducing complexity in its implementation.

#### How `ConcurrentHashMap` Ensures Concurrency Safety:

##### JDK 7 Implementation:

In JDK 7, `ConcurrentHashMap` ensures concurrency safety using a combination of `ReentrantLock`, Compare-And-Swap (CAS), and segmentation:

1. **Segmentation:** `ConcurrentHashMap` is divided into segments, where each segment can be thought of as a smaller, independent `HashMap`. This segmentation reduces lock contention by allowing multiple threads to operate on different segments simultaneously.
    
2. **ReentrantLock:** Each segment in JDK 7 extends `ReentrantLock`. When a thread attempts to modify a segment (e.g., using the `put` method), it first acquires the lock for that segment to ensure exclusive access. The thread then performs the operation and releases the lock afterward.
    
3. **CAS Operations:** `ConcurrentHashMap` uses CAS to manage atomic updates. For example, when inserting a new segment into the segment array, the CAS operation ensures that only one thread can perform the insertion at a time. This ensures that the segment array is updated safely, even under heavy concurrency.
    
4. **Segment Array:** `ConcurrentHashMap` maintains a segment array, where each segment is responsible for managing a portion of the key-value pairs. The `put` method locks the segment, inserts the key-value pair into the appropriate `HashEntry` array within the segment, and then releases the lock. The segmentation approach also includes a configurable concurrency level, which determines the number of segments.
    

##### JDK 8 Implementation:

In JDK 8, `ConcurrentHashMap` underwent significant changes, moving away from the segmented lock approach:

1. **Unified Node Array:** Instead of segments, JDK 8 uses a single `Node` array, where each `Node` holds a key, value, and hash code, similar to the `Entry` objects in `HashMap`.
    
2. **Synchronized Blocks:** The use of `synchronized` blocks replaces the segmented `ReentrantLock` mechanism. `synchronized` is used to lock specific `Node` objects within the array, ensuring that only one thread can modify a particular bucket at any given time. This approach provides thread safety at the bucket level.
    
3. **CAS Operations:** JDK 8 continues to use CAS for atomic operations on specific array positions. This ensures safe, lock-free updates in many cases, enhancing performance under concurrency.
    
4. **Memory Efficiency:** By moving away from segmented locks and using `synchronized` blocks, JDK 8 achieves better memory efficiency. The removal of multiple `ReentrantLock` objects reduces memory overhead. Additionally, the underlying JVM optimizations for `synchronized` blocks in JDK 8 make this approach more performant compared to JDK 7.
    
5. **Why Synchronized Instead of `ReentrantLock`:** The switch to `synchronized` in JDK 8 is due to its memory efficiency and the JVM's improved handling of `synchronized` blocks. While JDK 7 required multiple `ReentrantLock` objects for each segment, JDK 8's use of `synchronized` eliminates this need, reducing memory consumption and potentially improving performance. 

### ArrayList & LinkedList & Vector

**ArrayList:**
- ArrayList is backed by an array, which uses contiguous memory space.
- It has fast lookup operations, but insertions and deletions are slow because they require shifting elements.

**LinkedList:**
- LinkedList is a doubly-linked list that uses non-contiguous memory space.
- It has slower lookup times because each element must be accessed sequentially via pointers, but insertions and deletions are fast as they only require changing the pointers of the previous and next nodes.

However, the above is generally true under normal circumstances:
- When searching for the x-th element, ArrayList performs better.
- When searching for the position of an element, both ArrayList and LinkedList require traversal, so their performance is similar.
- For insertions or deletions at the beginning or in the middle of the list, LinkedList performs better. But at the end of the list, their performance is similar.

#### ArrayList Resizing

- The underlying structure of ArrayList is contiguous memory space, and its initial size is fixed after creation.
- When the size of the ArrayList exceeds its capacity, a new array is created with a size that is 1.5 times the original size (using bitwise operations, `(x + x << 2)` for efficiency). The elements from the original array are then copied into the newly expanded array.

#### ArrayList & Vector

- **ArrayList:** Not thread-safe, high efficiency, commonly used.
- **Vector:** Thread-safe (using synchronized locks), lower efficiency.

#### Inserting Node C Between Nodes A and B in a Doubly-Linked List

```java
C.pre = A;
C.next = A.next;
A.next.pre = C;
A.next = C;
```

#### Collections.sort and Arrays.sort Principles

- `Collections.sort` by default calls `Arrays.sort`.
- `Arrays.sort` uses two sorting methods: `LegacyMergeSort` (a traditional merge sort that can be used when specified by the user) and `TimSort` (the default).
- `TimSort` is an optimized version of the traditional merge sort, particularly for edge cases. While traditional merge sort performs poorly on arrays sorted in reverse order, `TimSort` optimizes for such cases and performs better on large datasets while maintaining stability.

### HashXXX
#### HashSet Storage Principle

- **HashSet** is based on `HashMap`, where elements are essentially the keys in a `HashMap`, and the values are dummy objects.
- The underlying data structure is a hash table, ensuring the uniqueness of elements.
- As elements are added, hash collisions may occur (i.e., different elements have the same hash value). When this happens, the `equals` method is called to compare the objects. If the objects are equal, the element is not inserted; if they are not equal, a linked list is formed using the "tail insertion method" (after JDK 1.8). If the linked list length exceeds 8, it is converted into a red-black tree.

This translation and explanation should accurately reflect your understanding of Java collections. Let me know if you need further clarification!

Here's your explanation translated into English:

#### HashTable & HashMap & ConcurrentHashMap

**HashTable:**
- Thread-safe, achieved through the use of the `synchronized` keyword for locking.

**HashMap:**
- High efficiency, but not thread-safe. When used in a multithreaded environment, `HashMap` can become unsafe, particularly during resizing, which can lead to infinite loops.
- Using `Collections.synchronizedMap()` returns a thread-safe wrapper, but it simply adds synchronized locking, making its efficiency similar to `HashTable`, with no significant performance improvement.

**ConcurrentHashMap:**
- Utilizes segmented locking, which reduces the granularity of locks, balancing both safety and performance.
- Operations on different segments can occur simultaneously without affecting each other, allowing concurrent operations without conflict.
![[Pasted image 20240819204035.png]]
**Choosing in Development:**
- Use `HashMap` for local variables where thread safety is not a concern.
- Use `ConcurrentHashMap` for global variables that require thread-safe shared access.
- Typically, other options are not commonly used.

#### Differences Between HashMap and HashTable

**Thread Safety:**
- `HashMap` is not thread-safe but is more efficient.
- `HashTable` uses synchronized locking, making it thread-safe but less efficient.

**Support for `null` Values:**
- `HashMap` allows `null` values for both keys and values.
- `HashTable` does not permit `null` for keys or values; attempting to use `null` will result in a runtime exception.

**Parent Classes:**
- `HashMap` inherits from the `AbstractMap` class.
- `HashTable` inherits from the `Dictionary` class.

**Available Methods:**
- `HashTable` provides two additional methods not found in `HashMap`: `elements()` and `contains()`.
  - `elements()` returns an enumeration of the values contained in the `HashTable`.
  - `contains()` checks if the `HashTable` contains the specified value.

**Initial Capacity and Resizing:**
- The initial size and resizing mechanisms differ between `HashMap` and `HashTable`.

**Hash Calculation:**
- The method of computing hash values differs between `HashMap` and `HashTable`.

#### Infinite Loop in HashMap

**The Issue:**
- In a multithreaded environment, `HashMap` can enter an infinite loop during resizing.

**Symptoms:**
- During concurrent execution, at an unknown time, CPU usage might spike to 100% and remain high. By examining the stack trace, you'll find that threads are hanging in the `HashMap`'s `get()` method. The problem temporarily disappears after restarting the service but may reoccur later.

**Cause:**
- When multiple threads concurrently perform `put` operations on a `HashMap`, if a thread exhausts its time slice before completing its operation, and the `HashMap` is in the middle of resizing, the linked list in the bucket can form a loop. Subsequent `get()` calls will enter this loop, leading to an infinite loop.

**Solution:**
- Use `HashTable` or, preferably, `ConcurrentHashMap` to avoid this issue.

**Additional Note:**
- Besides causing deadlocks, multithreaded operations on `HashMap` can also lead to data loss.

**Reference**:
- [老生常谈，HashMap的死循环 - 简书 (jianshu.com)](https://www.jianshu.com/p/1e9cf0ac07f4)

### Stack
**Stack:**
- The underlying implementation of a `Stack` is based on an array.
- Values are stored at the last position in the array.

**Key Operations:**
1. `stack.peek();`
    
    - Retrieves the top element of the stack without removing it.
    - This operation is used to inspect the top element, and it does not modify the stack.
    - Internally, it accesses the last element of the underlying array.
2. `stack.pop();`
    - Removes and returns the top element of the stack.
    - This operation not only retrieves the top element but also removes it from the stack.
    - The `pop()` method is essentially a combination of `peek()` and an array element removal operation.


### Concurrency in Collections

#### Concurrent Modification Exception
- **ConcurrentModificationException**: This exception occurs when multiple threads are adding data to a shared `List` object simultaneously. Specifically, it is common in `ArrayList` when a thread modifies the list while another thread is iterating over it.
![[Pasted image 20240819225841.png]]
**Solutions:**
1. **Manual Locking (Not Recommended)**
   - You can manually synchronize the code block that modifies the list, but this is not generally recommended due to complexity and potential performance issues.

2. **Using `Vector`:**
   - `Vector` is thread-safe as it uses internal synchronization for all operations. However, it is less efficient compared to `ArrayList` because of the overhead of synchronization.

3. **Using `Collections.synchronizedList(new ArrayList<>())`:**
   - The `Collections` utility class provides synchronized wrappers for collections, making them thread-safe. For example, `Collections.synchronizedList()` can wrap an `ArrayList` to make it thread-safe.

4. **Using `CopyOnWriteArrayList`:**
   - `CopyOnWriteArrayList` is a thread-safe variant of `ArrayList` that achieves thread safety by creating a new copy of the underlying array during any write operation (add, remove, etc.). This ensures that the original array is not modified, preventing `ConcurrentModificationException`.

**Tips:**
- **Differences between `SynchronizedList` and `Vector`:**
  - `Vector` doubles its size when it needs to grow, while `ArrayList` (and thus `SynchronizedList`) grows by 1.5 times.
  - `SynchronizedList` can wrap any `List` implementation, providing thread safety across different `List` subclasses.
  - When iterating over a `SynchronizedList`, you must manually synchronize to avoid issues, while `Vector` handles synchronization internally.
  - `SynchronizedList` allows specifying the object to lock on, providing more flexibility.

#### CopyOnWriteArrayList
![[Pasted image 20240819225915.png]]
- **Thread-Safety Mechanism:**
  - `CopyOnWriteArrayList` is thread-safe because it uses a **`ReentrantLock`** for exclusive locking during write operations, ensuring that only one thread can modify the list at a time.
  - Data in `CopyOnWriteArrayList` is stored in an internal `array` that dynamically changes size. When a modification occurs, a new array is created, and the old one is replaced after the modification.

- **Read vs. Write Operations:**
  - **Read Operations:** These are lock-free and directly operate on a snapshot of the array, which ensures that reading is fast and unaffected by concurrent modifications.
  - **Write Operations:** These involve locking, copying the array, modifying the copy, and then replacing the original array with the modified copy.

- **Iterator Safety:**
  - Unlike `ArrayList`, iterating over a `CopyOnWriteArrayList` does not require additional synchronization, and it does not throw `ConcurrentModificationException`. This is because the iterator works on a snapshot of the array, which remains unaffected by concurrent writes.

### HashSet

- **Underlying Implementation:**
  - `HashSet` is implemented internally using a `HashMap`. The elements added to a `HashSet` are stored as keys in this `HashMap`, while the values in the `HashMap` are all the same, specifically a static final constant named `PRESENT` (which typically has a value of `0`).

- **Uniqueness Guarantee:**
  - The uniqueness of elements in a `HashSet` is enforced by the underlying `HashMap`. Since `HashMap` does not allow duplicate keys, `HashSet` ensures that no duplicate elements are stored.

### Collection & Collections

- **Collection Interface:**
  - `Collection` is the root interface in the Java Collection Framework. It is the super interface of several other interfaces like `Set`, `List`, `Queue`, and `Deque`. These interfaces represent different types of collections (groups of objects).
  - Subinterfaces of `Collection` include:
    - **Set**: Represents a collection that does not allow duplicate elements (e.g., `HashSet`, `LinkedHashSet`, `TreeSet`).
    - **List**: Represents an ordered collection (e.g., `ArrayList`, `LinkedList`, `Vector`, `Stack`).

- **Collections Utility Class:**
  - `Collections` is a utility class in the Java Collection Framework. It contains static methods that operate on or return collections.
  - **Key Features:**
    - **Searching**: Methods like `binarySearch(List<? extends Comparable<? super T>> list, T key)` to perform binary searches on sorted lists.
    - **Sorting**: Methods like `sort(List<T> list)` to sort the specified list into ascending order.
    - **Synchronization**: Methods like `synchronizedList(List<T> list)` to return a synchronized (thread-safe) list backed by the specified list.
    - **Unmodifiable Collections**: Methods like `unmodifiableList(List<? extends T> list)` to create an unmodifiable view of the specified collection.
    - **Reverse Order**: Methods like `reverseOrder()` to return a comparator that imposes the reverse of the natural ordering on a collection of objects that implement the Comparable interface.
