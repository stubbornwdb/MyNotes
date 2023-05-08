## 一. Spring框架

#### 0.Spring的启动过程

Spring的启动过程主要可以分为两部分：

- 第一步：解析成BeanDefinition：将bean定义信息解析为BeanDefinition类，不管bean信息是定义在xml中，还是通过@Bean注解标注，都能通过不同的BeanDefinitionReader转为BeanDefinition类。
  - 这里分两种BeanDefinition，RootBeanDefintion和BeanDefinition。RootBeanDefinition这种是系统级别的，是启动Spring必须加载的6个Bean。BeanDefinition是我们定义的Bean。
- 第二步：参照BeanDefintion定义的类信息，通过BeanFactory生成bean实例存放在缓存中。
  - 这里的BeanFactoryPostProcessor是一个拦截器，在BeanDefinition实例化后，BeanFactory生成该Bean之前，可以对BeanDefinition进行修改。
  - BeanFactory根据BeanDefinition定义使用反射实例化Bean，实例化和初始化Bean的过程中就涉及到Bean的生命周期了，典型的问题就是Bean的循环依赖。接着，Bean实例化前会判断该Bean是否需要增强，并决定使用哪种代理来生成Bean。

#### 1.什么是Spring

Spring是一个开源的Java EE开发框架。Spring框架的核心功能可以应用在任何Java应用程序中，但对Java EE平台上的Web应用程序有更好的扩展性。Spring框架的目标是使得Java EE应用程序的开发更加简捷，通过使用POJO为基础的编程模型促进良好的编程风格。

#### 2.Spring有哪些优点
- **轻量级**：Spring在大小和透明性方面绝对属于轻量级的，基础版本的Spring框架大约只有2MB。
- **控制反转(IOC)**：Spring使用控制反转技术实现了松耦合。依赖被注入到对象，而不是创建或寻找依赖对象。
- **面向切面编程(AOP)**： Spring支持面向切面编程，同时把应用的**业务逻辑与系统的服务分离开来**。
- **容器**：Spring包含并管理应用程序对象的配置及生命周期。
- **MVC框架**：Spring的web框架是一个设计优良的web MVC框架，很好的取代了一些web框架。
- **事务管理**：Spring对下至本地业务上至全局业务(JAT)提供了统一的事务管理接口。
- **异常处理**：Spring提供一个方便的API将特定技术的异常(由JDBC, Hibernate, 或JDO抛出)转化为一致的、Unchecked异常。

#### 3.Spring框架有哪些模块
Spring框架至今已集成了20多个模块。这些模块主要被分如下图所示的核心容器、数据访问/集成、Web、AOP（面向切面编程）、测试模块。

- **核心容器模块**：是spring中最核心的模块。负责Bean的创建、配置和管理。主要包括：beans,core,context,expression等模块。
- **Spring的AOP模块**：主要负责对面向切面编程的支持，帮助应用对象解耦。
- **数据访问和集成模块**：包括JDBC，ORM，OXM，JMS和事务处理模块，其细节如下：
    - JDBC模块提供了不再需要冗长的JDBC编码相关了JDBC的抽象层。
    - ORM（Object Relational Mapping，对象关系映射）模块提供的集成层。流行的对象关系映射API，包括JPA，JDO，Hibernate和iBatis。
    - OXM模块提供了一个支持对象/ XML映射实现对JAXB，Castor，使用XMLBeans，JiBX和XStream 的抽象层。
    - Java消息服务JMS模块包含的功能为生产和消费的信息。
    - 事务模块支持编程和声明式事务管理实现特殊接口类，并为所有的POJO。
- **Web和远程调用**：包括web,servlet,struts,portlet模块。
- **测试模块**：test

#### 4.什么是控制反转(IOC)？什么是依赖注入？



传统模式中对象的调用者需要创建被调用对象，两个对象过于耦合，不利于变化和拓展．在spring中，直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理，从而实现对象之间的松耦合。**所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器**。

**依赖注入**：对象无需自行创建或管理它们的依赖关系，IoC容器在运行期间，动态地将某种依赖关系注入到对象之中。依赖注入能让相互协作的软件组件保持松散耦合。



控制反转(IoC，Inversion of Control)，也被称为依赖注入(DI，Dependency Injection). 指定义对象的过程中，它所依赖的对象只能通过构造器参数、工厂方法参数进行设置，或者构造器构造好的实例、工厂方法返回的实例通过set属性的方法进行设置。 IoC容器在创建bean的时候将这些依赖注入进来。 这个过程从根本上讲，就是bean自身不通过类直接构造依赖项的实例。（不控制依赖项的实例化、定位等） 控制反转是面向对象编程的一种设计原则，它可以用来降低代码之间的耦合度，而依赖注入是它最常见的一种方式，还有一种叫做依赖查找（Dependency Lookup） 使用DI原则后，代码更加清晰，当提供对象的依赖关系时，解耦会更加有效。对象不查找它的依赖项，也不知道依赖项的位置或类。因此，类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这些依赖关系允许在单元测试中使用存根或模拟实现。 IoC是目标，DI是实现方式。 

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中众多对象中的一个。bean及其之间的依赖关系反映在容器使用的配置元数据中



**SpringIoC的实现思路** 

Spring实现IOC的思路是提供一些配置信息用来描述类之间的依赖关系，然后由容器去解析这些配置信息，继而维护好对象之间的依赖关系，但前提是对象之间的依赖关系必须在类中定义好，比如A.class中有一个B.class的属性，那么我们可以理解为A依赖了B。这个依赖关系仅仅表明了A需要用到B，使用配置信息就是用于将A、B的依赖信息告诉容器，由容器去管理依赖信息，在需要的时候容器将依赖注入给这些对象，

**Spring实现IOC的思路大致可以拆分成3点**： 

应用程序中提供类，

提供依赖关系（属性或者构造方法） 把需要交给容器管理的对象通过配置信息告诉容器（xml、annotation，javaconfig） 

把各个类之间的依赖关系通过配置信息告诉容器

配置这些信息的方法有三种分别是xml，annotation和javaconfig维护的过程称为自动注入，自动注入的方法有两种构造方法和setter自动注入的值可以是对象、数组、map、list和常量比如字符串等。



#### 5.Spring有几种配置方式？

将Spring配置到应用开发中有以下三种方式：

 - **基于XML的配置**:
 - **基于注解的配置**： Spring在2.5版本以后开始支持用注解的方式来配置依赖注入。可以用注解的方式来替代XML方式的bean描述，可以将bean描述转移到组件类的内部，只需要在相关类上、方法上或者字段声明上使用注解即可。注解注入将会被容器在XML注入之前被处理，所以后者会覆盖掉前者对于同一个属性的处理结果
 - **基于Java的配置**： Spring对Java配置的支持是由@Configuration注解和@Bean注解来实现的。由@Bean注解的方法将会实例化、配置和初始化一个新对象，这个对象将由Spring的IoC容器来管理。@Bean声明所起到的作用与<bean/> 元素类似。被@Configuration所注解的类则表示这个类的主要目的是作为bean定义的资源。被@Configuration声明的类可以通过在同一个类的内部调用@bean方法来设置嵌入bean的依赖关系。

#### 6.BeanFactory和ApplicationContext有什么区别？

- Bean工厂(BeanFactory)是Spring框架最核心的接口，提供了高级IOC的配置机制．
- 应用上下文(ApplicationContext)建立在BeanFacotry基础之上，提供了更多面向应用的功能，如果国际化，属性编辑器，事件等等．
- beanFactory是spring框架的基础设施，是面向spring本身。ApplicationContext是面向使用Spring框架的开发者，几乎所有场合都会用到ApplicationContext.

#### 7.请解释自动装配模式的区别

- **no**：这是Spring框架的默认设置，在该设置下自动装配是关闭的，开发者需要自行在bean定义中用标签明确的设置依赖关系。

- **byName**：该选项可以根据bean名称设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的名称自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- **byType**：该选项可以根据bean类型设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的类型自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- **constructor**：造器的自动装配和byType模式类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。

- **autodetect**：该模式自动探测**使用构造器自动装配或者byType自动装配**。首先，首先会尝试找合适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在bean内部没有找到相应的构造器或者是无参构造器，容器就会自动选择byTpe的自动装配方式。

#### 8.Spring 框架中都用到了哪些设计模式

[Spring框架中的设计模式](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring-Design-Patterns)

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

#### 9.AOP相关概念

- **方面（Aspect）**：一个**关注点**的模块化，这个关注点实现可能横切多个对象。事务管理是J2EE应用中一个很好的横切关注点例子。方面用Spring的 Advisor或拦截器实现。

- **连接点（Joinpoint）**: 程序执行过程中明确的点，如方法的调用或特定的异常被抛出。可以理解什么时机执行aop的代码。

- **通知（Advice）:** 在特定的连接点，AOP框架执行的**动作**(怎么执行)。各种类型的通知包括“around”、“before”和“throws”等通知。通知类型将在下面讨论。许多AOP框架包括Spring都是以拦截器做通知模型，维护一个“围绕”连接点的拦截器链。Spring中定义了5个advice:
  -  Interception Around：JointPoint前后调用
  -  Before：JointPoint前调用
  -  After Returning：JointPoint后调用
  -  Throw：JoinPoint抛出异常时调用
  -  Introduction：JointPoint调用完毕后调用

- **切入点（Pointcut）**: 一系列连接点的集合。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解， MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上

- **引入（Introduction）**: 添加方法或字段到被通知的类。 Spring允许引入新的接口到任何被通知的对象。例如，你可以使用一个引入使任何对象实现 IsModified接口，来简化缓存。Spring中要使用Introduction, 可有通过DelegatingIntroductionInterceptor来实现通知，通过DefaultIntroductionAdvisor来配置Advice和代理类要实现的接口

- **目标对象（Target Object）**: 包含连接点的对象。也被称作被通知或被代理对象。POJO

- **AOP代理（AOP Proxy）**: AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。

- **织入（Weaving）**: 组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

#### 10.AOP（Aspect-Oriented Programming）是怎么实现的

实现AOP的技术，主要分为两大类：

 - 一是采用**动态代理技术**，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；
 - 二是采用**静态织入的方式**，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。

Spring AOP 的实现原理其实很简单：AOP 框架负责动态地生成 AOP 代理类，这个代理类的方法则由 Advice和回调目标对象的方法所组成, 并将该对象可作为目标对象使用。AOP 代理包含了目标对象的全部方法，但AOP代理中的方法与目标对象的方法存在差异，AOP方法在特定切入点添加了**增强处理，并回调了目标对象的方法**。

Spring AOP使用动态代理技术在运行期织入增强代码。使用两种代理机制：**基于JDK的动态代理**（JDK本身只提供接口的代理）和**基于CGlib的动态代理**。

- **(1) JDK的动态代理**

JDK的动态代理主要涉及java.lang.reflect包中的两个类：**Proxy和InvocationHandler**。其中InvocationHandler只是一个接口，可以通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态的将横切逻辑与业务逻辑织在一起。而Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。

其代理对象**必须是某个接口的实现**, 它是通过在运行期间创建一个接口的实现类来完成对目标对象的代理.只能实现接口的类生成代理,而**不能针对类**

- **(2)CGLib**

CGLib采用**底层的字节码技术**，**为一个类创建子类**，并在子类中采用方法拦截的技术拦截所有父类的调用方法，并顺势织入横切逻辑.它运行期间生成的代理对象是目标类的扩展子类.**所以无法通知final、private的方法**,因为它们不能被覆写.是针对类实现代理,主要是为指定的类生成一个子类,覆盖其中方法.

在spring中默认情况下使用JDK动态代理实现AOP,如果proxy-target-class设置为true或者使用了优化策略那么会使用CGLIB来创建动态代理.Spring　AOP在这两种方式的实现上基本一样．以JDK代理为例，会使用JdkDynamicAopProxy来创建代理，在invoke()方法首先需要织入到当前类的增强器封装到拦截器链中，然后递归的调用这些拦截器完成功能的织入．最终返回代理对象．

http://zhengjianglong.cn/2015/12/12/Spring/spring-source-aop/

#### 11.介绍spring的IOC实现

Spring　**IOC主要负责创建和管理bean及bean之间的依赖关系**．Spring　IOC的可分为:IOC容器的初始化和bean的加载．

在IOC容器初始化阶段主要是完成资源的加载(如定义bean的xml文件)，bean的解析及对解析后得到的beanDefinition的进行注册．以xmlBeanFactory为例，XmlBeanFactory继承了DefaultListableBeanFactory，XmlBeanFactory将读取xml配置文件，解析bean和注册解析后的beanDefinition工作交给XmlBeanDefinitionReader(是BeanDefinitionReader接口的一个个性化实现)来执行.

- 1) spring中定义了一套资源类，将文件，class等都看做资源．所以首先是将xml文件转化为资源然后用EncodeResouce来封装，该功能主要时考虑Resource可能存在编码要求的情况，如UTF-8等．

- 2) 然后根据xml文件判断xml的约束模式，是DTD还是Schema,以及寻找模式文档(验证文件)的方法(EntityResolver，这部分采用了代理模式和策略模式)．

完成了前面所有的准备工作以后就可以正式的加载配置文件，获取Document和解析注册BeanDefinition．Document的获取以及BeanDefinition的解析注册并不是由**XmlBeanDefinitionReader**完成，XmlBeanDefinitionReader只是将前面的工作完成以后文档加载交给**DefaultDocumentLoader**类来完成．

而解析交给了**DefaultBeanDefinitionDocumentReader**来处理.bean标签可以分为两种，一种是spring自带的默认标签，另一种就是用户自定义的标签．所以spring针对这两种情况，提供了不同的解析方式. 每种bean的解析完成后都会先注册到容器中然后最后发出响应事件，通知相关的监听器这个bean已经注册完成了．

#### 12.IoC注入的方式

- setter方法注入
- 构造器注入

```
<!--普通构造器注入-->
<bean id="helloAction" class="org.yoo.action.ConstructorHelloAction">
<!--type 必须为java.lang.String 因为是按类型匹配的，不是按顺序匹配-->
    
	<constructor-arg type="java.lang.String" value="yoo"/>
	<!-- 也可以使用index来匹配-->
	<!--<constructor-arg index="1" value="42"/>-->
	<constructor-arg><ref bean="helloService"/></constructor-arg>
</bean>
```

- 静态工厂注入  factory-method参数
- 实例工厂

http://blessht.iteye.com/blog/1162131

#### 13.你推荐哪种依赖注入？构造器依赖注入还是Setter方法依赖注入？

你可以同时使用两种方式的依赖注入，最好的选择是使用构造器参数实现强制依赖注入，使用setter方法实现可选的依赖关系。

#### 14.Spring中BeanFactory与FactoryBean的区别

> 在Spring中有BeanFactory和FactoryBean这2个接口，从名字来看很相似，比较容易搞混。

**BeanFactory**

`BeanFactory`是一个接口，它是Spring中工厂的顶层规范，是SpringIoc容器的核心接口，它定义了`getBean()`、`containsBean()`等管理Bean的通用方法。Spring的容器都是它的具体实现如：

- DefaultListableBeanFactory
- XmlBeanFactory
- ApplicationContext

这些实现类又从不同的维度分别有不同的扩展。

**源码：**

```
public interface BeanFactory {

	//对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
	//如果需要得到工厂本身，需要转义
	String FACTORY_BEAN_PREFIX = "&";

	//根据bean的名字，获取在IOC容器中得到bean实例
	Object getBean(String name) throws BeansException;

	//根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。
	<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	//提供对bean的检索，看看是否在IOC容器有这个名字的bean
	boolean containsBean(String name);

	//根据bean名字得到bean实例，并同时判断这个bean是不是单例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	//得到bean实例的Class类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	//得到bean的别名，如果根据别名检索，那么其原名也会被检索出来
	String[] getAliases(String name);
}
复制代码
```

**使用场景：**

- 从Ioc容器中获取Bean(byName or byType)
- 检索Ioc容器中是否包含指定的Bean
- 判断Bean是否为单例

**FactoryBean**

首先它是一个Bean，但又不仅仅是一个Bean。它是一个能生产或修饰对象生成的工厂Bean，类似于设计模式中的工厂模式和装饰器模式。它能在需要的时候生产一个对象，且不仅仅限于它自身，它能返回任何Bean的实例。

**源码**

```
public interface FactoryBean<T> {

	//从工厂中获取bean
	@Nullable
	T getObject() throws Exception;

	//获取Bean工厂创建的对象的类型
	@Nullable
	Class<?> getObjectType();

	//Bean工厂创建的对象是否是单例模式
	default boolean isSingleton() {
		return true;
	}
}
复制代码
```

从它定义的接口可以看出，`FactoryBean`表现的是一个工厂的职责。 **即一个Bean A如果实现了FactoryBean接口，那么A就变成了一个工厂，根据A的名称获取到的实际上是工厂调用`getObject()`返回的对象，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'`&`'符号。**

- getObject('name')返回工厂中的实例
- getObject('&name')返回工厂本身的实例

通常情况下，bean 无须自己实现工厂模式，Spring 容器担任了工厂的 角色；但少数情况下，容器中的 bean 本身就是工厂，作用是产生其他 bean 实例。由工厂 bean 产生的其他 bean 实例，不再由 Spring 容器产生，因此与普通 bean 的配置不同，不再需要提供 class 元素。

**示例**

先定义一个Bean实现FactoryBean接口

```
@Component
public class MyBean implements FactoryBean {
    private String message;
    public MyBean() {
        this.message = "通过构造方法初始化实例";
    }
    @Override
    public Object getObject() throws Exception {
        // 这里并不一定要返回MyBean自身的实例，可以是其他任何对象的实例。
        //如return new Student()...
        return new MyBean("通过FactoryBean.getObject()创建实例");
    }
    @Override
    public Class<?> getObjectType() {
        return MyBean.class;
    }
    public String getMessage() {
        return message;
    }
}
复制代码
```

MyBean实现了FactoryBean接口的两个方法，getObject()是可以返回任何对象的实例的，这里测试就返回MyBean自身实例，且返回前给message字段赋值。同时在构造方法中也为message赋值。然后测试代码中先通过名称获取Bean实例，打印message的内容，再通过`&+名称`获取实例并打印message内容。

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = TestApplication.class)
public class FactoryBeanTest {
    @Autowired
    private ApplicationContext context;
    @Test
    public void test() {
        MyBean myBean1 = (MyBean) context.getBean("myBean");
        System.out.println("myBean1 = " + myBean1.getMessage());
        MyBean myBean2 = (MyBean) context.getBean("&myBean");
        System.out.println("myBean2 = " + myBean2.getMessage());
        System.out.println("myBean1.equals(myBean2) = " + myBean1.equals(myBean2));
    }
}
复制代码
myBean1 = 通过FactoryBean.getObject()初始化实例
myBean2 = 通过构造方法初始化实例
myBean1.equals(myBean2) = false
复制代码
```

**使用场景**

说了这么多，为什么要有`FactoryBean`这个东西呢，有什么具体的作用吗？
FactoryBean在Spring中最为典型的一个应用就是用来**创建AOP的代理对象**。

我们知道AOP实际上是Spring在运行时创建了一个代理对象，也就是说这个对象，是我们在运行时创建的，而不是一开始就定义好的，这很符合工厂方法模式。更形象地说，AOP代理对象通过Java的反射机制，在运行时创建了一个代理对象，在代理对象的目标方法中根据业务要求织入了相应的方法。这个对象在Spring中就是——`ProxyFactoryBean`。

所以，FactoryBean为我们实例化Bean提供了一个更为灵活的方式，我们可以通过FactoryBean创建出更为复杂的Bean实例。

**区别**

- 他们两个都是个工厂，但`FactoryBean`本质上还是一个Bean，也归`BeanFactory`管理
- `BeanFactory`是Spring容器的顶层接口，`FactoryBean`更类似于用户自定义的工厂接口。

#### 15.解释核心容器(应用上下文)模块

这是Spring的基本模块，它提供了Spring框架的基本功能。BeanFactory 是所有Spring应用的核心。Spring框架是建立在这个模块之上的，这也使得Spring成为一个容器。

#### 16.BeanFactory – BeanFactory 实例

BeanFactory是工厂模式的一种实现，它使用控制反转将应用的配置和依赖与实际的应用代码分离开来。最常用的BeanFactory实现是XmlBeanFactory类。

#### 17.XmlBeanFactory

最常用的就是org.springframework.beans.factory.xml.XmlBeanFactory，它根据XML文件中定义的内容加载beans。该容器从XML文件中读取配置元数据，并用它来创建一个完备的系统或应用。

#### 18.解释AOP模块

AOP模块用来开发Spring应用程序中具有切面性质的部分。该模块的大部分服务由AOP Aliance提供，这就保证了Spring框架和其他AOP框架之间的互操作性。另外，该模块将元数据编程引入到了Spring。

#### 19.解释抽象JDBC和DAO模块

通过使用抽象JDBC和DAO模块保证了与数据库连接代码的整洁与简单，同时避免了由于未能关闭数据库资源引起的问题。它在多种数据库服务器的错误信息之上提供了一个很重要的异常层。它还利用Spring的AOP模块为Spring应用程序中的对象提供事务管理服务。

#### 20.解释对象/关系映射集成模块

Spring通过提供ORM模块在JDBC的基础上支持对象关系映射工具。这样的支持使得Spring可以集成主流的ORM框架，包括Hibernate, JDO, 及iBATIS SQL Maps。Spring的事务管理可以同时支持以上某种框架和JDBC。

#### 21.解释web模块

Spring的web模块建立在应用上下文(application context)模块之上，提供了一个适合基于web应用程序的上下文环境。该模块还支持了几个面向web的任务，如透明的处理多文件上传请求及将请求参数同业务对象绑定起来。

#### 22.解释Spring MVC模块

Spring提供MVC框架构建web应用程序。Spring可以很轻松的同其他MVC框架结合，但Spring的MVC是个更好的选择，因为它通过控制反转将控制逻辑和业务对象完全分离开来。

#### 23.Spring IoC容器

Spring IOC负责创建对象、管理对象(通过依赖注入)、整合对象、配置对象以及管理这些对象的生命周期。

#### 24.IOC有什么优点？

IOC或依赖注入减少了应用程序的代码量。它使得应用程序的**测试很简单**，因为在单元测试中不再需要单例或JNDI查找机制。简单的实现以及较少的干扰机制使得**松耦合**得以实现。IOC容器支持勤性单例及延迟加载服务。

#### 25.应用上下文是如何实现的？

ClassPathXmlApplicationContext 容器加载XML文件中beans的定义。XML Bean配置文件的完整路径必须传递给构造器。

FileSystemXmlApplicationContext 容器也加载XML文件中beans的定义。注意，你需要正确的设置CLASSPATH，因为该容器会在CLASSPATH中查看bean的XML配置文件。

WebXmlApplicationContext：该容器加载xml文件，这些文件定义了web应用中所有的beans。

#### 26.有哪些不同类型的IOC(依赖注入)

**接口注入**:接口注入的意思是通过接口来实现信息的注入，而其它的类要实现该接口时，就可以实现了注入

构造器依赖注入：构造器依赖注入在容器触发构造器的时候完成，该构造器有一系列的参数，每个参数代表注入的对象。

Setter方法依赖注入：首先容器会触发一个无参构造函数或无参静态工厂方法实例化对象，之后容器调用bean中的setter方法完成Setter方法依赖注入。

#### 27.如何向Spring 容器提供配置元数据

有三种方式向Spring 容器提供元数据:

 - XML配置文件
 - 基于注解配置
 - 基于Java的配置

#### 28.在Spring框架中如何更有效的使用JDBC？

使用Spring JDBC框架，资源管理以及错误处理的代价都会减轻。开发人员只需通过statements和queries语句从数据库中存取数据。Spring框架中通过使用模板类能更有效的使用JDBC，也就是所谓的JdbcTemplate。

#### 29.JdbcTemplate

JdbcTemplate类提供了许多方法，为我们与数据库的交互提供了便利。例如，它可以将数据库的数据转化为原生类型或对象，执行写好的或可调用的数据库操作语句，提供自定义的数据库错误处理功能。

#### 30.Spring对DAO的支持

Spring对数据访问对象(DAO)的支持旨在使它可以与数据访问技术(如 JDBC, Hibernate 及JDO)方便的结合起来工作。这使得我们可以很容易在的不同的持久层技术间切换，编码时也无需担心会抛出特定技术的异常。


#### 31.Spring支持的ORM

Spring支持一下ORM：

 - Hibernate
 - iBatis
 - JPA (Java -Persistence API)
 - TopLink
 - JDO (Java Data Objects)
 - OJB

#### 32. Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

#### 33.什么是织入？什么是织入应用的不同点？

织入是将切面和其他应用类型或对象连接起来创建一个通知对象的过程。织入可以在编译、加载或运行时完成。

#### 34.有几种不同类型的自动代理？

- **BeanNameAutoProxyCreator**：bean名称自动代理创建器

- **DefaultAdvisorAutoProxyCreator**：默认通知者自动代理创建器

- **Metadata autoproxying**：元数据自动代理

#### 35.Spring循环依赖问题？

1. Spring 创建 bean 主要分为两个步骤，创建原始 bean 对象，接着去填充对象属性和初始化
2. 每次创建 bean 之前，我们都会从缓存中查下有没有该 bean，因为是单例，只能有一个
3. 当我们创建 beanA 的原始对象后，并把它放到三级缓存中，接下来就该填充对象属性了，这时候发现依赖了 beanB，接着就又去创建 beanB，同样的流程，创建完 beanB 填充属性时又发现它依赖了 beanA，又是同样的流程，不同的是，这时候可以在三级缓存中查到刚放进去的原始对象 beanA，所以不需要继续创建，用它注入 beanB，完成 beanB 的创建
4. 既然 beanB 创建好了，所以 beanA 就可以完成填充属性的步骤了，接着执行剩下的逻辑，闭环完成。

**B 中提前注入了一个没有经过初始化的 A 类型对象不会有问题吗？**

虽然在创建 B 时会提前给 B 注入了一个还未初始化的 A 对象，但是在创建 A 的流程中一直使用的是注入到 B 中的 A 对象的引用，之后会根据这个引用对 A 进行初始化，所以这是没有问题的。

**Spring 是如何解决的循环依赖？**

Spring 为了解决单例的循环依赖问题，使用了三级缓存。其中一级缓存为单例池（`singletonObjects`），二级缓存为提前曝光对象（`earlySingletonObjects`），三级缓存为提前曝光对象工厂（`singletonFactories`）。

假设A、B循环引用，实例化 A 的时候就将其放入三级缓存中，接着填充属性的时候，发现依赖了 B，同样的流程也是实例化后放入三级缓存，接着去填充属性时又发现自己依赖 A，这时候从缓存中查找到早期暴露的 A，没有 AOP 代理的话，直接将 A 的原始对象注入 B，完成 B 的初始化后，进行属性填充和初始化，这时候 B 完成后，就去完成剩下的 A 的步骤，如果有 AOP 代理，就进行 AOP 处理获取代理后的对象 A，注入 B，走剩下的流程。

**为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？**

如果没有 AOP 代理，二级缓存可以解决问题，但是有 AOP 代理的情况下，只用二级缓存就意味着所有 Bean 在实例化后就要完成 AOP 代理，这样违背了 Spring 设计的原则，Spring 在设计之初就是通过 `AnnotationAwareAspectJAutoProxyCreator` 这个后置处理器来在 Bean 生命周期的最后一步来完成 AOP 代理，而不是在实例化后就立马进行 AOP 代理。



## 二.Spring Bean

#### 1.Spring Bean的生命周期
Bean在Spring中的生命周期如下：

- **实例化**：Spring通过new关键字将一个Bean进行实例化。
- **填入属性**：spring将值和bean引用注入到bean 的属性中。
- 如果Bean实现了**BeanNameAware**接口，工厂调用Bean的**setBeanName()**方法传递Bean的ID。
- 如果Bean实现了**BeanFactoryAware**接口，工厂调用**setBeanFactory()**方法传入工厂自身。
- 如果实现了**ApplicationContextAware**, spring将调用setApplicationContext()方法，将bean所在的上下文的引用 进来。
- 如果**BeanPostProcessor**和Bean关联，那么它们的**postProcessBeforeInitialization()**方法将被调用。
- 如果Bean指定了init-method方法，它将被调用。
- 如果有**BeanPostProcessor**和Bean关联，那么它们的postProcessAfterInitialization()方法将被调用
- 最后如果配置了destroy-method方法则注册**DisposableBean**.


**使用：**到这个时候，Bean已经可以被应用系统使用了，并且将被保留在Bean Factory中知道它不再需要。

**销毁**。如果Bean实现了DisposableBean接口，就调用其destroy方法。有两种方法可以把它从Bean Factory中删除掉：

1. 如果Bean实现了DisposableBean接口，destory()方法被调用。
2. 如果指定了订制的销毁方法，就调用这个方法。destory-method（）配置时指定。


对几个重要接口的解释：

- **BeanNameAware**: 实现该接口可以获得本身bean的id属性，获得在配置文件中定义好的Bean的ID名
- **BeanFactoryAware**：实现这个接口的bean其实是希望知道自己属于哪一个BeanFactory, 是哪个BeanFactory创建的。
- **ApplicationContextAware**：当一个类实现了这个接口之后，这个类就可以方便地获得 ApplicationContext 中的所有bean。换句话说，就是这个类可以直接获取Spring配置文件中，所有有引用到的bean对象。
- **BeanPostProcessor**是Spring中定义的一个接口，其与InitializingBean和DisposableBean接口类似，也是供Spring进行回调的。Spring将在初始化bean前后对BeanPostProcessor实现类进行回调，Spring容器通过BeanPostProcessor给了我们一个机会对Spring管理的bean进行再加工。比如：我们可以修改bean的属性，可以给bean生成一个动态代理实例等等。参考: [利用BeanPostProcessor做版本切换](https://www.jianshu.com/p/1417eefd2ab1)

#### 2.Spring Bean的作用域之间有什么区别

 - **singleton**：这种bean范围是默认的，这种范围确保不管接受到多少个请求，每个容器中只有一个bean的实例，单例的模式由bean factory自身来维护。
 - **prototype**：原形范围与单例范围相反，为每一个bean请求提供一个实例。
 - **request**：在请求bean范围内会每一个来自**客户端的网络请求**创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
 - **Session**：与请求范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
 - **global-session**：global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。

#### 3.spring中bean加载机制，bean生成的具体步骤

**bean的加载**:

http://zhengjianglong.cn/2015/12/06/Spring/spring-source-ioc-bean-parse/

1. 容器寻找Bean的定义信息并且将其实例化。
2. 如果允许提前暴露工厂，则提前暴露这个bean的工厂，这个工厂主要是返回该未完全处理的bean．主要是用于避免单例属性循环依赖问题．
3. 受用**依赖注入**，Spring按照Bean定义信息配置Bean的所有属性。
4. 如果Bean实现了**BeanNameAware**接口，工厂调用Bean的**setBeanName()**方法传递Bean的ID。
5. 如果Bean实现了**BeanFactoryAware**接口，工厂调用**setBeanFactory()**方法传入工厂自身。
6. 如果实现了**ApplicationContextAware**, spring将调用setApplicationContext()方法，将bean所在的上下文的引用 进来。
6. 如果**BeanPostProcessor**和Bean关联，那么它们的**postProcessBeforeInitialization()**方法将被调用。
7. 如果Bean指定了init-method方法，它将被调用。
8. 如果有**BeanPostProcessor**和Bean关联，那么它们的postProcessAfterInitialization()方法将被调用
9. 最后如果配置了destroy-method方法则注册**DisposableBean**.

http://www.cnblogs.com/ITtangtang/p/3978349.html

#### 4.什么是Spring Beans

Spring Beans是构成Spring应用核心的Java对象。这些对象由Spring IOC容器实例化、组装、管理。这些对象通过容器中配置的元数据创建，例如，使用XML文件中定义的创建。

在Spring中创建的beans都是单例的beans。在bean标签中有一个属性为”singleton”,如果设为true，该bean是单例的，如果设为false，该bean是原型bean。Singleton属性默认设置为true。因此，spring框架中所有的bean都默认为单例bean。

#### 5.Spring Bean中定义了什么内容？

Spring Bean中定义了所有的配置元数据，这些配置信息告知容器如何创建它，它的生命周期是什么以及它的依赖关系。

#### 6.Spring框架中单例beans是线程安全的吗？

不是，Spring框架中的单例beans不是线程安全的。

**Spring框架并没有对单例bean进行任何多线程的封装处理**。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。

#### 7.哪些是最重要的bean生命周期方法？能重写它们吗？

有两个重要的bean生命周期方法。

- 第一个是setup方法，该方法在容器加载bean的时候被调用。
- 第二个是teardown方法，该方法在bean从容器中移除的时候调用。
- bean标签有两个重要的属性(init-method 和 destroy-method)，你可以通过这两个属性定义自己的初始化方法和析构方法。Spring也有相应的注解：@PostConstruct 和 @PreDestroy。

#### 8.什么是Spring的内部bean

当一个bean被用作另一个bean的属性时，这个bean可以被声明为内部bean。在基于XML的配置元数据中，可以通过把元素定义在 或元素内部实现定义内部bean。内部bean总是匿名的并且它们的scope总是prototype。

#### 9.如何在Spring中注入Java集合类

Spring提供如下几种类型的集合配置元素：

 - list元素用来注入一系列的值，允许有相同的值。
 - set元素用来注入一些列的值，不允许有相同的值。
 - map用来注入一组”键-值”对，键、值可以是任何类型的。
 - props也可以用来注入一组”键-值”对，这里的键、值都字符串类型。

#### 10.什么是bean wiring？

Wiring，或者说bean Wiring是指beans在Spring容器中结合在一起的情况。当装配bean的时候，Spring容器需要知道需要哪些beans以及如何使用依赖注入将它们结合起来。

#### 11.什么是bean自动装配？

Spring容器可以自动配置相互协作beans之间的关联关系。这意味着Spring可以自动配置一个bean和其他协作bean之间的关系，通过检查BeanFactory 的内容里没有使用和< property>元素。

#### 12.自动装配有哪些局限性？

自动装配有如下局限性：

- 重写：你仍然需要使用 和< property>设置指明依赖，这意味着总要重写自动装配。
- 原生数据类型:你不能自动装配简单的属性，如原生类型、字符串和类。
- 模糊特性：自动装配总是没有自定义装配精确，因此，如果可能尽量使用自定义装配。

#### 13.你可以在Spring中注入null或空字符串吗

完全可以。

#### 14.Spring 中的单例 bean 的线程安全问题了解吗？

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

## 三、SpringMVC

#### 1.springMVC流程具体叙述下

**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

**详情：**

当应用启动时, 容器会加载servlet类并调用**init**方法. 在这个阶段，DispatcherServlet在init()完成初始化参数**init-param**的解析和封装、相关配置：

 - 完成了spring的**WebApplicationContext**的初始化，即完成xml文件的加载、bean的解析和注册等工作。
 - 另外为servlet功能所用的变量进行初始化, 如:handlerMapping, viewResolvers等.

当用户发送一个请求时，首先根据请求的类型调用DispatcherServlet不同的方法，这些方法都会转发到doService()中执行．在该方法内部完成以下工作：

 1. spring首先考虑**multipart**的处理, 如果是MultipartContent类型的request,则将该请求转换成MultipartHttpServletRequest类型的request.

 2. 根据request信息获取对应的**Handler**. 首先根据request获取访问路径,然后根据该路径可以选择直接匹配或通用匹配的方式寻找Handler,即用户定义的controller. Handler在init()方法时已经完成加载且保存到Map中了,只要根据路径就可以得到对应的Handler. 如果不存在则尝试使用默认的Handler. 如果还是没有找到那么就通过response向用户返回错误信息. 找到handler后会将其包装在一个**执行链**中,然后将所有的拦截器也加入到该链中.

 3. 如果存在handler则根据当前的handler寻找对应的**HandlerAdapter**. 通过遍历所有适配器来选择合适的适配器.

 4. SpringMVC允许你通过处理**拦截器**Web请求,进行前置处理和后置处理.所以在正式调用 Handler的逻辑方法时,先执行所有拦截器的**preHandle()**方法.

 5. 正式执行handle的业务逻辑方法**handle()**,返回ModelAndView.逻辑处理是通过适配器调用handle并返回视图.这过程其实是调用用户controller的业务逻辑.

 6. 调用拦截器的**postHandle()**方法,完成后置处理.

 7. 根据视图进行页面跳转.该过程首先会根据视图名字解析得到视图,该过程支持缓存,如果缓存中存在则直接获取,否则创建新的视图并在支持缓存的情况下保存到缓冲中. 该过程完成了像添加前缀后缀, 设置必须的属性等工作.最后就是进行页面跳转处理.

 8. 调用拦截器的**afterCompletion()**


 核心类说明：

 - **HandlerAdapter**：它的作用用一句话概括就是调用具体的方法对用户发来的请求来进行处理。当handlerMapping获取到执行请求的controller时，DispatcherServlet会根据controller对应的controller类型来调用相应的HandlerAdapter来进行处理。 

#### 2.listener是监听哪个事件(ServletContext创建事件)
ServletContextListener 接口用于监听 ServletContext 对象的创建和销毁事件:

 - 当 ServletContext 对象被创建时，激发contextInitialized (ServletContextEvent sce)方法
 - 当 ServletContext 对象被销毁时，激发contextDestroyed(ServletContextEvent sce)方法。

#### 3.过滤器与监听器的区别
Filter可认为是Servlet的一种“变种”，它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理，是个典型的处理链。它与Servlet的区别在于：它不能直接向用户生成响应。完整的流程是：Filter对用户请求进行预处理，接着将请求交给 Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。

 Java中的Filter 并不是一个标准的Servlet ，它不能处理用户请求，也不能对客户端生成响应。 主要用于对HttpServletRequest 进行预处理，也可以对HttpServletResponse 进行后处理，是个典型的处理链。优点：过滤链的好处是，执行过程中任何时候都可以打断，只要不执行chain.doFilter()就不会再执行后面的过滤器和请求的内容。而在实际使用时，就要特别注意过滤链的执行顺序问题
 http://blog.csdn.net/sd0902/article/details/8395641

Servlet,Filter都是针对url之类的，而Listener是针对对象的操作的，如session的创建，session.setAttribute的发生，或者在启动服务器的时候将你需要的数据加载到缓存等，在这样的事件发生时做一些事情。
http://www.tuicool.com/articles/bmqMjm

#### 4.请描述一下java事件监听机制。
 - Java的事件监听机制涉及到三个组件：事件源、事件监听器、事件对象
 - 当事件源上发生操作时，它将会调用事件监听器的一个方法，并在调用这个方法时，会传递事件对象过来
 - 事件监听器由开发人员编写，开发人员在事件监听器中，通过事件对象可以拿到事件源，从而对事件源上的操作进行处理。

#### 5.ContextLoaderListener是监听什么事件
ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。

#### 6.什么是Spring的MVC框架？

Spring提供了一个功能齐全的MVC框架用于构建Web应用程序。Spring框架可以很容易的和其他的MVC框架融合(如Struts)，该框架使用控制反转(IOC)将控制器逻辑和业务对象分离开来。它也允许以声明的方式绑定请求参数到业务对象上。

#### 7.DispatcherServlet

Spring的MVC框架围绕DispatcherServlet来设计的，它用来处理所有的HTTP请求和响应。

#### 8.WebApplicationContext

WebApplicationContext继承了ApplicationContext，并添加了一些web应用程序需要的功能。和普通的ApplicationContext 不同，WebApplicationContext可以用来处理主题样式，它也知道如何找到相应的servlet。

#### 9.什么是Spring MVC框架的控制器？

控制器提供对应用程序行为的访问，通常通过服务接口实现。控制器解析用户的输入，并将其转换为一个由视图呈现给用户的模型。Spring 通过一种极其抽象的方式实现控制器，它允许用户创建多种类型的控制器。

#### 10.@Controller annotation

@Controller注解表示该类扮演控制器的角色。Spring不需要继承任何控制器基类或应用Servlet API。

#### 11.@RequestMapping annotation

@RequestMapping注解用于将URL映射到任何一个类或者一个特定的处理方法上。

## 四、注解

#### 1.什么是Spring基于Java的配置？给出一些注解的例子
基于Java的配置允许你使用Java的注解进行Spring的大部分配置而非通过传统的XML文件配置。以注解@Configuration为例，它用来标记类，说明作为beans的定义，可以被Spring IOC容器使用。另一个例子是@Bean注解，它表示该方法定义的Bean要被注册进Spring应用上下文中。

#### 2.什么是基于注解的容器配置
另外一种替代XML配置的方式为基于注解的配置，这种方式通过字节元数据装配组件而非使用尖括号声明。开发人员将直接在类中进行配置，通过注解标记相关的类、方法或字段声明，而不再使用XML描述bean之间的连线关系。

#### 3.如何开启注解装配？
注解装配默认情况下在Spring容器中是不开启的。如果想要开启基于注解的装配只需在Spring配置文件中配置元素即可。<context:annotation-config>

#### 4.@Required 注解
@Required表明bean的属性必须在配置时设置，可以在bean的定义中明确指定也可通过自动装配设置。如果bean的属性未设置，则抛出BeanInitializationException异常。

#### 5.@Autowired 注解
@Autowired 注解提供更加精细的控制，包括自动装配在何处完成以及如何完成。它可以像@Required一样自动装配setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

#### 6.@Qualifier 注解
当有多个相同类型的bean而只有其中的一个需要自动装配时，将@Qualifier 注解和@Autowire 注解结合使用消除这种混淆，指明需要装配的bean。

#### 7.@Resource 和@Autowired区别

CommonAnnotationBeanPostProcessor是处理@ReSource注解的

AutoWiredAnnotationBeanPostProcessor是处理@AutoWired注解的

（2）@Autowired只按照byType 注入；@Resource默认按byName自动注入，也提供按照byType 注入；

（3）属性：@Autowired按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。@Resource有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。

@Resource装配顺序

　　1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常

　　2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常

　　3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常

　　4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；

推荐使用@Resource注解在字段上，这样就不用写setter方法了.并且这个注解是属于J2EE的，减少了与Spring的耦合,这样代码看起就比较优雅 。

#### 8. @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

#### 9.将一个类声明为Spring的 bean 的注解有哪些?

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

#### 10.[Spring注解总结](https://snailclimb.gitee.io/javaguide/#/./docs/system-design/framework/spring/spring-annotations)

## 五.事务

#### 1.Spring支持的事务管理类型
Spring支持如下两种方式的事务管理：

- **编码式事务管理**：sping对编码式事务的支持与EJB有很大区别，不像EJB与java事务API耦合在一起．spring通过回调机制将实际的事务实现从事务性代码中抽象出来．**你能够精确控制事务的边界，它们的开始和结束完全取决于你**．
- **声明式事务管理**：这种方式意味着你可以将事务管理和业务代码分离。你只需要通过注解或者XML配置管理事务。通过传播行为，隔离级别，回滚规则，事务超时，只读提示来定义．

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

#### 2.Spring框架的事务管理有哪些优点
- 它为不同的事务API(如JTA, JDBC, Hibernate, JPA, 和JDO)提供了统一的编程模型。
- 它为编程式事务管理提供了一个简单的API而非一系列复杂的事务API(如JTA).
- 它支持声明式事务管理。
- 它可以和Spring 的多种数据访问技术很好的融合。

#### 3.ACID
- **原子性(Atomic)**: 一个操作要么成功，要么全部不执行.
- **一致性(Consistent)**: 一旦事务完成，系统必须确保它所建模业务处于一致的状态
- **隔离性(Isolated)**: 事务允许多个用户对相同的数据进行操作，每个用户用户的操作相互隔离互补影响．
- **持久性(Durable)**: 一旦事务完成，事务的结果应该持久化．

#### 4.spring事务定义的传播规则
**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

#### 5.spring事务支持的隔离级别
**并发会导致以下问题：**

 - **脏读**：发生在一个事务读取了另一个事务改写但尚未提交的数据．
 - **不可重复读**：在一个事务执行相同的查询两次或两次以上，每次得到的数据不同．
 - **幻读**：与不可重复读类似，发生在一个事务读取多行数据，接着另一个并发事务插入一些数据，随后查询中，第一个事务发现多了一些原本不存在的数据．

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

#### 6.你更推荐那种类型的事务管理？
许多Spring框架的用户选择声明式事务管理，因为这种方式和应用程序的关联较少，因此更加符合轻量级容器的概念。声明式事务管理要优于编程式事务管理，尽管在灵活性方面它弱于编程式事务管理(这种方式允许你通过代码控制业务)。

#### 7.@Transactional(rollbackFor = Exception.class)注解了解吗？

我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

#### 8.事务失效（Spring AOP 自调用问题）

若同一类中的其他没有 `@Transactional` 注解的方法内部调用有 `@Transactional` 注解的方法，有`@Transactional` 注解的方法的事务会失效。

这是由于`Spring AOP`代理的原因造成的，因为只有当 `@Transactional` 注解的方法在类以外被调用的时候，Spring 事务管理才生效。

`MyService` 类中的`method1()`调用`method2()`就会导致`method2()`的事务失效。

#### 9.[Spring事务总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/spring-transaction)



