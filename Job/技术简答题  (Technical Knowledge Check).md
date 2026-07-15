## Tips
立自己的人设

讲故事的原则：PAR (Problem, Action, and Result)

TODO 讲关于技术问题的故事

如何定位OOM问题：结合B站回答，检查JVM各种区域

更新简历，更新Cover Letter模板

ELK日志平台


# 问题

Stream熟练使用 - TODO
AOP日志拦截器，记录入参出参耗时 - done
很多高级 API 会在 `ApiResponse` 里加入一个 **`requestId`**。 - TODO UUID定位每一个请求，方便日志查看，框架实现
- **做法**：在过滤器里生成一个 UUID 存入 `ThreadLocal`，然后在 `ApiResponse` 生成时放进去。 -- 用MDC实现了，其原理也是一个ThreadLocal
- 这个request_id没有必要入库，除非非常重要的交易类的请求





> "I prefer **Constructor Injection** via Lombok's `@RequiredArgsConstructor`. It's the recommended practice by Spring because it promotes **immutability** with `final` fields, makes **unit testing** much easier without needing a Spring context, and helps detect **circular dependencies** early at startup."




controller里各种注解，@RequestBody就是Body的数据，常见的就是form表单和raw JSON体？用哪种好？如何取舍？建议用body？ -- ok

参数前的@Valid（用于spring validation的校验），@RequestParam，@RequestBody要和不要都可以，怎么取舍，单独传一个对象也会被映射过去？ 
常见HTTP返回码，业务和系统级的，返回数据整理； -- done
RESTFUL 接口规范 - 尽量严格按照，post，put，get，delete这些都重新再学习学习 - done
请求体永远优先使用 **JSON + `@RequestBody`** 









- **Dependency Injection:** "Why do we use DI instead of creating objects manually with `new`?"
    
    - 主要是**Decoupling（解耦）**，降低了开发者维护对象的开销，更专注于业务开发。我个人感觉DI用处最大的就是使用Service之类的地方，@Autowired注解就能很好的使用了，以及spring会统一管理他们生命周期，无需程序员考虑...你觉得呢，再多列举点？以及这个问题应该回答用DI和用new的对比吧



- **Bean Scopes:** "What is the default scope of a Spring Bean, and can you name another one?"
    
    - _重点：_ **Singleton** (default) vs **Prototype**。还有request。。。但是没用到。。。这里要结合并发理解下



---
### **dive deeper 深入交流** - 技术面的开始，面试官可能根据经历问问题
I’ve worked in two main areas.

At **Huawei**, I developed software for **microwave products**. I worked on features like data monitoring, hardware deployment, and system upgrades. The tech stack included **Java microservices** and some **Android** development.

Later, I joined **China Minsheng Bank** and worked on the **Commercial Bill System**. This was a **very important transaction system** in the bank.

It was a full-stack project using **Vue** on the frontend and **Java** on the backend. We had many **upstream and downstream systems**, so we had many **RESTful APIs** to **interact with them**. We also used **Redis** for caching and **MQ** to **assure reliability**.

In my role, I was responsible for the **coding** and also **led a small team** to deliver core features like **acceptance, endorsement, and discounting**. I was involved in the **full lifecycle** of the project—from the early requirement discussions to the final deployment and support.



## 术语解释
### 1. Spring Boot

- **IoC (Inversion of Control):** The framework manages the objects instead of the developer. It makes the application more flexible and decoupled.
    
- **DI (Dependency Injection):** We pass dependencies into a class instead of creating them with `new`. This makes **Unit Testing** very easy because we can use Mocks.
    
- **AOP (Aspect-Oriented Programming):** We separate common tasks—like logging or security—from the main business logic. It keeps the code clean.
    
- **Bean Scopes:** **Singleton** means there is only one instance in the whole app. **Prototype** means a new instance is created every time you ask for it.
    
- **RestController vs Controller:** A **RestController** returns data (like JSON). A regular **Controller** usually returns a web page (HTML).
    
### 2. JPA / SQL

- **Indexing:** It is like a book index. It makes **Read** operations much faster, but it makes **Write** operations a bit slower.
    
- **1+N Problem:** This happens when you run one query for a list and then N more queries for the details. It is bad for performance. We fix it using **Join Fetch**.
    
- **Transaction Management:** It ensures that a group of database actions either all succeed or all fail together. This keeps the data safe.
    

### 3. AWS
// TODO
- **S3:** A simple service to store files like images or documents. It is cheap, safe, and can grow infinitely.
    
- **EC2:** A virtual server in the cloud. You can install anything on it and have full control.
    
- **Lambda:** A "serverless" function. You just upload the code and it runs. You only pay when it is active.
    
- **RDS:** A managed database service. AWS handles the backups, security patches, and updates for you.
    
- **IAM:** A tool to manage user permissions. We use it to follow the **"Least Privilege"** rule—only give users the access they really need.
    

### 4. Java

- **Stream API:** A modern way to process data collections. It makes the code shorter and much easier to read.
    
- **Optional:** A container for a value that might be null. It helps us avoid the famous **NullPointerException**.
    
- **Multithreading:** Running many tasks at the same time to save time. In Java, we use **Thread Pools** to manage this efficiently.
    
- **Exceptions:** **Checked** exceptions must be handled with a `try-catch`. **Unchecked** (Runtime) exceptions are usually logic errors that happen while the app is running.



### 并发，保证集合性能，保证线程安全
#### 高并发场景保证集合安全
选择集合类



#### 线程池优化并发性能
选择合适线程池，可缓存，配参数




#### 优化死锁
reentranctlock 竞争策略



---
## 基础问题，Engineering Culture

#### Why React over Angular?

> "I chose React because it has a **huge ecosystem** and a very strong community in Australia. It is **component-based**, which makes the code more reusable and easier to maintain. Also, React’s learning curve is smoother, allowing the team to deliver features faster."

#### Data Flow (Frontend to Database)

> "The React frontend sends a **JSON request** via a REST API. The Spring Boot **Controller** receives it and maps the data into a **DTO**. Then, the **Service layer** processes the business logic. Finally, we use Mybatis or Hibernate JPA framework to interact with the database and do data operation."


### How do you handle production issues?
客户经理收到客户反馈找到我们，给我们描述相关问题；我们根据票号、时间、类型等交易信息去日志平台定位到相关失败的日志；通过日志分析可能原因；同时也会在本地和测试环境复现相关错误，分析原因并给出解决方案；发现问题后我们会给客户经理将问题原因，提出临时方案降低影响，以及确定是我们的问题的话会创建相关修复的jira任务，并沟通好投产窗口完全修复这个问题；如果我们发现需要其它上下游系统也需要改，也会协调各方面一起排查及修改问题。
> "When a Customer Manager reports an issue based on client feedback, I would communicate with him thoroughly about the transaction scenario.
> 
> Then, I use transaction details—like **bill IDs, timestamps, and transaction types** to locate the failure logs on our logging platform (like ELK). After analyzing the logs for potential causes, I try to **reproduce the issue** in my local and environments to confirm the root cause.
> 
> Once the issue is identified, I communicate with the Customer Manager to explain the cause and provide a **temporary plan** to minimize the impact on the client. If the issue is on our side, I create a **Jira task** for a permanent fix and coordinate a **release window**.
> 
> If the problem involves other systems, I also take the lead in **coordinating with upstream and downstream teams** to investigate and fix the issue together."


#### What do you look for during a Code Review?


> "When I perform a Code Review, I focus on several areas:
> 
> - **Business Logic:** This is my top priority. I ensure the logic is correct and handles all the necessary validations, edge cases and business scenarios. 
>     
> - **Code Quality:** I look for clean structure and **clear naming conventions**. The code must be readable and easy for other developers to maintain in the long run.
>     
> - **Deployment Accuracy:** I carefully check everything related to the release, such as **SQL scripts, configuration files, and feature toggles**, to ensure nothing is missing or incorrect.
>     
> - **Testing and Coverage:** Finally, I verify the **unit tests** and check the **code coverage**. It's important to ensure that the new changes don't break any existing functionality."
>

#### How do you handle a situation where you need to deliver a feature fast, but the code quality might suffer?

如果这个是个紧急bug，我会以尽快修复上线为主；如果这是个正常需求，我会权衡取舍，尽量保证代码质量的情况下优先保证上线，并在后期记录为 **Technical Debt**，之后有机会会计划重构，不过我会尽量避免这种做法，一次做好设计保证代码质量，因为这种情况会导致团队花费更大投入在这个方案上。

"It depends on the situation. If it's an **urgent production bug**, my priority is to fix it and get it deployed as quickly as possible to minimize the impact.

However, for a **normal requirement**, I try to find a balance. I will prioritize the delivery but maintain as much code quality as possible. If we take any shortcuts, I make sure to record them as **Technical Debt** in the backlog so we can plan a refactor later.

That being said, I always prefer to **'do it right the first time'** with a solid design. In my experience, technical debt usually ends up costing the team much more time and effort in the long run, so I try to avoid it whenever I can."

#### How do you know if your application is healthy in Production? (Monitoring)
我们原来有专门的运维团队保障，不过我也知道可能需要看**Health checks**, 生产日志, Metrics (CPU/Memory/Latency)这些指标
"In my previous roles, we had a **dedicated Operations (Ops) team** to handle the heavy lifting of 24/7 monitoring.
However, as a developer, I would also actively monitor the application’s health. I mainly look at **Health Check Endpoints**, **Production Logs**, and **Key Metrics** like **CPU usage, memory consumption, and latency**."


#### Can you walk me through your development process? / Have you used Agile or Jira?
（能说下你们的开发流程吗？用过敏捷/Jira吗？）
讲讲需求如何一步步上线的

**Answer:**  
Our requirements come from different sources — client managers, other systems, and also our own optimizations or bug fixes. We have a release window every two weeks. The team leader decides which tasks will be released in which window, then creates Jira tickets and assigns them to developers.

From there, we follow the standard process: requirement analysis, solution design, coding, testing, and finally deployment. At each key step, we update the Jira ticket and move it to the next responsible person. We also have a fixed template and rules for creating Jira tasks, and we follow them strictly.

**关键词**：**release window**, **Jira tickets**, **requirement analysis**, **solution design**, **integration**, **deployment**, **templates & rules**




//TODO
描述一个你在工作中遇到的最有挑战的技术问题，你是如何定位（Debug）的？（这个问题区别于BQ中的挑战，这个是讲一个解决技术难点的故事）
这个问题我以前没有做过技术攻坚，不知道从何答起，有哪些思路吗？




---
## Java

### 多线程，并发
**Question: How do you handle thread safety in your Java application?**

> "In a Spring environment, most of my beans are **Stateless Services**, which are inherently thread-safe.
> 
> For shared data, I prefer using **Concurrent Collections** like `ConcurrentHashMap` or **Atomic variables**. I also try to keep objects **Immutable** as much as possible. If I really need to manage a shared resource, I use `synchronized` blocks or `ReentrantLock`, but I keep the locked section as small as possible to avoid performance issues."

### 锁


### 常见数据结构及其特性与选择（不知道这个会不会考到）




### 现代语法

- **Exception Handling:** "What is the difference between **Checked** and **Unchecked** exceptions? When would you use each?"
checked exception - 编译期间就要解决，否则代码飘红根本没法启动；
unchecked exception - 运行时异常，需要try catch捕获异常或使用统一异常处理：
我们以前系统中会有统一的异常处理。我自己开发也会倾向于包装一层`@RestControllerAdvice` 统一异常处理。业务异常建议封装暴露一些细节给前台，可以引导用户发现问题；而非业务的普遍性的运行异常，涉及系统底层的需要控制输出的内容，确保系统底层和内部技术细节不被过多暴露。


**Question: What is Type Erasure in Java?** (类型擦除)

> "Type Erasure means that Java removes all generic type information during **compilation**. For example, an `ArrayList<String>` becomes just a raw `ArrayList` in the bytecode. Java does this for **backward compatibility** with older versions that didn't have generics. This is why you cannot check generic types at runtime using `instanceof`."

// 这里底层原理很拗口，知道怎么用泛型就行了


**Question: What is a Functional Interface?** (函数式接口)

> "A Functional Interface is an interface that has exactly **one abstract method**. It is the foundation for **Lambda expressions** in Java 8. Common examples are `Runnable` or `Comparator`. You can use the `@FunctionalInterface` annotation to ensure the interface stays functional and doesn't accidentally get a second method."

// 使Java能够函数式驱动写代码，使用一段lambda表达式就能驱动，而不需要new一堆数据，代码更加简洁了

- **Streams:** "What are the benefits of using the **Stream API** over traditional `for` loops?"
	// TODO 上手体验 
    - _重点：_ 函数式编程、可读性、易于并行处理。并且底层有很多优化，处理集合数据时省去了很多样板代码。



- **Java 8+ Features:** "Why should we use `Optional` instead of returning `null`?"
	// TODO 自己去敲demo上手体验
    重点：_ 防止 `NullPointerException` 及其对代码可读性的提升。可做非空校验，代码清晰可读。但是正常逻辑还是需要判空，只是从传统的判断`xxx == null`, 变成`ifPresent()`,Optional 让人有意识去判空








## Spring


**Question: Difference between @Component, @Service, and @Repository?**

> "Technically, they are all the same—they register the class as a **Spring Bean**. However, we use them for different layers:
> 
> - **@Service** is for business logic.
>     
> - **@Repository** is for the data layer. It has a special feature: it automatically translates database-specific exceptions into Spring’s `DataAccessException`. 使用Mybatis框架不用额外标注@Repository
>     
> - **@Component** is a general-purpose annotation for any bean that doesn't fit into the other categories."
>     



**Question: What happens if a @Transactional method calls another method in the same class?**
// TODO 结合代码和结果进行理解
AOP 代理失效问题（Self-invocation issue）
> "The transaction will **fail to start** for the second method. This is the **self-invocation issue**. Spring uses **AOP proxies** to manage transactions. The proxy only intercepts calls coming from _outside_ the class. If a method calls another method in the same class, it bypasses the proxy. To fix this, you should move the second method to a different service."

如果这样做的话会导致事务失效，如果在类的内部调用，相当于绕过了代理机制。



### Explain the concept of Dependency Injection in Spring.

> _面试官：_ "What is Dependency Injection?"
> 
> _你：_ "It's a design pattern where objects don't create their own dependencies. Instead, Spring injects them at runtime. (**What**) This makes the code decoupled and much easier to unit test because I can easily mock the dependencies. (**Why**) In our project, they are used everywhere"

- **Dependency Injection:** "Why do we use DI instead of creating objects manually with `new`?"
    
    - 你提到的**Decoupling（解耦）**、易于单元测试（Mocking）。但是我觉得易于单元测试不值得单拎出来啊。。。我个人感觉DI用处最大的就是使用Service之类的地方，@Autowired注解就能很好的使用了，以及spring会统一管理他们生命周期，无需程序员考虑...你觉得呢，再多列举点？以及这个问题应该回答用DI和用new的对比吧


        
- **Bean Scopes:** "What is the default scope of a Spring Bean, and can you name another one?"
    
    - _重点：_ **Singleton** (default) vs **Prototype**。还有request。。。但是没用到。。。这里要结合并发理解下



---

## Database & SQL

### Database Indexing (Practical Usage)
**Question: What is an index, and when should you use it?**

> "An index is a data structure (like a B-Tree) that improves the speed of data retrieval. You should create indexes on columns that are frequently used in **WHERE** clauses, **JOIN** conditions, or **ORDER BY** statements. However, you should avoid indexing columns with very low cardinality, for example enumerate columns like 'gender' or 'type'."

**Question: When does an index fail (become ineffective)?**

> "An index usually fails when you use a **leading wildcard**, like `LIKE '%abc'`. It can also fail if you perform functions on the indexed column (e.g., `WHERE UPPER(name) = 'YORICK'`) or if the database engine decides a full table scan is faster because the table is very small."

// TODO explain 可以分析query plan，分析sql语句是否走索引


**Question: Can an index make the database slower?**

> "Yes, for **Write** operations. Every time you `INSERT`, `UPDATE`, or `DELETE`, the database must also update the index tree. If a table has too many indexes, write performance will drop significantly."


### Locking (Concurrency Control)

**Question: Optimistic vs. Pessimistic Locking?**

> "In practice, I usually prefer **Optimistic Locking** for high-concurrency web apps. It uses a **version column** and doesn't actually lock the database row, so it's much faster when conflicts are rare.
> 
> I only use **Pessimistic Locking** (like `SELECT FOR UPDATE`) for critical data—like financial transactions—where data consistency is more important than performance, or when we expect many users to edit the same record at the same time."

// TODO Java开发手册推荐金融敏感数据使用悲观锁


### JPA / Hibernate (Performance)

**Question: What is the 1+N problem, and how do you fix it?**
`@EntityGraph`或写原生SQL
> "The N+1 problem happens when you fetch a list of entities (1 query), and then the framework runs N more queries to fetch related data for each entity. It kills performance.
> 
> I detect it by checking the **Hibernate logs** or using monitoring tools. To fix it, I use **JOIN FETCH** in JPQL or an **EntityGraph** to load all the data in a single SQL query."

// TODO N+1问题是代码设计不合理导致的非常影响性能的问题；它是先进行一次查询，然后拿这次查询的结果再去数据库进行多次查询，由于跟数据库交互进行的开销很大，因此这种操作也极其影响性能，可以通过将查询优化在一次join操作解决。
这个是基础的代码质量问题，团队内是一定要避免的，使用联表查询就可以解决，并且代码审查也要着重查看不能出现这种情况；代码审查避免新出现这种问题，对于遗留的问题可以查看Mybatis的sql执行日志以及一些代码扫描工具
// todo `Entity Graph`是啥，怎么解决这个问题



#### 1. 为什么说“单向关联”不会自动导致 1+N？

你担心的逻辑是：如果 `User` 里有 `List<Task>`，查 `User` 就会把 `Task` 全带出来。

但在 JPA 中，这取决于 **`FetchType`（加载策略）**：

- **`@OneToMany` 默认是 `LAZY`（延迟加载）**：当你执行 `userRepository.findAll()` 时，Hibernate 只会生成一条查询 `users` 表的 SQL。它**绝对不会**去查 `tasks` 表。
    
- **Proxy（代理对象）**：Hibernate 会给 `tasks` 字段塞一个“占位符”。只有当你代码里真的调用 `user.getTasks().size()` 时，它才会去发第二条 SQL。
    

**那 1+N 是怎么来的？** 1+N 通常发生在：你查出 100 个 User，然后在循环里打印每个人的任务数量。这时会产生 1 条查 User 的 SQL + 100 条查 Task 的 SQL。


#### 2. `Task` 类里的 `@ManyToOne` 会有额外开销吗？

这是你最担心的点。我们来看 `Task` 实体：

Java

```
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;
```

##### A. 它是怎么查的？

当你查询 `Task` 时，Hibernate 已经从 `tasks` 表的 `user_id` 列拿到了那个 ID。

- **如果设为 `LAZY`（推荐）**：Hibernate **不会**去查 `users` 表。它会创建一个 `User` 的代理对象，里面只填好一个 `id`。此时，并没有跨表开销。
    
- **如果设为 `EAGER`（默认，不推荐）**：Hibernate 会在查 `Task` 的同时，自动发起一个 `LEFT JOIN` 或者再发一条 SQL 去查 `User`。**这才是你担心的“额外开销”**。
    

##### B. 这里的 `user_id` 是单独查的吗？

是的。因为它就在 `tasks` 表里。 当你调用 `task.getUser().getId()` 时，Hibernate 非常聪明，它知道 ID 已经有了，甚至都不会去触发数据库查询。只有当你调用 `task.getUser().getUsername()` 时，它才会真的去跑一次 `SELECT * FROM users WHERE id = ?`。









这个问题是使用JPA的时候存在关联关系，使用JPA的时候默认会把关联关系也给查出来，从而导致执行两次SQL，比如students表和results表存在关联关系，那么查students第一次查出3个学生，那么还会对这3个学生继续发送3次SQL查询命令查询成绩，这里就是"1+3"次查询。对性能会造成巨大的拖累。
解决办法：
1. 关系设计方面就要避免双向复杂的级联，尽量采用单向关联（Unidirectional），每个entity尽量保持自己本表的数据，这种设计还会导致JPA每次做不需要的查询拿到不需要的数据，比如只想简单查询本表信息；
2. 针对这种多条级联的查询我会自己单独提出来，写复杂查询以及加上分页等，这样还能规避了JPA自动关联查询出一堆不可控的数据
3. 这种设计领域模型更加清晰，性能更可控，并且双向级联可能出现一些框架问题（如Lombok @Data导致的死循环问题）
4. 我宁愿为了性能，可靠，安全性，忽略掉一些便利性，虽然JPA关联操作在某些场景能达到很便利的操作，但是我宁愿单独写查询，或者多几步操作来避免潜在危险。


|**场景**|**推荐做法**|**理由**|
|---|---|---|
|**中小型项目 / 快速迭代**|**单向关联 + LAZY**|利用 JPA 的便利，只要注意不写死循环，性能和开发速度平衡得最好。|
|**高并发 / 微服务架构**|**直接存 `Long userId`**|澳洲很多项目为了应对分布式下的分库分表，会强制要求实体间不能有对象引用。|
|**你的脚手架 (Scaffold)**|**单向关联 + LAZY**|**建议保留 `@ManyToOne(fetch = FetchType.LAZY)`**。因为在面试时，你能讲清楚什么是 `LAZY`，什么是 `Proxy`，这比你只写个 `Long userId` 更能体现你的 Java 深度。|




### SQL Relationship

**Question: What is a 'Joining Table' in a many-to-many relationship?**

> "A Joining Table (or Link Table) is a third table used to connect two other tables. For example, a 'Students' table and a 'Courses' table. The Joining Table stores the **Foreign Keys** from both, allowing one student to have many courses and one course to have many students."


## MQ
使用场景，优缺点，根据我的项目问

## Redis
使用场景，优缺点，结合我的项目...


我理解像购物系统这种多读少写的系统必用redis，如储存商品或者购物车信息等都用到；但是我之前的银行系统，要求强一致性，所以很多数据都是数据库直出；我们系统中唯一用到redis的地方就是业务配置缓存，如利率（还有哪些交易配置信息可以存缓存的），这种配置读多写少，能够快速读取减轻数据库压力。
这些配置数据会在应用启动，也就是系统上线时写入缓存中，并且只有高权限的业务经理能修改，修改后会同时刷新数据库与缓存中的数据。





## AWS & Devops

### AWS

TODO
- **AWS Services:** "If you need to store millions of user-uploaded images, which AWS service would you use and why?"
    
    - _重点：_ **S3** (Scalability, durability, cost-effective)。
        
- **Serverless:** "What are the pros and cons of using **AWS Lambda**?"
    
    - _重点：_ Pros: Scaling, No server management; Cons: **Cold start**, Execution time limits.


### CI/CD
#### How do you use Git and CI/CD (Continuous Integration/ Continuous Deployment/ Delivery)?
We create a branch from the last **production version** on master, develop locally, and then pull into the **release branch**. After all changes are ready, we merge it to master branch.  
Then we use our **internal pipeline** to **build**, check **test coverage**, run **code quality checks**, and finally **deploy** to production. We also update the **config center** and run **DB scripts** if needed.



- **CI/CD Pipeline:** "Describe the stages of a typical pipeline your code goes through before hitting Production."
    
    - _重点：_ Build -> Unit Tests -> Static Code Analysis (SonarQube) -> Deploy to Staging -> Integration Tests -> Prod.



### GIT
#### **Have you ever had problems using Git? How did you solve them?**

**✅ Answer:**

> Yes, our team had issues when several groups worked in parallel. Sometimes different versions caused conflicts or even overwriting.  
> To solve this, we always developed based on stable code. After each release, we merged unfinished work carefully.  
> Also, our team leader reviewed all merges, and we reminded each other to avoid mistakes.



“如果我们要发版了，突然发现线上有个紧急 Bug，你的 Git 操作流程是什么？”（考察你懂不懂 Hotfix 分支流）
定位并解决问题，基于master或现在的uat新拉一个分支，将代码修改完，测试无误后，合入uat再找测试人员测试，然后走正常发版流程啊？



### CLI
#### **Linux Commands**

I use commands like **ls**, **ll**, and **cd** for directories; **cat**, **grep**, and **tail** to check logs; **chmod** for permissions; and **ps** or **kill** to manage processes.


### 用过git高级指令吗
cherry pick




## Plugins / Tools / AI
用过AI工具吗...
GIT，版本控制相关...
效率提升...
接口调用，调试，测试相关








## API & 系统设计

// TODO 常见的HTTP请求码值
- **RESTful Status Codes:** "When would you return a `401` vs a `403`?"
    - _重点：_ `401` 是身份未验证（Who are you?）；`403` 是有身份但没权限（You can't do this）。


### 3. API Design: Idempotency (Deep Dive)

**Question: Which HTTP methods are idempotent, and why is this important?**

> "**Idempotency** means that making the same request multiple times will result in the same state on the server.
> 
> - **GET, PUT, and DELETE** are idempotent. For example, deleting the same resource twice still results in the resource being gone.
>     
> - **POST** is **not** idempotent. Every time you send a POST request, it usually creates a new record.
>     

// TODO 这里提一嘴我们普遍还是使用get和post方法，




**Deep Dive: How do you handle idempotency in a real project?**

> "In practice, idempotency is critical for **payments** or order processing. If a network error occurs and the client retries the request, we don't want to charge the user twice.
> 
> My solution is to use an **Idempotency Key** (usually a UUID) sent by the client in the header. On the server, we check if this key has already been processed in the database or Redis. If it has, we return the previous result without processing the logic again."




**Question: How does JWT work, and why is it called 'stateless'?**
// TODO
这里讲讲JWT项目中的用法


> "JWT is a token that contains user information in its payload. It’s called **stateless** because the server does not need to store any session data in memory or a database. Everything the server needs to identify the user is inside the token itself. This makes it very easy to **scale** the application horizontally."

**Question: What does 'Stateless' mean in the context of a REST API?**

> "It means each request from the client must contain all the information needed to understand and process that request. The server doesn't 'remember' previous requests. This simplifies the server design and makes it more **reliable** and easier to scale."























---
### 1. Spring Boot 实战避坑能力

// TODO
像我们刚才写的 `@RestControllerAdvice` 统一异常处理，这就是极其 standard 且实用的技能。除了这个，面试官最爱考的 Practical 问题是：

- **`@Transactional` 事务失效场景：** 比如同一个类里面，方法 A 调用方法 B，方法 B 上的事务为什么不生效？（这是澳洲 Java 面试的必考题）。
    
- **多环境配置：** 如何优雅地管理 `application-dev.yml` 和 `application-prod.yml`？
    

### Devops
//TODO
看懂简单的 Dockerfile，知道怎么把 Spring Boot 打包成镜像，或者了解一点 GitHub Actions 的基础 CI/CD 流程。


## 踩坑
### Lombok `@Data` 的背刺

**这是很多开发者踩的最大的坑。** `@Data` 自动生成了 `toString()`, `hashCode()` 和 `equals()`。

- 当你打印（或者调试器读取）`Task` 对象时，它会触发 `toString()`。
    
- `Task.toString()` 会去调用 `user.toString()`。
    
- `user.toString()` 又会去调用 `tasks.toString()`。
    
- **结果**：调试时直接触发了全表扫描，甚至导致 `StackOverflowError`








### TODO小故事
澳洲跟国内开发习惯不一样：接口上我国内团队遵循Action-Based URI convention，基本只使用了get和post请求，大量操作都在post的URI里面通过方法名区分，比如加子路径/saveTask的post请求；而了解到澳洲这边遵循规范的RESTFUL标准；