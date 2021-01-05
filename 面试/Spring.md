# Spring

1.Spring框架：

一般指的是Spring Framework，是多模块的集合，模块是核心容器，数据访问/集成，Web，AOP(面向切面编程)，工具，消息和测试模块。比如Core Container
中的core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。spring的6个特征是核心技术
(依赖注入DI，AOP，事件event，资源，i18n，验证，数据绑定，类型转换，SpEL)，测试(模拟对象，TestContext框架，Spring MVC测试，WebTestClient)，
数据访问(事务，DAO支持，JDBC，ORM，编组XML)，Web支持(Spring MVC和Spring WebFlux Web框架)，集成(远程处理，JMS，JCA，JMX，电子邮件，任务，
调度，缓存)，语言(Kotlin，Groovy，动态语言)。

2.Spring IOC和AOP：

IOC(Inverse of control，控制反转)是一种设计思想，就是将程序手动创建对象的控制权交给Spring框架来管理，IOC在其他语言中也有，IOC容器是Spring用来实现IOC的载体，
IOC容器实际上就是一个存放各种对象的Map。将对象之间的依赖关系交给IOC来管理，并且由IOC容器来完成对象的注入，可以简化开发的过程，IOC容器类似一个工厂，当要获取对象
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

![timewoo](https://timewoo.github.io/images/Bean1.jpg)

BeanFactory和ApplicationContext：BeanFactory和ApplicationContext是Spring的两个IOC容器，BeanFactory是最基础的，而ApplicationContext是在
BeanFactory的基础上实现了其他功能。BeanFactory在加载Bean时使用的是Lazy Loading，在XML文件中配置了Bean，并且通过new BeanFactory(xml)加载xml配置时，
只有BeanFactory进行了初始化，xml中的Bean还未加载，只有通过BeanFactory.getBean(BeanName)时才会加载相应的Bean，所以BeanFactory加载Bean时是Lazy Loading，
只在需要时才加载Bean。ApplicationContext加载Bean时使用的是Eager Loading，ApplicationContext在加载XML文件时会将配置Bean同时进行加载。ApplicationContext
相比较BeanFactory来说是比较重的IOC容器，因为ApplicationContext会将XML配置的所有Bean进行加载，而BeanFactory可以指定加载部分Bean，在内存有限的环境中比较方便。
ApplicationContext在BeanFactory的基础上增加了一些其他功能，包括消息传递(i18n或国际化)，事件发布，基于注解的依赖注入和Spring AOP的集成等，同时ApplicationContext
支持几乎所有的Bean类型的作用域，但是BeanFactory只支持Singleton和Prototype两种类型的Bean，所以尽管BeanFactory可以指定加载Bean，对内存更友好，但是一般
还是选择ApplicationContext，能够使用更多的功能。在Bean的生命周期中，需要注册BeanFactoryPostProcessor和BeanPostProcessor，BeanFactoryPostProcessor
主要是管理beanFactory内的BeanDefinition(未实例化)数据，可以在为实例化前修改bean的属性。BeanPostProcessor主要是对实例化的bean进行初始化方法的前后进行自定义
的逻辑处理，postProcessBeforeInitialization和postProcessAfterInitialization。由于BeanFactory是手动加载Bean，所以无法自动注册BeanFactoryPostProcessor
和BeanPostProcessor，需要手动创建BeanFactoryPostProcessor和BeanPostProcessor来调用相关方法。而ApplicationContext在会自动注册BeanFactoryPostProcessor
和BeanPostProcessor，在加载bean时自动调用相关方法。同时在使用BeanFactory时，AOP和事务都不会生效。因此在一般使用ApplicationContext，只有在需要严格
控制内存的情况下才使用BeanFactory。

Spring循环依赖问题：Spring循环依赖是指Bean之间循环引用，多个bean之间互相持有对方，形成闭环。A依赖B，B依赖C，C依赖A等。
```
public A{
    private B b;
}
public B{
    private C c;
}
public C{
    private A a;
}
```
在Spring中Bean的注入有两种方式，构造器注入，属性注入，属性注入又分为单例和非单例，通过构造器注入的Bean在发生循环依赖时会抛出异常。Spring中通过构造器注入bean时，
会维护一个正在创建的Bean池(Set结构存储的bean的name，singletonsCurrentlyInCreation)，存储的都是正在创建而未创建完成的bean，bean创建完成后就会从bean池中清除，
当出现循环依赖时Spring会发现注入的bean出现在Bean池中，代表依赖的bean也在创建，就会抛出BeanCurrentlylnCreationException异常，可以通过在构造方法的bean参数前添加@Lazy注解，
指明bean是懒加载，这样在构造时注入的是bean的代理对象，只有在真正使用时才通过代理对象调用真正的bean对象。当采用属性注入bean时，比如@setter，由于spring中bean是先实例化，
再进行属性的设置，所以spring对属性注入的bean采用三级缓存的方式来存储bean对象用来解决循环依赖问题。
```
/**
 * 一级缓存，存储的是完成初始化的bean
 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

/**
 * 二级缓存，存储的是完成实例化但是还未初始化的bean
 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/**
 * 三级缓存，存储的是进入实例化阶段的单例bean工厂
 */
private final Map<String, ObjectFactory> singletonFactories = new HashMap<String, ObjectFactory>(16);

```
spring对属性注入的bean先去一级缓存中获取，一级缓存没有就去二级缓存，二级缓存也没有就去三级缓存，比如A，B互相依赖，A在实例化时就会放入三级缓存中，在注入B时会去进行
B的创建，B在实例化后也会放入三级缓存中，然后B设置A依赖时发现一级和二级缓存都没有，就去三级缓存中获取，就拿到了A已经实例化还未初始化的对象，然后将A从三级缓存放入二级缓存，
并且删除三级缓存的A对象，B初始化完成后就将B对象放入一级缓存中，然后A在一级缓存中获取B对象，初始化完成后将A对象放入一级缓存中。其实我们可以看到，如果只使用两个缓存其实
也是可以解决循环依赖问题(只缓存实例化和初始化的bean)，但是spring采用三级缓存目的是防止在AOP的情况下导致注入的bean不是单例，意思就是在AOP的情况下，A创建生成的应该是代理对象，
即注入的应该是代理对象，但是A的代理对象是在bean创建完成后通过后置处理器生成的，如果只有二级缓存，B在获取A时拿到的是原始对象，A完成初始化后才生成代理对象，就违反了spring的设计，
所以新增一个三级缓存，存储的是beanFactory，在获取三级缓存的对象时会调用后置处理器，这样拿到的就是代理对象。三级缓存主要是针对AOP下的bean注入。
非单例bean在每次请求都会生成新的bean，无法使用三级缓存，所以无法解决循环依赖问题。

spring注入的@Resource和@Autowired的区别:@Resource和@Autowired都可以用来注入bean，@Resource是javaEE自带的注解，可以设置根据bean的name或type去注入，
name是指bean的名字，默认是参数名，type是bean的class类型。默认是根据bean的name注入，@Resource首先会根据bean的name去IOC容器中获取bean，如果没有找到指定name
的bean，就会使用type去匹配，当匹配不到或者匹配到多个type的bean，需要注意@Resource的type匹配时会使用向下匹配，即会将type的子类或者实现都匹配出来，如果type是父类或者接口，
则会将子类或者实现的bean也匹配出来，会抛出异常。使用向下匹配是java多态，因为向下匹配可以在注入抽象类或接口时可以使用指定的子类或实现，如果根据name匹配到bean并且指定了type的类型，
则会判断匹配的bean的type是否和指定type匹配，如果匹配则注入的是bean的type，指定的type可以向下兼容，即bean的type为指定type的子类也可以匹配，否则将匹配到的bean的类型转化为
指定的type类型，如果不能转化就会抛出异常。@Resource在注入时如果只设置type则会根据参数名作为bean的name去匹配，如果匹配到则会将bean的type和指定的type进行转化，所以指定了type
还是会先根据name去匹配。@Autowired是spring提供的注解，根据bean的type注入。@Autowired首先会根据bean的type进行匹配，若没有找到则抛出异常，若找到多个则根据参数名作为bean的name去匹配，
如果没有匹配到则抛出异常。@Autowired的type匹配会向下兼容，即会将type的子类和实现都匹配出来。@Autowired和@Resource的注入匹配是相反的，由于@Autowried没有指定bean的name的参数，
所以单独的@Autowried只能使用参数名作为name匹配，通过@Qualifier可以使用自定义的name来匹配。在注入有继承的父类或实现的接口时，若参数名和父类以及子类(接口和实现)的bean的name不匹配
则会注入失败，因为@Autowired的type匹配向下兼容，会将父类和子类(接口和实现)都匹配，需要指定bean的name，否则会抛出异常。

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

6.Spring事务：

spring管理事务的有两种，编程式事务和声明式事务，编程式事务是在代码里硬编码，声明式事务在配置文件中配置。主要推荐使用声明式事务，声明式事务又分为基于XML和注解两种。

spring的事务隔离级别和sql的隔离级别类似，TransactionDefinition.ISOLATION_DEFAULT使用后端数据库默认的事务隔离级别，Mysql使用REPEATABLE_READ,Oracle使用
READ_COMMITTED。TransactionDefinition.ISOLATION_READ_UNCOMMITTED使用最低的隔离级别，读未提交，可能脏读，幻读或不可重复读。TransactionDefinition.ISOLATION_READ_COMMITTED
使用读取已提交，可能幻读，不可重复读。TransactionDefinition.ISOLATION_REPEATABLE_READ使用可重复读，可能幻读。TransactionDefinition.ISOLATION_SERIALIZABLE使用最高级别的
隔离，串行化，影响性能。

事务的传播行为有7种，TransactionDefinition.PROPAGATION_REQUIRED是当前存在事务则加入该事务，否则创建一个新事务，事务会一起回滚。TransactionDefinition.PROPAGATION_SUPPORTS是当前存在事务
则加入该事务，否则已非事务的方式运行。TransactionDefinition.PROPAGATION_MANDATORY是当前存在事务则加入该事务，否则抛出异常。TransactionDefinition.PROPAGATION_REQUIRES_NEW
是创建一个新事务，如果当前存在事务则挂起该事务。只会回滚当前事务。TransactionDefinition.PROPAGATION_NOT_SUPPORTED是以非事务的方式运行，如果当前事务存在则挂起该事务。
TransactionDefinition.PROPAGATION_NEVER是以非事务的方式运行，如果存在当前事务则抛出异常。TransactionDefinition.PROPAGATION_NESTED是指当前存在事务则创建一个新事务作为当前事务
的嵌套事务运行，否则就和TransactionDefinition.PROPAGATION_REQUIRED行为一样。即外部事务的回滚会影响内部事务，而内部事务的回滚不会影响外部事务。

spring事务管理主要是3个接口，PlatformTransactionManager，TransactionDefinition，TransactionStatus。PlatformTransactionManager是事务的管理器，spring事务策略的核心，
TransactionDefinition是事务定义的信息(事务隔离级别，传播行为，超时，只读，回滚规则)，TransactionStatus是事务的运行状态。PlatformTransactionManager接口是事务的上层
管理者，TransactionDefinition和TransactionStatus两个接口是事务的描述。PlatformTransactionManager根据TransactionDefinition中定义的事务属性，比如超时时间，隔离级别，
传播行为来进行事务管理，而TransactionStatus接口提供获取事务状态，比如是否是新事务，是否可以回滚等。

PlatformTransactionManager事务管理接口，Spring并不直接管理事务，而是提供多种事务管理器，通过这个接口，spring为各个平台都提供了对应的事务管理器，比如JDBC的DataSourceTransactionManager，
Hibernate的HibernateTransactionManager，JPA的JpaTransactionManager。

![timewoo](https://timewoo.github.io/images/transaction.png)

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
	 *是当前存在事务，则加入该事务，否则已非事务的方式运行
	 */
	int PROPAGATION_SUPPORTS = 1;

	/**
	 *是当前存在事务存在则加入该事务，否则抛出异常
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
	 * 是指当前存在事务则创建一个新事务作为当前事务的嵌套事务运行，否则就和TransactionDefinition.PROPAGATION_REQUIRED行为一样
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
public interface SavepointManager{
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

7.过滤器(Filter)和拦截器(Interceptor)：

过滤器是在Servlet规范中定义的，是由Servlet容器支持的，能够对Web资源(html，Servlet)和客户端之间的请求和响应进行过滤。

拦截器是在Spring容器内，是由Spring框架支持，是AOP的一种实现策略，能够在方法或字段前后进行拦截。拦截器可以通过IOC容器进行管理，同时可以对spring中
的各种资源，对象进行调用。比如数据源，事务管理，Service对象等。

过滤器主要是基于函数的回调，拦截器是基于Java反射机制；过滤器可以修改request，拦截器不能；过滤器需要在servlet容器中实现，拦截器可以适用于javaEE和
javaSE等各种环境；拦截器可以调用IOC容器中各种依赖，过滤器不能；过滤器只能在请求前后进行使用，拦截器可以详情到每个方法前后。

过滤器和拦截器的触发时机：

![timewoo](https://timewoo.github.io/images/Filter-Interceptor.jpg)

过滤器和拦截器的运行步骤：

![timewoo](https://timewoo.github.io/images/Filter-Interceptor1.jpg)

8.springboot自动装配：

springboot的自动装配是指springboot通过注解或者特定配置能够自动加载一整个模块的功能，不需要开发者做太多的配置，在springmvc中需要自己配置模块的xml文件，
springboot将繁琐的配置工作优化为注解方式，提升开发效率。

springboot的自动配置一般是在启动类上加入@SpringBootApplication注解
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```
可以看到@SpringBootApplication内部又引入了@SpringBootConfiguration，@EnableAutoConfiguration，@ComponentScan，由于@ComponentScan没有指定
包路径，默认扫描的是当前类同级的或者同级包下的所有类。
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {}
```
@SpringBootConfiguration内部又引入@Configuration，因此@SpringBootApplication等同于@Configuration，@EnableAutoConfiguration，@ComponentScan。

@EnableAutoConfiguration注解用来开启自动装配，Spring会在classpath下找到所有配置的bean根据Conditional(定制规则)进行装配和初始化。
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	String[] excludeName() default {};

}
```
@EnableAutoConfiguration注解中引入了@Import(AutoConfigurationImportSelector.class)，@Import注解是通过导入的方法将一个或多个bean或者
bean的配置类注入到Spring容器中，可以在Configuration类中引入其他配置类和普通类，可以自主的选择bean进行注入。@EnableAutoConfiguration内引入了
AutoConfigurationImportSelector.class。AutoConfigurationImportSelector实现DeferredImportSelector接口，DeferredImportSelector接口
实现ImportSelector接口，主要实现selectImports方法。selectImports方法主要是为了导入@Configuration的配置项，DeferredImportSelector接口是延期
导入，当所有的@Configuration都处理过之后才执行。
```
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

	private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

	private static final String[] NO_IMPORTS = {};

	private static final Log logger = LogFactory.getLog(AutoConfigurationImportSelector.class);

	private static final String PROPERTY_NAME_AUTOCONFIGURE_EXCLUDE = "spring.autoconfigure.exclude";

	private ConfigurableListableBeanFactory beanFactory;

	private Environment environment;

	private ClassLoader beanClassLoader;

	private ResourceLoader resourceLoader;
       
    /**
     * 返回Configuration配置类数组
     */
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // 判断是否有@EnableAutoConfiguration注解，没有则直接返回空数组
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
        // 将META-INF/spring-autoconfigure-metadata.properties的配置文件加载到PropertiesAutoConfigurationMetadata对象返回
        // 配置文件包含自动配置类的生效条件，用作后面自动配置类的过滤
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
        // 经过筛选返回需要导入的autoConfigurationEntry，autoConfigurationEntry包含需要导入的配置configurations和排除的配置exclusions
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
        // 返回需要导入的配置类数组
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}

	protected boolean isEnabled(AnnotationMetadata metadata) {
		if (getClass() == AutoConfigurationImportSelector.class) {
			return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
		}
		return true;
	}
    /**
     * 返回需要导入配置configurations和排除的配置exclusions
     */    
	protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
        // 判断是否有@EnableAutoConfiguration注解，没有则直接返回空对象
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
        // 获取使用@EnableAutoConfiguration注解的属性值Map
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
        // 获取META-INF/spring.factories下所有配置了@EnableAutoConfiguration注解的类名数组
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        // 删除重复类名
		configurations = removeDuplicates(configurations);
        // 获取不需要自动配置的类名
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        // 判断忽略的类名是否是自动配置的类，不是则抛出异常
		checkExcludedClasses(configurations, exclusions);
        // 类名数组删除忽略的自动配置类
		configurations.removeAll(exclusions);
        // 根据自动配置类@Conditional条件类过滤，只保留符合条件，能够自动配置的类
		configurations = filter(configurations, autoConfigurationMetadata);
        // 触发自动配置导入事件
		fireAutoConfigurationImportEvents(configurations, exclusions);
        // 返回自动配置的类和排除的配置类
		return new AutoConfigurationEntry(configurations, exclusions);
	}

	protected AnnotationAttributes getAttributes(AnnotationMetadata metadata) {
        // 获取EnableAutoConfiguration类名
		String name = getAnnotationClass().getName();
        // 获取配置的EnableAutoConfiguration的属性值Map，AnnotationAttributes继承LinkedHashMap<String, Object>
        // key是属性名(exclude,excludeName等)，value是属性值
		AnnotationAttributes attributes = AnnotationAttributes.fromMap(metadata.getAnnotationAttributes(name, true));
        // 判断AnnotationAttributes是否为空
		Assert.notNull(attributes, () -> "No auto-configuration attributes found. Is " + metadata.getClassName()
				+ " annotated with " + ClassUtils.getShortName(name) + "?");
		return attributes;
	}

	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        // 获取META-INF/spring.factories下配置了@EnableAutoConfiguration注解的所有配置类的类名数组
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
        // 判断数组是否为空
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

	protected final <T> List<T> removeDuplicates(List<T> list) {
		return new ArrayList<>(new LinkedHashSet<>(list));
	}

	protected Set<String> getExclusions(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        // 新建set数组
		Set<String> excluded = new LinkedHashSet<>();
        // 获取注解配置的exclude忽略类名
		excluded.addAll(asList(attributes, "exclude"));
        // 获取注解配的excludeName忽略类名
		excluded.addAll(Arrays.asList(attributes.getStringArray("excludeName")));
        // 获取配置文件中配置的spring.autoconfigure.exclude忽略类名
		excluded.addAll(getExcludeAutoConfigurationsProperty());
		return excluded;
	}

	private void checkExcludedClasses(List<String> configurations, Set<String> exclusions) {
		List<String> invalidExcludes = new ArrayList<>(exclusions.size());
        // 遍历忽略类
		for (String exclusion : exclusions) {
            // 判断忽略的类是否是自动配置的类
			if (ClassUtils.isPresent(exclusion, getClass().getClassLoader()) && !configurations.contains(exclusion)) {
				invalidExcludes.add(exclusion);
			}
		}
        // 存在忽略类不是自动配置类则抛出异常
		if (!invalidExcludes.isEmpty()) {
			handleInvalidExcludes(invalidExcludes);
		}
	}

	protected void handleInvalidExcludes(List<String> invalidExcludes) {
		StringBuilder message = new StringBuilder();
		for (String exclude : invalidExcludes) {
			message.append("\t- ").append(exclude).append(String.format("%n"));
		}
        // 抛出异常
		throw new IllegalStateException(String.format(
				"The following classes could not be excluded because they are" + " not auto-configuration classes:%n%s",
				message));
	}

	private List<String> filter(List<String> configurations, AutoConfigurationMetadata autoConfigurationMetadata) {
		long startTime = System.nanoTime();
		String[] candidates = StringUtils.toStringArray(configurations);
		boolean[] skip = new boolean[candidates.length];
		boolean skipped = false;
		for (AutoConfigurationImportFilter filter : getAutoConfigurationImportFilters()) {
			invokeAwareMethods(filter);
			boolean[] match = filter.match(candidates, autoConfigurationMetadata);
			for (int i = 0; i < match.length; i++) {
				if (!match[i]) {
					skip[i] = true;
					candidates[i] = null;
					skipped = true;
				}
			}
		}
		if (!skipped) {
			return configurations;
		}
		List<String> result = new ArrayList<>(candidates.length);
		for (int i = 0; i < candidates.length; i++) {
			if (!skip[i]) {
				result.add(candidates[i]);
			}
		}
		if (logger.isTraceEnabled()) {
			int numberFiltered = configurations.size() - result.size();
			logger.trace("Filtered " + numberFiltered + " auto configuration class in "
					+ TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime) + " ms");
		}
		return new ArrayList<>(result);
	}

}
```
META-INF/spring-autoconfigure-metadata.properties和META-INF/spring.factories在springboot autoconfigure包路径下

![timewoo](https://timewoo.github.io/images/configuration.png)

springboot自动装配首先是springboot的启动类上添加@SpringBootApplication注解，@SpringBootApplication注解内部包含@SpringBootConfiguration，
@EnableAutoConfiguration，@ComponentScan，@ComponentScan注解默认扫描的是当前类同级的或者同级包下的所有类。@EnableAutoConfiguration注解内部
引入@Import(AutoConfigurationImportSelector.class)，AutoConfigurationImportSelector的selectImports方法会导入所有符合条件的自动配置类，
selectImports方法首先会判断是否有@EnableAutoConfiguration注解，然后会将springboot autoconfigure包路径下META-INF/spring-autoconfigure-metadata.properties
配置文件加载到PropertiesAutoConfigurationMetadata对象中，PropertiesAutoConfigurationMetadata对象主要加载的是自动配置类加载所需要的条件，
然后会获取@EnableAutoConfiguration注解内部设置的属性值AnnotationAttributes，然后通过SpringFactoriesLoader.loadFactoryNames()方法将
springboot autoconfigure包路径下META-INF/spring.factories内配置了@EnableAutoConfiguration注解的所有配置类的类名生成类名数组configurations，
然后将AnnotationAttributes属性值中设置的exclude忽略自动配置类生成Set类型exclusions，然后将configurations内包含的exclusions删除，同时根据
PropertiesAutoConfigurationMetadata对象的条件来过滤符合加载条件的configurations，然后触发自动配置导入事件，最后返回可以导入的自动配置类。











