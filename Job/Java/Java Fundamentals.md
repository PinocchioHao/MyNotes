
### Hashmap & Hashtable & Treemap

| Feature           | `HashMap`                                 | `Hashtable`                        | `TreeMap`                                                       |
| ----------------- | ----------------------------------------- | ---------------------------------- | --------------------------------------------------------------- |
| Null Keys/Values  | Allows one null key, multiple null values | Does not allow null keys or values | Allows null values, but null keys only with certain comparators |
| Thread-Safety     | Not synchronized                          | Synchronized                       | Not synchronized                                                |
| Order of Elements | No order guaranteed                       | No order guaranteed                | Sorted by keys                                                  |
| Performance       | O(1) for get/put                          | O(1) for get/put                   | O(log n) for get/put                                            |
| Use Case          | General-purpose, non-threaded scenarios   | Legacy, thread-safe scenarios      | When key order is important                                     |

### Memory Leaks & OutOfMemoryError (OOM)

**Causes of OOM:**

1. **Insufficient Allocation:** This occurs when the JVM has been allocated too little memory, typically through VM parameters set during startup.
 
2. **Excessive Usage Without Release:** If an application uses too much memory and fails to release it properly, it can lead to memory leaks or eventually an OutOfMemoryError.

**Memory Leak:** A memory leak occurs when memory that has been allocated is not properly released after it is no longer needed, preventing the JVM from reusing that memory. Essentially, the allocated memory is no longer in use but still cannot be reallocated by the JVM.

**OutOfMemoryError:** This happens when an application requests more memory than the JVM can provide, leading to an overflow situation.

**Causes of Memory Leaks:**

1. **Creating Large Numbers of Unused Objects:** For example, if you need to concatenate(连接) a large number of strings, using `String` instead of `StringBuilder` can lead to memory inefficiency:
   ```java
   String s = "";
   for (int i = 0; i < 10000; i++) {
       s += i; // This creates 10000 String objects.
   }
   ```

2. **Use of Static Collections:** Static collections such as `HashMap` and  `List` can easily cause memory leaks because their lifespan matches the entire application, meaning their objects cannot be garbage collected.

3. **Circular Dependencies in Spring Beans:** If there are circular dependencies in Spring beans, it can lead to an infinite loop of calls, eventually causing memory overflow.

4. **Unclosed Connection Objects:** Physical connections like IO streams, database connections, or network connections must be closed after use to prevent memory leaks.

5. **Usage of Listeners:** If a listener is not removed when an object is released, it can prevent garbage collection of that object, leading to memory leaks.

6. **Thread Pool Usage:** A large thread pool queue or too many threads in the pool can cause memory leaks, especially if using the Executors framework's default thread pools.


### JAVA Parameter Passing

For primitive data types (原始数据类型, e.g., int, float), what is passed is a copy of the value, not the value itself. This means changes to the parameter inside the method do not affect the original value.

For objects, what is passed is the value of the reference to the object, not the actual object. When an object is passed as a parameter, you are working with the reference to that object. Changes to the object’s fields or properties will affect the original object, but reassigning the reference itself will not change the original reference outside the method(e.g.  `obj = new MyObject();` in a method ).



### Constructor Methods in Java

- Constructors cannot be explicitly called(显式调用). They are invoked automatically when an object is created using the `new` keyword.
    
- Constructors can be overloaded. This means you can have multiple constructors in the same class with different parameter lists (different function signatures).
    
- Constructors cannot be overridden. This means you cannot redefine a constructor in a subclass to replace a constructor in the parent class. However, you can have different constructors in a subclass with different parameter lists.



### == vs .equals()

- `==` compares the memory addresses (references) of objects. For primitive data types(原始数据类型), it compares the values. For reference types, it compares whether the references point to the same object in memory.
    
- `equals()` is used to compare the content of two objects for equality. By default, `equals` in `java.lang.Object` compares the memory addresses, which is equivalent to `==`. However, many classes override the `equals` method to compare the actual contents of the objects.

``` java
String s1 = new String("zs"); // s1 refers to a new String object on the heap.
String s2 = new String("zs"); // s2 refers to another new String object on the heap.
System.out.println(s1 == s2); // Outputs false, as they are different objects in memory.

String s3 = "zs"; // s3 refers to a String object in the string literal pool (if not already exists, it will be created once).
String s4 = "zs"; // s4 refers to the same String object as s3 in the pool due to string interning.
System.out.println(s3 == s4); // Outputs true, as they refer to the same object in the string pool.

System.out.println(s3 == s1); // Outputs false, s1 refers to a heap address, s3 refers to the string pool address.

String s5 = "zszs"; // s5 refers to a new String object in the string pool.
String s6 = s3 + s4; // s6 is a new String object created by concatenation, not in the string pool.
System.out.println(s5 == s6); // Outputs false, as s6 is a new object created from concatenation.

System.out.println(s5 == "zs" + "zs"); // Outputs true, as the concatenation happens at compile time and both refer to the same object in the pool.

final String s7 = "zs"; // s7 is a constant reference to a String object in the pool.
final String s8 = "zs"; // s8 is also a constant reference to the same String object in the pool.
String s9 = s7 + s8; // Despite the concatenation, s9 refers to a new String object in the pool, as s7 and s8 are not literals.
System.out.println(s5 == s9); // Outputs true, because the compiler optimizes the concatenation of two string literals.

final String s10 = s3 + s4; // s10 is a new String object created by concatenation of non-literals, assigned to a final variable.
System.out.println(s5 == s10); // Outputs false, as s10 is a new object not in the string pool.
```

``` java
String s1 = new String("zs");
String s2 = new String("zs");
System.out.println(s1 == s2);    // false
String s3 = "zs";
String s4 = "zs";
System.out.println(s3 == s4);   // true   都在常量池
System.out.println(s3 == s1);   // false  s1指的是堆地址，s3指的是常量池的地址
String s5 = "zszs";
String s6 = s3+s4;
System.out.println(s5 == s6);  // false s3+s4是由String对象+String对象，会新建对象，新创建的对象不在常量池
System.out.println(s5 == "zs"+"zs");  // true "zs"+"zs"直接拼接，在常量池
final String s7 = "zs";
final String s8 = "zs";
String s9 = s7+s8;
System.out.println(s5 == s9);   // true  由于s7和s8是常量，编译器在处理常量的时候会进行优化，把s7+s8的新对象转化成一个常量，跟s5相等。
final String s10 = s3+s4;
System.out.println(s5 == s10);  // false  此处不用管修饰s10的final，因为String本身就是final，s3和s4是变量，此时+操作会新建一个对象
```

**Additional Note:** The `String` class is `final`, which means its instances cannot be modified once created. When performing concatenation with `+`, Java uses `StringBuilder` to create a new `String` object. Strings already present in the constant pool remain in the pool even after concatenation.


### String & StringBuilder & StringBuffer
- **`String`**: Each time a `String` is modified, a new `String` object is created and the reference points to the new `String` object. If you frequently modify string content, it is recommended to use `StringBuilder` or `StringBuffer` to improve performance.
    
- **`StringBuilder`**: This class is not thread-safe but offers better performance for single-threaded scenarios. It is typically used when you need to perform a lot of string manipulations in a single thread.
    
- **`StringBuffer`**: This class is thread-safe, which means it can be used safely in multi-threaded environments where multiple threads might access the same instance. However, this comes with some performance overhead due to synchronization.
	

**Tips on Thread Safety:**

- Consider thread safety when multiple threads access shared resources, typically shared global variables. Local variables, which are specific to a single thread, do not need to be thread-safe.
    
- `StringBuilder` is commonly used within a single method for string concatenation and other operations. For example, if a method in a server-side service uses `StringBuilder`, and multiple clients access the server, each service method call will run in its own thread. Each thread has its own stack frame and `StringBuilder` instance, so thread safety is not an issue in this context.
    
**In Summary:** In most practical development scenarios, `StringBuilder` is used due to its performance benefits in single-threaded situations. `StringBuffer` is used when thread safety is necessary.



### Four Types of References in Java

1. **Strong Reference**:
    
    - In Java, the default type of reference is a strong reference. The garbage collector will never reclaim an object that is strongly referenced, even if the JVM is running out of memory. In such a case, the JVM will throw an `OutOfMemoryError` instead of collecting the object. To break the connection between a strong reference and an object, you can set the reference to `null`.
    
    **Reference Relationship**:
        ![[Pasted image 20240815232125.png]]
    - An object `o` directly points to an `Object` instance in the heap memory. The `Object` instance will only be eligible for garbage collection when the reference `o` is no longer pointing to it (e.g., if `o` is set to `null`); otherwise, the object will remain in memory.
    
    **Usage**:
    
    - This is the most common reference type, typically used when creating new objects using the `new` keyword.
2. **Soft Reference**:
    
    - `SoftReference` is used to describe objects that are useful but not essential. When there is enough memory, objects with soft references will not be collected. However, if memory is low, the system will reclaim soft reference objects. If memory is still insufficient after reclaiming these objects, an `OutOfMemoryError` will be thrown.
    
    **Usage**:
    ![[Pasted image 20240815232210.png]]
    - Soft references are often used in caching mechanisms where you want to keep objects in memory as long as there is enough space.
    
    **Reference Relationship**:
    ![[Pasted image 20240815232231.png]]
    - Soft references allow an object to be collected if memory is needed, but they can be used to hold onto objects longer than weak references.
3. **Weak Reference**:
    
    - `WeakReference` refers to objects that are collected by the garbage collector as soon as they are no longer strongly referenced, regardless of the memory situation. Objects referenced only by weak references are collected during the next garbage collection cycle, even if memory is plentiful.
    
    **Usage**:

    - Weak references are useful for solving memory leak issues, such as in the case of `ThreadLocal` variables.
    
    **Reference Relationship**:
    
    - Weak references do not prevent their referents from being made eligible for GC.
4. **Phantom Reference (Less Common)**:

    - Phantom references are the weakest type of reference. An object referenced by a phantom reference may be collected at any time, and you cannot obtain the object through a phantom reference, even before garbage collection occurs. Phantom references must be used with a reference queue.
    
    **Phantom Reference Object Collection Process**:
        ![[Pasted image 20240815232322.png]]
    - Collection of phantom reference objects occurs in two steps:
        1. The reference is cleared, and the reference (which contains a direct memory address) is placed in a reference queue.
        2. The data pointed to by the reference in the queue is then reclaimed.
    
    **Usage**:
    
    - Phantom references are typically used for managing direct memory. For example, they are used in NIO and Netty for managing `DirectByteBuffer`.
    
    **Additional Note**:
    
    - Garbage collectors usually operate within the memory managed by the JVM. To improve efficiency when calling functions from the operating system kernel, some object data may need to be stored in the operating system's memory. Phantom references allow the JVM to reference this data in the operating system's memory without copying it again (e.g., in NIO for network data operations).


### Java Inner Classes

In Java, when a class is defined within another class, it is called an inner class. Inner classes are generally divided into four types: member inner classes, local inner classes, anonymous inner classes, and static inner classes.

1. **Member Inner Class**:
    
    - A member inner class can access all the members (including private members and static members) of its outer class unconditionally. Note: If the member inner class has a member variable or method with the same name as one in the outer class, a shadowing effect occurs, and by default, the member of the inner class is accessed.
        
    - To access the outer class’s member with the same name, use: `OuterClass.this.memberVariable` (or method).
        
    - The key difference between a member inner class and a static inner class is that a member inner class is highly dependent on an instance of the outer class and is not allowed to define any static members. In contrast, a static inner class is more independent of the outer class.
        
2. **Anonymous Inner Class**:
    
    - An anonymous inner class is the only type of class that does not have a constructor. Most anonymous inner classes are used for interface callbacks (e.g., creating a new `Runnable` instance in a `Thread`).
3. **Local Inner Class**:
    
    - A local inner class is defined within a method or a specific scope. Its access is limited to within the method or that scope. Like a local variable in a method, a local inner class cannot have `public`, `private`, `protected`, or `static` modifiers.
4. **Static Inner Class**:
    
    - A static inner class is an inner class modified with the `static` keyword. A static inner class does not need to depend on an instance of the outer class (similar to static member properties) and cannot access non-static variables or methods of the outer class. To access them, you would need to create an instance of the outer class first.

#### Interview Questions:

- **Why use inner classes?**
    
    - Inner classes can access all the data of the outer class, including private data.
    - Inner classes can be hidden from other classes in the same package.
    - Anonymous inner classes are convenient when defining callback interfaces without writing a lot of code.
    - They can be used to achieve "multiple inheritance" (by defining multiple inner classes to inherit from and calling them). However, this can break encapsulation.
- **Why must variables accessed by inner classes be `final`?**
    
    - This addresses the issue of the inconsistency between the lifecycle of local variables and the lifecycle of inner class objects.
    - Due to lifecycle differences, local variables in a method are released once the method finishes execution. The `final` keyword ensures that the variable always points to the same object. Although inner classes and outer classes are at the same level, the inner class won’t be destroyed just because it is defined within a method. The problem arises if the variable in the outer class method is not defined as `final`. When the outer class method finishes execution, the local variable would be garbage collected (GC), but if an inner class method that references this variable hasn’t finished executing, it would no longer be able to find the variable. If the variable is defined as `final`, Java will copy this variable as a member variable inside the inner class. Since the value modified by `final` cannot be changed, the memory region pointed to by this variable will remain constant.


### Static:

- `static` can modify inner classes, methods, variables, and code blocks.
    
- **Static Inner Class**:  
    A class modified by `static` is a static inner class.
    
- **Static Method**:  
    A method modified by `static` is a static method, which means the method belongs to the class itself rather than to any specific instance. Static methods cannot be overridden but can be overloaded. They are called using the format `ClassName.methodName()`. In static methods, you cannot use the `this` or `super` keywords.
    
- **Static Variable**:  
    A variable modified by `static` is a static variable (also called a class variable). Static variables are shared among all instances of a class and do not depend on any object. There is only one copy of a static variable in memory, and it is allocated memory only once when the JVM loads the class.
    
- **Static Code Block**:  
    A code block modified by `static` is called a static code block. Static code blocks are executed only once when the class is loaded, and they are typically used for program optimization.
    
### Final:

- **Final Class**:  
    A class modified by `final` cannot be inherited.
    
- **Final Method**:  
    A method modified by `final` cannot be overridden.
    
- **Final Variable**:  
    A variable modified by `final` is a constant.
    
**Note**:  
When `final` modifies a reference type, the reference itself cannot be changed (the address cannot be redirected to another address), but the internal values of the object can be modified (as long as the internal values are not themselves `final`).

e.g.
![[Pasted image 20240816141624.png]]


### Interfaces & Abstract Classes

**Syntax**:

- **Abstract Class**:  
    An abstract class can have both abstract and non-abstract methods. It can also have constructors.
    
- **Interface**:  
    In an interface, all methods are abstract by default, and all properties are constants, automatically marked as `public static final` (before Java 8). From Java 8 onwards, interfaces can have methods with implementations (marked with `default` or `static`), but these implementations are typically minimal or empty.
    

**Design**:

- **Abstract Class**:  
    Abstract classes are used to extract and reuse methods that are common to multiple classes.
    
    For example, if both `userDao` and `otherDao` have the same `crud` method, you would need to write the `crud` method in both classes if using interfaces. However, by using an abstract class, you can extract the `crud` method into the abstract class (with a concrete implementation, not an empty one), so `userDao` and `otherDao` don't need to implement this common method separately.
e.g.
![[Pasted image 20240816142107.png]]

- **Interface**:  
    Interfaces are often used in frameworks or architectures to define methods that should be implemented, without worrying about the specific implementation. This allows different layers of an application to expose interfaces for others to call, and even if the implementation changes, the original interface can still be used to make calls.
    
    Interfaces can be **multiple inherited**, whereas classes can only be **singly inherited**.
e.g.
![[Pasted image 20240816142121.png]]

### Auto-Boxing and Unboxing with `Integer` and `int`

```java
Integer i1 = new Integer(12);
Integer i2 = new Integer(12);
System.out.println(i1 == i2);    //false, because they point to different addresses in heap memory.
```
- **Explanation**: When using `new Integer(12)`, two distinct `Integer` objects are created in heap memory, hence `i1` and `i2` do not refer to the same object, resulting in `false` when compared using `==`.

```java
Integer i3 = 126;
Integer i4 = 126;
int i5 = 126;
System.out.println(i3 == i4);    //true, because values within the range -128 to 127 are cached, and i3 and i4 point to the same object.
System.out.println(i3 == i5);   //true, because auto-unboxing occurs, comparing the primitive values.
```
- **Explanation**: For values between `-128` and `127`, Java caches these values in the `Integer` pool. Thus, `i3` and `i4` refer to the same cached `Integer` object. When comparing `i3` (an `Integer`) with `i5` (an `int`), auto-unboxing occurs, and the comparison is based on the primitive value, which is `126` in both cases.

```java
Integer i6 = 128;
// Integer i6 = new Integer(128);   When the value is outside the -128 to 127 range, auto-boxing is equivalent to new Integer(128)
Integer i7 = 128;
int i8 = 128;
System.out.println(i6 == i7);    //false, because they point to different objects in the heap as 128 is not cached.
System.out.println(i6 == i8);    //true, because auto-unboxing compares the primitive values.
```
- **Explanation**: For values outside the `-128` to `127` range, `Integer` objects are not cached. Hence, `i6` and `i7` refer to different objects, making `i6 == i7` return `false`. However, `i6 == i8` returns `true` because `i6` is unboxed to compare the primitive values.

**Key Point**:
- Auto-unboxing compares the numeric values, so when comparing an `Integer` with an `int`, the comparison is based on the actual numeric value, not the object reference.


### Widening Conversion (Implicit Conversion)

Widening conversion happens when a data type of smaller size is automatically converted to a data type of larger size. This type of conversion is safe because the larger data type can easily accommodate the smaller one without losing any information.

Here’s the order of widening conversions:
![[Pasted image 20240816144803.png]]
- **Lower-level types (smaller types)** can be **automatically converted** to higher-level types (larger types) without any explicit casting. This is known as **widening conversion** and is safe because there's no risk of data loss.
    
- **Higher-level types (larger types)** cannot be automatically converted to lower-level types (smaller types) because there's a risk of losing information (e.g., losing precision or range). This requires **explicit casting** by the programmer, and without it, the code will cause a compilation error.


### serialVersionUID in Java

- **Role of serialVersionUID**:
    - `serialVersionUID` is a member variable that can be declared when implementing the `Serializable` interface.
    - The `Serializable` interface is used for object serialization and deserialization in Java. `serialVersionUID` provides a consistent identification mechanism to differentiate between classes during serialization and deserialization.

### Example:

1. **Identification Role**:
    
    - Suppose client A and server B both have a `Person` class for network or file transfer. If both classes specify the same `serialVersionUID`, such as `1L`, they can successfully deserialize each other’s objects.
    - However, if another program C also uses a `Person` class but specifies a different `serialVersionUID`, such as `2L`, then C's `Person` class will not be deserialized by A or B. The `serialVersionUID` serves as an identifier to ensure compatibility during deserialization.
2. **Version Management**:
    
    - If the `Person` class is updated, for instance, by adding a new field, but the `serialVersionUID` remains as `1L`, it will still be compatible with the previous version and can deserialize objects created by the old version.
    - If after the update, the `serialVersionUID` is changed to `2L`, the new `Person` class will not be able to deserialize objects created by the old version. This is a manual version management mechanism that helps control class evolution.

In summary, `serialVersionUID` is crucial for ensuring compatibility between different versions of a serialized class. It helps in managing object versions during the serialization and deserialization process, making it a key aspect of Java's serialization mechanism.


### Exception Hierarchy in Java
![[Pasted image 20240816152609.png]]
- **Error**: These are exceptions generated by the JVM, such as `StackOverflowError` and `OutOfMemoryError` (OOM). They indicate serious problems that a reasonable application should not try to catch.
    
- **Exception**: These are exceptions that occur due to less robust code, often reflecting issues that can be handled or corrected by the program.
    

#### Examples of Common Exceptions:

- **Runtime Exceptions**:
    
    1. **ArithmeticException** - Occurs when an illegal arithmetic operation is performed, such as division by zero.
    2. **NullPointerException** - Occurs when attempting to use `null` in a case where an object is required.
    3. **ClassCastException** - Occurs when attempting to cast an object to a subclass of which it is not an instance.
    4. **ArrayIndexOutOfBoundsException** - Occurs when trying to access an array element with an invalid index.
    5. **NumberFormatException** - Occurs when trying to convert a string into a numeric type, but the string doesn't have an appropriate format.
- **Checked Exceptions (Non-Runtime Exceptions)**:
    
    1. **IOException** - Related to input/output operations, like reading from or writing to a file.
    2. **SQLException** - Occurs when there is a database access error.
    3. **FileNotFoundException** - Occurs when attempting to access a file that does not exist.
    4. **NoSuchFileException** - Similar to `FileNotFoundException`, occurs when an attempt is made to access a non-existent file.
    5. **NoSuchMethodException** - Occurs when a particular method cannot be found.

#### Why Do Frameworks Often Throw Runtime Exceptions?

Frameworks often throw runtime exceptions because they are designed to enforce a set of rules or conventions. When a developer violates these rules—such as by misconfiguring the framework or using it in an unintended way—the framework considers it a logical error. By throwing a runtime exception, the framework signals that the program is in a state where it cannot continue to function correctly, leaving the developer responsible for fixing the issue.

Runtime exceptions allow for cleaner code, as they do not require explicit handling like checked exceptions, and are often used in scenarios where the problem is considered a programming error rather than a recoverable situation.


Your understanding of these Java concepts is comprehensive. Here's a refined translation with some additional clarifications:

### Reflection
Reflection in Java allows a program to dynamically discover and interact with the properties and methods of a class at runtime. This includes the ability to dynamically invoke methods and assign values to fields. Reflection adds a level of dynamism to Java programs and is a fundamental feature underlying many frameworks.

- **Key API**: The `Class` class is central to reflection in Java.

### Ways to Create Objects in Java
1. **Using the `new` keyword**: The most common way to create an object.
2. **Reflection**: Creating objects dynamically at runtime using reflection, typically through the `Class.newInstance()` method or constructors.
3. **Cloning**: Creating a new object as a copy of an existing object using the `clone()` method.
4. **Serialization**: Deserializing an object from a stream, which creates a new instance.

### `hashCode` in Java
- `hashCode` is a method that returns an integer representation of an object’s memory address. It is often used in hash-based collections like `HashMap`, `HashSet`, and `Hashtable` to optimize lookup times.
  
- **Key Points**:
  - If two objects are equal according to the `equals(Object)` method, they must have the same `hashCode`.
  - If `equals()` is overridden, `hashCode()` should also be overridden to maintain consistency.
  - Having the same `hashCode` doesn’t guarantee that two objects are equal, but it indicates that they might be stored in the same bucket in a hash-based structure.

### JDBC Operation Steps
1. **Load the Driver**: Load the JDBC driver for the specific database.
2. **Establish a Connection**: Connect to the database using the `DriverManager.getConnection()` method.
3. **Execute SQL Statements**: Use a `Statement` or `PreparedStatement` object to execute SQL queries.
4. **Process Results**: Handle the results returned from the SQL queries.
5. **Close Resources**: Ensure all resources such as connections, statements, and result sets are closed.

### Data Source (DataSource)
- A **DataSource** in Java is an object that provides connections to a database. It abstracts the details of obtaining a database connection and can either represent a connection pool or a single connection to the database.

- **Common Data Source Technologies**:
  - **C3P0**
  - **DBCP** (Apache Database Connection Pool)
  - **Proxool**
  - **Druid** (from Alibaba)

- **Advantages of Using Data Sources**:
  - **Druid** offers several benefits:
    - **Performance Monitoring**: Tracks and monitors database access performance.
    - **SQL Execution Logging**: Logs SQL execution details for analysis.
    - **SQL Firewall**: Provides security features like blocking malicious SQL queries.

### Access Modifiers in Java
Java provides four access levels for class members (fields, methods, etc.):

1. **public**: Accessible from any class.
2. **protected**: Accessible within the same package and by subclasses.
3. **default (package-private)**: Accessible only within the same package. If no modifier is specified, this is the default.
4. **private**: Accessible only within the same class.

- **Visibility Chart**:

| Modifier    | Same Class | Same Package | Subclass | Other Packages |
|-------------|------------|--------------|----------|----------------|
| `public`    | Yes        | Yes          | Yes      | Yes            |
| `protected` | Yes        | Yes          | Yes      | No             |
| `default`   | Yes        | Yes          | No       | No             |
| `private`   | Yes        | No           | No       | No             |













---

### New Features of Each JDK Version
#### JDK 8：
1. Lambda表达式
2. Stream流操作
3. 函数式接口
4. 方法引用
5. Optional类
6. Base64编解码
7. 时间API Date类优化

Lambda表达式，配合函数式接口在创建线程这方面很方便；

Stream适合对一些集合类进行操作，首先创建流，然后是中间链操作（筛选(filter, limit, distinct...)、映射(map)、修改(peek)），最后结束操作（匹配与查找(find, match, foreach... )、约束(reduce)、收集(collect）；  
比如`List<Integer>`和`int[]`之间的转换可以采用Stream流直接操作。

Optional类就是一个可以为null的容器对象，原对象如果为null可能会抛异常，需要严格的非空校验的情况可以用到。

Java8还整改优化ConcurrentHashMap与HashMap（每个hash槽上对应的链表大小超过8变为红黑树）、优化synchronized使其开销更小、JVM底层放弃永久代改为元空间；
#### JDK 9：
- **模块化系统（Project Jigsaw）**：
    
    - JDK 9 中最大的变化是引入了模块化系统（Java Platform Module System），通过 `module` 声明可以将 Java 程序划分为模块。这样可以更好地组织代码，减少类之间的耦合，并且提高了应用程序的可维护性和性能。
- **JShell：交互式Java工具**：
    
    - JDK 9 引入了 JShell，这是一个交互式的 REPL（Read-Eval-Print Loop）工具，使得开发者可以在没有创建完整程序的情况下，快速地测试 Java 代码片段。
- **改进的JVM编译器接口（JVMCI）**：
    
    - JVMCI 是一个新的 Java 编译器接口，允许新的动态编译器（如 Graal）与 JVM 一起运行，为高性能应用程序提供更灵活的编译和优化功能。
- **多版本兼容的JAR文件**：
    
    - JDK 9 引入了多版本 JAR 文件的支持，使得开发者能够在同一个 JAR 文件中包含不同 JDK 版本的类，以便在不同的 Java 版本中运行。
- **流（Stream API）改进**：
    
    - Stream API 在 JDK 9 中得到了扩展，增加了新的方法，例如 `takeWhile`、`dropWhile` 和 `iterate`，这些方法使得处理流操作更加方便和直观。
- **HTTP/2 客户端**：
    
    - JDK 9 引入了对 HTTP/2 的支持，并且提供了一个新的 HttpClient API，尽管它在 JDK 9 中是一个孵化器模块（Incubator Module），但为之后的正式支持奠定了基础。
- **集合工厂方法**：
    
    - JDK 9 提供了简单的工厂方法来创建不可变集合，例如 `List.of()`、`Set.of()` 和 `Map.of()`，这使得集合的创建更加简洁和高效。
- **私有接口方法**：
    
    - JDK 9 允许在接口中定义私有方法，帮助开发者更好地组织代码逻辑，同时隐藏实现细节。
- **G1垃圾收集器的改进**：
    
    - JDK 9 将 G1 设为默认的垃圾收集器，并且在其基础上进行了性能优化，尤其是降低了全局暂停时间。


#### 之后的版本主要就是进行一些性能上的优化了，例如优化GC等




















---



1. What is the difference between a Stack and a Queue?

●  A Stack is a Last In First Out (LIFO) data structure, where elements can be added or removed only from one end. A Queue is a First In First Out (FIFO) data structure, where elements are added at the rear and removed from the front.

2. Can you explain the concept of SOLID principles?

●  SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable..

●  Single Responsibility: A class should have one reason to change.

●  Open/Closed: Software entities should be open for extension, but closed for modification.

●  Liskov Substitution: Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.

●  Interface Segregation: No client should be forced to depend on methods it does not use.

●  Dependency Inversion: Depend upon abstractions, not concretions.

3. What is the difference between checked and unchecked exceptions in Java?

●  Checked exceptions are checked at compile-time and must be either caught or declared in the method's throws clause. Unchecked exceptions are not checked at compile-time and are typically caused by programming errors, such as `NullPointerException` or `ArrayIndexOutOfBoundsException`.

4. How do you handle and manage exceptions in your code?

●  Use try-catch blocks to handle exceptions. Declare throws for methods that might throw an exception but do not handle it. Use finally or try-with-resources for cleanup. Avoid catching overly broad exceptions like `Exception` or `Throwable`.

5. What is the difference between HashMap and Hashtable in Java?

●  `HashMap` allows one null key and multiple null values, is not synchronized, and allows non-deterministic order. `Hashtable` does not allow null keys or values, is synchronized, and ensures thread safety.

6. Can you explain the Java Memory Model and its components?

●  The Java Memory Model defines how threads interact through main memory. It consists of the method area, heap, and thread stacks. The method area is shared among threads, while each thread has its own stack and part of the heap.

7. What is the difference between a local variable and an instance variable in Java?

●  Local variables are declared within methods and are only accessible within that method. Instance variables are declared in the class but outside any method and are accessible from any instance method of the class.

8. How does Java manage garbage collection?

●  Java uses a garbage collector to automatically reclaim memory by identifying objects that are no longer reachable in the program and freeing the memory they occupy.

9. What is the difference between an interface and an abstract class in Java?

●  An interface can contain only abstract methods (methods without an implementation) and default methods (since Java 8). An abstract class can contain both abstract and concrete methods and cannot be instantiated.

10. Can you explain the concept of multithreading in Java?

●  Multithreading is the ability of a program to execute multiple threads concurrently. Each thread represents a separate path of execution and can run in parallel with other threads.

11. What is the purpose of the 'synchronized' keyword in Java?

●  The `synchronized` keyword is used to lock a method or a block of code so that only one thread can execute it at a time, preventing race conditions in multithreaded environments.

12. How do you design a RESTful API?

●  Design RESTful APIs by using standard HTTP methods (GET, POST, PUT, DELETE), organize resources into a hierarchical URL structure, use status codes and headers appropriately, and ensure statelessness.

13. What is the difference between SOAP and REST?

●  SOAP (Simple Object Access Protocol) is a protocol for exchanging structured information using XML. REST (Representational State Transfer) is an architectural style for building web services using HTTP methods and without requiring a specific protocol.

14. Can you explain the concept of Singleton pattern and its use cases?

●  The Singleton pattern restricts the instantiation of a class to one single instance and provides a global point of access to that instance. It's useful for managing shared resources like configuration settings or database connections.

15. What is Dependency Injection and how is it used in Java?

●  Dependency Injection is a design pattern that allows objects to be created with their dependencies provided externally, rather than hard-coding them within the object. It's used to increase modularity and make the system easier to test.

16. What is the difference between a Servlet and a JSP in Java EE?

●  A Servlet is a Java class that generates dynamic content and is part of the Java EE specification. A JSP (JavaServer Pages) is a technology that uses HTML mixed with Java code to dynamically generate web content.

17. Can you explain the concept of MVC (Model-View-Controller) architecture?

●  MVC is a design pattern that separates an application into three interconnected components: Model (data and business logic), View (user interface), and Controller (handles user input and manages communication between Model and View).

18. What is the role of the Spring Framework in Java development?

●  The Spring Framework is a comprehensive programming and configuration model for modern Java applications. It provides infrastructure support for developing Java applications, including inversion of control, data access, transaction management, and more.

19. Can you explain the different types of Spring Beans and their scopes?

●  Spring Beans are objects that form the backbone of a Spring application. They are configured and managed by the Spring container. Scopes include singleton (one instance per Spring container), prototype (a new instance every time an object is asked for), request, session, and application.

20. What is the difference between Spring MVC and Spring Boot?

●  Spring MVC is a part of the Spring Framework that provides model-view-controller framework for building web applications. Spring Boot is a project that simplifies the bootstrapping and development of new Spring applications.





