# Spring夺命连环100问——Spring核心技术深度指南

> 本文档面向Spring学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是Spring？为什么要用Spring？

#### 1.1 Spring定义

> Spring是JavaEE的轻量级替代品，由Rod Johnson创建。2004年发布Spring 1.0，现为Spring Framework + Spring Boot + Cloud生态。核心是控制反转（IoC）和面向切面（AOP）。

```plaintext
Spring生态：
├── Spring Boot      # 快速开发
├── Spring Cloud    # 分布式
├── Spring Data    # 数据访问
├── Spring Security# 安全
├── Spring Batch   # 批处理
└── Spring AMQP   # 消息队列
```

#### 1.2 核心价值

```plaintext
Spring优势：
1. 简化开发：POJO + IoC容器
2. 解耦：Dependency Injection
3. 面向切面：AOP分离横切关注点
4. 集成：整合第三方框架
5. 生态：完整的生态体系
```

#### 1.3 回答模板

> Spring是Java企业级开发的生态系统，核心是IoC容器和AOP。Spring Boot快速开发、Spring Cloud分布式、Spring Data数据访问等。Spring让Java开发更简单、解耦、可测试，是后端开发必备技能。

---

### 2. Spring Boot和Spring区别？

#### 2.1 概念区分

```plaintext
Spring Framework：框架
- IOC/AOP容器
- 兼容Java EE
- 需要XML/Java配置
- 手动搭建

Spring Boot：脚手架
- 自动配置
- 嵌入式服务器（Tomcat/Jetty）
- 起动器Starter
- Actuator监控
```

#### 2.2 自动配置原理

```java
// @SpringBootApplication入口
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan

// @EnableAutoConfiguration
// 1. META-INF/spring.factories
// 2. 自动配置类xxxAutoConfiguration
// 3. @ConditionalOnxxx条件判断
// 4. 配置属性绑定@ConfigurationProperties
```

#### 2.3 回答模板

> Spring是框架本身需手动配置，Spring Boot是脚手架基于Spring自动配置。@EnableAutoConfiguration加载 spring.factories中的配置类，@ConditionalOn*判断条件满足才配置。Spring Boot约定大于配置，让开发更简单。

---

### 3. IoC和DI是什么？

#### 3.1 IoC概念

```plaintext
IoC（Inversion of Control）控制反转：
- 对象的创建管理权反转给容器
- 不new对象
- 被动接收注入

DI（Dependency Injection）依赖注入：
- 容器注入对象依赖
- 构造器/Setter/字段注入
- 方式：XML/注解/JavaConfig
```

#### 3.2 注入方式

```java
// 1. 构造器注入（推荐）
@Autowired
public UserService(UserDao userDao) {
    this.userDao = userDao;
}

// 2. Setter注入
@Autowired
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}

// 3. 字段注入（不推荐）
@Autowired
private UserDao userDao;
```

#### 3.3 回答模板

> IoC是控制反转，对象不由new而由容器管理；DI是依赖注入，容器把依赖对象注入到目标。推荐构造器注入便于测试和不变性。Spring Boot推荐使用构造函数注入@RequiredArgsConstructor + @NonNull。

---

### 4. Bean生命周期？

#### 4.1 Bean生命周期

```java
// Bean生命周期（简化）：
1. 实例化 → new对象
2. 属性填充 → @Autowired注入
3. 初始化 → @PostConstruct / InitializingBean.afterPropertiesSet()
4. 使用 → 业务方法
5. 销毁 → @PreDestroy / DisposableBean.destroy()
```

```plaintext
Bean后置处理器：
├── BeanPostProcessor.postProcessBeforeInitialization
├── InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation
├── BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry
└── BeanFactoryPostProcessor.postProcessBeanFactory
```

#### 4.2 Bean作用域

```java
// 六种作用域
@Scope("singleton") // 默认，整个容器一个实例
@Scope("prototype") // 每次获取新实例
@Scope("request")   // 每个HTTP请求一个实例
@Scope("session")  // 每个HTTP会话一个实例
@Scope("application") // ServletContext级别
@Scope("websocket") // WebSocket级别

// 懒加载
@Lazy // 延迟加载
```

#### 4.3 回答模板

> Bean生命周期：实例化→属性填充→初始化→使用→销毁。有后置处理器扩展点，@PostConstruct和init-method是初始化前，@PreDestroy和destroy-method是销毁前。单例Bean容器启动创建，原型每次获取新实例。

---

### 5. Bean创建方式？

#### 5.1 @Component/@Bean

```java
// @Component：自动扫描
@Component
public class UserService { }

// @Bean：Java配置
@Configuration
public class Config {
    @Bean
    public UserService userService() {
        return new UserService();
    }
}
```

```java
// @Import
@Import({UserService.class, UserDao.class})
public class AppConfig { }

// @ImportResource
@ImportResource("classpath:beans.xml")
```

#### 5.2 FactoryBean

```java
// FactoryBean创建复杂Bean
public class ProxyFactoryBean implements FactoryBean<Object> {
    @Override
    public Object getObject() {
        return createProxy();
    }

    @Override
    public Class<?> getObjectType() {
        return Object.class;
    }
}
```

#### 5.3 回答模板

> Bean创建：@Component自动扫描或@Bean手动定义。@Component加@Service/@Controller/@Repository细分。@Import快速导入类。Bean多或复杂用@Configuration+@Bean，FactoryBean创建复杂对象（代理、装饰）。

---

### 6. Spring常用注解？

#### 6.1 Core注解

```java
// Bean定义
@Component @Service @Controller @Repository
@Configuration @Bean @Import @Scope

// 注入
@Autowired @Qualifier @Primary @Inject

// MVC
@RestController @RequestMapping @GetMapping @PostMapping
@RequestParam @RequestBody @PathVariable @ResponseBody

// 配置
@ConfigurationProperties @EnableConfigurationProperties
@Value ${property:default}
```

#### 6.2 Spring Boot注解

```java
// 自动配置
@SpringBootApplication
@EnableAutoConfiguration

// 配置
@ConditionalOnClass @ConditionalOnProperty
@ConditionalOnMissingBean @ConditionalOnBean

// 激活
@Profile("dev") // 环境profile

// YAML
@ConfigurationProperties(prefix = "user")
```

#### 6.3 回答模板

> Spring核心注解分：Bean定义@Component+Service+Controller+Repository、注入@Autowired+Qualify+Primary、配置@Value+@@ConfigurationProperties。Spring Boot @Conditional条件配置。

---

### 7. ApplicationContext？

#### 7.1 容器接口

```java
// Spring核心容器
ApplicationContext ctx = new ClassPathXmlApplicationContext("bean.xml");
ApplicationContext ctx = new AnnotationConfigApplicationContext(Config.class);

// Spring Boot容器
ApplicationContext ctx = SpringApplication.run(App.class, args);
```

```java
// 获取Bean
UserService us = ctx.getBean(UserService.class);
UserService us = ctx.getBean("userService", UserService.class);
UserService us = ctx.getBean(UserService.class);
List<UserService> list = ctx.getBeansOfType(UserService.class);
```

#### 7.2 容器事件

```java
// 发布事件
applicationEventPublisher.publishEvent(new OrderEvent(order));

// 监听事件
@Component
public class OrderListener implements ApplicationListener<OrderEvent> {
    @Override
    public void onApplicationEvent(OrderEvent event) {
        // 处理
    }
}

// 或@EventListener
@EventListener
public void handle(OrderEvent event) { }
```

#### 7.3 回答模板

> ApplicationContext是Spring核心容器，getBean获取Bean。它是BeanFactory的扩展，有国际化、事件、资源加载。publishEvent发布事件，Listener是观察者模式。

---

### 8. AOP是什么？

#### 8.1 AOP概念

```java
// Aspect切面：横切关注点模块化
// Join Point连接点：可织入的方法（方法执行）
// Pointcut切入点：哪些Join Point需要织入
// Advice通知：具体逻辑（Before/After/Around）
// Weaving织入：切面应用到目标的过程
```

```java
@Aspect
@Component
public class LogAspect {

    @Before("execution(* com.demo..*.save(..))")
    public void beforeSave(JoinPoint jp) {
        // 方法执行前的逻辑
    }

    @Around("@annotation(Login)")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object ret = pjp.proceed();
        System.out.println(System.currentTimeMillis() - start);
        return ret;
    }
}
```

#### 8.2 通知类型

```java
@Before // 前置通知
@AfterReturning // 返回后通知
@AfterThrowing // 异常通知
@After // 后置通知（finally）
@Around // 环绕通知（最强）
```

#### 8.3 回答模板

> AOP是面向切面编程，把日志、事务等横切关注点分离。@Aspect+@Component定义切面，@Before等是通知，execution是切入点和切面表达式。Spring AOP是动态代理，只能拦截public方法。

---

### 9. 静态代理和动态代理？

#### 9.1 代理模式

```java
// 静态代理：编译时已确定
public class UserServiceProxy implements UserService {
    private UserService target;

    @Override
    public void save(User u) {
        System.out.println("before");
        target.save(u);
        System.out.println("after");
    }
}
```

```java
// JDK动态代理：运行时生成接口实现类
// 基于InvocationHandler
UserService proxy = (UserService) Proxy.newProxyInstance(
    cl.getClassLoader(),
    new Class[]{UserService.class},
    (proxy, method, args) -> {
        // 反射调用
        return method.invoke(target, args);
    }
);
```

#### 9.2 CGLib动态代理

```java
// CGLib：继承父类override
// Enhancer创建
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(UserService.class);
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    // 代理逻辑
    return proxy.invokeSuper(obj, args);
});
UserService proxy = (UserService) enhancer.create();
```

#### 9.3 区别

| 特性 | JDK代理 | CGLib |
|------|--------|-------|
| 原理 | 实现接口 | 继承父类 |
| 性能 | 稍慢 | 快 |
| 类 | 接口实现类 | 子类 |
| 限制 | 接口方法 | final/static不可 |

#### 9.4 回答模板

> 静态代理编译确定，JDK动态代理实现相同接口，CGLib继承父类。Spring AOP根据是否实现接口选择JDK/CGLib（spring.aop.proxy-target-class=true强制cglib）。CGLib不能代理final/static方法。

---

### 10. Spring MVC工作流程？

#### 10.1 请求流程

```plaintext
1. 请求 → DispatcherServlet
2. HandlerMapping找Controller
3. Controller执行业务 → 返回ModelAndView
4. ViewResolver渲染View
5. 响应浏览器
```

```java
// @Controller
@RestController // = @Controller + @ResponseBody
@RequestMapping("/user")
public class UserController {

    @GetMapping("/{id}")
    public User get(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping
    public User create(@RequestBody @Valid User user) {
        return userService.save(user);
    }
}
```

#### 10.2 核心组件

```plaintext
核心组件：
├── DispatcherServlet          前端控制器
├── HandlerMapping            映射器（RequestMappingHandlerMapping）
├── HandlerInterceptor       拦截器
├── Controller              处理器
├── HandlerAdapter          适配器
├── ModelAndView             模型视图
├── ViewResolver           视图解析器（InternalResourceViewResolver）
└── View                  视图（Thymeleaf/Freemarker/JSP）
```

#### 10.3 回答模板

> Spring MVC流程：DispatcherServlet接收→HandlerMapping找Controller→Controller返回Model→ViewResolver渲染→响应。@RestController=@Controller+@ResponseBody，返回对象自动转JSON/XML。

---

## 第二章 数据访问篇（高频 ★★★★★）

### 11. Spring Data JPA？

#### 11.1 JPA定义

```java
// 定义实体
@Entity
@Table(name = "t_user")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String name;

    @Column(unique = true)
    private String email;

    @Temporal(TemporalType.DATE)
    private Date birthday;
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 自动实现
    User findByName(String name);
    List<User> findByEmailContaining(String email);
    @Query("select u from User u where u.name = ?1")
    User findByName2(String name);
}
```

#### 11.2 Repository继承

```java
// Repository继承层级
Repository → CrudRepository → PagingAndSortingRepository → JpaRepository

// 常用方法
save()      saveAll()
findById()  findAll()
deleteById() delete()
existsById()
count()
```

#### 11.3 回答模板

> JPA是Java持久化API，@Entity定义实体，@Id主键。JpaRepository自动生成CRUD，只需按方法名约定命名（findBy、countBy）或@Query写JPQL。注意@Entity的@Table，Column对应表字段。

---

### 12. Spring事务传播？

#### 12.1 传播行为

```java
@Transactional(propagation = Propagation.REQUIRED)
// REQUIRED：存在事务加入，不存在创建（默认）
// REQUIRES_NEW： always创建新事务，挂起旧的
// SUPPORTS：跟随，无则不在事务
// NOT_SUPPORTED：无事务执行，挂起
// MANDATORY：必须存在事务，没有抛异常
// NEVER：必须没事务，有抛异常
// NESTED：嵌套事务（SAVEPOINT）
```

#### 12.2 隔离级别

```java
@Transactional(isolation = Isolation.DEFAULT)
// DEFAULT：跟随数据库
// READ_UNCOMMITTED：读未提交（脏读）
// READ_COMMITTED：读已提交（防脏）
// REPEATABLE_READ：可重复读（防不可重复）
// SERIALIZABLE：串行（表锁，最慢）
```

```sql
-- MySQL默认REPEATABLE_READ
-- PostgreSQL默认READ_COMMITTED
```

#### 12.3 回滚和只读

```java
// 回滚
@Transactional(rollbackFor = Exception.class)
@Transactional(noRollbackFor = BusinessException.class)

// 只读
@Transactional(readOnly = true) // 优化，对主从有利
```

#### 12.4 回答模板

> 传播行为：REQUIRED默认有加入无创建；REQUIRES_NEW总是新事务。隔离级别READ_COMMITTED防脏读，Mysql默认REPEATABLE_READ。rollbackFor指定回滚异常，readOnly优化只读事务。

---

### 13. 声明式事务原理？

#### 13.1 @Transactional

```java
// 方法级
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    fromAccount.withdraw(amount);
    toAccount.deposit(amount);
}

// 类级：所有public方法都事务
@Transactional(propagation=REQUIRED)
public class AccountService { }

// 传播+隔离+回滚
```

#### 13.2 原理

```java
// 基于AOP
// 1. @Transactional -> TransactionInterceptor
// 2. 解析属性propagation/isolation/rollbackFor
// 3. 获取DataSource>getConnection
// 4. setAutoCommit(false)
// 5. 执行目标方法
// 6. 异常rollback，正常commit
// 7. finally close connection
```

#### 13.3 回答模板

> @Transactional是声明式事务，基于AOP动态代理。原理：获取连接→关闭自动提交→执行业务→异常回滚/正常提交。注意：public方法、无异常不算回滚、代理调用、同类调用不生效。

---

### 14. 如何自定义Repository？

#### 14.1 自定义接口

```java
public interface UserRepositoryCustom {
    List<User> findByComplexQuery(UserQuery query);
}

// 实现
public class UserRepositoryImpl implements UserRepositoryCustom {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<User> findByComplexQuery(...) {
        // JPQL/原生SQL
    }
}

// 继承
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom { }
```

#### 14.2 EntityManager

```java
@PersistenceContext
EntityManager em;

// Criteria API
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> root = query.from(User.class);
query.where(cb.equal(root.get("name"), name));
return em.createQuery(query).getResultList();
```

#### 14.3 回答模板

> 自定义Repository：定义接口+Impl实现类+JpaRepository继承。EntityManager用Criteria API做动态查询。适合复杂查询场景。

---

### 15. 乐观锁和悲观锁？

#### 15.1 乐观锁

```java
// @Version
@Entity
public class Product {
    @Id
    private Long id;

    @Version
    private Long version;
}
```

```java
// JPA自动：SELECT ... WHERE id=? AND version=?
// UPDATE product SET ... version=?+1WHERE id=? AND version=?
// 乐观锁冲突抛ObjectOptimisticLockingFailureException
```

#### 15.2 悲观锁

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select p from Product p where p.id = ?1")
Product findProductByIdWithLock(Long id);

// SELECT ... FOR UPDATE（数据库）
```

#### 15.3 回答模板

> 乐观锁@Version字段，更新时version+1，version不匹配抛异常。适合并发不太高的场景。悲观锁SELECT FOR UPDATE，加行锁，适合高并发写场景。注意MySQL innoDB支持行锁，MyISAM只支持表锁。

---

## 第三章 Web开发篇（高频 ★★★★★）

### 16. RESTful API设计？

#### 16.1 REST原则

```java
// 资源命名用名词
GET    /users           获取用户列表
GET    /users/{id}      获取单个用户
POST   /users           创建用户
PUT    /users/{id}      全量更新
PATCH /users/{id}      部分更新
DELETE /users/{id}      删除

// 路径嵌套表示关联
GET /users/{uid}/orders    用户的订单
GET /orders/{oid}/items   订单的商品
```

#### 16.2 响应状态码

```java
// 成功2xx
200 OK
201 Created
204 No Content // DELETE

// 3xx重定向
304 Not Modified

// 4xx客户端错误
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
422 Unprocessable Entity

// 5xx服务端错误
500 Internal Server Error
503 Service Unavailable
```

#### 16.3 回答模板

> RESTful用名词Endpoint：GET查、POST创、PUT全量、PATCH部分、DELETE删。状态码：200成功、201创、204删无返回、401无认证、403禁止、404找不到、500/503错误。

---

### 17. @RequestMapping变体？

#### 17.1 简化注解

```java
// @RequestMapping变体
@GetMapping("/users")    // @RequestMapping(method = GET)
@PostMapping("/users")   // @RequestMapping(method = POST)
@PutMapping("/users")   // @RequestMapping(method = PUT)
@DeleteMapping("/users") // @RequestMapping(method = DELETE)
@PatchMapping("/users") // @RequestMapping(method = PATCH)
```

#### 17.2 参数绑定

```java
// 路径参数
@GetMapping("/users/{id}")
public User get(@PathVariable Long id) { }

// Query参数
@GetMapping("/users")
public List<User> list(@RequestParam(defaultValue = "10") Integer page,
                    @RequestParam(defaultValue = "10") Integer size) { }

// 请求体
@PostMapping("/users")
public User create(@RequestBody @Valid User user) { }

// 请求头
@GetMapping("/users")
public List<User> list(@RequestHeader("Authorization") String token) { }
```

#### 17.3 回答模板

> @GetMapping等是@RequestMapping简写。@PathVariable路径变量、@RequestParam查询参数、@RequestBody请求体JSON映射、@RequestHeader请求头。@Valid+Hibernate Validator校验。

---

### 18. 统一异常处理？

#### 18.1 @ControllerAdvice

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Result handle(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public Result handle(ValidationException e) {
        return Result.error(400, e.getBindingResult().getFieldError().getDefaultMessage());
    }
}
```

#### 18.2 @ResponseStatus

```java
@ResponseStatus(HttpStatus.FORBIDDEN)
public class ForbiddenException extends RuntimeException { }
```

#### 18.3 回答模板

> @ControllerAdvice+@ExceptionHandler统一异常处理。返回Result统一响应格式。@ResponseStatus给异常指定HTTP状态码。注意异常被处理才会返回正确码。

---

### 19. Interceptor和Filter区别？

#### 19.1 HandlerInterceptor

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                        HttpServletResponse response,
                        Object handler) {
        // 预处理，返回true继续，false中断
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                        HttpServletResponse response,
                        Object handler,
                        ModelAndView modelAndView) { }

    @Override
    public void afterCompletion(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           Exception ex) { }
}
```

```java
// 配置拦截器
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(Interceptors Registry) {
        registry.addInterceptor(new AuthInterceptor())
               .addPathPatterns("/api/**");
    }
}
```

#### 19.2 Filter

```java
@WebFilter(urlPatterns = "/*")
public class LogFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request,
                       ServletResponse response,
                       FilterChain chain) {
        chain.doFilter(request, response);
    }
}
```

#### 19.3 区别

| 特性 | Filter | Interceptor |
|------|-------|-----------|
| 归属 | Servlet规范 | Spring MVC |
| 作用范围 | 所有请求 | 精确Controller |
| 可获Bean | 不能直接 | 可直接获 |
| 执行顺序 | 先Filter后Interceptor | 后Filter |

#### 19.4 回答模版

> Filter是Servlet规范，Interceptor是Spring MVC。Filter先执行，无法直接获@Bean需从容器取。Interceptor在Controller处理前后执行，可用。Filter适合编码/日志/安全，Interceptor适合权限/Cookie。

---

### 20. Spring Boot Starter？

#### 20.1 官starter

```java
// 常用 Starter
spring-boot-starter-web      // Web和Tomcat
spring-boot-starter-data-jpa  // JPA + Hibernate
spring-boot-starter-validation // Validation
spring-boot-starter-test     // 测试JUnit/Mockito
spring-boot-starter-actuator // 监控
spring-boot-starter-aop      // AOP
```

#### 20.2 自定义Starter

```java
// 1. 模块Jar
// 2. spring factories
// org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
//   XxxAutoConfiguration

// 3. @ConfigurationProperties
@Data
@ConfigurationProperties(prefix = "xxx")
public class XxxProperties { }

// 4. @EnableXxxProperties
@EnableConfigurationProperties(XxxProperties.class)
```

#### 20.3 回答模板

> Starter是预配置的依赖集合，xxx-starter-web=web+tomcat+jackson。一个jar包含配置+JAVA类。用spring.factories注册自动配置@EnableAutoConfiguration，@ConfigurationProperties绑定属性。

---

## 第四章 整合与配置篇（中高级 ★★★★）

### 21. Spring Boot自动配置原理？

#### 21.1 启动流程

```java
// 1. SpringApplication.run()
// 2. 创建ApplicationContext
// 3. 初始化
// -> SpringFactoriesLoader加载META-INF/spring.factories
// -> 加载EnableAutoConfiguration
// -> 反射创建配置类实例
// -> @Conditional判断
// -> @AutoConfigureBefore/After排序
// 4. refresh()启动容器
```

#### 21.2 条件注解

```java
// 条件注解
@ConditionalOnClass(DataSource.class) // classpath有
@ConditionalOnMissingBean(UserService.class) // 无该Bean
@ConditionalOnProperty(prefix = "spring.datasource", name = "url") // 属性满足
@ConditionalOnBean(DataSource.class) // 容器中有Bean
@Conditional(Expression) // SpEL为true
```

#### 21.3 回答模板

> 启动时SpringFactoriesLoader加载META-INF/spring.factories的EnableAutoConfiguration，反射创建@Configuration bean，用@ConditionalOn*判断是否实例化。配置类@@EnableConfigurationProperties绑定属性。

---

### 22. Spring Boot配置文件？

#### 22.1 多环境配置

```plaintext
application.properties
application-dev.properties
application-test.properties
application-prod.properties

# 激活 profile
spring.profiles.active=dev
```

#### 22.2 配置属性

```properties
# 默认配置
server.port=8080
spring.application.name=user-service

# 自定义
user.prefix=/api
user.timeout=30

# random
user.secret=${random.uuid}
user.number=${random.int}
```

#### 22.3 YAML配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: ${DB_PASSWORD}

user:
  id: ${random.int}

server:
  port: ${PORT:8080}
```

#### 22.4 回答模板

> Spring Boot用application.properties/yml + profile多环境。用spring.profiles.active激活，server.port��预设配置，自定义用@ConfigurProperties映射。${random}可生成随机值。

---

### 23. 全局异常处理？

#### 23.1 @ExceptionHandler

```java
@RestControllerAdvice // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public Result handle(Exception e) {
        log.error(e.getMessage(), e);
        return Result.error(500, "系统错误");
    }

    @ExceptionHandler(BusinessException.class)
    public Result handle(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }
}
```

#### 23.2 @ControllerAdvice

```java
// @ControllerAdvice可指定basePackages
@RestControllerAdvice(basePackages = "com.demo.controller")

// @InitBinder 预处理Binder
@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.registerCustomEditor(Date.class, new CustomDateEditor(...));
}
```

#### 23.3 回答模板

> @RestControllerAdvice+@ExceptionHandler统一异常处理。全局处理所有Controller异常，返回统一Result。指定value只处理特定Controller，basePackages扫描指定包。

---

### 24. 拦截器实现权限验证？

#### 24.1 Interceptor权限

```java
@Component
public class TokenInterceptor implements HandlerInterceptor {
    @Autowired UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request,
                          HttpServletResponse response,
                          Object handler) {
        String token = request.getHeader("Authorization");
        if (token != null) {
            User user = userService.getUserByToken(token);
            request.setAttribute("currentUser", user);
        }
        return true;
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor())
               .addPathPatterns("/api/**")
               .excludePathPatterns("/api/login");
    }
}
```

#### 24.2 回答模板

> 用HandlerInterceptor.preHandle做权限校验，Token从Header获取，校验后setAttribute传值。在WebConfig配置addPathPatterns路径匹配，excludePathPatterns排除不需要拦截的路径如登录。

---

### 25. CORS跨域？

#### 25.1 @CrossOrigin

```java
// 方法级
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/users")
public List<User> list() { }

// 或全局配置
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
               .allowedOrigins("http://localhost:3000")
               .allowedMethods("GET", "POST")
               .allowedHeaders("*")
               .allowCredentials(true)
               .maxAge(3600);
    }
}
```

#### 25.2 Filter跨域

```java
@Bean
public CorsFilter corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:3000"));
    source.registerCorsConfiguration("/api/**", config);
    return new CorsFilter(source);
}
```

#### 25.3 回答模板

> @CrossOrigin/CrossRegistry加CORS配置，allowedOrigins origins，allowedMethods方法，allowedHeaders头，allowCredentials withCredentials=true，maxAge预检请求缓存时间。Nginx代理也可配置CORS。

---

### 26. Spring Security？

#### 26.1 基本配置

```java
// 继承WebSecurityConfigurerAdapter
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
           .antMatchers("/login", "/register").permitAll()
           .antMatchers("/admin/**").hasRole("ADMIN")
           .anyRequest().authenticated()
           .and()
           .formLogin().loginPage("/login")
           .and()
           .logout().logoutSuccessUrl("/");
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
           .withUser("admin").password("{noop}123456").roles("ADMIN");
    }
}
```

#### 26.2 核心概念

```java
// 认证 Authentication
// 授权 Authorization

// UserDetails
// UserDetailsService
// GrantedAuthority
// AuthenticationProvider

// 过滤器链
// UsernamePasswordAuthenticationFilter
// BasicAuthenticationFilter
// ExceptionTranslationFilter
// FilterSecurityInterceptor
```

#### 26.3 回答模板

> Spring Security用filter链处理认证授权，authorizeRequests()配置路径权限，formLogin()表单登录，hasRole("ADMIN")角色检查。UserDetailsService加载用户信息，PasswordEncoder密码编码。JWT可在UsernamePasswordAuthenticationFilter前加JWT Filter。

---

### 27. Spring Cache？

#### 27.1 @Cache

```java
// 启用缓存
@EnableCaching

// @Cacheable
@Cacheable(value = "users", key = "#id", unless = "#result == null")
public User findById(Long id) { }

// @CacheEvict
@CacheEvict(value = "users", key = "#id")
public User save(User user) { }

// @CachePut
@CachePut(value = "users", key = "#result.id")
public User update(User user) { }

// @Caching
@Caching(evict = {@CacheEvict(...), @CachePut(...)})
public User saveOrUpdate(User user) { }
```

#### 27.2 RedisCacheManager

```java
// 配置
@Bean
public CacheManager cacheManager(RedisTemplate<String, Object> template) {
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofHours(1));
    return RedisCacheManager.builder(template)
        .cacheDefaults(config).build();
}
```

#### 27.3 回答模板

> @Cacheable缓存查询方法结果，@CacheEvict删除缓存，@CachePut更新并缓存新值。@CacheConfig类级统一配置。Spring Boot RedisCacheManager自动配置，需要引入spring-boot-starter-data-redis。

---

### 28. Schedule定时任务？

#### 28.1 @Scheduled

```java
// 启动.EnableScheduling
@SpringBootApplication
@EnableScheduling
public class App { }

// @Scheduled
@Scheduled(cron = "0 0/5 * * * ?") // 每5分钟
@Scheduled(fixedRate = 5000) // 每5秒
@Scheduled(fixedDelay = 5000) // 上次完成后5秒
@Scheduled(initialDelay = 10000) // 首次延迟10秒
```

```cron
# Cron表达式
秒 分 时 日 月 周 [年]

0 0 12 * * ?     // 每天12点
0 */5 * ? * *   // 每5分钟
0 0 10,18 ? * * // 10点和18点
0 0/5 14 ? * *  // 下午2点每5分钟
```

#### 28.2 异步执行

```java
// @Async
@EnableAsync

@Async
@Scheduled(cron = "0 0 2 * * ?")
public void backup() {
    // 异步执行，不阻塞主线程
}
```

#### 28.3 回答模板

> @Scheduled(cron)或fixedRate执行定时任务。cron是6位秒分有时日月周，可省略年。@Async异步执行。@EnableScheduling启用。任务耗时可用@Async。

---

### 29. 国际化i18n？

#### 29.1 国际化配置

```plaintext
messages.properties        # 默认
messages_zh_CN.properties # 中文
messages_en_US.properties # 英文
```

```properties
# messages.properties
user.not.found=User not found
user.save.success=User saved successfully
```

#### 29.2 使用

```java
// MessageSource
@Autowired MessageSource ms;

String msg = ms.getMessage("user.not.found",
                          new Object[]{name},
                          Locale.US); // 指定locale
```

#### 29.3 回答模板

> 创建messages.properties + localesuffix定义多语言文件，MessageSource.getMessage(key, locale)获取。@Value("${spring.messages.basename:}")改默认。

---

### 30. Validation校验？

#### 30.1 注解校验

```java
// 实体
public class User {
    @NotNull(message = "ID不能为空")
    @Min(1)
    private Long id;

    @NotBlank(message = "用户名不能为空")
    @Size(min = 4, max = 20)
    private String name;

    @Email
    private String email;

    @Pattern(regexp = "^1[3-9]\\d{9}$")
    private String phone;
}

// Controller
@PostMapping
public User create(@Valid @RequestBody User user) {
    // @Valid 激活校验
}
```

```java
// 分组校验
@PostMapping public User create(@Validated(Create.class) User u) {}
@PutMapping public User update(@Validated(Update.class) User u) {}

public interface Create {}
public interface Update {}

// 自定义
@Constraint(validatedBy = PhoneValidator.class)
@Target({FIELD})
@Retention(RUNTIME)
public @interface Phone { }

public class PhoneValidator implements ConstraintValidator<Phone, String> {
    @Override public boolean isValid(String value, ConstraintValidatorContext context) {...}
}
```

#### 30.2 回答模板

> @Valid+Hibernate Validator校验。常用@NotNull/Blank/Min/Max/Size/Email/Pattern。分组@Validated(Create.class)配合@ValidatedOnDefault groups不同场景。自定义@Constraint+Validator。

---

## 第五章 微服务篇（中高级 ★★★★）

### 31. Spring Cloud核心组件？

#### 31.1 Netflix Components

```plaintext
Spring Cloud Netflix：
├── Eureka           // 服务注册中心
├── Ribbon           // 客户端负载均衡
├── Feign            // 声明式HTTP客户端
├── Hystrix          // 熔断器
├── Zuul/Gateway     // 网关
└── Config          // 配置中心
```

#### 31.2 Alibaba Component

```plaintext
Spring Cloud Alibaba：
├── Nacos           // 注册+配置中心
├── Sentinel         // 熔断+限流
├── Seata           // 分布式事务
└── Dubbo          // RPC框架
```

#### 31.3 服务通信

```java
// Feign
@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/user/{id}")
    User getById(@PathVariable("id") Long id);
}

// Dubbo
@DubboReference
private UserService userService;
```

#### 31.4 回答模板

> Spring Cloud有Netflix和Alibaba两大生态。Eureka/Nacos服务注册、Feign/Sentinel调用、Ribbon/Nginx负载均衡、Hystrix/Sentinel熔断。Alibaba生态Nacos+Sentinel+Seata。

---

### 32. 服务注册与发现？

#### 32.1 Nacos

```properties
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
```

```java
// 服务提供者
@RestController
public class ProviderController {
    @Value("${spring.application.name}")
    String name;
}

// 服务消费者
@LoadBalanced
@Bean
public RestTemplate restTemplate() { return new RestTemplate(); }

// 或 Feign
@FeignClient("user-service")
public interface UserClient { }
```

#### 32.2 Eureka

```properties
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
spring:
  application:
    name: user-service
```

#### 32.3 回答模板

> 服务注册中心有Eureka（Huawei闭源）和Nacos（Alibaba开源+配置中心）。配置spring.cloud.nacos.discovery.server-addr启用。@EnableDiscoveryClient让服务能被被发现。

---

### 33. 熔断器Hystrix？

#### 33.1 @HystrixCommand

```java
@Service
public class UserService {
    @Autowired RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "getUserFallback")
    public User getUser(Long id) {
        return restTemplate.getForObject("http://user-service/user/" + id, User.class);
    }

    public User getUserFallback(Long id) {
        return User.builder().id(id).name("default").build();
    }
}
```

```java
// @Feign + Hystrix
@FeignClient(name = "user-service", fallback = UserFallback.class)
public interface UserClient { }

@Component
public class UserFallback implements UserClient { ... }
```

#### 33.2 配置

```properties
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 5000
  circuitBreaker:
    requestVolumeThreshold: 20
    sleepWindowInMilliseconds: 5000
```

#### 33.3 回答模板

> @HystrixCommand(fallbackMethod)定义降级逻辑，服务不可用时执行fallback。Ribbon超时+Hystrix超时=超长时间，Ribbon先超时也会failback。@FeignClient加fallback开启Hystrix。

---

### 34. 网关Gateway？

#### 34.1 Gateway

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/user/**
          filters:
            - StripPrefix=1
        - id: baidu
          uri: https://www.baidu.com
          predicates:
            - Query=url,baidu
```

```java
// 路由 Predicate
Predicates:
- Path=/user/**
- Method=GET
- Header=X-Request-Id, \d+
- Query=url, baidu
- Cookie=name, john
- Before=2020-01-01T00:00:00.000Z
- After=...

// Filter
filters:
- AddRequestHeader=X-Request-Id,abc
- StripPrefix=1
- PrefixPath=/api
- Redirect=302,/login
```

#### 34.2 回答模板

> Spring Cloud Gateway用Predicate路由、Filter处理。Path路由predicates，StripPrefix去掉前缀，lb://负载均衡ribbon或loadbalancer。RouteLocatorBuilder可Java代码定义路由。

---

### 35. Seata分布式事务？

#### 35.1 AT模式

```java
// 开启seata
@GlobalTransactional
@Transactional
public void purchase(String userId, String commodityCode, int count) {
    // 自动管理分支事务
}

//undo_log表记录前后镜像
//seata_at_inner_undo_log table
```

```properties
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: my_test_tx_group
```

```java
// 配置
@DependsOn("seataProperties")
@Bean
public GlobalTransactionScanner globalTransactionScanner() {
    return new GlobalTransactionScanner("app", "tx-group");
}
```

#### 35.2 三种模式

```plaintext
AT：自动 undo_log
TCC：Try/Confirm/Cancel
SAGA：状态机编排
XA：数据库XA
```

#### 35.3 回答模板

> Seata（Alibaba分布式事务）三种模式：AT（自动undo_log）、TCC（手动的Try Confirm Cancel）、SAGA（状态机），XA（依赖数据库XA）。AT是最简单的模式，需undo_log表和@GlobalTransactional。

---

### 36. Sleuth链路追踪？

#### 36.1 添加依赖

```xml
<!-- Spring Cloud Sleuth -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>

<!-- Zipkin -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### 36.2 使用

```java
// 注入Tracer
@Autowired Tracer tracer;

Span span = tracer.nextId().start();
span.tag("userId", id);
try {
    // do work
} finally {
    span.finish();
}

// 日志会自动添加traceId、spanId
// [app] traceId spanId INFO - message
```

#### 36.3 回答模板

> Sleuth埋点自动追踪，log输出traceId和spanId。Zipkin收集展示Trace。spring.zipkin.base-url配置Zipkin地址，zipkin.sender.type=messaging可选web/stream/rabbit。

---

### 37. Config配置中心？

#### 37.1 配置

```properties
# bootstrap.yml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      name: app-name
      profile: dev
      # label: master
      # discovery: service-id
```

#### 37.2 动态刷新

```java
// 刷新Config
@RefreshScope
@RestController
public class ConfigController {

    @Value("${user.name}")
    String name;

    // POST /actuator/refresh 刷新
}
```

```properties
# 暴露端点
management.endpoints.web.exposure.include=refresh
```

#### 37.3 回答模板

> Spring Cloud Config用bootstrap.yml优先级高先加载配置服务端。@RefreshScope动态刷新。Git后端（默认）存储配置，配置服务热部署用POST /actuator/refresh。Spring Cloud Bus广播刷新。

---

### 38. Feign原理？

#### 38.1 Feign

```java
// 定义接口
@FeignClient(name = "user-service", path = "/user")
public interface UserClient {

    @GetMapping("/{id}")
    User getById(@PathVariable("id") Long id);

    @PostMapping
    Result create(@RequestBody User user);
}

// 调用
@Autowired UserClient userClient;
User user = userClient.getById(1L);
```

#### 38.2 原理

```java
// 1. @EnableFeignClients扫描@FeignClient
// 2. 生成JDK动态代理
// 3. InvokeHandler处理方法
// 4. RequestTemplate构造请求
// 5. Ribbon负载均衡
// 6. HTTP Client执行
```

#### 38.3 回答模板

> @FeignClient定义HTTP接口，编译后生成代理，方法调用生成RequestTemplate + LoadBalancer负载均衡 + HTTP Client发送_request。@EnableFeignClients开启。

---

### 39. Ribbon负载均衡？

#### 39.1 @LoadBalanced

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// 使用
String url = "http://user-service/user/1";
User user = restTemplate.getForObject(url, User.class);
```

#### 39.2 负载均衡策略

```java
@Bean
public IRule ribbonRule() {
    // RoundRobinRule 轮询（默认）
    // RandomRule 随机
    // BestAvailableRule 最空闲
    // WeightedResponseTimeRule 根据响应时间加权
    return new RandomRule();
}
```

#### 39.3 回答模板

> @LoadBalanced RestTemplate实现负载均衡。默认RoundRobin轮询，也支持Random、BestAvailable等。Ribbon 7种Rule可自定义。

---

### 40. OAuth2+JWT？

#### 40.1 JWT

```java
// JJWT生成
JwtBuilder builder = Jwts.builder()
    .setId(UUID.randomUUID().toString())
    .setSubject(username)
    .claim("roles", roles)
    .signWith(SignatureAlgorithm.HS256, secretKey);
String token = builder.setExpiration(new Date(System.currentTimeMillis() + EXPIRE)).compact();

// JJWT解析
Claims claims = Jwts.parser()
    .setSigningKey(secretKey)
    .parseClaimsJws(token)
    .getBody();
```

#### 40.2 Spring Security OAuth2

```java
// 授权服务器
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    // tokenStore: JwtTokenStore
    // accessTokenConverter: JwtAccessTokenConverter
    // tokenEnhancer: TokenEnhancer
}

// Resource服务器
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    // resourceId
    // tokenStore
}
```

#### 40.3 回答模板

> JWT token分三部分：Header.Payload.Signature用HS256签名。Token可Base64Decode解码获取Payload，Spring Security OAuth2提供完整实现。公钥私钥非对称RS256更安全。

---

## 第六章 测试与优化（中高级 ★★★★）

### 41. 单元测试Mock？

#### 41.1 @SpringBootTest

```java
@SpringBootTest
class UserServiceTest {
    @Autowired UserService userService;

    @Test
    void testFindById() {
        User u = userService.findById(1L);
        assertNotNull(u);
    }
}
```

#### 41.2 Mock

```java
// @MockBean
@SpringBootTest
class DemoTest {
    @MockBean
    UserDao userDao;

    @Test
    void test() {
        when(userDao.findById(1L)).thenReturn(user);
        // 验证
    }
}
```

#### 41.3 Slice Test

```java
// WebMvcTest：Controller层测试
@WebMvcTest(UserController.class)

// DataJpaTest：数据层测试
@DataJpaTest

// WebClientTest：http client
@WebClientTest

// RestDocs测试
```

#### 41.4 回答模板

> @SpringBootTest全容器，@MockBean Mock依赖，Mockito.when模拟。Slice test @WebMvcTest只启动Web层，@DataJpaTest只Data层。测试隔离用@DirtiesContext。

---

### 42. 性能优化建议？

#### 42.1 SQL优化

```java
// 避免N+1
@EntityGraph(attributePaths = {"roles"})
@Query("select u from User u left join fetch u.roles")

// 分页
Page<User> findByName(String name, Pageable pageable);

// 投影
interface UserDTO { String getName(); Long getId(); }
@Query("select new com.demo.UserDTO(u.id, u.name) from User u")
List<UserDTO> findUserDTO();
```

#### 42.2 缓存

```java
@Cacheable(value = "users", key = "#id")
public User findById(Long id) { }

// 索引
@Table(indexes = @Index(columnList = "email"))
```

#### 42.3 回答模板

> 减少N+1：@EntityGraph/@Query join fetch。分页用Page。缓存@Cacheable。数据库加索引@Table indexes。JPA batch-size配置批量操作。

---

### 43. Spring Boot Actuator？

#### 43.1 Endpoints

```properties
# 暴露
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=health,metrics

# 或单个
management.endpoint.health.show-details=always
management.endpoint.health.show-details=when-authorized
```

```java
// 常用端点
/actuator/health        # 健康检查
/actuator/beans         # Bean列表
/actuator/mappings      # URL映射
/actuator/env          # 环境变量
/actuator/configprops  # 配置属性
/actuator/threaddump   # 线程dump
/actuator/heapdump    # 堆dump
/actuator Metrics     # 指标
/actuator/scheduledtasks # 定时任务
/actuator/conditions   # 自动配置条件
```

#### 43.2 HealthIndicator

```java
@Component
public class MyHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // check
        if (unhealthy) {
            return Health.down().withDetail("msg", "error").build();
        }
        return Health.up().build();
    }
}
```

#### 43.3 回答模板

> Spring Boot Actuator提供监控端点，/actuator/health健康检查。management.endpoints.web.exposure.include暴露端点。HealthIndicator可自定义健康检查逻辑。

---

### 44. 常用配置参数？

#### 44.1 DataSource

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/demo
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=30000
```

#### 44.2 JPA

```properties
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=none
spring.jpa.open-in-view=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.batch_size=20
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.properties.hibernate.generate_statistics=true
```

#### 44.3 Logging

```properties
logging.level.root=INFO
logging.level.com.demo=DEBUG
logging.pattern.console=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
logging.file.name=/var/log/app.log
logging.file.max-size=10MB
logging.file.max-history=30
```

#### 44.4 回答模板

> 常用配置分DataSource（连接池hikari性能好）、JPA（ddl-auto=none安全）、日志。Hikari minimum-idle最小空闲连接。show-sql开发看SQL。

---

### 45. 常用起步依赖？

#### 45.1 Web

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### 45.2 Data

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 45.3 Dev

```xml
<!-- devtools -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- actuator -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 45.4 回答模板

> Starter分Web（web+tomcat+json）、Data（JPA+Hibernate+Redis+Lettuce）、Security、AOP、Actuator（监控）、DevTools（dev热重启）。生产不加devtools。

---

## 附录：面试追问

1. **Spring Boot启动流程？**
   SpringApplication.run() → create ApplicationContext → refresh() → afterRefresh → callRunners

2. **Spring 如何解决循环依赖？**
   三级缓存+singletonObjects + earlySingletonObjects + singletonFactory。Spring 5.1后有的Bean不能解决

3. **BeanFactory vs ApplicationContext？**
   BeanFactory是底层，ApplicationContext是扩展有国际化+AOP+事件+web等功能

4. **Spring事务失效原因？**
   非public方法、try catch、异常被catch、代理调用同类、传播NOT_SUPPORTED

5. **SpringBoot自动配置原理？**
   @EnableAutoConfiguration → META-INF/spring.factories加载配置类 → @Conditional条件判断

---

## 参考资料

- 《Spring Boot实战》
- 《Spring源码深度解析》
- Spring官方文档

---

> 整理by Claude Code | Spring面试高频100问