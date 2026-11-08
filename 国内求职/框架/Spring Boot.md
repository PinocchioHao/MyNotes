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
自定义 Starter 的核心目的就是“解耦与复用”。它将繁琐的底层 Bean 实例化和组装逻辑封装在 Jar 包内部，仅将必要的动态参数暴露给业务方的 `application.yml`，从而实现开箱即用。

### 什么是 SPI 机制？

**SPI (Service Provider Interface)** 是一种**动态服务发现机制**。

- **原生 Java SPI：** 核心思想是“接口定义在核心库，实现类在外部包”。为了让核心库能找到外部的实现类，要求外部包必须在 `META-INF/services/` 目录下创建一个以接口全限定名命名的文件，里面写上具体的实现类。Java 启动时会自动读取并加载。
    
- **Spring Boot SPI (Spring 自动装配底层)：** Spring 借用了这个思想。由于业务代码的 `@ComponentScan` 默认只能扫描当前项目的包路径，根本扫不到外部引入的 Starter Jar 包里的配置类。为了打破这个物理隔离，Spring Boot 约定：只要外部 Jar 包在 `META-INF/spring.factories`（或 Spring Boot 2.7+ 的 `AutoConfiguration.imports`）文件中声明了自动配置类的全路径，Spring Boot 启动时就会跨包扫描并加载这些类。这就是 Starter 能实现“零代码侵入”的底层原动力。
    
### 自定义 Starter 开发流程 (实战笔记)

开发一个自定义 Starter 主要分为 4 个标准步骤：

**1. 创建工程与引入核心依赖** 创建一个独立的 Maven 工程，并在 `pom.xml` 中引入 Spring Boot 的自动配置基础依赖。这是使用相关装配注解的前提。

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

**2. 定义配置属性绑定类 (Properties)** 定义一个类用于接收业务方 `application.yml` 中的配置参数，使用 `@ConfigurationProperties` 声明配置前缀。

```Java
@ConfigurationProperties(prefix = "my.component")
public class MyComponentProperties {
    private String host;
    private int port;
    // 省略 getter / setter
}
```

**3. 编写自动配置类 (AutoConfiguration)** 编写核心组装逻辑，将底层的组件实例化并注入 Spring 容器。

- 必须使用 `@Configuration` 标识为配置类。
    
- 使用 `@EnableConfigurationProperties` 使第二步的属性绑定类生效。
    
- **核心防御编程：** 配合 `@Conditional` 系列注解（如 `@ConditionalOnMissingBean`）实现条件装配。当业务方自身未定义同类型 Bean 时，当前默认配置才生效，避免产生冲突报错。
    

```Java
@Configuration
@EnableConfigurationProperties(MyComponentProperties.class)
public class MyComponentAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(MyTargetClient.class) // 条件装配：如果容器没有该 Bean，则执行初始化
    public MyTargetClient myTargetClient(MyComponentProperties properties) {
        return new MyTargetClient(properties.getHost(), properties.getPort());
    }
}
```

**4. 暴露 SPI 发现接口 (注册配置类)** 在 `resources` 目录下创建约定的 SPI 注册文件，使 Spring Boot 启动时能够跨包扫描到该配置类。

- **Spring Boot 2.7 及以后：** 创建文件 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 内容为自动配置类的全限定名（每行一个）：
    
    ```
    com.yourcompany.config.MyComponentAutoConfiguration
    ```

完成以上 4 步后，执行 `mvn clean install` 打包。业务方只需在 `pom.xml` 中引入该依赖，并在 `yml` 中配置参数，即可直接注入目标组件。


### Spring Boot 启动流程
Spring Boot 的启动流程本质上是一场“从拉取配置、构建工厂、生产 Bean，到最终开门迎客”的接力赛。核心分为四大阶段：

**1. 准备环境阶段 (Environment)**

- **底层在干嘛：** 推断应用类型（比如是 Web 还是普通服务），然后收集所有来源的配置：操作系统的环境变量、JVM 启动参数，以及我们的 `application.yml`。
    
- **实战扩展点：`EnvironmentPostProcessor` (环境后置处理器)**
    
    - **应用场景（配置前置加载）：** 假设公司使用了 Nacos 或 Apollo 作为配置中心。为了安全，数据库的密码是不写在代码里的。我们必须实现这个钩子，在 Spring 刚加载完基础配置、但**还没开始创建任何 Bean 之前**，去远程服务器拉取数据库密码，并强行塞进 Environment 中。这样后续实例化数据库连接池时，才能顺利拿到密码。
        

**2. 创建容器阶段 (Context)**

- **底层在干嘛：** 根据第一步推断的环境，创建一个空白的 IoC 容器（不用背全名，面试直接说“创建 Web 版本的 ApplicationContext 容器”即可）。此时它只是个空壳。
    

**3. 核心刷新阶段 (Refresh) —— 【最重头戏】**

- **底层在干嘛：** 直接调用 Spring 核心的 `refresh()` 方法。这是整个启动流程的心脏。
    
    - **扫描与装配：** 触发自动装配机制（SPI），扫描所有包，将 Bean 的定义读取进来。
        
    - **生命周期大爆发：** 依次执行 Bean 的实例化、依赖注入、Aware 回调、AOP 代理生成（就是咱们在 Bean 生命周期里聊的那一套）。
        
    - **启动 Tomcat：** 实例化 Web 服务器 Bean，并在这一步把**内置的 Tomcat 真正跑起来**，绑定端口。
        

**4. 启动收尾阶段 (Runners)**

- **底层在干嘛：** 此时 Spring 容器完全建好，Tomcat 已就绪，马上就要开始接收用户的外部 HTTP 请求了。
    
- **实战扩展点：`ApplicationRunner` 或 `CommandLineRunner`**
    
    - **应用场景（缓存预热，防止缓存击穿）：** 金融系统刚启动时，Redis 可能是空的。如果此时直接放行大流量进来，海量请求会因为 Redis 没数据而全部打到 MySQL 上，瞬间把数据库打死（缓存击穿）。
        
    - **落地操作：** 我们实现 `ApplicationRunner`，在这里写一段查库的代码，把高频访问的“业务字典表”、“黑白名单”等数据提前加载并刷进 Redis。等这个 Runner 跑完，系统才算真正启动成功，安全地迎接外部流量。
        

**面试话术总结：** “其实不要把 Spring Boot 启动流程看成死板的源码，它就是一个提供了各种 Hook 的生命周期。在环境刚准备好时，我们可以用 `EnvironmentPostProcessor` 介入拉取远程敏感配置；在容器大刷新（`refresh`）时，它把 Spring 的扩展点和 Tomcat 融合在了一起；而在完全启动前的最后一刻，我们必须利用 `ApplicationRunner` 做好核心缓存的预热，防止流量瞬间打崩底层数据库。懂了这些，启动流程就成了我们调优和扩展系统的武器。”

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

目前**最推荐的是构造器注入**。业界标准的写法是结合 Lombok 的 `@RequiredArgsConstructor`。

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


**为什么官方强烈抛弃 `@Autowired` 字段注入，推崇 构造器 + Lombok？**
> 
> 1. **单测无缝且安全：** 字段注入依赖 Mockito 反射机制，新增依赖时单测不报错，运行时报 NPE；构造器注入直接 `new`，新增依赖会导致单测编译标红，强制开发者补全 Mock，排雷于编译期。
>     
> 2. **捍卫不可变性 (Immutability)：** 字段注入无法使用 `final`；构造器注入强制使用 `private final`，将核心组件焊死，避免运行期被意外篡改。
>     
> 3. **架构纠错（拒绝循环依赖）：** 字段注入会利用 Spring 三级缓存掩盖循环依赖的坏味道；构造器注入一旦遇到 A 和 B 互相依赖，启动直接抛 `BeanCurrentlyInCreationException` 崩溃，倒逼开发者重构、拆分代码。
>     
> 4. **告别初始化 NPE：** 字段注入的对象是先实例化再反射赋值，构造方法中调用依赖必报空指针；构造器注入确保了对象在创建成功的那一刻，所有依赖必定已就绪。

新手最容易犯的错，就是在构造方法或者声明变量时，去使用 `@Autowired` 的字段。Spring 的生命周期是**先通过空构造器把对象 new 出来（实例化阶段），然后才去利用反射填充 `@Autowired` 字段（属性赋值阶段）**。所以在对象刚创建时，`patientMapper` 绝对是 `null`。

```Java
@Service
public class PatientService {
    @Autowired
    private PatientMapper patientMapper;

    private String defaultPatientName = patientMapper.getDefaultName(); // 💥直接 NullPointerException!

    public PatientService() {
        patientMapper.init(); // 💥直接 NullPointerException!
    }
}
```

### Bean的生命周期
Spring Bean 的生命周期非常复杂，但我通常把它浓缩为四个核心大阶段，这也是源码 `doCreateBean` 里的执行顺序：

1. **实例化 (Instantiation)：** 相当于在内存中 `new` 出了一个对象，此时它是个一无所有的空壳。
    
2. **属性赋值 (Populate)：** **【这就是 IOC 的依赖注入发生的地方！】** Spring 解析 `@Autowired` 、`@Value` 以及 XML，完成依赖的解析和注入。
    
3. **初始化 (Initialization)：** 这是扩展点最密集的地方，按顺序执行：
	    
	- **步骤 3.1：Aware 接口回调（注入上下文感知能力）**
	    
	    - **底层：** 调用 `BeanNameAware`, `ApplicationContextAware` 等接口。
	        
	    - **实战痛点（逃生舱）：** 在实际开发中，有些非 Spring 管理的对象（如底层多线程 `Runnable`、JPA 实体类、第三方框架的回调类）需要调用 Spring 容器内的业务 Bean，此时 `@Autowired` 失效。
	        
	    - **实战落地：** 编写一个全局静态工具类 `SpringContextUtil implements ApplicationContextAware`，在这个回调阶段主动获取并保存 `ApplicationContext` 容器引用，提供统一的静态 `getBean()` 方法供外部类捞取业务 Bean。这是架构上面对边界问题的常见妥协与解法。
	        
	- **步骤 3.2：BeanPostProcessor 的 `postProcessBeforeInitialization`**
	    
	    - **底层：** 在自定义初始化逻辑之前执行拦截。
	        
	    - **实战落地（极少直接使用）：** 多用于一些极端的参数校验或者注解解析的前置处理，业务开发较少触及。
	        
	- **步骤 3.3：执行自定义初始化逻辑**
	    
	    - **底层：** 按顺序执行 `@PostConstruct` 注解 -> `InitializingBean` 接口的 `afterPropertiesSet` -> 自定义的 `init-method`。
	        
	    - **实战痛点（动态路由与预热）：** 消除代码中的大量 `if-else`。
	        
	    - **实战落地（策略模式自动注册）：** 各个支付渠道的策略类实现统一的 `PayStrategy` 接口，并在其 `@PostConstruct` 或 `afterPropertiesSet()` 方法中，将自身实例自动注册到“路由工厂类”的 Map 缓存中。实现新增策略完全无需修改核心路由代码。此外，也常用于 Redis 缓存字典表的启动预热。
	        
	- **步骤 3.4：BeanPostProcessor 的 `postProcessAfterInitialization` —— 【核心：AOP 与代理对象的诞生地】**
	    
	    - **底层：** 初始化的最后一步，Spring 通过 `AnnotationAwareAspectJAutoProxyCreator`（一个核心后置处理器）检查该 Bean 是否有 `@Transactional`、`@Async` 等注解，或是否匹配了切面表达式。
	        
	    - **实战落地（AOP）：** 如果匹配，Spring 在这里会利用 CGLIB（或 JDK 动态代理）为该对象生成一个**代理对象**，将原始对象替换掉塞进容器中。这解释了为什么**自己 `new` 出来的对象注解不生效**，因为没有经过这个后置处理器的“偷梁换柱”。
	        
	    - **实战落地（MyBatis Mapper 注入原理）：** MyBatis 只写了接口（连实现类都没有），为什么能执行 SQL？因为 MyBatis 通过实现特定的 `BeanPostProcessor`，在这一阶段动态生成了该接口的 JDK 代理对象（底层调用 `SqlSession`），并将其作为 Bean 塞入 Spring 容器中。
        
4. **销毁 (Destruction)：** 容器关闭时，执行 DisposableBean 等销毁逻辑。 【应用关闭】


静态工具类或非spring bean实现了ApplicationContextAware接口就可以用它调用spring容器里的bean，如
```java
// 这里需要调用 UserService 查数据库 
// @Autowired 在这里完全无效！只能主动捞取： 
UserService userService = SpringContextUtil.getBean(UserService.class); 
userService.doSomething();
```
这种虽然违背了单一职责，但是也是项目架构的一种妥协，会有MQ、Redis或者定时任务组件可能需要这种调用方式。


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
	    
    - **统一接口防重：** 比如利用自定义注解 + AOP，拦截所有 Controller 层方法，请求进来时，利用 AOP 获取请求参数，生成唯一 Key，去 Redis 执行 `SETNX` 获取分布式锁，获取成功就执行，失败就说明接口重复，抛异常阻断。
	    
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

### **Spring 本地事务底层原理**
Spring `@Transactional` 底层**绝对没有**去解析 SQL 或者写 Undo Log。它干的活非常纯粹，就是**管理 Connection**。

> “Spring 事务的本质是 **AOP 动态代理 + ThreadLocal**。 当你调用加了注解的方法时，AOP 代理拦截了这个请求。它去数据源拿一个 `Connection`，把自动提交关掉（`setAutoCommit(false)`），然后把这个 Connection 挂载到当前线程的 `ThreadLocal` 上。 接着执行你的业务代码，所有的 MyBatis/Hibernate 操作只要在同一个线程，就会去 `ThreadLocal` 里拿到同一个 Connection 执行 SQL。 方法结束时，代理逻辑捕获有没有抛出异常，决定是调用 `Connection.commit()` 还是 `rollback()`。 **真正的回滚，靠的是 MySQL 数据库自身的 Undo Log，Spring 只是发号施令。**”

不管上层框架如何演进（Spring、JFinal 还是原生 Servlet），在 Java 层控制本地事务的物理底座永远只有两个核心要素：

1. **关闭自动提交：** `connection.setAutoCommit(false)`，将控制权收归业务。
    
2. **保证线程内 Connection 唯一：** 确保跨多个 DAO 层的 SQL 操作，拿到的是同一根数据库连接。
    
**不同框架仅仅是“拦截时机”与“封装形式”的不同：**

- **脱离 Spring 的纯 Servlet 玩法（过滤器模式）：** 我们在最外层定义一个 Servlet Filter 拦截 HTTP 请求。请求进来时，从连接池取出一个 `Connection`，关闭自动提交，并塞入我们手写的 `ThreadLocal` 中。后续业务代码全从该 `ThreadLocal` 拿连接执行 SQL。等 HTTP 响应返回前，Filter 统一在 `finally` 块中执行 `commit` 或 `rollback`，并清空 ThreadLocal。
    
- **Spring 事务的玩法（AOP 模式）：** Spring 只是把这个过程封装成了 `TransactionSynchronizationManager`（底层本质就是一个大 ThreadLocal）。它通过 AOP 动态代理，把拦截时机从“HTTP 请求进出”缩小到了“加了 `@Transactional` 注解的方法进出”。**总结：Spring 只是发号施令的门卫，真正执行回滚的，是底层数据库的原生 Undo Log。**


### 传播机制
> 简单来讲，就是当系统中存在两个事务方法时（我们暂称为方法A和方法B），如果方法B在方法A中被调用，那么将采用什么样的事务形式，就叫做事务的传播特性  
> 比如，A方法调用了B方法（B方法必须使用事务注解），那么B事务可以是一个在A中嵌套的事务，或者B可以新开一个独立的事务，或是使用与A事务相同的事务，又或者B事务不使用事务，这些均可以通过指定事务传播特性来实现

#### 核心3种传播机制与时序特性总结

1. **`REQUIRED` (默认：同生共死)**
    
    - **连接状态：** 内外层共用同一 Connection。
        
    - **时序影响（无影响）：** 无论报错发生在调用前还是调用后，只要有一处抛出异常且未被处理，整个 Connection 标记为 `rollback-only`，**全局回滚**。
        
    - **避坑：** 若内层报错，即使外层 `try-catch` 捕获，最终 Commit 时仍会抛出 `UnexpectedRollbackException` 导致全局回滚。
        
2. **`REQUIRES_NEW` (完全隔离：独立提交)**
    
    - **连接状态：** 强制挂起外层事务，获取**全新 Connection**。
        
    - **时序影响（绝对隔离）：** 只要该方法执行结束，数据立刻**物理 Commit 落盘**。外层在此**之后**发生的任何报错回滚，都无法撤销该内层数据。
        
    - **避坑：** 极其**消耗数据库连接**，高并发下慎用。
        
3. **`NESTED` (嵌套事务：Savepoint 控制)**
    
    - **连接状态：** 共用外层 Connection，但利用底层的 **Savepoint**（保存点）机制。
        
    - **时序影响（单向隔离）：**
        
        - **内层报错：** 外层若 `try-catch`，仅回滚到该 Savepoint，外层之前的数据保留并可正常提交。
            
        - **外层后置报错：** 若外层在内层执行**之后**报错回滚，由于主事务清零，依附于主事务的 Savepoint 会被全部抹除，**内层已执行的数据必定连带回滚**。
            

#### 伪代码举例

通过以下伪代码的时间轴，可以清晰看到异常发生位置对三种事务的不同影响：

```Java
@Service
public class LoanService {

    // 核心入口：贷款申请 (大事务开始)
    @Transactional(propagation = Propagation.REQUIRED)
    public void applyLoan() {
        
        // ---------------------------------------------------------
        // 1. 调用子服务：冻结额度 (REQUIRED)
        // 融入当前事务。只要本方法内任何地方最终回滚，额度一定回滚。
        // ---------------------------------------------------------
        accountService.freezeLimit(); 

        // ---------------------------------------------------------
        // 2. 调用子服务：记录风控日志 (REQUIRES_NEW)
        // 开启新连接。方法执行完毕的瞬间，日志直接物理 Commit 入库！
        // ---------------------------------------------------------
        auditService.saveAuditLog(); 

        // ⚠️【时序截断点 A】：假设此时发生 OOM 或业务报错抛出异常
        // 结果：
        // 1. applyLoan 大事务回滚 -> freezeLimit 回滚。
        // 2. saveAuditLog 绝对不回滚（因为它已经独立 Commit 完了）。
        // if (error) throw new RuntimeException("外层报错");

        // ---------------------------------------------------------
        // 3. 调用子服务：发放营销积分 (NESTED)
        // 使用 Savepoint。允许局部失败，不影响主流程。
        // ---------------------------------------------------------
        try {
            bonusService.addBonusPoints(); 
        } catch (Exception e) {
            // 【内层报错】：只会回滚 addBonusPoints 这一步的代码。
            // 因为被 catch 住了，外层的 freezeLimit 不受影响，主事务继续往下走。
            log.warn("积分发放失败，但不影响贷款主流程");
        }

        // ⚠️【时序截断点 B】：假设此时外层校验失败，抛出异常
        // 结果：
        // 1. applyLoan 大事务回滚 -> freezeLimit 回滚。
        // 2. addBonusPoints 连带回滚（哪怕它刚才执行成功了，因为依附的主连接回滚了）。
        // 3. saveAuditLog 依然不回滚（绝对独立）。
        // if (finalCheckFailed) throw new RuntimeException("最终校验失败，全局回滚");
        
    } // 大事务 Commit
}
```


```Java
@Service
public class SubService {
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void freezeLimit() { /* 扣减额度 SQL */ }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveAuditLog() { /* 插入审计日志 SQL */ }

    @Transactional(propagation = Propagation.NESTED)
    public void addBonusPoints() { /* 增加积分 SQL */ }
}
```


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
            // 【关键点】：必须重新抛出，让 Spring AOP 监听到！ 
            // throw new RuntimeException("扣款失败", e);
        }
    }
    ```
    **此时事务绝对不会回滚！** 因为 Spring AOP 的代理逻辑是在方法外面包了一层，如果异常在方法内部被 `catch` 处理掉了，代理层以为方法执行成功了，就会执行 `commit`。
这个例子错就错在异常被`try-catch`块给吞掉了，而spring就是靠拦截异常来进行事务回滚的。最简单的改法是去掉`try-catch`块，最严谨的改法是记录现场再手动抛出异常。


#### 使用规则
- **`REQUIRED`（同生共死）：** 外层**禁止**用 `try-catch` 吞没异常。只要有一处报错，全体必须回滚。强行吞异常必报 `UnexpectedRollbackException` 冲突。
    
- **`REQUIRES_NEW`（绝对物理隔离）：** 外层**可以且经常**使用 `try-catch`。视业务情况而定需不需要内外绑定回滚。
    - **作用：** 既能保证“内层失败不阻断外层”，又能保证“内层成功后，绝不受外层后续回滚的牵连”。常用于**风控留痕、独立流水号生成、批处理防连坐**。
        
- **`NESTED`（单向嵌套依赖）：** 外层**必须/推荐**使用 `try-catch`。（不包这层的话效果跟required一样同生共死了）
    - **作用：** 发挥 Savepoint 局部回滚的威力，实现“内层旁路失败不影响主线，但主线一旦失败，内层必须跟着陪葬”。常用于**发积分、发优惠券**等依附型业务。


#### `try-catch`了REQUIRED 模式下的 `UnexpectedRollbackException` 到底是怎么回事？

**案发现场还原：**

- **方法 A**（`REQUIRED`）：开启了物理事务（连接 Connection-1）。
    
- **方法 B**（`REQUIRED`）：因为是 `REQUIRED`，它没有新建连接，而是**加入了 A 的 Connection-1**。两人同乘一条船。
    
**灾难发生的流程：**

1. 船开到方法 B 的区域，B 突然爆炸了（抛出异常）。
    
2. 方法 B 门口也有一个“小安检员”（B 的 AOP 代理）。B 的安检员抓到了这个异常，他发现 B 和 A 用的是同一个物理事务，于是他**在这个共享的底层 Connection-1 上，狠狠地盖了一个大大的红戳：`rollback-only`（此船已漏水，只能沉没，绝对不许靠岸提交）。**
    
3. 然后 B 把异常扔给了上层的方法 A。
    
4. **骚操作来了：** 方法 A 里写了个 `try-catch`，把 B 扔过来的异常给吞了。A 觉得自己把问题摆平了，继续往下走，直到正常结束。
    
5. 方法 A 走到大门口，A 的大安检员一看 A 没有抛异常，于是高兴地对数据库下达指令：**“一切顺利，给我 Commit！”**
    
6. **底层的冲突爆发：** Spring 事务管理器准备 Commit 时，低头一看物理连接 Connection-1，上面赫然盖着 B 之前打的 `rollback-only` 红戳！
    
7. Spring 当场崩溃：**“你 A 叫我 Commit，但底层已经被 B 标记为只能 Rollback 了，你们俩指令冲突，我该听谁的？！”**
    
8. 于是，Spring 只能抛出一个巨大的异常：`UnexpectedRollbackException`（意想不到的回滚异常），强行中止整个程序，执行全局回滚。
    

**一句话总结：**`try-catch` 本身没有错，错在**破坏了 Spring AOP 通过异常来判定事务成功与否的通信机制**。只要理解了 AOP 是在方法外部“监听”异常的，以及 `REQUIRED` 模式下状态是“共享”的，这笔糊涂账就彻底算清了。


### 事务隔离级别
Spring 并不自己实现隔离级别，而是直接透传给底层数据库（如 MySQL）。

- **`Isolation.DEFAULT` (默认值)：** 这是 Spring 的默认配置，意思是**完全听从底层数据库的默认隔离级别**。在 MySQL (InnoDB) 中，这就等同于 `REPEATABLE_READ` (可重复读)；在 Oracle 中等同于 `READ_COMMITTED` (读已提交)。
    
- **其余四个级别：** 与数据库理论完全一致（`READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`）。在实战中，我们极少在 Spring 层去硬性指定隔离级别，绝大多数情况直接使用默认值交由 DBA 在数据库服务端统一调优。


###  `@Transactional` 失效的场景 

1. **同类内部方法调用：** 方法 A 没有事务，调用同类中有事务的方法 B，事务为什么失效？（因为绕过了 Spring 生成的代理对象，直接走了 `this` 原生调用。解法：注入自身或使用 `AopContext.currentProxy()`）。（广义AOP失效）
    
2. **方法不是 `public`：** `@Transactional` 标注在 `private` 或 `protected` 方法上直接失效。（广义AOP失效）
    
3. **异常被 `try-catch` 吞没：** 方法里写了 `try-catch` 但没有把异常 `throw` 出来，AOP 以为执行成功了，就不会回滚。catch了异常可以重新throw出来，或者指定异常。
	
4. **抛出的异常不匹配：** 默认只回滚 `RuntimeException` 和 `Error`，如果抛了受检异常（如 `IOException`）且没配 `rollbackFor = Exception.class`，同样失效。
    
5. **数据库引擎不支持：** 表的引擎如果是 MyISAM，Spring 配置得再好也没用。



 
### **MyBatis 自定义封装一个查询怎么做？**
题意是在架构层面，如何拦截 MyBatis 的底层执行链路，把一些全公司通用的查询逻辑（如数据隔离、分页、脱敏）统一下沉，做到对上层业务代码零侵入？

要封装/修改查询，你必须知道 MyBatis 底层执行 SQL 需要经过四个核心对象：

1. **Executor（执行器）：** 负责调度整个过程，管理缓存。
    
2. **StatementHandler（语句处理器）：** 负责把 SQL 和参数拼接好（**拦截 SQL 最常改造的地方**）。
    
3. **ParameterHandler（参数处理器）：** 负责给 SQL 里的 `?` 占位符赋值。
    
4. **ResultSetHandler（结果处理器）：** 负责把查出来的结果集映射成 Java List/对象。

#### 实战场景：为什么要用拦截器封装？

假设你们银行/金融系统有一个极度严格的安全规定：**所有查询必须带上当前租户的 ID（多租户隔离），或者查出来的敏感字段（手机号、身份证）必须脱敏打码。**

- **低级做法：** 让所有开发人员在每一个 XML 的 SQL 后面手动加 `WHERE tenant_id = xxx`。极容易漏写，一旦漏写直接造成跨租户数据泄露（P0级安全生产事故）。
    
- **架构解法（面试官想要的）：** 使用 MyBatis 拦截器。
    

> **💡 高分面试话术总结：** “如果是指架构层面的底层封装，我会利用 MyBatis 的 **Interceptor（插件机制）** 来实现对业务零侵入的查询拦截。
> 
> MyBatis 底层有四大核心对象，我们通常会拦截 **StatementHandler**。 比如在我们的系统中，为了实现绝对安全的多租户数据隔离，需要自定义拦截器，实现`ibatis`的`Interceptor`接口，并重写`intercept`方法，拦截目标方法。 当一条原生的 `SELECT * FROM order` 准备发送给数据库前，拦截器会被触发。我会利用 **JSqlParser** 这个语法树解析工具，把 SQL 解析出来，强行在 `WHERE` 条件后面拼上 `AND tenant_id = ?`，并从 `ThreadLocal` 里取出当前登录用户的租户 ID 塞进去。
> 
> 这样封装的好处是：上面的业务开发同事只需要写最纯粹的业务 SQL，不用管任何租户隔离逻辑，拦截器在底层完成了 SQL 的重写和封装，从根源上杜绝了人为漏写导致的数据越权事故。”