# Spring

1.Spring框架：

一般指的是Spring Framework，是多模块的集合，模块是核心容器，数据访问/集成，Web，AOP(面向切面编程)，工具，消息和测试模块。比如Core Container
中的core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。spring的6个特征是核心技术
(依赖注入DI，AOP，事件event，资源，i18n，验证，数据绑定，类型转换，SpEL)，测试(模拟对象，TestContext框架，Spring MVC测试，WebTestClient)，
数据访问(事务，DAO支持，JDBC，ORM，编组XML)，Web支持(Spring MVC和Spring WebFlux Web框架)，集成(远程处理，JMS，JCA，JMX，电子邮件，任务，
调度，缓存)，语言(Kotlin，Groovy，动态语言)。

2.Spring IOC和AOP：

IOC(Inverse of control，控制反转)是一种设计思想，就是将程序手动创建对象的控制权交给Spring框架来管理，IOC在其他语言中也有，IOC容器是Spring用来实现IOC的载体，
IOC容器实际上就是一个存放各种对象的Map。将对象之间的依赖关系交给IOC来管理，并且由IOC容器来完成对象的注入，可见简化开发的过程，IOC容器类似一个工厂，当要获取对象
时，只需要配置文件或注解即可，不用考虑对象是如何创建的，比如最常用的用@Service注解的实现类，调用时不需要直接new实现类，直接注入实现类的接口即可使用，这样就不需要
考虑具体的实现类，全部交给IOC容器即可。IOC在spring中具体的实现是DI，即依赖注入。IOC的初始化过程是读取XML文件形成resource资源，然后解析resource生成BeanDefinition
即各种Bean的信息，比如指向类，是否单例，是否懒加载，bean之间的依赖关系等等。最后将BeanDefinition注册到BeanFactory中，形成IOC容器。

AOP(Aspect-Oriented Programming，面向切面编程)是指将那些和业务无关，却为业务模块所共同调用的逻辑或责任(事务处理，日志管理，权限控制等)封装起来，减少代码的重复性，
降低模块间的耦合，有利于未来的拓展和维护。代理就是指设置中间代理对象，通过代理对象来访问原对象，从而在不改变原对象方法的情况下提供额外的功能操作，拓展原对象，
java中的代理实现有三种，静态代理，动态代理和cglib代理，静态代理需要原对象和代理对象实现同一个接口，代理对象在需要的拓展的方法上调用原对象的实现方法。比如
```
public interface Subject{
    void test();
}

public class Target implements Subject{

    @Override
    public void test(){
        System.out.println("target);
    }
}

public class Proxy implements Subject{
    
    private Subject subject;

    public Proxy(Subject subject){
        this.subject = subject
    }
    @Override
    public void test(){
        System.out.println("proxy);
        subject.test();
    }
}

public class client{
    public static void main(String[] args){
        Subject target = new Target();
        Proxy proxy = new Proxy(target);
        proxy.test();
    }
}
```
第二种是动态代理，使用的是JDK自带的API，动态代理不需要和原对象实现同一接口，但是原对象必须实现接口，静态代理在编译时会生成代理类的class文件，
动态代理是在运行时动态生成的，即在编译时没有class文件，而是在运行时才生成类字节码，并加载到JVM中。
```
public interface Subject{
    void test();
}

public class Target implements Subject{

    @Override
    public void test(){
        System.out.println("target);
    }
}

public class DynamicProxy implements InvocationHandler {

    private Object target;

    public DynamicProxy(Object target){
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("dynamicProxy");
        Object result = method.invoke(target, args);
        return result;
    }
}

public class client{
    public static void main(String[] args){
        Target target = new Target();
        DynamicProxy dynamicProxy = new DynamicProxy(target);
        ClassLoader classLoader = subject.getClass().getClassLoader();
        Subject subject1 = (Subject) Proxy.newProxyInstance(classLoader,new Class[]{Subject.class},dynamicProxy);
        subject1.test();
    }
}
```
第三种是cglib代理，需要引入额外的三方库。cglib代理的对象不需要实现任何接口，cglib主要是通过继承原对象生成子类来实现代理，所以不能代理有final修饰的对象。
Spring AOP是基于动态代理实现的，如果要代理的对象实现了接口则使用JDK的动态代理，没有则使用cglib代理。除了Spring AOP外，还可以使用AspectJ框架在实现AOP，
Spring AOP属于运行时增强，AspectJ属于编译时增强，Spring AOP是基于代理，AspectJ是基于字节码，Spring AOP已经集成AspectJ。

3.spring bean：

Bean的作用域有singleton，prototype，request，session，global-session。singleton是唯一的bean实例，spring默认都是单例bean，对于所有的bean请求，只要id和该bean定义
相匹配，则只会返回bean的同一实例。prototype是每次请求都会创建新的bean实例。request是每次http请求都会产生一个bean实例，仅在当前http request内有效。
session是每次http请求都会产生一个bean实例，仅在当前http session内有效。global-session是指全局session作用域，仅仅在基于portlet的web应用中才有意义，
Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，
每个 portlet 都有不同的会话。spring的singleton bean在多线程下对bean的非静态变量的写操作会出现线程安全问题，推荐将变量用ThreadLocal保存。

@Component和@Bean区别：@Component作用于类，@Bean作用于注解。@Component通常是通过类路径扫描来自动侦测以及自动装配到spring bean容器中，也可以使用@ComponentScan
来定义要扫描的类自动装配到spring bean容器中，@Bean只在方法中产生bean，@Bean告诉了Spring这是某个类的示例，当我需要用它的时候还给我。@Bean比@Component自定义更强，
当引用第三方库的类需要装配到spring bean容器中时，只能通过@Bean。@Component修饰的类能够自动装配到spring bean中，@Bean修饰的方法返回的对象将自动装配到spring bean中。
```
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
除了使用@Component来声明类自动装配到spring bean中外，还可以使用@Repository，@Service，@Controller。

bean的生命周期：Bean容器找到配置文件中spring bean的定义，Bean容器利用Java Reflection API创建bean的实例，如果涉及到属性值则利用set()方法设置属性值，如果bean
实现了BeanNameAware接口则调用setBeanName()方法设置bean的名字，如果bean实现了BeanClassLoaderAware接口，则调用setBeanClassLoader()方法传入ClassLoader对象
的实例，如果bean实现的其他Aware接口则调用对应接口的方法。如果有和加载这个bean的spring容器相关的BeanPostProcessor对象则执行postProcessBeforeInitialization()
方法，如果bean实现了InitializingBean接口则执行afterPropertiesSet()方法，如果bean在配置文件中包含init-method属性则执行指定方法，如果有和加载这个bean的spring
容器相关的BeanPostProcessor对象则执行postProcessAfterInitialization()方法，当要销毁bean时，如果bean实现了DisposableBean接口，执行destroy()方法，当要销毁bean
时如果Bean在配置文件中包含destroy-method属性则执行指定方法。大致就是先实例化bean，设置属性，检查bean是否有实现Aware相关接口，有则调用接口方法，然后检查spring容器是否有
BeanPostProcessor对象，有则执行postProcessBeforeInitialization()前置处理，然后检查bean是否实现InitializingBean接口，有则执行afterPropertiesSet()方法，然后
检查bean是否有配置自定义init-method方法，有则调用，然后检查spring容器是否有BeanPostProcessor对象，有则执行postProcessAfterInitialization()后置处理，然后注册
必要Destruction相关回调接口，在销毁bean时检查bean是否实现DisposableBean接口，有则调用执行destroy()方法，然后检查配置文件中包含destroy-method属性则执行指定方法。

![timewoo](https://timewoo.github.io/images/Bean.jpg)

![timewoo](https://timewoo.github.io/images/Bean2.jpg)

Spring循环依赖问题：

4.spring mvc：

mvc是一种设计模式，spring mvc是一款优秀的mvc框架，spring mvc一般将项目分成service层(处理业务)，Dao层(数据库操作)，Entity层(实体类)，Controller层(控制层，
返回数据给前端)。spring mvc主要的原理是用户发起一个请求，到达controller层，将请求交给模型层service,dao,entity处理，将结果返回controller，controller再判断
是直接返回用户还是进行视图渲染。

![timewoo](https://timewoo.github.io/images/SpringMVC.jpg)

spring mvc主要流程是用户发送请求，请求到达前端控制器(DispatcherServlet)，DispatcherServlet根据请求调用HandlerMapping，解析请求对应的Handler。解析到对应的
Handler，即Controller控制器后，开始由HandlerAdapter适配器处理。HandlerAdapter适配器根据Handler来调用真正的处理器来处理请求，并且处理相应的业务逻辑。处理器
处理完业务之后会返回一个ModelAndView对象，Mode是返回的数据对象，View是逻辑上的视图。ViewResolver根据逻辑View找到实际的View。DispatcherServlet将返回的Model
传给View进行渲染，最后将View返回给用户。

![timewoo](https://timewoo.github.io/images/SpringMVC1.jpg)

5.Spring框架中的设计模式：

工厂模式：通过BeanFactory，ApplicationContext创建bean对象。

代理模式：Spring AOP

单例模式：spring bean默认是单例

模板方法：JdbcTemplate，hibernateTemplate等即template结尾的对数据库操作的类使用模板方法

包装器设计模式：动态数据源

观察者模式：Spring事件驱动模型是观察者模式。

适配器模式：Spring AOP的增强获取通知使用适配器模式，Spring MVC也用到了适配器模式的Controller。

6.Spring 事务：

spring管理事务的有两种，编程式事务和声明式事务，编程式事务是在代码里硬编码，声明式事务在配置文件中配置。主要推荐使用声明式事务，声明式事务又分为基于XML和注解两种。

spring的事务隔离级别和sql的隔离级别类似，TransactionDefinition.ISOLATION_DEFAULT使用后端数据库默认的事务隔离级别，Mysql使用REPEATABLE_READ,Oracle使用
READ_COMMITTED。TransactionDefinition.ISOLATION_READ_UNCOMMITTED使用最低的隔离级别，读未提交，可能脏读，幻读或不可重复读。TransactionDefinition.ISOLATION_READ_COMMITTED
使用读取已提交，可能幻读，不可重复读。TransactionDefinition.ISOLATION_REPEATABLE_READ使用可重复读，可能幻读。TransactionDefinition.ISOLATION_SERIALIZABLE使用最高级别的
隔离，串行化，影响性能。

事务的传播行为有7种，TransactionDefinition.PROPAGATION_REQUIRED是当前存在事务则加入该事务，否则创建一个新事务，事务会一起回滚。TransactionDefinition.PROPAGATION_SUPPORTS是当前存在事务
则加入该事务，否则已非事务的方式运行。TransactionDefinition.PROPAGATION_MANDATORY是当前存在事务存在则加入该事务，否则抛出异常。TransactionDefinition.PROPAGATION_REQUIRES_NEW
是创建一个新事务，如果当前存在事务则挂起该事务。只会回滚当前事务。TransactionDefinition.PROPAGATION_NOT_SUPPORTED是以非事务的方式运行，如果当前事务存在则挂起该事务。
TransactionDefinition.PROPAGATION_NEVER是以非事务的方式运行，如果存在当前事务则抛出异常。TransactionDefinition.PROPAGATION_NESTED是指当前存在事务则创建一个新事务作为当前事务
的嵌套事务运行，否则就和TransactionDefinition.PROPAGATION_REQUIRED行为一样。即外部事务的回滚会影响内部事务，而内部事务的回滚不会影响外部事务。

spring事务管理主要是3个接口，PlatformTransactionManager，TransactionDefinition，TransactionStatus。PlatformTransactionManager是事务的管理器，spring事务策略的核心，
TransactionDefinition是事务定义的信息(事务隔离级别，传播行为，超时，只读，回滚规则)，TransactionStatus是事务的运行状态。PlatformTransactionManager接口是事务的上层
管理者，TransactionDefinition和TransactionStatus两个接口是事务的描述。PlatformTransactionManager根据TransactionDefinition中定义的事务属性，比如超时时间，隔离级别，
传播行为来进行事务管理，而TransactionStatus接口提供获取事务状态，比如是否是新事务，是否可以回滚等。

PlatformTransactionManager事务管理接口，Spring并不直接管理事务，而是提供多种事务管理器，通过这个接口，spring为各个平台都提供了对应的事务管理器，比如JDBC的DataSourceTransactionManager，
Hibernate的HibernateTransactionManager，JPA的JpaTransactionManager。

![timewoo](https://timewoo.github.io/images/transaction.jpg)

PlatformTransactionManager主要定义了三个方法
```
public interface PlatformTransactionManager {

	/**
	 * 获取事务
	 */
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
			throws TransactionException;

	/**
	 * 提交事务
	 */
	void commit(TransactionStatus status) throws TransactionException;

	/**
	 * 回滚事务
	 */
	void rollback(TransactionStatus status) throws TransactionException;

}
```

TransactionDefinition是事务属性，PlatformTransactionManager通过getTransaction(TransactionDefinition definition)来获取事务，TransactionDefinition
定义的就是事务的一些基本属性，一般是隔离级别，传播行为，回滚规则，是否只读，事务超时。
```
public interface TransactionDefinition {

	/**
	 * 是当前存在事务则加入该事务，否则创建一个新事务
	 */
	int PROPAGATION_REQUIRED = 0;

	/**
	 * 是当前存在事务
       则加入该事务，否则已非事务的方式运行
	 */
	int PROPAGATION_SUPPORTS = 1;

	/**
	 * 是当前存在事务存在则加入该事务，否则抛出异常
	 */
	int PROPAGATION_MANDATORY = 2;

	/**
	 * 是创建一个新事务，如果当前存在事务则挂起该事务
	 */
	int PROPAGATION_REQUIRES_NEW = 3;

	/**
	 * 是以非事务的方式运行，如果当前事务存在则挂起该事务
	 */
	int PROPAGATION_NOT_SUPPORTED = 4;

	/**
	 * 是以非事务的方式运行，如果存在当前事务则抛出异常
	 */
	int PROPAGATION_NEVER = 5;

	/**
	 * 是指当前存在事务则创建一个新事务作为当前事务
       的嵌套事务运行，否则就和TransactionDefinition.PROPAGATION_REQUIRED行为一样
	 */
	int PROPAGATION_NESTED = 6;


	/**
	 * 数据库默认事务隔离
	 */
	int ISOLATION_DEFAULT = -1;

	/**
	 * 最低级别隔离
	 */
	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;

	/**
	 * 读取已提交
	 */
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;

	/**
	 * 可重复读
	 */
	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;

	/**
	 * 串行化
	 */
	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;


	/**
	 * 默认超时
	 */
	int TIMEOUT_DEFAULT = -1;


	/**
	 * 返回事务传播行为
	 */
	int getPropagationBehavior();

	/**
	 * 返回事务隔离级别
	 */
	int getIsolationLevel();

	/**
	 * 返回事务超时时间，超时会自动回滚事务
	 */
	int getTimeout();

	/**
	 * 是否是只读事务
	 */
	boolean isReadOnly();

	/**
	 * 获取事务名称
	 */
	@Nullable
	String getName();

}
```
TransactionDefinition可以设置事务超时时间，超时未提交事务则会回滚事务。TransactionDefinition可以设置事务为只读属性，设置为只读事务后
事务内的只能进行查询操作，写入操作会抛出异常，同时该事务内的查询不会受其他事务的影响，即事务内的多次查询数据是一致的，所以如果事务内需要多次查询，
查询的结果需要保证一致性的话，可以开启只读事务。

TransactionStatus接口用来保存事务状态。
```
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

	/**
	 * 是否有保存点，主要是对嵌套事务的支持，保存操作位置，回滚时可以返回指定位置而不需要全部回滚
	 */
	boolean hasSavepoint();

	/**
	 * 
	 */
	@Override
	void flush();

}

public interface TransactionExecution {

	/**
	 * 是否是新事务
	 */
	boolean isNewTransaction();

	/**
	 * 设置为只回滚
	 */
	void setRollbackOnly();

	/**
	 * 是否是只回滚
	 */
	boolean isRollbackOnly();

	/**
	 * 是否已完成
	 */
	boolean isCompleted();

}
	/**
	 * 创建一个新的保存点
	 */
	Object createSavepoint() throws TransactionException;

	/**
	 * 回滚到指定保存点
	 */
	void rollbackToSavepoint(Object savepoint) throws TransactionException;

	/**
	 * 释放指定保存点
	 */
	void releaseSavepoint(Object savepoint) throws TransactionException;

}

public interface Flushable {

    /**
     * 刷新
     */
    void flush() throws IOException;
}

```
spring事务的传播级别是指多个事务嵌套时事务传播方式，TransactionDefinition.PROPAGATION_REQUIRED下内外事务时同一个，回滚会影响所有事务方法。
TransactionDefinition.PROPAGATION_REQUIRES_NEWS下内外事务独立，事务的回滚互不影响。TransactionDefinition.PROPAGATION_NESTED下内外事务
属于嵌套事务，外事务的回滚影响内事务，内事务的回滚不影响外事务。TransactionDefinition.PROPAGATION_SUPPORTS，TransactionDefinition.PROPAGATION_NOT_SUPPORTED，
TransactionDefinition.PROPAGATION_NEVER下的都不是事务方法，不会回滚。事务默认只有在RuntimeException时才会回滚，Error也会导致事务回滚，但是在检查型(Checked)异常
时不会回滚，可以通过rollbackFor来指定回滚的异常。事务主要是根据AOP来实现，生成事务方法的代理类。事务方法类实现了接口则采用jdk动态代理，没有的话采用CGLIB动态代理。
因此当和事务方法同一个类的其他方法调用事务方法时，事务会失效，默认的调用会调用类的方法而不是代理类的方法。







