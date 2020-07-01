# Spring
## Spring的优良特性
- 非侵入式：基于Spring开发的应用中的对象可以不依赖于Spring的API
- 依赖注入: DI,反转控制(IOC)的最经典的实现
- 面向切面编程: AOP
- 容器: Spring是一个容器，i难为他包含并且管理应用对象的生命周期
- 组件化: Spring实现了使用简单的组件配置组合成一个复杂的应用
- 一站式: 可以整合各种ie应用的开源框架和优秀的第三方库类  
## Spring配置文件
- bean元素定义一个有IOC容器创建的对象
- id= 非必须 唯一
- class= 管理的类
- property 为bean的属性赋值
  ```java
   <property name="id" value="100"></property>
   ```
   或通过构造器赋值
   ```java
     <constructor-arg value="10086"></constructor-arg>
   ```
- 初始化 
   ``` java
    ApplicationContext ac=new ClassPathXmlApplicationContext("配置文件位置");
    ```
### bean可以使用的值
1. 字面量 字符串，数字类型，若有特殊字符 可以用 &lt;![CDATA[]]&gt;把字面值包裹起来
2. null值
3. 给bean的级联属性赋值
4. 外部已经声明的bean 通过ref来引用
5. 内部bean 内部bean直接包含在property元素中，不需要设置id或name  **内部bean不能使用在其他任何地方**
```java
<property name="teacher">
           <bean id="tt" class="lxy.di.Teacher">
               <property name="tid" value="2222"></property>
               <property name="tname" value="小张"></property>
           </bean>
       </property>
```
6. Map 通过&lt;map&gt;标签定义，map标签中可以使用多个entry作为子标签，每个条目报包含一个键和一个值
```java
<property name="bossMap">
            <map>
                <entry>
                    <key>
                        <value>1001</value>
                    </key>
                    <value>刘老师</value>
                </entry>
                <entry>
                    <key>
                        <value>1002</value>
                    </key>
                    <value>茉老师</value>
                </entry>
            </map>
        </property>
```
---

## Bean的作用域  
| 类别 | 说明 |  
| :---: | :---: |
| singleton | 在springIOC容器中仅存在一个Bean实例，Bean以单实例的方式存在 |
| prototype | 每次调用getBean()时都会返回一个新的实例 |
| request | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session | 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean。该作用域仅适用于WebApplicationContext环境 |

## Bean的生命周期
1. 通过构造器或者工厂方法创建bean
2. 为bean属性赋值和对其他bean的引用
3. 调用bean初始话方法`ApplicationContext ac=new ClassPathXmlApplicationContext("配置文件位置");`
4. 使用
5. 当容器关闭时，销毁bean

## 引用外部属性文件
1. 创建propeties属性文件
2. 指定propeties属性文件的位置 **context:property-placeholder location=**
````java
  <!--加载资源文件-->
        <context:property-placeholder location="classpath:conf/db.properties"/>
````
3. 从properties属性文件中引入属性值
````java
 <bean id="datasource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${jdbc.driver}"></property>
            <property name="url" value="${jdbc.url}"></property>
            <property name="username" value="${jdbc.username}"></property>
            <property name="password" value="${jdbc.password}"></property>
        </bean>
````


## 通过注解来配置bean
1. 普通组件 **@Component** 标识一个受Spring IOC容器管理的组件
2. 持久化层组件 **@Repository** 标识一个受Spring IOC容器管理的持久化组件
3. 业务逻辑层组件 **@Service** 标识一个受Spring IOC容器管理的业务逻辑层组件
4. 表述层控制组件 **@Controller** 标识一个受Spring IOC容器管理的表述层控制组件
5. 命名规则
    1. 默认情况： 使用组件的简单类名首字母小写后得到的字符串作为bean的id
    2. 使用组件注解的value属性指定bean的id
    3. **@Component，@Repository，@Service，@Controller这几个注解没有区别，仅仅是为了让开发人员了解当前组件所扮演的角色**

## 扫描组件
组件被注解标识后需要通过Spring扫描才能够侦测到
1. 指定被扫描的包 **context:component-scan base-package=扫描位置**
`<context:component-scan base-package="com.lxy.ioc.userMod" use-default-filters="true">`
2. 详细说明
    1. base-package属性指定一个需要扫描的基类包，Spring容器将会扫描这个基类包以及报下所有的类
    2. 当需要扫描多个包时可以用逗号分隔
    3. 如果仅希望扫描特定的类而非基类包下的所有的类，可以使用resource-pattern属性过滤特定的类
    4. 包含于排除
       -  <context:include-filter>:在设定的包结构下，再次通过注解或类型具体包含到某个或某几个类
     注意：在使用包含时一定要将use-default-filters设置为默认的关闭(即扫描包下所有的类)
       -  <context:exclude-filter>:在设定的包结构下，再次通过注解或类型排除某个或某几个类
      注意：在使用排除时一定要将use-default-filters设置为默认的打开(即扫描包下所有的类）
        - use-default-filters默认的过滤方式
        - **切记 可以出现多个包含，或多个排除，但是包含和排除不嫩那个同时出现**

## 自动装配 @Autowired
**默认按照byType不行的话为byName**
1. 根据类实现自动装配
2. 构造器，普通字段(即使是非public)一切参数与方法都可以应用@Autowired注解
3. 默认情况下，所有使用@Autowired注解的属性都需要被设置，当Spring找不到匹配的bean装配属性时会抛出异常
4. 若某一属性允许不被设置，可以设置@Autowired注解的required属性为false
5. 默认情况下，当IOC容器里存在多个类型兼容的bean时，Spring会尝试匹配bean		  的id值是否与变量名相同，如果相同则进行装配。如果bean的id值不相同，通过类型的自动装配将无法工作。此时可以在@Qualifier注解里提供bean的名称。Spring	  甚至允许在方法的形参上标注@Qualifiter注解以指定注入bean的名称
6. @Autowired注解也可以应用在数组类型的属性上，此时Spring将会把所有匹配的bean进行自动装配
7. @Autowired注解也可以应用在集合属性上，此时Spring读取该集合的类型信息，然后自动装配所有与之兼容的bean
8. @Autowired注解用在java.util.Map上时，若该Map的键值为String，那么 Spring将自动装配与值类型兼容的bean作为值，并以bean的id值作为键

---

## AOP(面向切面编程)
### AOP的术语
1. 横切关注点 每个方法中抽取出来的同一类非核心业务
2. 切面(Aspect) 封装横切关注点信息的类，每个关注点体现为一个通知方法
3. 通知(Advice) 切面必须要完成的各个具体工作
4. 目标(Target) 被通知的对象
5. 代理(Proxy) 向目标对象应用通知之后创建的代理对象
6. 连接点(JointPoint) 横切关注点在程序代码中的具体体现，对应程序执行的某个特定位置
7. 切入点(Pointcut) 定位连接的方式 切入点可以通过org.springframework.aop.Pointcut 接口进行描述，它使用类和方法作为连接点的查询条件。

## ApectJ(不属于Spring)
#### 配置
````java
    <context:component-scan base-package="com.lxy.spring.aop"></context:component-scan>
    <!--开启aspectj的自动代理功能-->
        <aop:aspectj-autoproxy />
````

### 用AspectJ注解申明切面
1. 要在Spring中声明AspectJ切面，只需要在IOC容器中将切面声明为bean实例。
2. 当在Spring IOC容器中初始化AspectJ切面之后，Spring IOC容器就会为那些与 AspectJ切面相匹配的bean创建代理。
3. 在AspectJ注解中，切面只是一个带有@Aspect注解的Java类，它往往要包含很多通知。
4. 通知是标注有某种注解的简单的Java方法。
5. AspectJ支持5种类型的通知注解。
     1. @Before：前置通知，在方法执行之前执行
     2. @After：后置通知，在方法执行之后执行
     3. @AfterRunning：返回通知，在方法返回结果之后执行
     4. @AfterThrowing：异常通知，在方法抛出异常之后执行
     5. @Around：环绕通知，围绕着方法执行

#### 切入点表达式的语法细节
`execution([权限修饰符] [返回值类型] [简单类名/全类名] [方法名]([参数列表]))`

重用切入点表达式：
````java
    @Pointcut(value ="execution(* com.lxy.spring.aop.*.*(..))" )
    public void test() {

    }
````

## 通知
### 前置通知
1. 前置通知：在方法执行之前执行的通知
2. 使用@Before注解

### 后置通知
1. 后置通知：后置通知是在连接点完成之后执行的，即连接点返回结果或者抛出异常的时候(**相当于写在finally中**)
2. 使用@After注解
   
### 返回通知
1. 返回通知：无论连接点是正常返回还是抛出异常，后置通知都会执行。如果只想在连接点返回的时候记录日志，应使用返回通知代替后置通知。
2. 使用@AfterReturning注解,在返回通知中访问连接点的返回值
     1. 在返回通知中，只要将returning属性添加到@AfterReturning注解中，就可以访问连接点的返回值。该属性的值即为用来传入返回值的参数名称
     2. 必须在通知方法的签名中添加一个同名参数。在运行时Spring AOP会通过这个参数传递返回值
     3. 原始的切点表达式需要出现在pointcut属性中

### 异常通知
1. 异常通知：只在连接点抛出异常时才执行异常通知
2. 将throwing属性添加到@AfterThrowing注解中，也可以访问连接点抛出的异常。Throwable是所有错误和异常类的顶级父类，所以在异常通知方法可以捕获到任何错误和异常。
3. 如果只对某种特殊的异常类型感兴趣，可以将参数声明为其他异常的参数类型。然后通知就只在抛出这个类型及其子类的异常时才被执行

### 环绕通知
1. 环绕通知是所有通知类型中功能最为强大的，能够全面地控制连接点，甚至可以控制是否执行连接点。
2. 对于环绕通知来说，连接点的参数类型必须是ProceedingJoinPoint。它是 JoinPoint的子接口，允许控制何时执行，是否执行连接点。
3. 在环绕通知中需要明确调用ProceedingJoinPoint的proceed()方法来执行被代理的方法。如果忘记这样做就会导致通知被执行了，但目标方法没有被执行。
4. 注意：环绕通知的方法需要返回目标方法执行之后的结果，即调用 joinPoint.proceed();的返回值，否则会出现空指针异常。
   
### 用XML来配置切面
````java
    <aop:config>
        <aop:aspect ref="myLogger">
            <aop:pointcut id="cut" expression="execution(* com.lxy.aopxml.*.*(..))"/>
        <!--<aop:before method="before" pointcut="execution(* com.lxy.aopxml.*.*(..))"/>-->
        <aop:before method="before" pointcut-ref="cut"/>
        </aop:aspect>

    </aop:config>
````

## JdbcTemplate 
### 在Spring配置文件中配置相关的bean
1. 数据源对象
````java
 <context:property-placeholder location="conf/db.properties"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>

    </bean>
````
2. JdbcTemplate对象
3. ````java
 <!--配置jdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>


### 持久化操作
1. 增删改 JdbcTemplate.update(String, Object...)
2. 批量增删改 JdbcTemplate.batchUpdate(String, List<Object[]>)
		Object[]封装了SQL语句每一次执行时所需要的参数
		List集合封装了SQL语句多次执行时的所有参数

3. 查询单行 JdbcTemplate.queryForObject(String, RowMapper<Department>, Object...)
4. 查询多行 JdbcTemplate.query(String, RowMapper<Department>, Object...)
RowMapper对象依然可以使用BeanPropertyRowMapper

5. 查询单一值 JdbcTemplate.queryForObject(String, Class, Object...)

## 事务概述
1. 在JavaEE企业级开发的应用领域，为了保证数据的完整性和一致性，必须引入数据库事务的概念，所以事务管理是企业级应用程序开发中必不可少的技术。
2. 事务就是一组由于逻辑上紧密关联而合并成一个整体(工作单元)的多个数据库操作，这些操作要么都执行，要么都不执行。 
3. 事务的四个关键属性(ACID)
     1. 原子性(atomicity)：“原子”的本意是“不可再分”，事务的原子性表现为一个事务中涉及到的多个操作在逻辑上缺一不可。事务的原子性要求事务中的所有操作要么都执行，要么都不执行。 
     2. 一致性(consistency)：“一致”指的是数据的一致，具体是指：所有数据都处于满足业务规则的一致性状态。一致性原则要求：一个事务中不管涉及到多少个操作，都必须保证事务执行之前数据是正确的，事务执行之后数据仍然是正确的。如果一个事务在执行的过程中，其中某一个或某几个操作失败了，则必须将其他所有操作撤销，将数据恢复到事务执行之前的状态，这就是回滚。
     3. 隔离性(isolation)：在应用程序实际运行过程中，事务往往是并发执行的，所以很有可能有许多事务同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。隔离性原则要求多个事务在并发执行过程中不会互相干扰。
     4. 持久性(durability)：持久性原则要求事务执行完成后，对数据的修改永久的保存下来，不会因各种系统错误或其他意外情况而受到影响。通常情况下，事务对数据的修改应该被写入到持久化存储器中。

### spring事务的管理
#### 事务管理器的主要实现
1. 配置文件
   ````java 
    <!--配置事务管理器 不管使用注解的方式还是xml的方式配置事务，一定要有dataSourceTransactionManager的支持-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--配置事务通知-->
    <tx:advice id="tx" transaction-manager="dataSourceTransactionManager">
        <tx:attributes>
            <!--在设置好的切入点表达式下再次进行事务设置-->
            <tx:method name="buybook"/>
            <tx:method name="checkout"/>
            <!--只有select开头的方法才会被事务处理-->
            <tx:method name="select*" read-only="true"/>
            <tx:method name="insert*"/>
            <tx:method name="update*"/>
            <tx:method name="delete*"/>


            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <!--配置切入点表达式-->
    <aop:config>
        <aop:pointcut id="pointCut" expression="execution(* com.lxy.book_xml.service.Impl.*.*(..))"/>
        <aop:advisor advice-ref="tx" pointcut-ref="pointCut"></aop:advisor>
    </aop:config>
    ````

    @Transactional :对方法中所有的操作为一个事务进行管理
     * 在方法上使用只对方法有效果，在类上使用，对类中所有的方法都有效果
     * @Transactional中可以设置的属性：
     * propagation:A方法和B方法都有事务，当A在调用B时会将A中的事务传播给B方法，B方法对于事务的处理方式就是事务的传播行为
     * Propagation.REQUIRED:必须使用调用者的事务(默认)
     * Propagation.REQUIRES_NEW:将调用者的事务挂起 不使用调用者的事务，使用新的事务进行处理
     * isolation：
     *      读未提交 :脏读 1
     *      读已提交 :不可重复读 2
     *      可重复读 :幻读 4
     *      串行话 :单线程 性能低 消耗大  8
     *
     *
     * timeout:在事务强制回滚前，最多可以执行(等待)的时间
     *
     * readOnly:指定当前事务中的一系列的操作是否为只读
     * 若设置为只读，mysql就会在请求访问数据的时候不加锁，提高性能
     * 若设置为只读，不管事务中有没有写的操作，mysql都不会加锁提高性能
     * 但是如果有写操作的情况，建议一定不能设置为只读
     *
     * rollBackFor
     *
     * rollBackForClassName|noRollBackFor|noRollBackForClassName


