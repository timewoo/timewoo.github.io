# Spring

1.Spring框架：

一般指的是spring Framework，是多模块的集合，模块是核心容器，数据访问/集成，Web，AOP(面向切面编程)，工具，消息和测试模块。比如Core Container
中的core组件时Spring所有组件的核心，Beans组件和Context组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。spring的6个特征是核心技术
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
Spring AOP是基于动态代理实现的，如果要代理的对象实现了接口则使用JDK的动态代理，没有则使用cglib代理。除了spring AOP外，还可以使用AspectJ框架在实现AOP，
Spring AOP属于运行时增强，AspectJ属于编译时增强，Spring AOP是基于代理，AspectJ是基于字节码，Spring AOP已经集成AspectJ。
