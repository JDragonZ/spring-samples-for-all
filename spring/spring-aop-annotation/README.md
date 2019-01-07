# spring AOP（注解方式）

## 一、说明

### 1.1 项目结构说明

1. 切面配置位于com.heibaiying.config下AopConfig.java文件；
2. 自定义切面位于advice下，其中CustomAdvice是标准的自定义切面，FirstAdvice和SecondAdvice用于测试多切面共同作用于同一个被切入点时的执行顺序；
3. OrderService是待切入方法。

![spring+redis项目目录结构](D:\spring-samples-for-all\pictures\spring-aop-annotation.png)



### 1.2 依赖说明

除了spring的基本依赖外，需要导入aop依赖包

```xml
 <!--aop 相关依赖-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>${spring-base-version}</version>
</dependency>
```



## 二、spring aop

#### 2.1 创建待切入接口及其实现类

```java
public interface OrderService {

    Order queryOrder(Long id);

    Order createOrder(Long id, String productName);
}
```

```java
public class OrderServiceImpl implements OrderService {

    public Order queryOrder(Long id) {
        return new Order(id, "product", new Date());
    }

    public Order createOrder(Long id, String productName) {
        // 模拟抛出异常
        // int j = 1 / 0;
        return new Order(id, "new Product", new Date());
    }
}

```

#### 2.2 创建自定义切面类

注：@Pointcut的值可以是多个切面表达式的组合。

```java
/**
 * @author : heibaiying
 * @description : 自定义切面
 */
@Aspect
@Component //除了加上@Aspect外 还需要声明为spring的组件 @Aspect只是一个切面声明
public class CustomAdvice {


    /**
     * 使用 || , or  表示或
     * 使用 && , and 表示与
     * ! 表示非
     */
    @Pointcut("execution(* com.heibaiying.service.OrderService.*(..)) && !execution(* com.heibaiying.service.OrderService.deleteOrder(..))")
    private void pointCut() {

    }

    @Before("pointCut()")
    public void before(JoinPoint joinPoint) {
        //获取节点名称
        String name = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        System.out.println(name + "方法调用前：获取调用参数" + Arrays.toString(args));
    }

    // returning 参数用于指定返回结果与哪一个参数绑定
    @AfterReturning(pointcut = "pointCut()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("后置返回通知结果" + result);
    }

    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕通知-前");
        //调用目标方法
        Object proceed = joinPoint.proceed();
        System.out.println("环绕通知-后");
        return proceed;
    }

    // throwing 参数用于指定抛出的异常与哪一个参数绑定
    @AfterThrowing(pointcut = "pointCut()", throwing = "exception")
    public void afterThrowing(JoinPoint joinPoint, Exception exception) {
        System.err.println("后置异常通知:" + exception);
    }


    @After("pointCut()")
    public void after(JoinPoint joinPoint) {
        System.out.println("后置通知");
    }
}

```

#### 2.3 配置切面

```java
/**
 * @author : heibaiying
 * @description : 开启切面配置
 */
@Configuration
@ComponentScan("com.heibaiying.*")
@EnableAspectJAutoProxy // 开启@Aspect注解支持 等价于<aop:aspectj-autoproxy>
public class AopConfig {
}
```

#### 2.4 测试切面

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = AopConfig.class)
public class AopTest {


    @Autowired
    private OrderService orderService;

    @Test
    public void saveAndQuery() {
        orderService.createOrder(1283929319L, "手机");
        orderService.queryOrder(4891894129L);
    }

    /**
     * 多个切面作用于同一个切入点时，可以用@Order指定切面的执行顺序
     * 优先级高的切面在切入方法前执行的通知(before)会优先执行，但是位于方法后执行的通知(after,afterReturning)反而会延后执行
     */
    @Test
    public void delete() {
        orderService.deleteOrder(12793179L);
    }
}
```

#### 2.5  切面执行顺序

- 多个切面作用于同一个切入点时，可以用@Order指定切面的执行顺序

- 优先级高的切面在切入方法前执行的通知(before)会优先执行，但是位于方法后执行的通知(after,afterReturning)反而会延后执行，类似于同心圆原理。

  ![aop执行顺序](D:\spring-samples-for-all\pictures\aop执行顺序.png)