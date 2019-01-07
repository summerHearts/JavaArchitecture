##1、基础概念简介
- 1、AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。另外还有面向过程编程如C，函数式编程，事件驱动编程等等。
- 2、AOP是一种编程范式，不是编程语言。解决了特定问题，不能解决所有问题。是OOP的补充，不是替代。
- 3、AOP的初衷：
  - DRY： 不要重复你的代码
  - SOC： Separtion of Concerns
  
    - 水平分离： 展示层--> 服务层 --> 持久层
    - 垂直分离： 模块划分(订单、库存)
    - 切面分离：  分离功能性需求和非功能性需求
- 4、使用AOP的好处
  - 1、集中处理某一关注点/横切逻辑
  - 2、可以很方便的添加/删除关注点
  - 3、侵入性少，增强代码可读性及维护性。

- 5、AOP的应用场景
  权限控制、缓存控制、事务控制、审计日志、性能监控、分布式追踪、异常处理
  
- 6、Aspect(切面)

   - aspect 由 pointcount 和 advice 组成, 它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP就是负责实施切面的框架, 它将切面所定义的横切逻辑织入到切面所指定的连接点中.
   - AOP的工作重心在于如何将增强织入目标对象的连接点上, 这里包含两个工作:

      - 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
      - 如何在 advice 中编写切面代码.
      - 可以简单地认为, 使用 @Aspect 注解的类就是切面.

- 7、advice(增强)
  - 由 aspect 添加到特定的 join point(即满足 point cut 规则的 join point) 的一段代码.
许多 AOP框架, 包括 Spring AOP, 会将 advice 模拟为一个拦截器(interceptor), 并且在 join point 上维护多个 advice, 进行层层拦截.

  - 例如 HTTP 鉴权的实现, 我们可以为每个使用 RequestMapping 标注的方法织入 advice, 当 HTTP 请求到来时, 首先进入到 advice 代码中, 在这里我们可以分析这个 HTTP 请求是否有相应的权限, 如果有, 则执行 Controller, 如果没有, 则抛出异常. 这里的 advice 就扮演着鉴权拦截器的角色了.

- 8、连接点(join point)
 
      -  程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理.
      -  在 Spring AOP 中, join point 总是方法的执行点, 即只有方法连接点.
      
 - 9、切点(point cut)
     - 在 Spring 中, 所有的方法都可以认为是 joinpoint, 但是我们并不希望在所有的方法上都添加 Advice, 而 pointcut 的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配joinpoint, 给满足规则的 joinpoint 添加 Advice.

 - 10、关于join point 和 point cut 的区别
     -  在 Spring AOP 中, 所有的方法执行都是 join point. 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 因此 join point 和 point cut 本质上就是两个不同纬度上的东西.
     -  advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice
 - 11、introduction
     - 为一个类型添加额外的方法或字段. Spring AOP 允许我们为 目标对象 引入新的接口(和对应的实现). 例如我们可以使用 introduction 来为一个 bean 实现 IsModified 接口, 并以此来简化 caching 的实现.
  - 12、目标对象(Target)
     - 织入 advice 的目标对象. 目标对象也被称为 advised object.
     - 因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)
     - 注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类.
     
  - 13、AOP proxy

     - 一个类被 AOP 织入 advice, 就会产生一个结果类, 它是融合了原类和增强逻辑的代理类.
     - 在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象.

     - 14、织入(Weaving)

     - 将 aspect 和其他对象连接起来, 并创建 adviced object 的过程.
     - 根据不同的实现技术, AOP织入有三种方式:
         - 编译器织入, 这要求有特殊的Java编译器.
         - 类装载期织入, 这需要有特殊的类装载器.
         - 动态代理织入, 在运行期为目标类添加增强(Advice)生成子类的方式.
         - Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入.

    - 15、advice 的类型
     
     - before advice, 在 join point 前被执行的 advice. 虽然 before advice 是在 join point 前被执行, 但是它并不能够阻止 join point 的执行, 除非发生了异常(即我们在 before advice 代码中, 不能人为地决定是否继续执行 join point 中的代码)
     - after return advice, 在一个 join point 正常返回后执行的 advice
     - after throwing advice, 当一个 join point 抛出异常后执行的 advice
     - after(final) advice, 无论一个 join point 是正常退出还是发生了异常, 都会被执行的 advice.
     - around advice, 在 join point 前和 joint point 退出后都执行的 advice. 这个是最常用的 advice.

  - 16、关于 AOP Proxy
     
     - Spring AOP 默认使用标准的 JDK 动态代理(dynamic proxy)技术来实现 AOP 代理, 通过它, 我们可以为任意的接口实现代理.
     
     - 如果需要为一个类实现代理, 那么可以使用 CGLIB 代理. 当一个业务逻辑对象没有实现接口时, 那么Spring AOP 就默认使用 CGLIB 来作为 AOP 代理了. 即如果我们需要为一个方法织入 advice, 但是这个方法不是一个接口所提供的方法, 则此时 Spring AOP 会使用 CGLIB 来实现动态代理. 鉴于此, Spring AOP 建议基于接口编程, 对接口进行 AOP 而不是类.

     
     
##2、彻底理解 aspect, join point, point cut, advice

```
看完了上面的理论部分知识, 我相信还是会有不少朋友感觉到 AOP 的概念还是很模糊, 对 AOP 中的各种概念理解的还不是很透彻. 其实这很正常, 因为 AOP 中的概念是在是太多了, 我当时也是花了老大劲才梳理清楚的.
下面我以一个简单的例子来比喻一下 AOP 中 aspect, jointpoint, pointcut 与 advice 之间的关系.

让我们来假设一下, 从前有一个叫爪哇的小县城, 在一个月黑风高的晚上, 这个县城中发生了命案. 作案的凶手十分狡猾, 现场没有留下什么有价值的线索. 不过万幸的是, 刚从隔壁回来的老王恰好在这时候无意中发现了凶手行凶的过程, 但是由于天色已晚, 加上凶手蒙着面, 老王并没有看清凶手的面目, 只知道凶手是个男性, 身高约七尺五寸. 爪哇县的县令根据老王的描述, 对守门的士兵下命令说: 凡是发现有身高七尺五寸的男性, 都要抓过来审问. 士兵当然不敢违背县令的命令, 只好把进出城的所有符合条件的人都抓了起来.

来让我们看一下上面的一个小故事和 AOP 到底有什么对应关系.
首先我们知道, 在 Spring AOP 中 join point 指代的是所有方法的执行点, 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 对应到我们在上面举的例子, 我们可以做一个简单的类比, join point 就相当于 爪哇的小县城里的百姓, point cut 就相当于 老王所做的指控, 即凶手是个男性, 身高约七尺五寸, 而 advice 则是施加在符合老王所描述的嫌疑人的动作: 抓过来审问.
为什么可以这样类比呢?
```   
  
```
join point --> 爪哇的小县城里的百姓: 因为根据定义, join point 是所有可能被织入 advice 的候选的点, 在 Spring AOP中, 则可以认为所有方法执行点都是 join point. 而在我们上面的例子中, 命案发生在小县城中, 按理说在此县城中的所有人都有可能是嫌疑人.
```

```
point cut --> 男性, 身高约七尺五寸: 我们知道, 所有的方法(joint point) 都可以织入 advice, 但是我们并不希望在所有方法上都织入 advice, 而 pointcut 的作用就是提供一组规则来匹配joinpoint, 给满足规则的 joinpoint 添加 advice. 同理, 对于县令来说, 他再昏庸, 也知道不能把县城中的所有百姓都抓起来审问, 而是根据凶手是个男性, 身高约七尺五寸, 把符合条件的人抓起来. 在这里 凶手是个男性, 身高约七尺五寸 就是一个修饰谓语, 它限定了凶手的范围, 满足此修饰规则的百姓都是嫌疑人, 都需要抓起来审问.
```
```
advice --> 抓过来审问, advice 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 join point 上的. 同理, 对比到我们的例子中, 抓过来审问 这个动作就是对作用于那些满足 男性, 身高约七尺五寸 的爪哇的小县城里的百姓.
```
```
aspect: aspect 是 point cut 与 advice 的组合, 因此在这里我们就可以类比: "根据老王的线索, 凡是发现有身高七尺五寸的男性, 都要抓过来审问" 这一整个动作可以被认为是一个 aspect.
```
     
##3、AOP使用

案例背景:

- 产品管理的服务
- 产品添加/删除的操作只能管理员进行
- 普通实现 vs AOP实现

假设现在有一个商品服务，提供商品的增加和删除功能

```
public class Product {
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```
@Service
public class ProductService {

    @Autowired
    AuthService authService;

    public void insert(Product product){
        System.out.println("insert product");
    }

    public void delete(Long id){
        System.out.println("delete product");
    }
}
```
如果要判断当前用户是否有权限去操作商品服务，一般的做法是： 在用户进行操作商品服务之前，进行权限判断。创建一个认证服务，用来判断用户是否有权限操作

```
@Component
public class AuthService {
    //权限校验
    public void checkAccess(){
        String user = CurrentSetHolder.get();
        if(!"admin".equals(user)){
            throw new RuntimeException("operation not allow");
        }
    }
}
```
在商品服务中添加判断操作

```
public void insert(Product product){
     authService.checkAccess();//传统方式进行权限校验
     System.out.println("insert product");
}
```
这种方式属于硬编码方式，对业务有侵入性。不利于后期的维护。

那么使用AOP就方便很多。

首先创建一个注解

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)  //仅在方法上有效
public @interface AdminOnly {
}
```

其次创建一个切面

```
@Aspect
@Component
public class SecurityAspect {

    @Autowired
    AuthService authService;

    @Pointcut("@annotation(AdminOnly)") // 切入的点是带有AdminOnly 注解的方法
    public void adminOnly(){

    }

    @Before("adminOnly()")     // 在方法执行之前首先执行check方法
    public void check(){
        authService.checkAccess();
    }
}
```

然后在需要校验权限的地方添加注解即可

```
@Service
public class ProductService {

    @Autowired
    AuthService authService;

    @AdminOnly
    public void insert(Product product){
        System.out.println("insert product");
    }

    @AdminOnly
    public void delete(Long id){
        System.out.println("delete product");
    }
}
```

 - 1、Spring AOP使用方式
   - 1、XML配置
   - 2、注解方式
   - 都要对切面表达式进行描述 Pointcut expression 

   ![](https://upload-images.jianshu.io/upload_images/325120-0be098e99de45dd4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
   
   ![](https://upload-images.jianshu.io/upload_images/325120-5f50a632c27358a9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
   
   ![](https://upload-images.jianshu.io/upload_images/325120-6f1c372d6eda938d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
   
   ![](https://upload-images.jianshu.io/upload_images/325120-b9d7417a26437ab4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
   
 ```
@Aspect
@Component
public class PkgTypeAspectConfig {

    //匹配com.imooc.service.sub包及子包下所有类的方法
    @Pointcut("within(com.imooc.service.sub.*)")
    public void matchType(){}

    @Before("matchType()")
    public void before(){
        System.out.println("");
        System.out.println("###before");
    }

    //匹配ProductService类里的所有方法
    @Pointcut("within(com.imooc.service.ProductService)")
    public void matchTypes(){}

    @Before("matchTypes()")
    public void befores(){
        System.out.println("");
        System.out.println("###before");
    }
}
 ```
    
  ![](https://upload-images.jianshu.io/upload_images/325120-a6819002d35d6834.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
  
  接下来我们开始分析对象的切面使用方法
  
 ```
 public interface Loggable {
        public void log();
 }
 ```
  
 ```
 @Component
 public class LogService implements Loggable{
 
     @Override
     @AdminOnly
     public void log() {
         System.out.println("log from LogService");
     }

     public void annoArg(Product product){
         System.out.println("execute log service annoArg");
     }
 }
 ```
 
 ```
 @Pointcut("this(com.imooc.log.Loggable)")
 public void matchCondition(){}

    @Before("matchCondition()")
    public void before(){
        System.out.println("");
        System.out.println("###before");
}
 ```
 
 从结果来看，凡是实现了Loggable接口方法，都将被AOP命中。
  
 ```
 @Aspect
 @Component
 public class ObjectAspectConfig {

     @Pointcut("bean(logService)")
     public void matchCondition(){}

     @Before("matchCondition()")
     public void before(){
         System.out.println("");
         System.out.println("###before");
     }
 }
 ```
从结果来看，拦截了logService中Bean的方法。

分析参数的切面使用方法

  ![](https://upload-images.jianshu.io/upload_images/325120-2026ac2aa1be73f2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
  
分析注解的切面使用方法

  ![](https://upload-images.jianshu.io/upload_images/325120-6eb7f905ea2702fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
  
 execution表达式的切面使用方法
 
 ```
  //匹配任何公共方法
 @Pointcut("execution(public * com.imooc.service.*.*(..))")

 //匹配com.imooc包及子包下Service类中无参方法
 @Pointcut("execution(* com.imooc..*Service.*())")

 //匹配com.imooc包及子包下Service类中的任何只有一个参数的方法
 @Pointcut("execution(* com.imooc..*Service.*(*))")

 //匹配com.imooc包及子包下任何类的任何方法
 @Pointcut("execution(* com.imooc..*.*(..))")

 //匹配com.imooc包及子包下返回值为String的任何方法
 @Pointcut("execution(String com.imooc..*.*(..))")

 //匹配异常
 execution(public * com.imooc.service.*.*(..) throws java.lang.IllegalAccessException)
 ```
 
 5种Advice注解
 
 ![](https://upload-images.jianshu.io/upload_images/325120-5f18604e5603ef65.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
 
 1：@Before，前置通知；
 2：@After(finally),后置通知，方法执行完成之后；
 3：@AfterReturning，返回通知，成功执行之后执行；
 4：@AfterThrowing，异常通知，抛出异常之后执行
 5：@Around，环绕通知，环绕着方法执行；
  
  






