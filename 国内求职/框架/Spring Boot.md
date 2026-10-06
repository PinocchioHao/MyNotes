## Spring Boot 简介 & 进阶
### Spring Boot 的优点
首先，它**继承了 Spring 所有的优秀基因**，包括轻量级、IOC 解耦、AOP 切面以及声明式事务。 但更重要的是，它是基于 Spring 封装的脚手架，核心理念是‘约定大于配置’。它解决了以前 Spring 开发的两大痛点：一是通过提供 Starter 机制，统一了依赖版本管理，解决了 Jar 包冲突；二是通过内置 Web 容器（如 Tomcat）和自动装配机制，干掉了繁琐的 XML 配置，让我们能通过一个 main 方法直接跑起一个服务，真正实现了开箱即用。
### Spring Boot 自动装配原理
自动装配的入口是 `@SpringBootApplication` 这个复合注解，里面包含三大核心：

1. **`@SpringBootConfiguration`**：标明这是一个配置类。
    
2. **`@ComponentScan`**：负责扫描当前主类包及子包下的 Bean。
    
3. **`@EnableAutoConfiguration`**：**这是自动装配的核心。** 它通过 `@Import` 机制，读取所有引入的 jar 包里的 `spring.factories`（或 `.imports`）文件。把里面声明的配置类全限定名拿出来。接着，利用 `@Conditional` 系列注解进行‘疯狂过滤’，剔除当前环境不需要的类，最后把剩下的有效组件注入到 IOC 容器中。


### 按需加载 @Conditional
它是动态装配的核心，把硬编码变成了可配置。

在自动装配把成百上千个配置类拉进内存后，如果不做过滤，不仅启动慢，还会报错（比如没引入 Redis 却实例化 RedisTemplate）。这时候就需要 `@Conditional` 来做‘门卫’。常见的有：

1. **`@ConditionalOnClass`**：判断当前 classpath 下有没有指定的类。比如只有 pom 里引入了 Jedis 的依赖，才会去实例化 Redis 相关的配置。
2. **`@ConditionalOnProperty`**：判断 `application.yml` 里有没有开启某个配置。比如 `xxx.enable=true` 时才生效。常用来做**多环境组件的开关隔离**。
3. **`@ConditionalOnMissingBean`**：用于提供默认的**兜底方案**。意思是‘如果开发者没有自己定义这个组件，框架才给你创建一个默认的’。


### 自定义 Starter 的落地实现
为公司封装一个公共的 Redis 或 MQ Starter，你需要哪几步？（核心：`@ConfigurationProperties` 绑定配置 + `@Conditional` 条件注解控制加载 + `spring.factories` 暴露入口）。

在实际项目中，我们经常需要把公司通用的鉴权、日志或定制版的 MQ 客户端封装成 Starter 给其他微服务复用。实现步骤非常标准化：

1. **建工程与引依赖：** 创建一个独立的 Maven 项目，引入 `spring-boot-autoconfigure` 依赖。
    
2. **定义属性绑定类：** 写一个 `XxxProperties` 类，打上 `@ConfigurationProperties(prefix = "my.mq")`，用来和各个业务线服务里的 `application.yml` 配置文件进行属性绑定。
    
3. **编写自动配置类：** 写一个 `XxxAutoConfiguration` 类，使用 `@Bean` 实例化核心业务逻辑组件。这里**必须**配合使用 `@Conditional`注解（让业务侧可以选择是否开启）。
    
4. **暴露发现接口：** 在 `resources/META-INF/` 目录下创建 `spring.factories`（或 `.imports` 文件），把刚才的 `XxxAutoConfiguration` 类的全路径写进去。 最后 `mvn install` 打成 jar 包，业务方只要在 pom 里引入这个依赖，配置一下 yml，就能直接无缝使用了。



### Spring Boot 启动流程
- **准备环境 (Environment)：** 首先推断应用类型（比如是 Servlet 还是 Reactive），然后读取所有的系统环境变量、启动参数，以及加载我们的 `application.yml/properties` 配置文件。
    
- **创建上下文 (Context)：** 根据应用类型，通过反射创建对应的 ApplicationContext（通常是 `AnnotationConfigServletWebServerApplicationContext`）。
    
- **准备上下文：** 将之前的 Environment 绑定到 Context 上，并执行一系列 ApplicationContextInitializer 的回调。
    
- **刷新上下文 (Refresh)：这是最核心的一步！** 其实就是调用的 Spring 核心的 `refresh()` 方法。这里面完成了扫描包、解析配置类、实例化所有的单例 Bean，并且**启动内置的 Tomcat 服务器**。
    
- **回调 Runners：** 容器启动完成后，回调实现了 `CommandLineRunner` 或 `ApplicationRunner` 接口的 Bean，执行我们自定义的启动后置逻辑。”



---
## IOC & Spring Bean
### 基本概念，使用场景

IOC就是控制反转的设计思想。传统开发中，我们需要什么对象就自己去 `new`，对象之间的耦合度极高。而 IOC 将对象的创建、生命周期管理以及依赖关系的绑定，全部交给了 Spring 容器（也就是大名鼎鼎的 IOC 容器）。 它的**核心使用场景**就是**解耦**。在我们的微服务或单体架构中，Controller 依赖 Service，Service 依赖 Mapper，我们通过 IOC 配合 DI（依赖注入），实现了组件级别的拔插和替换，极其方便进行单元测试和后续的扩展。


### 三级缓存与循环依赖
Spring 解决单例 Bean 循环依赖的核心机制是**三级缓存**加**提前暴露对象**。

三个缓存分别是：

1. **一级缓存 (`singletonObjects`)：** 存放完全经历过生命周期的成品 Bean。
    
2. **二级缓存 (`earlySingletonObjects`)：** 存放早期暴露的半成品 Bean（刚实例化完，还没赋值属性）。
    
3. **三级缓存 (`singletonFactories`)：** 存放一个 ObjectFactory 对象工厂。
    

**流转过程：** 当 A 实例化后发现需要注入 B，A 会把自己封装成一个 Factory 放进**三级缓存**，然后去创建 B。B 创建时需要 A，就会去缓存里找，最终在三级缓存拿到 A 的 Factory，借此拿到 A 的早期引用（此时 A 被移入二级缓存）。B 顺利创建完毕放入一级缓存，接着 A 拿到成品的 B，也顺利完成整个生命周期。

**核心追问：为什么非得三级缓存？两级不行吗？** 如果系统没有 AOP，两级绝对够了。三级缓存存在的唯一意义，就是**处理 AOP**。Spring 的原则是：AOP 代理对象应该在初始化的最后一步（`BeanPostProcessor`）再去生成。但遇到循环依赖时，B 在填充 A 时，如果 A 配置了 AOP，B 必须拿到 A 的**代理对象**！三级缓存里的 Factory 就是用来打破常规，**提前把 A 的代理对象生成出来暴露给 B** 的。

**循环依赖的局限性与实战应对：** 三级缓存不是万能的，它有前提：

1. **不支持 Prototype (多例)：** 多例模式每次都要创建新对象，无法缓存，遇到循环直接报错。
    
2. **不支持构造器注入：** 因为构造器注入连实例化这个‘空壳’都造不出来，直接死锁。 **实战解法：** 最根本的是**代码重构**，将产生循环依赖的逻辑抽离到第三个类中解耦。如果是遗留老代码应急，可以在注入的属性上加 `@Lazy` 注解，注入一个代理对象来延迟加载，打破循环。

### 几种依赖注入对比

主要有三种：字段注入（`@Autowired`标在属性上）、Setter 注入、构造器注入。 以前习惯用 `@Autowired` 直接标在字段上，但现在 Spring 官方和阿里规约都**强烈不推荐**。它容易掩盖循环依赖的坏味道，并且脱离了 Spring 容器（比如跑单测时）连对象都没法传。

目前**最推荐的是构造器注入**。在实际业务中（比如复杂的医疗健康平台），业界标准的写法是结合 Lombok 的 `@RequiredArgsConstructor`。

**反面教材：字段注入**

```Java
@Service
public class PatientService {
    @Autowired
    private PatientMapper patientMapper; // 脱离Spring容器无法注入，且变量可变
}
```

**工业级规范：构造器注入 + Lombok (强烈推荐)**

```Java
@Service
@RequiredArgsConstructor // Lombok 在编译阶段自动生成包含所有 final 字段的构造方法
public class PatientService {
    // 优势1：final 保证依赖的不可变性
    private final PatientMapper patientMapper;
    
    // 优势2：单测时可以直接 new PatientService(mockMapper)
    // 优势3：如果有循环依赖，项目启动直接报错 BeanCurrentlyInCreationException，逼迫你重构
}
```

### Bean的生命周期
Spring Bean 的生命周期非常复杂，但我通常把它浓缩为四个核心大阶段，这也是源码 `doCreateBean` 里的执行顺序：

1. **实例化 (Instantiation)：** 相当于在内存中 `new` 出了一个对象，此时它是个一无所有的空壳。
    
2. **属性赋值 (Populate)：** **【这就是 IOC 的依赖注入发生的地方！】** Spring 解析 `@Autowired` 或 XML 里的依赖，把 B 对象塞给 A。
    
3. **初始化 (Initialization)：** 这是扩展点最密集的地方，按顺序执行：
    
    - 调用各种 `Aware` 接口（如 BeanNameAware）。
        
    - 执行 `BeanPostProcessor` 的 `postProcessBeforeInitialization` 方法。
        
    - 执行 `InitializingBean` 接口的 `afterPropertiesSet` 和自定义的 `init-method`。
        
    - 执行 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法。如果需要 AOP，这里会利用 JDK 动态代理或 CGLIB 针对原始对象生成一个增强的代理对象，并将代理对象返回放入容器。**【这就是 AOP 发生的地方！】** 
        
4. **销毁 (Destruction)：** 容器关闭时，执行 DisposableBean 等销毁逻辑。 【应用关闭】



### Bean的线程安全性
**Spring 容器本身绝对不保证单例 Bean 的线程安全性！** Spring 中的 Bean 默认是单例（Singleton）的。如果这个 Bean 是**无状态**的（比如常规的 Controller、Service、Dao，里面只有方法，没有可变的成员变量），那它在多线程环境下天然是线程安全的。 但如果在 Bean 里定义了**可变的实例变量**，并发访问时绝对会发生线程安全问题。我们的解法通常是：避免在单例 Bean 中定义状态；如果必须存状态，就使用 `ThreadLocal` 隔离上下文，或者干脆把 Bean 的作用域改成 `prototype` 或者是 `request`。

### Bean的作用域
最常用的就是 `singleton`（单例，容器中唯一）和 `prototype`（多例，每次注入或 getBean 都会创建一个新实例）。如果是 Web 环境，还有 `request` 和 `session`。

需要注意**单例 Bean 中注入多例 Bean 失效**：如果你在一个 Singleton 的 A 里面，注入了一个 Prototype 的 B。由于 A 只会被初始化一次，所以 A 里面的 B 永远是最初注入的那同一个 B，`prototype` 直接失效了！ 解决办法是：不在 A 里面直接注入 B，而是每次需要用 B 的时候，通过注入 `ApplicationContext` 去 `getBean(B.class)`，或者使用 Spring 提供的 `@Lookup` 注解修饰一个方法来动态获取新实例。


### BeanFactory与 ApplicationContext 的区别
这两者是 Spring 底层最核心的两个接口。`BeanFactory` 是低级容器，本质上是个大 HashMap；`ApplicationContext` 是继承自它的高级容器。主要有三大区别：

1. **层级与定位：** `ApplicationContext` 是 `BeanFactory` 的子接口。`BeanFactory` 提供最基础的注册和获取功能；`ApplicationContext` 面向企业级开发。
    
2. **加载时机：** `BeanFactory` 采用**懒加载 (Lazy Load)**，调用 `getBean()` 时才实例化，启动快。`ApplicationContext` 采用**饿汉式加载**，容器启动时一次性实例化所有单例 Bean，启动慢但运行快，能提前暴露配置错误。
    
3. **功能丰富度：** `ApplicationContext` 是自动注册后置处理器的，且额外支持了国际化 (MessageSource)、事件发布机制 (ApplicationEventPublisher)、环境配置解析 (Environment) 等高级特性。实战中我们 99% 都是直接使用 `ApplicationContext`。

---
## AOP
### 基本概念，使用场景
- AOP即面向切面编程，我们能将那些与业务无关，但却被各个业务模块共同调用的逻辑（交叉业务）封装起来，提取成一个“切面”。目的是**减少系统中的重复代码，降低模块间的耦合度**。
    
- **高频实战场景：**
    
    - **统一日志记录：** 比如利用自定义注解 + AOP，拦截所有 Controller 层方法，记录出入参和执行时间。
        
    - **权限校验：** 拦截特定接口，检查 Header 中的 Token 并解析用户角色。
        
    - **全局异常处理机制**（底层也有 AOP 思想）。比如`@RestControllerAdvice`加上`@ExceptionHandler`做统一异常拦截
        
    - **Spring 声明式事务 (`@Transactional`)** 本身就是 AOP 最经典的官方实现。


### 实现AOP的几种方法
- **使用一些 Spring AOP 线程切面注解**

- **基于 Spring AOP (运行时织入)：**
    
    - **注解方式（最主流）：** 使用 `@Aspect` 配合 `@Before`, `@Around` 等注解，结合自定义注解或 Pointcut 表达式来拦截目标方法。
        
    - **XML 配置方式：** 在旧项目中通过 `<aop:config>` 标签配置，现在极少使用。
        
- **基于 AspectJ (编译时/类加载时织入)：**
    
    - Spring AOP 只是借用了 AspectJ 的注解语法，底层依然是动态代理。如果你需要拦截类的 `new` 实例化过程，或者拦截 `private` 字段的修改，就必须引入真正的 AspectJ 编译器（ajc）进行字节码级的静态织入。


### AOP执行顺序
如果只有一个切面，正常执行顺序就像套娃一样：先 `@Around` 前半截 -> `@Before` -> 目标方法 -> `@AfterReturning` -> 最后是 `@After` 和 `@Around` 的后半截。 但如果是多个切面拦截同一个方法，执行顺序就变成了**洋葱模型**。我们可以通过 `@Order(数字)` 来控制，数字越小，优先级越高。 比如 `@Order(1)` 的切面是最外层的洋葱皮，`@Order(2)` 是内层。进去的时候 1 先执行，2 后执行；但是出来的时候，2 会先执行完毕，最后才轮到最外层的 1 执行结束，这就叫‘先进后出’。



### 底层机制 CGLIB VS 动态代理
- **JDK 动态代理：**
    
    - **机制：** 基于 Java 反射机制实现。它要求目标类**必须实现至少一个接口**。代理类会实现相同的接口，把方法调用转发给 `InvocationHandler`。
        
    - **局限：** 只能代理接口中定义的方法，如果目标类没有实现接口，直接报错。以下代码报错`ClassCastException: $Proxy0 cannot be cast to UserServiceImpl`
```JAVA
    @Autowired 
    private UserServiceImpl userService;
```

- **CGLIB 动态代理：**
    
    - **机制：** 基于 ASM 字节码技术。它会在内存中动态**生成目标类的一个子类**，并重写（Override）父类的方法，在重写的方法里织入增强逻辑。
        
    - **局限：** 因为是基于继承的，所以**无法代理被 `final` 修饰的类和方法**，也无法代理 `private` 方法。
        
Spring AOP 的底层核心就是动态代理。 如果是旧版 Spring，目标类有接口就用 JDK 反射代理，没接口就用 CGLIB 字节码生成子类。 **但在 Spring Boot 2.0 之后，官方默认全部改用 CGLIB 了。** 这里面有个经典的坑：如果我们习惯把实现类直接 `@Autowired` 注入进去，因为 JDK 代理类只实现了接口，跟我们的实现类没半毛钱关系，启动时就会直接报 `ClassCastException`。而 CGLIB 生成的是目标类的子类，完美支持多态，包容性更强。当然，CGLIB 的局限就是不能代理 `final` 和 `private` 方法。




---
## 事务控制

### 传播机制
> 简单来讲，就是当系统中存在两个事务方法时（我们暂称为方法A和方法B），如果方法B在方法A中被调用，那么将采用什么样的事务形式，就叫做事务的传播特性  
> 比如，A方法调用了B方法（B方法必须使用事务注解），那么B事务可以是一个在A中嵌套的事务，或者B事务不使用事务，又或是使用与A事务相同的事务，这些均可以通过指定事务传播特性来实现

Spring 提供了 7 种传播机制，但在实际企业级开发中，我们绝大多数情况只关注 3 种：

- **`REQUIRED` (默认)：** 最常用，同生共死。如果外层有事务就加入，没有就新建。
    
- **`REQUIRES_NEW` (实战利器)：** 强制新建一个完全独立的事务。**典型场景：** 核心交易报错回滚了，但我必须把错误日志写进数据库。这时候写日志的方法必须用 `REQUIRES_NEW`，这样无论外层怎么回滚，日志都能独立提交。
    
- **`NESTED` (嵌套事务)：** 依赖于 JDBC 的 Savepoint（保存点）。它允许内层事务失败时只回滚自己，不影响外层。**注意前提：** 外层方法必须 `try-catch` 住内层抛出的异常，否则异常继续向上抛，外层依然会全局回滚。

### 事务回滚规则
- **默认规则：** Spring 默认只在遇到 `RuntimeException`（运行时异常）和 `Error` 时才会回滚。遇到 `Exception`（如 `IOException`, `SQLException` 等受检异常）**默认不回滚**！
- **正规写法：** 阿里开发规范强制要求，使用 `@Transactional` 时必须显式指定 `rollbackFor = Exception.class`，确保所有异常都能**兜底回滚**。
    
- **致命踩坑（异常被吞没）：** 如果你的代码写成了这样：
    ```Java
    @Transactional(rollbackFor = Exception.class)
    public void pay() {
        try {
            // 扣款逻辑发生异常
        } catch (Exception e) {
            log.error("扣款失败", e);
            // 异常被吃掉了，没有重新 throw e;
        }
    }
    ```
    **此时事务绝对不会回滚！** 因为 Spring AOP 的代理逻辑是在方法外面包了一层，如果异常在方法内部被 `catch` 处理掉了，代理层以为方法执行成功了，就会执行 `commit`。

**引申：如果 B 是默认的 `REQUIRED`，A `catch` 了 B 的异常会怎样？** 
会直接报错 `UnexpectedRollbackException`！因为 `REQUIRED` 模式下 A 和 B 共用一个物理事务。B 报错时，B 的代理对象会把底层的共享事务打上 `rollback-only` 标记。然后 A 假装没事人一样 catch 了异常并尝试 commit，Spring 底层一查标记，发现已经被 B 标记为只能回滚了，直接抛出异常阻止提交。


### 事务隔离级别
Spring 并不自己实现隔离级别，而是直接透传给底层数据库（如 MySQL）。

- **`Isolation.DEFAULT` (默认值)：** 这是 Spring 的默认配置，意思是**完全听从底层数据库的默认隔离级别**。在 MySQL (InnoDB) 中，这就等同于 `REPEATABLE_READ` (可重复读)；在 Oracle 中等同于 `READ_COMMITTED` (读已提交)。
    
- **其余四个级别：** 与数据库理论完全一致（`READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`）。在实战中，我们极少在 Spring 层去硬性指定隔离级别，绝大多数情况直接使用默认值交由 DBA 在数据库服务端统一调优。


###  `@Transactional` 失效的场景 

1. **同类内部方法调用：** 方法 A 没有事务，调用同类中有事务的方法 B，事务为什么失效？（因为绕过了 Spring 生成的代理对象，直接走了 `this` 原生调用。解法：注入自身或使用 `AopContext.currentProxy()`）。（广义AOP失效）
    
2. **方法不是 `public`：** `@Transactional` 标注在 `private` 或 `protected` 方法上直接失效。（广义AOP失效）
    
3. **异常被 `try-catch` 吞没：** 业务代码里 catch 了异常却没有重新 throw，Spring 代理感知不到异常，无法回滚。
    
4. **数据库引擎不支持：** 表的引擎如果是 MyISAM，Spring 配置得再好也没用。