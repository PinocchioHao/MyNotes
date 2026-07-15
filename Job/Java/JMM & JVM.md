
## JMM vs JVM
The Java Memory Model (JMM) and the Java Virtual Machine (JVM) are both fundamental concepts in Java, but they **serve different purposes and operate at different levels** within the Java runtime environment. Here’s how they are related:

### 1. Java Memory Model (JMM):
   - **What it is**: The Java Memory Model defines how threads in a Java program interact through memory. It specifies the rules for reading and writing variables (including when changes made by one thread are visible to other threads) and provides the foundation for Java’s concurrency and synchronization mechanisms.
   - **Characteristics**:
     - **Visibility**: In a Java program, multiple threads may access and modify shared variables at the same time. To ensure that each thread is able to see the modifications made to shared variables by other threads, the Java memory model provides a number of rules. For example, the volatile keyword ensures that variables are visible, i.e., when a thread modifies the value of a volatile variable, other threads are able to see the modification immediately. In addition, visibility is also ensured by the synchronized block, which ensures that a thread's operations on a shared variable are visible to other threads when entering and exiting the synchronized block.
     - **Ordering**: To optimise program performance, compilers and processors may reorder instructions. However, this reordering may lead to unexpected results in multithreaded programs. To address this problem, the Java memory model defines the happens-before rule to ensure that the order of operations between multiple threads is as expected. Simply put, if one operation happens-before another, then the result of the first operation will be visible to the second.
     - **Atomicity**: Atomicity ensures that operations on variables are performed entirely or not at all. 

#### Relationship Between Visibility and Atomicity
- **Visibility**: Focuses on ensuring that changes made by one thread are visible to other threads promptly.
- **Atomicity**: Focuses on ensuring that operations are completed fully without interruption, preventing other threads from seeing an intermediate(中间的) state.

- **Atomicity does not include Visibility**: Atomicity pertains to whether operations are performed as an indivisible unit, while visibility pertains to whether changes are visible to other threads. They address different issues: atomicity prevents interruptions in an operation, whereas visibility ensures that changes are propagated(传播的) across threads.
    
- **Synchronization for Both**: Using synchronization mechanisms can address both issues(同步机制处理这两个问题). For example:
    
    - **Synchronized Blocks**: Ensure both atomicity (for compound operations) and visibility (ensuring that changes within a block are visible to other threads).
    - **Volatile Keyword**: Guarantees visibility but does not ensure atomicity for compound actions.



### 2. Java Virtual Machine (JVM):
   - **What it is**: The JVM is an abstract machine that provides a runtime environment for executing Java bytecode. It’s responsible for managing memory, executing code, and providing the necessary services to run Java applications. The JVM abstracts the underlying(底层的) hardware and operating system, making Java applications platform-independent.
   - **Responsibilities**:
     - **Memory Management**: The JVM manages memory allocation and garbage collection.
     - **Execution of Bytecode**: The JVM interprets or compiles bytecode into native machine code to execute on the host system.
     - **Class Loading**: The JVM loads classes dynamically as needed by the application.
     - **Thread Management**: The JVM handles the creation, synchronization, and scheduling of threads.

### Relationships Between JMM and JVM:
- **Implementation in JVM**: The JMM is a part of the Java language specification that the JVM must implement. The JVM enforces the rules and constraints of the JMM when executing Java code, particularly in multithreaded environments.
- **Concurrency Control**: The JVM uses the JMM to manage memory consistency between threads. When you use synchronized blocks, `volatile` variables, or atomic operations in your code, the JVM ensures that these operations adhere to the rules defined by the JMM.
- **Memory Visibility**: The JMM ensures that changes made by one thread to shared variables are visible to other threads in a predictable manner(以一种可预测的方式). The JVM implements these rules, ensuring that the program runs correctly according to the Java specification(规范).

### **Summary:**
- **JVM** is the engine that runs Java applications, managing memory, execution, and threading.
- **JMM** defines the low-level details of how Java handles memory visibility, atomicity and ordering in a multithreaded context.
- The **JMM is implemented by the JVM** to ensure that multithreaded Java programs behave correctly and consistently across different platforms and hardware architectures.


## JVM

[JVM（Java虚拟机）详解（JVM 内存模型、堆、GC、直接内存、性能调优）_jvm内存-CSDN博客](https://blog.csdn.net/footless_bird/article/details/128921448)

The JVM is a virtual computer that can execute Java bytecode, providing the necessary environment for running Java programs.

JDK1.8
![[Pasted image 20240811122254.png]]

![[Pasted image 20240811122332.png]]



Before JDK.1.8
![[Pasted image 20240811003652.png]]
![[Pasted image 20240809210937.png]]

### Thread Private Area
**Program Counter (PC Register)**  
The program counter holds the address of the next instruction to be executed in the bytecode for each thread. This memory area is the only one in the JVM that does not encounter an OutOfMemoryError (OOM).

**JVM Stack (Java Virtual Machine Stack)**  
The JVM stack is also thread-private. It describes the memory model for Java method execution. Each method execution creates a stack frame, which contains the local variable table, operand stack, dynamic linking, and method exit (return address). In Java, the execution of each method corresponds(对应) to the process of pushing and popping the stack frames in the JVM stack.

**Native Method Stack**  
Similar to the JVM stack, but it’s used for executing native methods (methods written in languages other than Java, such as C or C++).

### Thread Shared Area
**Java Heap**  
The heap stores object information. This is a memory area shared among all threads. The heap is the largest area in the JVM, and nearly all object instances are allocated memory here. The Java heap is the main area managed by the Garbage Collector (GC). 

Before JDK1.8, the heap is divided into the Young Generation (which includes the Eden space, From space, and To space), the Old Generation, and the Permanent Generation (only Hot Spot virtual merchin called Perm Gem, it is renamed to Meta-space in Java 8, and is located in native memory rather than the heap). **After JDK1.8, the heap is divided into the Young Generation and the Old Generation.**  The Young Generation mainly stores newly created objects, taking up about one-third of the heap. It is a frequent area for object creation and destruction, making it a frequent target for GC. The Old Generation stores objects with longer lifecycles, and it is more stable.

The Young Generation is further divided into approximately 80% Eden space and two Survivor spaces (From and To spaces) that each take up about 10%. Survivor-From holds survivors from the last GC cycle and is scanned during the current GC, while Survivor-To is the target space where surviving objects from Eden and Survivor-From are copied during GC. During each GC in the Young Generation, the Eden and Survivor-From spaces are cleared, and any surviving objects are copied to Survivor-To.

**Method Area (also known as Permanent Generation(Hot Spot) before JDK1.8, Meta Space after JDK1.8)**  
This is also a thread-shared memory area used to store **class information, constants, static variables, and JIT(Just in Time - 即时编译器)-compiled code** that have been loaded by the JVM.

### Method Area(Meta Space vs Permanent Generation)

#### Method Area (logically)
- **The Method Area is a specification of JVM that all virtual machines must comply with**. Common JVM virtual machines include Hotspot, JRockit (Oracle), J9 (IBM)
- The method area **physically belongs to the heap**, but in order to distinguish it from the heap, it is usually called the non heap area
- Shared among threads, mainly used to store type information, constants, static variables, and JIT-compiled code that have already been loaded by the virtual machine
- The size of the method area determines how many classes the system can store. If the system defines too many classes and causes the method area to overflow, the virtual machine will also throw a memory overflow error. Closing JVM will release the memory in this area.

Before Java 8, it was stored in JVM memory and implemented permanently in heap space, limited by JVM memory size parameters
Java8 has moved away from the permanent generation and method area, and introduced meta space


#### Meta-Space (Metadata Area)
Meta-space is a new implementation of the method area in the HotSpot Virtual Machine from JDK 1.8 and later.

Meta-space is not in the virtual machine, but is implemented directly in physical (local) memory, and is no longer limited by the JVM memory size parameter. The JVM will no longer have the memory overflow problem of the method area, but if the physical memory is full, the meta-space will also report OOM.

The difference between the meta-space and the method area is that the contents of the meta-space are slightly different during compilation and after the class is loaded, but in general, they are divided into these two parts:

**Class meta information**
- Class meta information is put into the meta space during the compilation of the class. It contains basic information about the class: version, fields, methods, interfaces, and the constant pool table.
- Constant pool table: mainly stores the literals and symbolic references generated during the compilation of the class, which will be resolved into the runtime constant pool after the class is loaded.

**Runtime Constant Pool**
- Runtime Constant Pool mainly holds literals and symbolic references that are resolved after the class is loaded, but not only that.
- Runtime Constant Pool is dynamic, you can add data to it, the most common use is the intern() method of the String class.

#### Direct Memory

**Direct Memory** refers to a memory area in the JVM that is not part of the Java heap but is still managed by the JVM. It's used primarily for **off-heap memory allocation**, which allows Java applications to allocate memory outside the traditional Java heap, providing more direct access to the memory. This is particularly useful for applications that require **high-performance I/O operations**, such as those dealing with large amounts of data or needing to interface closely with native code.

It is expensive to allocate and reclaim, but has high read/write performance.
##### Key Points about Direct Memory:

1. **Off-Heap Memory**:
    - Direct memory is part of the off-heap memory, which is not managed by the Garbage Collector (GC). This can help reduce GC pressure, especially for applications that need large amounts of memory.
2. **ByteBuffer Allocation**:
    - One common way to use direct memory is through the `java.nio` package, specifically with `ByteBuffer.allocateDirect()`. This allows you to allocate memory outside of the heap for buffers that can be directly accessed by native I/O operations.
3. **Memory Management**:
    - Since direct memory is not managed by the GC, it requires explicit deallocation. Failing to release direct memory properly can lead to memory leaks, even though the Java heap appears to be under control.
4. **Performance**:
    - Direct memory can offer performance benefits, particularly for I/O operations, because it allows the JVM to interact more closely with the underlying operating system, bypassing some of the overhead associated with heap memory.
5. **JVM Options**:
		Direct memory is not managed by the JVM memory reclamation (the allocation and release of direct memory is managed by Java through the **UnSafe object**), but the system memory is limited, and will be reported as OOM when there is insufficient physical memory.
    - The maximum amount of direct memory that can be allocated is controlled by the JVM option `-XX:MaxDirectMemorySize`. If this option is not specified, it defaults to the maximum heap size.

##### Usage Scenarios:

- **High-Performance I/O**: Direct memory is often used in scenarios where fast, low-latency I/O operations are required, such as in network servers, databases, and other systems that manage large amounts of data.
    
- **Native Libraries**: Applications that use native libraries (e.g., through JNI) might use direct memory for better performance and more direct control over memory allocation.
    



### Example

![[Image.png]]


---
## Garbage Collection

### How to determine if an object is a "garbage"

#### Reference Counting(引用计数法)

In reference counting, an object's reference count is incremented when it is referenced by another object. If no references point to the object, its reference count drops to zero, and it is considered a candidate for garbage collection. While reference counting is simple to implement and efficient, it cannot resolve circular dependencies. As a result, Java does not use reference counting for garbage collection (though Python does).

#### Reachability Analysis(可达性分析)

Java uses reachability analysis for garbage collection. In this approach, the garbage collector starts from a set of "GC Roots" and attempts to build a chain of references, starting from these roots. If no reference chain can be established between the GC Roots and a given object, that object is considered unreachable. If an object is determined to be unreachable in at least two consecutive checks, it is marked as eligible for garbage collection.

##### What Can Be Considered as "GC Roots"?

- Objects referenced by the JVM stack (i.e., local variables or method parameters).
- Objects referenced by the native method stack (JNI references).
- Objects referenced by static properties of loaded classes in the method area.
- Objects referenced by constants in the method area (such as final constant values).


### Garbage Collection Algorithms
#### Mark-Sweep Algorithm

The Mark-Sweep algorithm is the most basic and easiest to implement. In this algorithm, the marking phase(阶段) identifies all objects that need to be collected, and the sweeping phase reclaims the space occupied by these marked objects.

However, the Mark-Sweep algorithm can lead to a lot of fragmented memory. If there is too much fragmentation, it may become difficult to find enough contiguous space for allocating large objects, which could trigger another GC cycle.

![[Pasted image 20240811205334.png]]
#### Copying Algorithm

The Copying algorithm divides the memory into two regions of equal size (or in some cases, proportionally). Only one region is used at a time. When that region is full, the surviving objects are copied to the other region, and the space previously occupied is entirely cleared.

The Copying algorithm is simple to implement and addresses the memory fragmentation issue present in the Mark-Sweep algorithm. However, this approach effectively reduces the available memory by half. Additionally, the efficiency of the algorithm is dependent on the number of surviving objects; if there are many, the performance of the Copying algorithm can significantly decrease.

![[Pasted image 20240811200234.png]]
#### Mark-Compact Algorithm (Compression Method)

To address the shortcomings of the Copying algorithm, the Mark-Compact algorithm first marks the memory that can be reclaimed and then moves the surviving objects toward one end of the memory space, clearing the space beyond the boundary.

![[Pasted image 20240811205503.png]]

#### Generational Collection Algorithm

The Generational Collection algorithm is used in most modern JVMs.

The Young Generation is characterized by a high turnover of objects—many are created and quickly become eligible for GC. Therefore, the Copying algorithm is used in this region (with an 8:1:1 ratio). The Old Generation, where fewer objects need to be collected, uses the Mark-Compact algorithm.

The Young Generation is divided into approximately 80% Eden space and two Survivor spaces (From and To), each accounting for about 10%. During each GC cycle in the Young Generation, the Eden space and the Survivor-From spaces are cleared, and any surviving objects are copied to the Survivor-To space.

---
### Triggering conditions for GC

#### 1. Minor GC (Young Generation GC)

**Triggering Conditions**:

- **Minor GC** is triggered when the Eden space in the Young Generation becomes full.
- The Eden space is the memory area within the Young Generation used for allocating new objects. Most newly created objects are initially allocated memory in the Eden space.
- **Minor GC** will clean up unused objects in the Eden space and move still-active objects to the Survivor spaces (S0 or S1).
- When objects survive multiple collections in the Survivor space (reaching a set threshold, 15 as usual), they are moved to the Old Generation.

**Note**:

- **Minor GC** occurs relatively frequently and is generally fast because it only focuses on the Young Generation's memory.
- Since most objects have a short lifespan, **Minor GC** usually recovers a significant amount of memory.

#### 2. Major GC (Old Generation GC)

**Triggering Conditions**:

- **Major GC** is triggered when the Old Generation becomes full.
- The Old Generation mainly stores objects that have survived multiple **Minor GCs** in the Young Generation.
- **Major GC** is slower because it needs to check more objects and reclaim a larger amount of memory.

**Note**:

- **Major GC** significantly impacts application performance as it typically causes a Stop-The-World (STW) pause.
- **Major GC** cleans up garbage objects in the Old Generation and reclaims the memory they occupied.

#### 3. Full GC

**Triggering Conditions**:

- **System Calls**: A **Full GC** can be triggered when the developer explicitly calls the `System.gc()` method, requesting the JVM to perform **Full GC**.
- **Old Generation Space Insufficiency**: When objects in the Young Generation are frequently promoted to the Old Generation, leading to insufficient space in the Old Generation, **Full GC** might be triggered.
- **PermGen/Metaspace Full**: If the JVM uses Permanent Generation (PermGen) or Metaspace to store class metadata, and these areas become full, **Full GC** might be triggered.
- **CMS GC**: For Concurrent Mark-Sweep (CMS) GC, if there is excessive memory fragmentation and it cannot allocate large objects, **Full GC** might be triggered.

**Note**:

- **Full GC** is the most expensive garbage collection operation, as it cleans the Young Generation, Old Generation, and possibly the Permanent Generation or Metaspace.
- **Full GC** usually results in long STW pauses, significantly affecting application performance, so frequent triggering should be avoided as much as possible.


---

### Garbage Collector

**Serial Collector** (Single-threaded, Copying Algorithm, Young Generation)
- The Serial Collector is a single-threaded garbage collector that uses the copying algorithm. It is designed for the Young Generation.

**ParNew Collector** (Multithreading Serial Collector, Young Generation)
- The ParNew Collector is a multi-threaded version of the Serial Collector. It is also used for the Young Generation.

**Parallel Scavenge Collector** (Multithreading, Copying Algorithm, Young Generation)
- The Parallel Scavenge Collector is a multi-threaded garbage collector that uses the copying algorithm and is intended for the Young Generation. Its primary focus is on achieving a controlled throughput.

**Serial Old Collector** (Single-threaded, Mark-Compact Algorithm, Old Generation)
- The Serial Old Collector is a single-threaded garbage collector that uses the Mark-Compact algorithm and is designed for the Old Generation.

**Parallel Old Collector** (Multithreading, Mark-Compact Algorithm, Old Generation)
- The Parallel Old Collector is a multi-threaded garbage collector that uses the Mark-Compact algorithm. It is used for the Old Generation and often works together with the Parallel Scavenge Collector.

**CMS Collector** (Multithreading, Mark-Sweep Algorithm, Old Generation)
- The CMS (Concurrent Mark-Sweep) Collector is a multi-threaded garbage collector that uses the Mark-Sweep algorithm. It is designed for the Old Generation and aims to minimize pause times.

**G1 Collector** (Multithreading, Mark-Compact Algorithm, Mixed Collection for Old and Young Generations, Highly Efficient, Introduced in JDK 7)
- The G1 (Garbage-First) Collector is a multi-threaded garbage collector that uses the Mark-Compact algorithm. It is designed to perform mixed collection for both the Old and Young Generations. G1 is highly efficient and became the default garbage collector in JDK 9.



**JDK 1.8 Default Garbage Collector**: Parallel Scavenge (Young Generation) + Parallel Old (Old Generation)
- In JDK 1.8, the default garbage collectors are the Parallel Scavenge Collector for the Young Generation and the Parallel Old Collector for the Old Generation.

**JDK 1.9 Default Garbage Collector**: G1
- In JDK 9, the default garbage collector is G1.


---
### Object Creating Process

**Object Allocation**:
- Objects are initially allocated in the **Eden space**. If there isn't enough space in the Eden area, a **Minor GC** (Garbage Collection) is triggered.

**Large Objects**:
- **Large objects** (those requiring a significant amount of contiguous memory) are allocated directly in the **Old Generation**. This approach avoids the overhead of copying large objects between the Eden and Survivor spaces.

**Long-Lived Objects**:
- Objects that survive for a long time are promoted to the **Old Generation**.
- The JVM assigns an **age counter(usually 15)** to each object. After an object survives its first Minor GC, it is moved to the **Survivor space**. Each time it survives another Minor GC, its age increases by one. When its age reaches a certain threshold, it is promoted to the **Old Generation**.

**Dynamic Age Determination**:
- The JVM dynamically determines the age threshold for promotion. If the total size of objects with the same age in the Survivor space exceeds half of the Survivor space, then objects of that age or older are promoted directly to the **Old Generation**.

**Space Allocation Guarantee**:
- Before a **Minor GC**, the JVM calculates the average size of objects that are likely to be promoted from the **Survivor space** to the **Old Generation**. If this value exceeds the remaining space in the Old Generation, the JVM will trigger a **Full GC** to reclaim memory in the Old Generation.


---
### JAVA Class Loading Process

#### Java Source File to Class File
1. **Java Source File** is compiled into a **.class file**.
2. The **ClassLoader** reads this **.class file** and converts it into an instance of `java.lang.Class`. Once the `Class` instance is created, the JVM can use it to create objects, invoke methods, and perform other operations.

#### ClassLoader Mechanism:

**Sources of Class Files**:

- **Core Java Classes**: Located in `$JAVA_HOME/jre/lib`, with the most notable being `rt.jar`. These are loaded by the **BootstrapClassLoader**.
- **Java Extension Classes**: Located in `$JAVA_HOME/jre/lib/ext`. These are loaded by the **ExtClassLoader**.
- **User-Defined Classes and Third-Party Jars**: Located in the project directory, such as `WEB-INF/lib`. These are loaded by the **AppClassLoader** (also known as the System ClassLoader).

**Note**: These three class loaders do not extend `ClassLoader` and are implemented internally by the JVM. They are not accessible via Java programs and will return `null` if queried.

#### Parent Delegation Mechanism:

The **Parent Delegation Mechanism** works as follows:

- When loading a class, the system first tries to get an instance of the **AppClassLoader**.
- It then delegates the class loading request to its parent, which is the **BootstrapClassLoader**.
- If the **BootstrapClassLoader** cannot find the class, the request is passed to the **ExtClassLoader**.
- If the **ExtClassLoader** cannot find the class, the request is finally passed to the **AppClassLoader**.
- If the class is not found by any of these loaders, an error is reported.

This mechanism ensures the security of core Java classes by preventing user-defined classes from replacing them.

For example:
![[Pasted image 20240813205349.png]]

#### When to Override ClassLoader:

**Custom ClassLoader** may be needed for scenarios such as:

- Loading database drivers, custom frameworks, or server containers (e.g., Tomcat).
- Loading classes from non-standard sources or custom bytecode.

To implement a custom class loader, you need to:

- **Extend `ClassLoader`** and override the methods `findClass()` and `loadClass()`.






--- 
## Interview Questions

### **Why It’s Not Recommended to Override the `finalize` Method**

1.  **Extending Object Lifecycle**: 
	- Overriding the `finalize` method often involves placing time-consuming operations within it. This can unintentionally extend the object's lifecycle. If objects are being created and garbage collected rapidly, this delay in finalization can slow down the garbage collection process, potentially leading to OutOfMemoryError (OOM) and other exceptions.
2.   **Unpredictable Behavior**:
    - The timing of when the `finalize` method is called is unpredictable. The JVM does not guarantee when or even if `finalize` will be invoked, leading to unreliable resource management.
3.  **Performance Impact**:
    - Finalization adds extra overhead to the garbage collection process. Objects with a `finalize` method are not immediately reclaimed and must undergo an additional step, further delaying memory recovery.
4.  **Potential Resource Leaks**:
    - If an exception is thrown in the `finalize` method or if the JVM exits before finalization occurs, resources might not be properly released, leading to resource leaks.
5.  **Deprecation in Java 9**:
    - Starting with Java 9, the `finalize` method is deprecated. Java recommends using alternative approaches like `try-with-resources` and `java.lang.ref.Cleaner`, which provide more reliable and efficient resource management.







## 其它杂七杂八面试题
对象分配：对象先会分配在eden（大对象直接进老年代），当eden满了，触发GC，存活的对象进入from survivor，from survivor满了，触发GC，存活的对象进入to survivor，to survivor满了，触发GC，存活的对象进入from survivor，这样from survivor和to survivor总有一个为空，并且每次GC对象的年龄阈值都会加一，默认到15时对象进入老年代，当老年代到达一定百分比时触发GC


Full GC 、Major GC（Old GC）
Minor GC、Major GC、Full GC 的区别

新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
老年代收集（Major GC/Old GC ）：只是老年代的垃圾收集
整堆收集（Full GC）：收集整个 java 堆（young gen + old gen）和方法区的垃圾收集
Full GC 触发机制：

调用 System.gc 时，系统建议执行 Full GC，但是不必然执行
老年代空间不足
方法区空间不足
通过 Minor GC 后进入老年代的平均大小大于老年代的可用内存
由 Eden 区、survivor space1（From Space）区向 survivor space2（To Space）区复制时，对象大小大于 To Space 可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
当永久代满时也会引发 Full GC，会导致 Class、Method 元信息的卸载

堆空间分成不同区的原因
堆空间分为新生代和老年代的原因

根据对象存活的时间，有的对象寿命长，有的对象寿命短。应该将寿命长的对象放在一个区，寿命短的对象放在一个区。不同的区采用不同的垃圾收集算法。寿命短的区清理频次高一点，寿命长的区清理频次低一点。

新生代分为了 eden、Survivor 区的原因

为了更好的管理堆内存中的对象，方便GC算法（复制算法）来进行垃圾回收。

如果没有 Survivor 区，那么 Eden 每次满了清理垃圾，存活的对象被迁移到老年区，老年区满了，就会触发 Full GC，而 Full GC 是非常耗时的。

将 Eden 区满了的对象，添加到 Survivor 区，等对象反复清理几遍之后都没清理掉，再放到老年区，这样老年区的压力就会小很多。即 Survivor 相当于一个筛子，筛掉生命周期短的，将生命周期长的放到老年代区，减少老年代被清理的次数。

新生代的 Survivor 区又分为 s0 和 s1 区的原因：

分两个区的好处就是解决内存碎片化。

为什么一个 Survivor 区不行？

假设现在只有一个survivor区，模拟一下流程：

新建的对象在 Eden 中，一旦 Eden 满了，触发一次 Minor GC，Eden 中的存活对象就会被移动到 Survivor 区。这样继续循环下去，下一次 Eden 满了的时候，问题来了，此时进行 Minor GC，Eden和 Survivor 各有一些存活对象，如果此时把 Eden 区的存活对象硬放到 Survivor 区，很明显这两部分对象所占有的内存是不连续的，也就导致了内存碎片化。

GC 优化的本质，也是为什么分代的原因：减少GC次数和GC时间，避免全区扫描。




