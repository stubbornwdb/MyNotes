# werpc

## 0.项目介绍 -STAR模型

**Situation**:  简短的项目背景，比如项目的规模，开发的软件的功能、目标用户等。
**Task** ：自己完成的任务。
**Action**：为了完成任务自己做了哪些工作，是怎么做的
**Result**：自己的贡献





## 1. RPC的由来/什么是RPC？
随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

**单一应用架构**
当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。
此时，用于简化增删改查工作量的 数据访问框架(ORM) 是关键。
**垂直应用架构**
当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。
此时，用于加速前端页面开发的 Web框架(MVC) 是关键。
**分布式服务架构**
当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
此时，用于提高业务复用及整合的 分布式服务框架(RPC)，提供统一的服务是关键。
例如：各个团队的服务提供方就不要各自实现一套序列化、反序列化、网络框架、连接池、收发线程、超时处理、状态机等“业务之外”的重复技术劳动，造成整体的低效。

所以，统一RPC框架来解决提供统一的服务。

**什么是 RPC**
RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。比如两个不同的服务 A、B 部署在两台不同的机器上，那么服务 A 如果想要调用服务 B 中的某个方法该怎么办呢？使用 HTTP请求 当然可以，但是可能会比较慢而且一些优化做的并不好。 RPC 的出现就是为了解决这个问题。
最终解决的问题：让分布式或者微服务系统中不同服务之间的调用像本地调用一样简单。

##  2.RPC的实现原理

两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

比如说，A服务器想调用B服务器上的一个方法：

Employee getEmployeeByName(String fullName)

整个调用过程，主要经历如下几个步骤：建立通信、服务寻址、网络传输、服务调用

### 2.1 建立通信

首先要解决通讯的问题：即A机器想要调用B机器，首先得建立起通信连接。

主要是通过在客户端和服务器之间建立TCP连接，远程过程调用的所有交换的数据都在这个连接里传输。连接可以是按需连接，调用结束后就断掉，也可以是长连接，多个远程过程调用共享同一个连接。

### 2.2 服务寻址

要解决寻址的问题，也就是说，A服务器上的应用怎么告诉底层的RPC框架，如何连接到B服务器（如主机或IP地址）以及特定的端口，方法的名称名称是什么。

通常情况下我们需要提供B机器（主机名或IP地址）以及特定的端口，然后指定调用的方法或者函数的名称以及入参出参等信息，这样才能完成服务的一个调用。

可靠的寻址方式（主要是提供服务的发现）是RPC的实现基石，比如可以采用redis或者zookeeper来注册服务等等。


从服务提供者的角度看：当提供者服务启动时，需要自动向注册中心注册服务；
当提供者服务停止时，需要向注册中心注销服务；
提供者需要定时向注册中心发送心跳，一段时间未收到来自提供者的心跳后，认为提供者已经停止服务，从注册中心上摘取掉对应的服务。
从调用者的角度看：调用者启动时订阅注册中心的消息并从注册中心获取提供者的地址；
当有提供者上线或者下线时，注册中心会告知到调用者；
调用者下线时，取消订阅。

### 2.3 网络传输

2.3.1 序列化

当A机器上的应用发起一个RPC调用时，调用方法和其入参等信息需要通过底层的网络协议如TCP传输到B机器，由于网络协议是基于二进制的，所有我们传输的参数数据都需要先进行序列化（Serialize）或者编组（marshal）成二进制的形式才能在网络中进行传输。然后通过寻址操作和网络传输将序列化或者编组之后的二进制数据发送给B机器。

2.3.2 反序列化

当B机器接收到A机器的应用发来的请求之后，又需要对接收到的参数等信息进行反序列化操作（序列化的逆操作），即将二进制信息恢复为内存中的表达方式，然后再找到对应的方法（寻址的一部分）进行本地调用（一般是通过生成代理Proxy去调用,
通常会有JDK动态代理、CGLIB动态代理、Javassist生成字节码技术等），之后得到调用的返回值。

### 2.4 服务调用

B机器进行本地调用（通过代理Proxy）之后得到了返回值，此时还需要再把返回值发送回A机器，同样也需要经过序列化操作，然后再经过网络传输将二进制数据发送回A机器，而当A机器接收到这些返回值之后，则再次进行反序列化操作，恢复为内存中的表达方式，最后再交给A机器上的应用进行相关处理（一般是业务逻辑处理操作）。

通常，经过以上四个步骤之后，一次完整的RPC调用算是完成了。

## 3. RPC架构组件
一个基本的RPC架构里面应该至少包含以下4个组件：

1、客户端（Client）:服务调用方（服务消费者）

2、客户端存根（Client Stub）:存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端

3、服务端存根（Server Stub）:接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理

4、服务端（Server）:服务的真正提供者

## 4. RPC调用过程

1、服务消费者（client客户端）通过本地调用的方式调用服务

2、客户端存根（client stub）接收到调用请求后负责将方法、入参等信息序列化（组装）成能够进行网络传输的消息体

3、客户端存根（client stub）找到远程的服务地址，并且将消息通过网络发送给服务端

4、服务端存根（server stub）收到消息后进行解码（反序列化操作）

5、服务端存根（server stub）根据解码结果调用本地的服务进行相关处理

6、本地服务执行具体业务逻辑并将处理结果返回给服务端存根（server stub）

7、服务端存根（server stub）将返回结果重新打包成消息（序列化）并通过网络发送至消费方

8、客户端存根（client stub）接收到消息，并进行解码（反序列化）

9、服务消费方得到最终结果


##  5. 业界常用的 RPC 框架
Dubbo: Dubbo 是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。目前 Dubbo 已经成为 Spring Cloud Alibaba 中的官方组件。
gRPC ：gRPC 是可以在任何环境中运行的现***源高性能RPC框架。它可以通过可插拔的支持来有效地连接数据中心内和跨数据中心的服务，以实现负载平衡，跟踪，运行状况检查和身份验证。它也适用于分布式计算的最后一英里，以将设备，移动应用程序和浏览器连接到后端服务。
Hessian： Hessian是一个轻量级的 remoting-on-http 工具，使用简单的方法提供了 RMI 的功能。 相比 WebService，Hessian 更简单、快捷。采用的是二进制 RPC协议，因为采用的是二进制协议，所以它很适合于发送二进制数据。
## 6. 为什么用 RPC，不用 HTTP
首先需要指正，这两个并不是并行概念。RPC 是一种设计，就是为了解决不同服务之间的调用问题，完整的 RPC 实现一般会包含有 传输协议 和 序列化协议 这两个。

而 HTTP 是一种传输协议，RPC 框架完全可以使用 HTTP 作为传输协议，也可以直接使用 TCP，使用不同的协议一般也是为了适应不同的场景。

使用 TCP 和使用 HTTP 各有优势：

传输效率：

TCP，通常自定义上层协议，可以让请求报文体积更小
HTTP：如果是基于HTTP 1.1 的协议，请求中会包含很多无用的内容
性能消耗，主要在于序列化和反序列化的耗时

TCP，可以基于各种序列化框架进行，效率比较高
HTTP，大部分是通过 json 来实现的，字节大小和序列化耗时都要更消耗性能
跨平台：

TCP：通常要求客户端和服务器为统一平台
HTTP：可以在各种异构系统上运行
总结：
  RPC 的 TCP 方式主要用于公司内部的服务调用，性能消耗低，传输效率高。HTTP主要用于对外的异构环境，浏览器接口调用，APP接口调用，第三方接口调用等。


## 7. 调用如何在客户端无感（动态代理）
基于动态代理生成代理对象，当调用代理对象的方法时，由代理进行相关信息（方法、参数等）的组装并发送到服务器进行远程调用，并由代理接收调用结果并返回。

### 7.1 动态代理和静态代理的区别
静态代理的代理对象和被代理对象在代理之前就已经确定，它们都实现相同的接口或继承相同的抽象类。静态代理模式一般由业务实现类和业务代理类组成，业务实现类里面实现主要的业务逻辑，业务代理类负责在业务方法调用的前后作一些你需要的处理，以实现业务逻辑与业务方法外的功能解耦，减少了对业务方法的入侵。静态代理又可细分为：基于继承的方式和基于聚合的方式实现。

静态代理模式的代理类，只是实现了特定类的代理，代理类对象的方法越多，你就得写越多的重复的代码。动态代理就可以动态的生成代理类，实现对不同类下的不同方法的代理。

JDK 动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用业务方法前调用InvocationHandler 处理。代理类必须实现 InvocationHandler 接口，并且，JDK 动态代理只能代理实现了接口的类

### 7.2 JDK 动态代理的步骤
使用 JDK 动态代理类基本步骤：

1、编写需要被代理的类和接口

2、编写代理类，需要实现 InvocationHandler 接口，重写 invoke() 方法；

3、使用Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)动态创建代理类对象，通过代理类对象调用业务方法。

如果想代理没有实现接口的对象
CGLIB 框架实现了对无接口的对象进行代理的方式。JDK 动态代理是基于接口实现的，而 CGLIB 是基于继承实现的。它会对目标类产生一个代理子类，通过方法拦截技术对过滤父类的方法调用。代理子类需要实现 MethodInterceptor 接口。

CGLIB 底层是通过 asm 字节码框架实时生成类的字节码，达到动态创建类的目的，效率较 JDK 动态代理低。Spring 中的 AOP 就是基于动态代理的，如果被代理类实现了某个接口，Spring 会采用 JDK 动态代理，否则会采用 CGLIB。

可以写一个动态代理的例子吗
不可以

```
interface DemoInterface {
    String hello(String msg);
}

class DemoImpl implements DemoInterface {
    @Override
    public String hello(String msg) {
        System.out.println("msg = " + msg);
        return "hello";
    }
}

class DemoProxy implements InvocationHandler {

​    private DemoInterface service;

​    public DemoProxy(DemoInterface service) {
​        this.service = service;
​    }

​    @Override
​    public Object invoke(Object obj, Method method, Object[] args) throws Throwable {
​        System.out.println("调用方法前...");
​        Object returnValue = method.invoke(service, args);
​        System.out.println("调用方法后...");
​        return returnValue;
​    }

}

public class Solution {
    public static void main(String[] args) {
        DemoProxy proxy = new DemoProxy(new DemoImpl());
        DemoInterface service = Proxy.newInstance(
            DemoInterface.class.getClassLoader(),
            new Class<?>[]{DemoInterface.class},
            proxy
        );
        System.out.println(service.hello("呀哈喽！"));
    }
}
```

输出：
调用方法前...
msg = 呀哈喽！
调用方法后...
hello

## 8.对象是怎么在网络中传输的（序列化）
通过将对象序列化成字节数组，即可将对象发送到网络中。

序列化（serialization）在计算机科学的资料处理中，是指将数据结构或对象状态转换成可取用格式（例如存成文件，存于缓冲，或经由网络中发送），以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始对象相同语义的副本。对于许多对象，像是使用大量引用的复杂对象，这种序列化重建的过程并不容易。这种过程也称为对象编组（marshalling）。从一系列字节提取数据结构的反向操作，是反序列化（也称为解编组、deserialization、unmarshalling）。

在 Java 中，想要序列化一个对象，这个对象所属的类必须实现了 Serializable 接口，并且其内部属性必须都是可序列化的。如果有一个属性不是可序列化的，则该属性必须被声明为 transient。

JDK 中提供了 ObjectOutStream 类来对对象进行序列化。

### 8.1 你的框架实现了哪几种序列化方式，可以介绍下吗

实现了 JSON、Kryo、Hessian 和 Protobuf 的序列化及自定义序列化方式。

JSON 是一种轻量级的数据交换语言，该语言以易于让人阅读的文字为基础，用来传输由属性值或者序列性的值组成的数据对象，类似 xml，Json 比 xml更小、更快更容易解析。JSON 由于采用字符方式存储，占用相对于字节方式较大，并且序列化后类的信息会丢失，可能导致反序列化失败。

剩下的都是基于字节的序列化。

Kryo 是一个快速高效的 Java 序列化框架，旨在提供快速、高效和易用的 API。无论文件、数据库或网络数据 Kryo 都可以随时完成序列化。 Kryo 还可以执行自动深拷贝、浅拷贝。这是对象到对象的直接拷贝，而不是对象->字节->对象的拷贝。kryo 速度较快，序列化后体积较小，但是跨语言支持较复杂。

Hessian 是一个基于二进制的协议，Hessian 支持很多种语言，例如 Java、python、c++,、net/c#、D、Erlang、PHP、Ruby、object-c等，它的序列化和反序列化也是非常高效。速度较慢，序列化后的体积较大。

protobuf（Protocol Buffers）是由 Google 发布的数据交换格式，提供跨语言、跨平台的序列化和反序列化实现，底层由 C++ 实现，其他平台使用时必须使用 protocol compiler 进行预编译生成 protoc 二进制文件。性能主要消耗在文件的预编译上。序列化反序列化性能较高，平台无关。

### 8.2 哪种快？为什么

1、XML序列化（Xstream）无论在性能和简洁性上比较差。

2、Thrift与Protobuf相比在时空开销方面都有一定的劣势。

3、Protobuf和Avro在两方面表现都非常优越。

**选型建议**

以上描述的五种序列化和反序列化协议都各自具有相应的特点，适用于不同的场景：

1、对于公司间的系统调用，如果性能要求在100ms以上的服务，基于XML的SOAP协议是一个值得考虑的方案。

2、基于Web browser的Ajax，以及Mobile app与服务端之间的通讯，JSON协议是首选。对于性能要求不太高，或者以动态类型语言为主，或者传输数据载荷很小的的运用场景，JSON也是非常不错的选择。

3、对于调试环境比较恶劣的场景，采用JSON或XML能够极大的提高调试效率，降低系统开发成本。

4、当对性能和简洁性有极高要求的场景，Protobuf，Thrift，Avro之间具有一定的竞争关系。

5、对于T级别的数据的持久化应用场景，Protobuf和Avro是首要选择。如果持久化后的数据存储在Hadoop子项目里，Avro会是更好的选择。

6、由于Avro的设计理念偏向于动态类型语言，对于动态语言为主的应用场景，Avro是更好的选择。

7、对于持久层非Hadoop项目，以静态类型语言为主的应用场景，Protobuf会更符合静态类型语言工程师的开发习惯。

8、如果需要提供一个完整的RPC解决方案，Thrift是一个好的选择。

9、如果序列化之后需要支持不同的传输层协议，或者需要跨防火墙访问的高性能场景，Protobuf可以优先考虑。



### 8.3 自定义序列化



**传统的序列化**

典型的序列化和反序列化过程往往需要如下组件：

- IDL（Interface description language）文件：参与通讯的各方需要对通讯的内容需要做相关的约定（Specifications）。为了建立一个与语言和平台无关的约定，这个约定需要采用与具体开发语言、平台无关的语言来进行描述。这种语言被称为接口描述语言（IDL），采用IDL撰写的协议约定称之为IDL文件。
- IDL Compiler：IDL文件中约定的内容为了在各语言和平台可见，需要有一个编译器，将IDL文件转换成各语言对应的动态库。
- Stub/Skeleton Lib：负责序列化和反序列化的工作代码。Stub是一段部署在分布式系统客户端的代码，一方面接收应用层的参数，并对其序列化后通过底层协议栈发送到服务端，另一方面接收服务端序列化后的结果数据，反序列化后交给客户端应用层；Skeleton部署在服务端，其功能与Stub相反，从传输层接收序列化参数，反序列化后交给服务端应用层，并将应用层的执行结果序列化后最终传送给客户端Stub。
- Client/Server：指的是应用层程序代码，他们面对的是IDL所生存的特定语言的class或struct。
- 底层协议栈和互联网：序列化之后的数据通过底层的传输层、网络层、链路层以及物理层协议转换成数字信号在互联网中传递。



**smartbuf**

smartbuf是一种新颖、高效、智能、易用的跨语言序列化框架，它既拥有不亚于protobuf的高性能，也拥有与json相仿的通用性、可扩展性、可调试性等。

与json、xml类似，smartbuf编码后的数据中仍然保留着schema信息， 无需预先协定任何数据模型，接收方即可直接完成数据的解码。这个特性赋予了smartbuf类似于json的通用、可扩展、可读性、兼容性等。

为提高数据的压缩与传输的效率，smartbuf内部采用分区序列化的策略，即将对象拆分为多个不同的分区， 针对不同的分区按照不同的规则进行紧凑的序列化，然后各个分区之间通过id引用并组成最终的实体对象。 关于分区序列化的技术细节，可以参考分区序列化章节。

分区序列化的设计策略在确保扩展性、兼容性的同时，也提供了非常高的压缩率和性能，在实际测试中它甚至比protobuf要更高一些。 关于性能测试的具体细节，可以参考性能测试章节。

smartbuf针对不同的业务场景提供两种了不同的模式packet与stream，它们分别适用于不同的实际场景，具体细节可以参考后续章节。

分区序列化
在smartbuf的设计理念中，对象由以下三部分组成：

property：组成对象的底层属性，比如整数、浮点数、字符串等
struct：描述对象实体的数据结构，包括属性名、属性列表等
body：对象实体，以引用的形式将struct和property组装为完整的实例。
针对这样的设计理念，smartbuf引入了分区序列化的概念。 即对象编码过程中，将不同的部分以独立分区的形式独立编码，从而形成若干个紧凑的分区，各个分区之间通过唯一ID进行关联以完成对象的组装。

property分区
属性分区负责存储通用、标准的属性值，支持const、float、double、varint、string、symbol等类型， 并为这些属性值分配递增且唯一的ID。

在其他分区中引用这些属性的地方，直接引用ID即可，这个ID往往都很小，只需要1~2byte。

在实际情况下，对象树中的某些属性有可能重复出现，对于数组和集合尤其如此。此时使用json和protobuf的话，重复出现的数据会被重复序列化。 而对于smartbuf，得益于分区的设计，它不需要再重复序列化相同的属性，从而显著提高空间利用率。

struct分区
对象在smartbuf编码中的表现形式类似于动态语言中的弱类型结构体，这一点与json类似。 即只负责维护松散的字段列表，而不需要考虑每个属性的具体类型，字段的真实属性取决于它的值。

struct分区包括两部分: field池与struct池，前者类似于string[]，后者类似于int[][]， 即结构体的表现形式为int[]，其中int表示field池的某个元素ID。

这样的设计有两个优点：

字段名复用：编码时不同对象可以复用相同的字段名，尤其是常用名称如name、id、timestamp、url等。
上下文复用：在长连接通信时，可以将整个struct分区缓存在上下文中重复使用，从而降低数据报文的冗余数据，提供数据传输效率。
针对这个特点，smartbuf内部会维护两套结构体，分别为temporary和context， 前者用于临时性地描述类似Map的松散对象，而后者用于描述上下文复用的POJO这样的固定对象，

这也是前文中提到的packet和stream两种模式的最大区别，由于不支持上下文概念，packet模式会把所有对象都视为temporary类型。

数据分区
数据分区事实上就等同于常规的数据体Body。

如上文所言，对象的property与struct都已经提取至独立的分区中， 因此在body中只需要通过极小的空间即可实现对property和struct的引用，从而组装成一个完整的对象。

以上规则主要针对普通Object，对于数组则有一套特殊的处理规则：

原生数组：原生数组作为特殊的实体，并不需要提取至property分区，以免property分区过于臃肿。
数组分片：一个数组可能包含不同的数据类型、甚至null，而数组分片技术就是专门设计用于这个场景
smartbuf对数组的处理有一套非常巧妙的算法，这个算法可以提高编码空间利用率。

实例演示
为加深大家对分区序列化的理解，也为了更清晰地展示smartbuf序列化的最终效果，本章节通过一个简单的对象演示上文所述的分区编码的细节。

这是一个简单的User模型数据结构：

message User {
    int32 id = 1;
    string name = 2;
    int64 time = 3;
}
首次使用smartbuf对此模型的一个实例User{id=1001, name="hello", time=10000L}进行编码，其最终输出字节码结构如下所示：

smartbuf-first-packet

上图字节码中包括了对象结构元数据，所以显得臃肿了一些。

如果在stream模式下重复使用该模型的话，由于上下文可以缓存这些元数据，则不需要附加这些额外的描述性数据，最终编码效果如下所示：

smartbuf-following

你可以想象一下，在实际的系统开发中，我们传输的数据（尤其是数组）内部常常存在许多重复属性， 使用smartbuf序列化这些重复属性的话，往往只需要额外的一个字节。

除此之外，从这个例子也可以看出，即便是对于没有缓存完整上下文schema元数据的任何人，都可以正常解析这个报文。 当然，由于数据体中缺乏辅助性的字段信息，仍不能正常解析id, name, time这些字段名，只能将它们解析为无意义的序号。

这个特点对于数据可读性、可调试性都有很大的帮助，你可以通过网络抓包的方式， 直接查看smartbuf的编码数据报文，这一点对于protobuf等是很难做到的。



**性能测试**

优势与劣势
从上面这个例子中，我们可以直观的看到，smartbuf最大限度地将schema信息保留在序列化结果中， 这就导致它面对小数据集时，尤其是100B左右的测试性小对象时，难以发挥设计上的优势。

但是对于正常的数据对象，比如2K至20K这样常用的系统数据而言，smartbuf算法设计上的优势便可以充分体现， 对于数组类的较大的数据对象而言，smartbuf的空间利用率将明显超出protobuf。

使用smartbuf不需要预定义任何类似*.proto的IDL，它可以直接将普通的POJO编码为byte[]，整个过程与常用的json序列化工具非常相似。

总而言之，使用smartbuf可能带来以下益处：

数据传输效率更高
相比于json，它可以降低30%~70%的网络资源消耗。

相比于protobuf，它也可以降低10%左右的网络资源消耗。

这对于互联网产品而言，尤其是网络环境可能不太好的移动互联网，它可以一定程度地提高接口响应速度、降低设备耗电量、提高系统吞吐率等。

提高开发调试灵活性
相比于protobuf，使用smartbuf不再需要手动维护IDL，这对于快速迭代的早中期产品而言非常重要。

还有一点不容忽视的就是IDL对产品的影响，比如我亲身接触到的一个使用protobuf的Android应用， 伴随着快速迭代而频繁修改proto，该产品上线一年之后，由proto编译而成的jar包甚至达到了惊人的3.8MB，而整个APP才不到12MB。



## 9.  消息体的定义

### 9.1 WeRpcRequest

```
/**
 * 消费者向提供者发送的请求对象
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class WeRpcRequest implements Serializable {

    /**
     * 请求号
     */
    private String requestId;
    /**
     * 待调用接口名称
     */
    private String interfaceName;
    /**
     * 待调用方法名称
     */
    private String methodName;
    /**
     * 调用方法的参数
     */
    private Object[] parameters;
    /**
     * 调用方法的参数类型
     */
    private Class<?>[] paramTypes;

    /**
     * 是否是心跳包
     */
    private Boolean heartBeat;

}
```

### 9.2 WeRpcResponse

```
/**
 * 提供者执行完成或出错后向消费者返回的结果对象
 */
@Data
@NoArgsConstructor
public class WeRpcResponse<T> implements Serializable {

    /**
     * 响应对应的请求号
     */
    private String requestId;
    /**
     * 响应状态码
     */
    private Integer statusCode;
    /**
     * 响应状态补充信息
     */
    private String message;
    /**
     * 响应数据
     */
    private T data;
    public static <T> WeRpcResponse<T> success(T data, String requestId) {
        WeRpcResponse<T> response = new WeRpcResponse<>();
        response.setRequestId(requestId);
        response.setStatusCode(ResponseCode.SUCCESS.getCode());
        response.setData(data);
        return response;
    }
    public static <T> WeRpcResponse<T> fail(ResponseCode code, String requestId) {
        WeRpcResponse<T> response = new WeRpcResponse<>();
        response.setRequestId(requestId);
        response.setStatusCode(code.getCode());
        response.setMessage(code.getMessage());
        return response;
    }

}
```



## 10. RPC客户端的调用过程

### 10.1 客户端调用逻辑

```
//创建RPC客户端
WeRpcClient client = new WeRpcNettyClient(CommonSerializer.SMARTBUF_SERIALIZER);
//动态代理器，将客户端传入动态代理器
WeRpcClientProxy rpcClientProxy = new WeRpcClientProxy(client);
//获取动态代理对象
HelloService helloService = rpcClientProxy.getProxy(HelloService.class);

//使用动态代理对象调用服务
HelloObject object = new HelloObject(12, "This is a message");
String res = helloService.hello(object);
System.out.println(res);
```

首先是创建netty通信客户端WeRpcNettyClient,构造参数可传入序列化器以及负载均衡器，默认使用KRYO序列化方式以及随机的负载均衡算法。另外这个通信的客户端主要封装了一个sendRequest的方法，sendRequest方法是一个一步调用的方法，返回CompletableFuture,主要用于异步发送调用请求。这个方法首先要根据调用接口的名称获取到服务的地址（ip + 端口号 InetSocketAddress）。Netty使用这个InetSocketAddress获取到Channnel，通过Channel发送网络请求。另外，我还使用ConcurrentHashMap（服务调用号(K) ----  CompletableFuture<WeRpcRequest>  (V)）装载客户端的异步调用请求，当异步调用结束的时候就从map中移除。

然后就是关于怎么远程过程调用的问题。其实直接将要调用的接口、方法、参数那些放入到通信的客户端中也是可以的。但是就会出现一些问题，就是每次远程调用的时候都要重新写这个通信客户端代码，非常麻烦，通信的底层细节和调用服务的过程耦合在了一起。为了解决这个问题，就需要用到动态代理了。使用动态代理可以将通信的底层细节剥离开来。当调用代理对象的方法时，由代理进行相关信息（通信和服务调用信息）的组装并发送到服务器进行远程调用，并由代理接收调用结果并返回。主要就是用一个动态代理器实现，这个动态代理器实现InvocationHandler接口，动态代理器调用getProxy方法得到远程调用的代理对象，而前面说的通信客户端作为动态代理器的构造参数传入，并在invoke方法中用sendRequest方法来发送调用请求。

### 10.2 异步调用CompletableFuture

CompletableFuture是jdk8以来出现的一个专门处理并发的类库，是Future的加强版，集成了更多的方法以应对更加复杂的业务场景

```
CompletableFuture<WeRpcResponse> resultFuture = new CompletableFuture<>();
```



## 11. 自定义注解怎么实现的

自定义注解主要是通过Java反射机制实现的。

### 11.0 自定义注解

**1.是什么**

注解是一种元数据形式。即注解是属于java的一种数据类型，和类、接口、数组、枚举类似。
注解用来修饰，类、方法、变量、参数、包。
注解不会对所修饰的代码产生直接的影响。

**2.作用范围**

注解有许多用法，其中有：**为编译器提供信息** - 注解能被编译器检测到错误或抑制警告。**编译时和部署时的处理** - 软件工具能处理注解信息从而生成代码，XML文件等等。**运行时的处理** - 有些注解在运行时能被检测到。

**3.怎么用**

注解其实就是一种标记，可以在程序代码中的关键节点（类、方法、变量、参数、包）上打上这些标记，然后程序在编译时或运行时可以检测到这些标记从而执行一些特殊操作。因此可以得出自定义注解使用的基本流程：

第一步，定义注解——相当于定义标记；
第二步，配置注解——把标记打在需要用到的程序代码中；
第三步，解析注解——在编译期或运行时检测到标记，并进行特殊操作。



@Target注解，是专门用来限定某个自定义注解能够被应用在哪些Java元素上面的。它使用一个枚举类型定义

    /** 类，接口（包括注解类型）或枚举的声明 */
        TYPE,
    /** 属性的声明 */
    FIELD,
    
    /** 方法的声明 */
    METHOD,
    
    /** 方法形式参数声明 */
    PARAMETER,
    
    /** 构造方法的声明 */
    CONSTRUCTOR,
    
    /** 局部变量声明 */
    LOCAL_VARIABLE,
    
    /** 注解类型声明 */
    ANNOTATION_TYPE,
    
    /** 包的声明 */
    PACKAGE
@Retention注解，翻译为持久力、保持力。即用来修饰自定义注解的生命力。
注解的生命周期有三个阶段：1、Java源文件阶段；2、编译到class文件阶段；3、运行期阶段。同样使用了RetentionPolicy枚举类型定义了三个阶段

**4.自定义注解的运行时解析**

在运行期探究和使用编译期的内容（编译期配置的注解），要用到Java中的灵魂技术——反射！



### 11.1 项目中的自定义注解

@WeRpcServiceScan注解

```
/**
 * 服务扫描的基包
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface WeRpcServiceScan {
    public String value() default "";
}
```

@WeRpcService注解

```
/**
 * 表示一个服务提供类，用于远程接口的实现类
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface WeRpcService {
    public String name() default "";
}
```

 AbstractWeRpcServer中的 scanServices方法：利用反射机制解析注解

```
public void scanServices() {
    String mainClassName = ReflectUtil.getStackTrace();
    Class<?> startClass;
    try {
        startClass = Class.forName(mainClassName);
        if(!startClass.isAnnotationPresent(WeRpcServiceScan.class)) {
            logger.error("启动类缺少 @ServiceScan 注解");
            throw new WeRpcException(WeRpcError.SERVICE_SCAN_PACKAGE_NOT_FOUND);
        }
    } catch (ClassNotFoundException e) {
        logger.error("出现未知错误");
        throw new WeRpcException(WeRpcError.UNKNOWN_ERROR);
    }
    //从定义的扫描包下进行扫描
    String basePackage = startClass.getAnnotation(WeRpcServiceScan.class).value();
    if("".equals(basePackage)) {
        basePackage = mainClassName.substring(0, mainClassName.lastIndexOf("."));
    }
    Set<Class<?>> classSet = ReflectUtil.getClasses(basePackage);
    for(Class<?> clazz : classSet) {
        //找到使用@WeRpcService注解的类
        if(clazz.isAnnotationPresent(WeRpcService.class)) {
        	//获取注解的值 name
            String serviceName = clazz.getAnnotation(WeRpcService.class).name();
            Object obj;
            try {
               //尝试实例化
                obj = clazz.newInstance();
            } catch (InstantiationException | IllegalAccessException e) {
                logger.error("创建 " + clazz + " 时有错误发生");
                continue;
            }
            //如果name为空
            if("".equals(serviceName)) {
            	//找到该对象实现的所有接口
                Class<?>[] interfaces = clazz.getInterfaces();
                for (Class<?> oneInterface: interfaces){
                	//使用接口名注册服务
                    publishService(obj, oneInterface.getCanonicalName());
                }
            } else {
                //使用注解定义的name注册服务
                publishService(obj, serviceName);
            }
        }
    }
}

/**
 * 发布服务
 * @param service 服务实例
 * @param serviceName 服务名称
 * @param <T>
 */
 @Override
 public <T> void publishService(T service, String serviceName) {
     //向本地的服务注册表添加服务(ConcurrentHashMap)
     serviceProvider.addServiceProvider(service, serviceName);
     //向注册中心注册服务
     serviceRegistry.register(serviceName, new InetSocketAddress(host, port));
 }
```

### 11. 2 反射机制

JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 java 语言的反射机制。

**获取 Class 对象的四种方式**

如果我们动态获取到这些信息，我们需要依靠 Class 对象。Class 类对象将一个类的方法、变量等信息告诉运行的程序。Java 提供了四种方式获取 Class 对象:

1.知道具体类的情况下可以使用：

```java
Class alunbarClass = TargetObject.class;
```

但是我们一般是不知道具体类的，基本都是通过遍历包下面的类来获取 Class 对象，通过此方式获取Class对象不会进行初始化

2.通过 `Class.forName()`传入类的路径获取：

```java
Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");
```

Class.forName(className)方法，内部实际调用的是一个native方法 forName0(className, true, ClassLoader.getClassLoader(caller), caller);

第2个boolean参数表示类是否需要初始化，Class.forName(className)默认是需要初始化。

一旦初始化，就会触发目标对象的 static块代码执行，static参数也会被再次初始化。

3.通过对象实例`instance.getClass()`获取：

```java
Employee e = new Employee();
Class alunbarClass2 = e.getClass();
```

4.通过类加载器`xxxClassLoader.loadClass()`传入类路径获取

```java
class clazz = ClassLoader.LoadClass("cn.javaguide.TargetObject");
```

通过类加载器获取Class对象不会进行初始化，意味着不进行包括初始化等一些列步骤，静态块和静态对象不会得到执行



**静态编译和动态编译**

- **静态编译：** 在编译时确定类型，绑定对象
- **动态编译：** 运行时确定类型，绑定对象

**反射机制优缺点**

- **优点：** 运行期类型的判断，动态加载类，提高代码灵活度。
- **缺点：** 1,性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的 java 代码要慢很多。2,安全问题，让我们可以动态操作改变类的属性同时也增加了类的安全隐患。

**反射的应用场景**

反射是框架设计的灵魂。

在我们平时的项目开发过程中，基本上很少会直接使用到反射机制，但这不能说明反射机制没有用，实际上有很多设计、开发都与反射机制有关，例如模块化的开发，通过反射去调用对应的字节码；动态代理设计模式也采用了反射机制，还有我们日常使用的 Spring／Hibernate 等框架也大量使用到了反射机制。



##  12. 服务端的通信逻辑

### 12.1 构造器，初始化

```
/**
 * 服务端构造器
 * @param host 地址
 * @param port 端口
 * @param serializer 序列化器
 * serviceRegistry 服务注册中心
 * serviceProvider 本地注册表
 */
public WeRpcNettyServer(String host, int port, Integer serializer) {
    this.host = host;
    this.port = port;
    serviceRegistry = new NacosServiceRegistry(); 
    serviceProvider = new ServiceProviderImpl();
    this.serializer = CommonSerializer.getByCode(serializer);
    //扫描服务并完成注册，参考 #19
    scanServices();
}
```

### 12.2 启动方法

```
/**
 * 启动
 */
@Override
public void start() {
    ShutdownHook.getShutdownHook().addClearAllHook();
    //接收请求的线程池
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    //处理请求的线程池
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        //启动类
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .handler(new LoggingHandler(LogLevel.INFO))
                .option(ChannelOption.SO_BACKLOG, 256)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .childOption(ChannelOption.TCP_NODELAY, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ChannelPipeline pipeline = ch.pipeline();
                        //加入心跳处理器、自定义编码器、解码器、服务处理器
                        pipeline.addLast(new IdleStateHandler(30, 0, 0, TimeUnit.SECONDS))
                                .addLast(new CommonEncoder(serializer))
                                .addLast(new CommonDecoder())
                                .addLast(new NettyServerHandler());
                    }
                });
        //绑定地址、端口
        ChannelFuture future = serverBootstrap.bind(host, port).sync();
        future.channel().closeFuture().sync();

    } catch (InterruptedException e) {
        logger.error("启动服务器时有错误发生: ", e);
    } finally {
        bossGroup.shutdownGracefully();
        workerGroup.shutdownGracefully();
    }
}
```

### 12.3 服务处理器 NettyServerHandler 与 RequestHandler

 **NettyServerHandler**

```
/**
 * Netty中处理RpcRequest的Handler
 */
public class NettyServerHandler extends SimpleChannelInboundHandler<WeRpcRequest> {
    private static final Logger logger = LoggerFactory.getLogger(NettyServerHandler.class);
    private final RequestHandler requestHandler;

    public NettyServerHandler() {
        //使用RequestHandler(单例)
        this.requestHandler = SingletonFactory.getInstance(RequestHandler.class);
    }
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, WeRpcRequest msg) throws Exception {
        try {
        	
            if(msg.getHeartBeat()) {
                logger.info("接收到客户端心跳包...");
                return;
            }
            logger.info("服务器接收到请求: {}", msg);
            Object result = requestHandler.handle(msg);
            if (ctx.channel().isActive() && ctx.channel().isWritable()) {
            	//响应调用结果
                ctx.writeAndFlush(WeRpcResponse.success(result, msg.getRequestId()));
            } else {
                logger.error("通道不可写");
            }
        } finally {
            ReferenceCountUtil.release(msg);
        }
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        logger.error("处理过程调用时有错误发生:");
        cause.printStackTrace();
        ctx.close();
    }
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleState state = ((IdleStateEvent) evt).state();
            if (state == IdleState.READER_IDLE) {
                logger.info("长时间未收到心跳包，断开连接...");
                ctx.close();
            }
        } else {
            super.userEventTriggered(ctx, evt);
        }
    }
}
```

 **RequestHandler**

```
/**
 * 进行过程调用的请求处理器
 */
public class RequestHandler {

    private static final Logger logger = LoggerFactory.getLogger(RequestHandler.class);
    private static final ServiceProvider serviceProvider;

    static {
        serviceProvider = new ServiceProviderImpl();
    }

    /**
     * 调用 invokeTargetMethod 方法处理请求
     * @param weRpcRequest
     * @return
     */
    public Object handle(WeRpcRequest weRpcRequest) {
        //从本地服务表中获取服务类对象
        Object service = serviceProvider.getServiceProvider(weRpcRequest.getInterfaceName());
        return invokeTargetMethod(weRpcRequest, service);
    }

    private Object invokeTargetMethod(WeRpcRequest weRpcRequest, Object service) {
        Object result;
        try {
            //传过来的方法名和参数，构造Method对象
            Method method = service.getClass().getMethod(weRpcRequest.getMethodName(), weRpcRequest.getParamTypes());
            //调用invoke执行方法调用
            result = method.invoke(service, weRpcRequest.getParameters());
            logger.info("服务:{} 成功调用方法:{}", weRpcRequest.getInterfaceName(), weRpcRequest.getMethodName());
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            //调用失败
            return WeRpcResponse.fail(ResponseCode.METHOD_NOT_FOUND, weRpcRequest.getRequestId());
        }
        //返回调用结果
        return result;
    }

}
```

## 13.编码器与解码器(解决TCP粘包问题)

### 13.1 自定义协议格式

```
+---------------+---------------+-----------------+-------------+
|  Magic Number |  Package Type | Serializer Type | Data Length |
|    4 bytes    |    4 bytes    |     4 bytes     |   4 bytes   |
+---------------+---------------+-----------------+-------------+
|                          Data Bytes                           |
|                   Length: ${Data Length}                      |
+---------------------------------------------------------------+
```

| 字段            | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| Magic Number    | 魔数，表识一个 MRF 协议包，0xCAFEBABE                        |
| Package Type    | 包类型，标明这是一个调用请求还是调用响应                     |
| Serializer Type | 序列化器类型，标明这个包的数据的序列化方式                   |
| Data Length     | 数据字节的长度                                               |
| Data Bytes      | 传输的对象，通常是一个`RpcRequest`或`RpcClient`对象，取决于`Package Type`字段，对象的序列化方式取决于`Serializer Type`字段。 |

### 13.2 CommonEncoder

```
/**
 * 通用的编码拦截器
 */
public class CommonEncoder extends MessageToByteEncoder {

    private static final int MAGIC_NUMBER = 0xCAFEBABE;

    private final CommonSerializer serializer;

    public CommonEncoder(CommonSerializer serializer) {
        this.serializer = serializer;
    }

    @Override
    protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
        //魔数
        out.writeInt(MAGIC_NUMBER);
        //消息类型
        if (msg instanceof WeRpcRequest) {
            out.writeInt(PackageType.REQUEST_PACK.getCode());
        } else {
            out.writeInt(PackageType.RESPONSE_PACK.getCode());
        }
        //序列化类型
        out.writeInt(serializer.getCode());
        byte[] bytes = serializer.serialize(msg);
        //序列化后的消息长度
        out.writeInt(bytes.length);
        //序列化后的消息体
        out.writeBytes(bytes);
    }
}
```

### 13.3 CommonDecoder

```
/**
 * 通用的解码拦截器
 */
public class CommonDecoder extends ReplayingDecoder {

    private static final Logger logger = LoggerFactory.getLogger(CommonDecoder.class);
    private static final int MAGIC_NUMBER = 0xCAFEBABE;

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        //读四个字节的魔数
        int magic = in.readInt();
        //魔数校验
        if (magic != MAGIC_NUMBER) {
            logger.error("不识别的协议包: {}", magic);
            throw new WeRpcException(WeRpcError.UNKNOWN_PROTOCOL);
        }
        //读四个字节的消息包类型
        int packageCode = in.readInt();
        Class<?> packageClass;
        //消息类型校验
        if (packageCode == PackageType.REQUEST_PACK.getCode()) {
            packageClass = WeRpcRequest.class;
        } else if (packageCode == PackageType.RESPONSE_PACK.getCode()) {
            packageClass = WeRpcResponse.class;
        } else {
            logger.error("不识别的数据包: {}", packageCode);
            throw new WeRpcException(WeRpcError.UNKNOWN_PACKAGE_TYPE);
        }
        //读四个字节的序列化类型
        int serializerCode = in.readInt();
        CommonSerializer serializer = CommonSerializer.getByCode(serializerCode);
        if (serializer == null) {
            logger.error("不识别的反序列化器: {}", serializerCode);
            throw new WeRpcException(WeRpcError.UNKNOWN_SERIALIZER);
        }
        //读四个字节的消息长度
        int length = in.readInt();
        byte[] bytes = new byte[length];
        //读定长的消息体
        in.readBytes(bytes);
        //进行反序列化
        Object obj = serializer.deserialize(bytes, packageClass);
        out.add(obj);
    }

}
```

## 14. 单例工厂与线程池工厂

### 14.1 单例工厂

```
/**
 * 单例工厂
 */
public class SingletonFactory {
    private static Map<Class, Object> objectMap = new HashMap<>();

    private SingletonFactory() {}

    public static <T> T getInstance(Class<T> clazz) {
        Object instance = objectMap.get(clazz);
        synchronized (clazz) {
            if(instance == null) {
                try {
                    instance = clazz.newInstance();
                    objectMap.put(clazz, instance);
                } catch (IllegalAccessException | InstantiationException e) {
                    throw new RuntimeException(e.getMessage(), e);
                }
            }
        }
        return clazz.cast(instance);
    }
}
```

### 14.2 线程池工厂

```
/**
 * 通过 threadNamePrefix 来区分不同线程池（我们可以把相同 threadNamePrefix 的线程池看作是为同一业务场景服务）。
 * key: threadNamePrefix
 * value: threadPool
 */
public class ThreadPoolFactory {
    /**
     * 线程池参数
     */
    private static final int CORE_POOL_SIZE = 10;
    private static final int MAXIMUM_POOL_SIZE_SIZE = 100;
    private static final int KEEP_ALIVE_TIME = 1;
    private static final int BLOCKING_QUEUE_CAPACITY = 100;

    private final static Logger logger = LoggerFactory.getLogger(ThreadPoolFactory.class);

    private static Map<String, ExecutorService> threadPollsMap = new ConcurrentHashMap<>();

    private ThreadPoolFactory() {
    }

    public static ExecutorService createDefaultThreadPool(String threadNamePrefix) {
        return createDefaultThreadPool(threadNamePrefix, false);
    }

    public static ExecutorService createDefaultThreadPool(String threadNamePrefix, Boolean daemon) {
        ExecutorService pool = threadPollsMap.computeIfAbsent(threadNamePrefix, k -> createThreadPool(threadNamePrefix, daemon));
        if (pool.isShutdown() || pool.isTerminated()) {
            threadPollsMap.remove(threadNamePrefix);
            pool = createThreadPool(threadNamePrefix, daemon);
            threadPollsMap.put(threadNamePrefix, pool);
        }
        return pool;

    }

    public static void shutDownAll() {
        logger.info("关闭所有线程池...");
        threadPollsMap.entrySet().parallelStream().forEach(entry -> {
            ExecutorService executorService = entry.getValue();
            executorService.shutdown();
            logger.info("关闭线程池 [{}] [{}]", entry.getKey(), executorService.isTerminated());
            try {
                executorService.awaitTermination(10, TimeUnit.SECONDS);
            } catch (InterruptedException ie) {
                logger.error("关闭线程池失败！");
                executorService.shutdownNow();
            }
        });
    }

    private static ExecutorService createThreadPool(String threadNamePrefix, Boolean daemon) {
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(BLOCKING_QUEUE_CAPACITY);
        ThreadFactory threadFactory = createThreadFactory(threadNamePrefix, daemon);
        return new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE_SIZE, KEEP_ALIVE_TIME, TimeUnit.MINUTES, workQueue, threadFactory);
    }


    /**
     * 创建 ThreadFactory 。如果threadNamePrefix不为空则使用自建ThreadFactory，否则使用defaultThreadFactory
     *
     * @param threadNamePrefix 作为创建的线程名字的前缀
     * @param daemon           指定是否为 Daemon Thread(守护线程)
     * @return ThreadFactory
     */
    private static ThreadFactory createThreadFactory(String threadNamePrefix, Boolean daemon) {
        if (threadNamePrefix != null) {
            if (daemon != null) {
                return new ThreadFactoryBuilder().setNameFormat(threadNamePrefix + "-%d").setDaemon(daemon).build();
            } else {
                return new ThreadFactoryBuilder().setNameFormat(threadNamePrefix + "-%d").build();
            }
        }

        return Executors.defaultThreadFactory();
    }
}
```

## 15.  Netty相关

### 15.1 Netty简单介绍

Netty 是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。Netty 基于 NIO 的，封装了 JDK 的 NIO，让我们使用起来更加方法灵活。
特点和优势：
使用简单：封装了 NIO 的很多细节，使用更简单。
功能强大：预置了多种编解码功能，支持多种主流协议。
定制能力强：可以通过 ChannelHandler 对通信框架进行灵活地扩展。
性能高：通过与其他业界主流的 NIO 框架对比，Netty 的综合性能最优。

### 15.2 为什么 Netty 性能高

IO 线程模型：同步非阻塞，用最少的资源做更多的事。
内存零拷贝：尽量减少不必要的内存拷贝，实现了更高效率的传输。
内存池设计：申请的内存可以重用，主要指直接内存。内部实现是用一颗二叉查找树管理内存分配情况。
串行化处理读写：避免使用锁带来的性能开销。
高性能序列化协议：支持 protobuf 等高性能序列化协议。

### 15.3 简单说下 BIO、NIO 和 AIO

BIO：一个连接一个线程，客户端有连接请求时服务器端就需要启动一个线程进行处理。线程开销大。

伪异步IO：将请求连接放入线程池，一对多，但线程还是很宝贵的资源。

NIO：一个请求一个线程，但客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。

AIO：一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。

**阻塞和非阻塞I/O区别？**

- 如果内核缓冲没有数据可读时，read()系统调用会一直等待有数据到来后才从阻塞态中返回，这就是阻塞I/O。
- 非阻塞I/O在遇到上述情况时会立即返回给用户态进程一个返回值，并设置erro为EAGAIN。
- 对于往缓冲区写的操作同理。

**同步和异步区别**？

- 同步I/O指处理I/O操作的进程和处理I/O操作的进程是同一个。
- 异步I/O中I/O操作由操作系统完成，并不由产生I/O的用户进程执行。

**Reactor和Proactor区别？**

- Reactor模式已经是同步I/O，处理I/O操作的依旧是产生I/O的程序；Proactor是异步I/O，产生I/O调用的用户进程不会等待I/O发生，具体I/O操作由操作系统完成。
- 异步I/O需要操作系统支持，Linux异步I/O为AIO，Windows为IOCP。

**epoll和select及poll区别？**

- 文件描述符数量限制：select文件描述符数量受到限制，最大为2048（FD_SETSIZE），可重编内核修改但治标不治本；poll没有最大文件描述符数量限制；epoll没有最大文件描述符数量限制。
- 检查机制：select和poll会以遍历方式（轮询机制）检查每一个文件描述符以确定是否有I/O就绪，每次执行时间会随着连接数量的增加而线性增长；epoll则每次返回后只对活跃的文件描述符队列进行操作（每个描述符都通过回调函数实现，只有活跃的描述符会调用回调函数并添加至队列中）。**当大量连接是非活跃连接时epoll相对于select和poll优势比较大，若大多为活跃连接则效率未必高（设计队列维护及红黑树创建）**
- 数据传递方式：select和poll需要将FD_SET在内核空间和用户空间来回拷贝；epoll则避免了不必要的数据拷贝。

**epoll中ET和LT模式的区别与实现原理？**

- LT：默认工作方式，同时支持阻塞I/O和非阻塞I/O，LT模式下，内核告知某一文件描述符读、写是否就绪了，然后你可以对这个就绪的文件描述符进行I/O操作。如果不作任何操作，内核还是会继续通知。这种模式编程出错误可能性较小但由于重复提醒，效率相对较低。传统的select、poll都是这种模型的代表。
- ET：高速工作方式（因为减少了epoll_wait触发次数），适合高并发，只支持非阻塞I/O，ET模式下，内核告知某一文件描述符读、写是否就绪了，然后他假设已经知道该文件描述符是否已经就绪，内核不会再为这个文件描述符发更多的就绪通知（epoll_wait不会返回），直到某些操作导致文件描述符状态不再就绪。

**ET模式下要注意什么（如何使用ET模式）？**

- 对于读操作，如果read没有一次读完buff数据，下一次将得不到就绪通知（ET特性），造成buff中数据无法读出，除非有新数据到达。
  - 解决方法：将套接字设置为非阻塞，用while循环包住read，只要buff中有数据，就一直读。一直读到产生EAGIN错误。
- 对于写操作主要因为ET模式下非阻塞需要我们考虑如何将用户要求写的数据写完。
  - 解决方法：只要buff还有空间且用户请求写的数据还未写完，就一直写。

- 文件描述符数量限制：select文件描述符数量受到限制，最大为2048（FD_SETSIZE），可重编内核修改但治标不治本；poll没有最大文件描述符数量限制；epoll没有最大文件描述符数量限制。
- 检查机制：select和poll会以遍历方式（轮询机制）检查每一个文件描述符以确定是否有I/O就绪，每次执行时间会随着连接数量的增加而线性增长；epoll则每次返回后只对活跃的文件描述符队列进行操作（每个描述符都通过回调函数实现，只有活跃的描述符会调用回调函数并添加至队列中）。**当大量连接是非活跃连接时epoll相对于select和poll优势比较大，若大多为活跃连接则效率未必高（设计队列维护及红黑树创建）**
- 数据传递方式：select和poll需要将FD_SET在内核空间和用户空间来回拷贝；epoll则避免了不必要的数据拷贝。



### 15.4 Netty的线程模型？

Netty 通过 Reactor 模型基于多路复用器接收并处理用户请求，内部实现了两个线程池， boss 线程池和 worker 线程池，其中 boss 线程池的线程负责处理请求的 accept 事件，当接收到 accept 事件的请求时，把对应的 socket 封装到一个 NioSocketChannel 中，并交给 worker 线程池，其中 worker 线程池负责请求的 read 和 write 事件，由对应的Handler 处理。
单线程模型：所有I/O操作都由一个线程完成，即多路复用、事件分发和处理都是在一个Reactor 线程上完成的。既要接收客户端的连接请求,向服务端发起连接，又要发送/读取请求或应答/响应消息。一个NIO 线程同时处理成百上千的链路，性能上无法支撑，速度慢，若线程进入死循环，整个程序不可用，对于高负载、大并发的应用场景不合适。
多线程模型：有一个NIO 线程（Acceptor） 只负责监听服务端，接收客户端的TCP 连接请求；NIO 线程池负责网络IO 的操作，即消息的读取、解码、编码和发送；1 个NIO 线程可以同时处理N 条链路，但是1 个链路只对应1 个NIO 线程，这是为了防止发生并发操作问题。但在并发百万客户端连接或需要安全认证时，一个Acceptor 线程可能会存在性能不足问题。
主从多线程模型：Acceptor 线程用于绑定监听端口，接收客户端连接，将SocketChannel 从主线程池的 Reactor 线程的多路复用器上移除，重新注册到Sub 线程池的线程上，用于处理I/O 的读写等操作，从而保证 mainReactor 只负责接入认证、握手等操作；

### 15.5 如何解决 TCP 的粘包拆包问题

TCP 是以流的方式来处理数据，一个完整的包可能会被 TCP 拆分成多个包进行发送，也可能把小的封装成一个大的数据包发送。
TCP 粘包/分包的原因：应用程序写入的字节大小大于套接字发送缓冲区的大小，会发生拆包现象，而应用程序写入数据小于套接字缓冲区大小，网卡将应用多次写入的数据发送到网络上，这将会发生粘包现象；

Netty 自带解决方式：
消息定长：FixedLengthFrameDecoder 类
包尾增加特殊字符分割：
行分隔符类：LineBasedFrameDecoder
自定义分隔符类 ：DelimiterBasedFrameDecoder
将消息分为消息头和消息体：LengthFieldBasedFrameDecoder 类。分为有头部的拆包与粘包、长度字段在前且有头部的拆包与粘包、多扩展头部的拆包与粘包。

框架解决方式：
自定义协议，其中有字段标明包长度。

**参考 [13.编码器与解码器 ](#13)**

### 15.6 说下 Netty 零拷贝

Netty 的零拷贝主要包含三个方面：

Netty 的接收和发送 ByteBuffer 采用 DIRECT BUFFERS，使用堆外直接内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。如果使用传统的堆内存（HEAP BUFFERS）进行 Socket 读写，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
Netty 提供了组合 Buffer 对象，可以聚合多个 ByteBuffer 对象，用户可以像操作一个 Buffer 那样方便的对组合 Buffer 进行操作，避免了传统通过内存拷贝的方式将几个小 Buffer 合并成一个大的 Buffer。
Netty 的文件传输采用了 transferTo 方法，它可以直接将文件缓冲区的数据发送到目标 Channel，避免了传统通过循环 write 方式导致的内存拷贝问题。



---

NIO的零拷贝

NIO的零拷贝由transferTo方法实现。transferTo方法将数据从FileChannel对象传送到可写的字节通道（如Socket Channel等）。在transferTo方法内部实现中，由native方法transferTo0来实现，它依赖底层操作系统的支持。在UNIX和Linux系统中，调用这个方法会引起sendfile()系统调用，实现了数据直接从内核的读缓冲区传输到套接字缓冲区，避免了用户态(User-space) 与内核态(Kernel-space) 之间的数据拷贝。

使用NIO零拷贝，流程简化为两步：

1. transferTo方法调用触发DMA引擎将文件上下文信息拷贝到内核读缓冲区，接着内核将数据从内核缓冲区拷贝到与套接字相关联的缓冲区。
2. DMA引擎将数据从内核套接字缓冲区传输到协议引擎（第三次数据拷贝）。

相比传统IO，使用NIO零拷贝后改进的地方：

1. 我们已经将上下文切换次数从4次减少到了2次；
2. 将数据拷贝次数从4次减少到了3次（其中只有1次涉及了CPU，另外2次是DMA直接存取）。

如果底层NIC（网络接口卡）支持gather操作，可以进一步减少内核中的数据拷贝。在Linux 2.4以及更高版本的内核中，socket缓冲区描述符已被修改用来适应这个需求。这种方式不但减少上下文切换，同时消除了需要CPU参与的重复的数据拷贝。

用户这边的使用方式不变，依旧通过transferTo方法，但是方法的内部实现发生了变化：

1. transferTo方法调用触发DMA引擎将文件上下文信息拷贝到内核缓冲区。
2. 数据不会被拷贝到套接字缓冲区，只有数据的描述符（包括数据位置和长度）被拷贝到套接字缓冲区。DMA 引擎直接将数据从内核缓冲区拷贝到协议引擎，这样减少了最后一次需要消耗CPU的拷贝操作。

NIO零拷贝适用于以下场景：

1. 文件较大，读写较慢，追求速度
2. JVM内存不足，不能加载太大数据
3. 内存带宽不够，即存在其他程序或线程存在大量的IO操作，导致带宽本来就小



### 15.7 简单说下 Netty 中的重要组件

Channel：Netty 网络操作抽象类，它除了包括基本的 I/O 操作，如 bind、connect、read、write 等。
EventLoop：主要是配合 Channel 处理 I/O 操作，用来处理连接的生命周期中所发生的事情。
ChannelFuture：Netty 框架中所有的 I/O 操作都为异步的，因此我们需要 ChannelFuture 的 addListener()注册一个 ChannelFutureListener 监听事件，当操作执行成功或者失败时，监听就会自动触发返回结果。
ChannelHandler：充当了所有处理入站和出站数据的逻辑容器。ChannelHandler 主要用来处理各种事件，这里的事件很广泛，比如可以是连接、数据接收、异常、数据转换等。
ChannelPipeline：为 ChannelHandler 链提供了容器，当 channel 创建时，就会被自动分配到它专属的 ChannelPipeline，这个关联是永久性的。

### 15.8 Netty 中责任链

首先说明责任链模式：
适用场景:
对于一个请求来说,如果有个对象都有机会处理它,而且不明确到底是哪个对象会处理请求时,我们可以考虑使用责任链模式实现它,让请求从链的头部往后移动,直到链上的一个节点成功处理了它为止
优点:
发送者不需要知道自己发送的这个请求到底会被哪个对象处理掉,实现了发送者和接受者的解耦
简化了发送者对象的设计
可以动态的添加节点和删除节点
缺点:
所有的请求都从链的头部开始遍历,对性能有损耗
极差的情况,不保证请求一定会被处理

Netty的责任链：

netty 的 pipeline 设计,就采用了责任链设计模式, 底层采用双向链表的数据结构, 将链上的各个处理器串联起来

客户端每一个请求的到来，netty 都认为，pipeline 中的所有的处理器都有机会处理它，因此，对于入栈的请求，全部从头节点开始往后传播，一直传播到尾节点（来到尾节点的 msg 会被释放掉）。

责任终止机制：
在pipeline中的任意一个节点，只要我们不手动的往下传播下去，这个事件就会终止传播在当前节点
对于入站数据，默认会传递到尾节点，进行回收，如果我们不进行下一步传播，事件就会终止在当前节点

### 15.9 Netty 是如何保持长连接的（心跳）

首先 TCP 协议的实现中也提供了 keepalive 报文用来探测对端是否可用。TCP 层将在定时时间到后发送相应的 KeepAlive 探针以确定连接可用性。

ChannelOption.SO_KEEPALIVE, true 表示打开 TCP 的 keepAlive 设置。

TCP 心跳的问题：

考虑一种情况，某台服务器因为某些原因导致负载超高，CPU 100%，无法响应任何业务请求，但是使用 TCP 探针则仍旧能够确定连接状态，这就是典型的连接活着但业务提供方已死的状态，对客户端而言，这时的最好选择就是断线后重新连接其他服务器，而不是一直认为当前服务器是可用状态一直向当前服务器发送些必然会失败的请求。

Netty 中提供了 IdleStateHandler 类专门用于处理心跳。

IdleStateHandler 的构造函数如下：

```
public IdleStateHandler(long readerIdleTime, long writerIdleTime,
                        long allIdleTime,TimeUnit unit){
}
```


第一个参数是隔多久检查一下读事件是否发生，如果 channelRead() 方法超过 readerIdleTime 时间未被调用则会触发超时事件调用 userEventTrigger() 方法；

第二个参数是隔多久检查一下写事件是否发生，writerIdleTime 写空闲超时时间设定，如果 write() 方法超过 writerIdleTime 时间未被调用则会触发超时事件调用 userEventTrigger() 方法；

第三个参数是全能型参数，隔多久检查读写事件；

第四个参数表示当前的时间单位。

所以这里可以分别控制读，写，读写超时的时间，单位为秒，如果是0表示不检测，所以如果全是0，则相当于没添加这个 IdleStateHandler，连接是个普通的短连接。

**项目中的设置：**

```
new IdleStateHandler(30, 0, 0, TimeUnit.SECONDS); 
```

### 15.10 详解IO

**最基本的socket模型**
要想客户端和服务器能在网络中通信，那必须得使用 Socket  编程，它是进程间通信里比较特别的方式，特别之处在于它是可以跨主机间通信。

Socket  的中文名叫作插口，咋一看还挺迷惑的。事实上，双方要进行网络通信前，各自得创建一个 Socket，这相当于客户端和服务器都开了一个“口子”，双方读取和发送数据的时候，都通过这个“口子”。这样一看，是不是觉得很像弄了一根网线，一头插在客户端，一头插在服务端，然后进行通信。

创建 Socket 的时候，可以指定网络层使用的是 IPv4 还是 IPv6，传输层使用的是 TCP 还是 UDP。

UDP 的 Socket 编程相对简单些，这里我们只介绍基于 TCP 的 Socket 编程。

服务器的程序要先跑起来，然后等待客户端的连接和数据，我们先来看看服务端的 Socket 编程过程是怎样的。

服务端首先调用 `socket()` 函数，创建网络协议为 IPv4，以及传输协议为 TCP 的 Socket ，接着调用 `bind()` 函数，给这个 Socket 绑定一个 **IP 地址和端口**。
**绑定这两个的目的是什么？**

- 绑定端口的目的：当内核收到 TCP 报文，通过 TCP 头里面的端口号，来找到我们的应用程序，然后把数据传递给我们。
- 绑定 IP 地址的目的：一台机器是可以有多个网卡的，每个网卡都有对应的 IP 地址，当绑定一个网卡时，内核在收到该网卡上的包，才会发给我们；

绑定完 IP 地址和端口后，就可以调用 `listen()` 函数进行监听，此时对应 TCP 状态图中的 `listen`，如果我们要判定服务器中一个网络程序有没有启动，可以通过 `netstate` 命令查看对应的端口号是否有被监听。

服务端进入了监听状态后，通过调用 `accept()` 函数，来从内核获取客户端的连接，如果没有客户端连接，则会阻塞等待客户端连接的到来。

**那客户端是怎么发起连接的呢？**

客户端在创建好 Socket 后，调用 `connect()` 函数发起连接，该函数的参数要指明服务端的 IP 地址和端口号，然后万众期待的 TCP 三次握手就开始了。

在  TCP 连接的过程中，服务器的内核实际上为每个 Socket 维护了两个队列：

- 一个是还没完全建立连接的队列，称为 **TCP 半连接队列**，这个队列都是没有完成三次握手的连接，此时服务端处于 `syn_rcvd` 的状态；
- 一个是一件建立连接的队列，称为 **TCP 全连接队列**，这个队列都是完成了三次握手的连接，此时服务端处于 `established` 状态；

当 TCP 全连接队列不为空后，服务端的 `accept()` 函数，就会从内核中的 TCP 全连接队列里拿出一个已经完成连接的  Socket 返回应用程序，后续数据传输都用这个 Socket。

注意，监听的 Socket 和真正用来传数据的 Socket 是两个：

- 一个叫作**监听 Socket**；
- 一个叫作**已连接 Socket**；

连接建立后，客户端和服务端就开始相互传输数据了，双方都可以通过 `read()` 和 `write()` 函数来读写数据。

至此， TCP 协议的 Socket 程序的调用过程就结束了。

>读写 Socket  的方式，好像读写文件一样。基于 Linux 一切皆文件的理念，在内核中 Socket 也是以「文件」的形式存在的，也是有对应的文件描述符。
>
>**文件描述符的作用是什么？**每一个进程都有一个数据结构 `task_struct`，该结构体里有一个指向「文件描述符数组」的成员指针。该数组里列出这个进程打开的所有文件的文件描述符。数组的下标是文件描述符，是一个整数，而数组的内容是一个指针，指向内核中所有打开的文件的列表，也就是说内核可以通过文件描述符找到对应打开的文件。
>
>然后每个文件都有一个 inode，Socket 文件的 inode 指向了内核中的 Socket 结构，在这个结构体里有两个队列，分别是**发送队列**和**接收队列**，这个两个队列里面保存的是一个个 `struct sk_buff`，用链表的组织形式串起来。
>
>sk_buff 可以表示各个层的数据包，在应用层数据包叫 data，在 TCP 层我们称为 segment，在 IP 层我们叫 packet，在数据链路层称为 frame。
>
>**为什么全部数据包只用一个结构体来描述呢？**协议栈采用的是分层结构，上层向下层传递数据时需要增加包头，下层向上层数据时又需要去掉包头，如果每一层都用一个结构体，那在层之间传递数据的时候，就要发生多次拷贝，这将大大降低 CPU 效率。
>
>于是，为了在层级之间传递数据时，不发生拷贝，只用 sk_buff 一个结构体来描述所有的网络包，那它是如何做到的呢？是通过调整 sk_buff 中 `data` 的指针，比如：
>
>- 当接收报文时，从网卡驱动开始，通过协议栈层层往上传送数据报，通过增加 skb->data 的值，来逐步剥离协议首部。
>- 当要发送报文时，创建 sk_buff 结构体，数据缓存区的头部预留足够的空间，用来填充各层首部，在经过各下层协议时，通过减少 skb->data 的值来增加协议首部。

**如何服务更多用户**

前面提到的 TCP Socket 调用流程是最简单、最基本的，它基本只能一对一通信，因为使用的是同步阻塞的方式，当服务端在还没处理完一个客户端的网络 I/O 时，或者 读写操作发生阻塞时，其他客户端是无法与服务端连接的。

可如果我们服务器只能服务一个客户，那这样就太浪费资源了，于是我们要改进这个网络 I/O 模型，以支持更多的客户端。

在改进网络 I/O 模型前，我先来提一个问题，你知道服务器单机理论最大能连接多少个客户端？

相信你知道 TCP 连接是由四元组唯一确认的，这个四元组就是：**本机IP, 本机端口, 对端IP, 对端端口**。

服务器作为服务方，通常会在本地固定监听一个端口，等待客户端的连接。因此服务器的本地 IP 和端口是固定的，于是对于服务端 TCP 连接的四元组只有对端 IP 和端口是会变化的，所以**最大 TCP 连接数 = 客户端 IP 数×客户端端口数**。

对于 IPv4，客户端的 IP 数最多为 2 的 32 次方，客户端的端口数最多为 2 的 16 次方，也就是**服务端单机最大 TCP 连接数约为 2 的 48 次方**。

这个理论值相当“丰满”，但是服务器肯定承载不了那么大的连接数，主要会受两个方面的限制：

- **文件描述符**，Socket 实际上是一个文件，也就会对应一个文件描述符。在 Linux 下，单个进程打开的文件描述符数是有限制的，没有经过修改的值一般都是 1024，不过我们可以通过 ulimit 增大文件描述符的数目；
- **系统内存**，每个 TCP 连接在内核中都有对应的数据结构，意味着每个连接都是会占用一定内存的；

那如果服务器的内存只有 2 GB，网卡是千兆的，能支持并发 1 万请求吗？

并发 1 万请求，也就是经典的 C10K 问题 ，C 是 Client 单词首字母缩写，C10K 就是单机同时处理 1 万个请求的问题。

从硬件资源角度看，对于 2GB 内存千兆网卡的服务器，如果每个请求处理占用不到 200KB 的内存和 100Kbit 的网络带宽就可以满足并发 1 万个请求。

不过，要想真正实现 C10K 的服务器，要考虑的地方在于服务器的网络 I/O 模型，效率低的模型，会加重系统开销，从而会离 C10K 的目标越来越远。

**多进程模型**
基于最原始的阻塞网络 I/O， 如果服务器要支持多个客户端，其中比较传统的方式，就是使用**多进程模型**，也就是为每个客户端分配一个进程来处理请求。

服务器的主进程负责监听客户的连接，一旦与客户端连接完成，accept() 函数就会返回一个「已连接 Socket」，这时就通过 `fork()` 函数创建一个子进程，实际上就把父进程所有相关的东西都**复制**一份，包括文件描述符、内存地址空间、程序计数器、执行的代码等。

这两个进程刚复制完的时候，几乎一摸一样。不过，会根据**返回值**来区分是父进程还是子进程，如果返回值是 0，则是子进程；如果返回值是其他的整数，就是父进程。

正因为子进程会**复制父进程的文件描述符**，于是就可以直接使用「已连接 Socket 」和客户端通信了，

可以发现，子进程不需要关心「监听 Socket」，只需要关心「已连接 Socket」；父进程则相反，将客户服务交给子进程来处理，因此父进程不需要关心「已连接 Socket」，只需要关心「监听 Socket」。

另外，当「子进程」退出时，实际上内核里还会保留该进程的一些信息，也是会占用内存的，如果不做好“回收”工作，就会变成**僵尸进程**，随着僵尸进程越多，会慢慢耗尽我们的系统资源。

因此，父进程要“善后”好自己的孩子，怎么善后呢？那么有两种方式可以在子进程退出后回收资源，分别是调用 `wait()` 和 `waitpid()` 函数。

这种用多个进程来应付多个客户端的方式，在应对 100 个客户端还是可行的，但是当客户端数量高达一万时，肯定扛不住的，因为每产生一个进程，必会占据一定的系统资源，而且进程间上下文切换的“包袱”是很重的，性能会大打折扣。

进程的上下文切换不仅包含了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的资源。

**多线程模型**

既然进程间上下文切换的“包袱”很重，那我们就搞个比较轻量级的模型来应对多用户的请求 —— **多线程模型**。

线程是运行在进程中的一个“逻辑流”，单进程中可以运行多个线程，同进程里的线程可以共享进程的部分资源的，比如文件描述符列表、进程空间、代码、全局数据、堆、共享库等，这些共享些资源在上下文切换时是不需要切换，而只需要切换线程的私有数据、寄存器等不共享的数据，因此同一个进程下的线程上下文切换的开销要比进程小得多。

当服务器与客户端 TCP 完成连接后，通过 `pthread_create()` 函数创建线程，然后将「已连接 Socket」的文件描述符传递给线程函数，接着在线程里和客户端进行通信，从而达到并发处理的目的。

如果每来一个连接就创建一个线程，线程运行完后，还得操作系统还得销毁线程，虽说线程切换的上写文开销不大，但是如果频繁创建和销毁线程，系统开销也是不小的。

那么，我们可以使用**线程池**的方式来避免线程的频繁创建和销毁，所谓的线程池，就是提前创建若干个线程，这样当由新连接建立时，将这个已连接的 Socket 放入到一个队列里，然后线程池里的线程负责从队列中取出已连接 Socket 进程处理。需要注意的是，这个队列是全局的，每个线程都会操作，为了避免多线程竞争，线程在操作这个队列前要加锁。

上面基于进程或者线程模型的，其实还是有问题的。新到来一个 TCP 连接，就需要分配一个进程或者线程，那么如果要达到 C10K，意味着要一台机器维护 1 万个连接，相当于要维护 1 万个进程/线程，操作系统就算死扛也是扛不住的。

**IO多路复用**
既然为每个请求分配一个进程/线程的方式不合适，那有没有可能只使用一个进程来维护多个 Socket 呢？答案是有的，那就是 **I/O 多路复用**技术。一个进程虽然任一时刻只能处理一个请求，但是处理每个请求的事件时，耗时控制在 1 毫秒以内，这样 1 秒内就可以处理上千个请求，把时间拉长来看，多个请求复用了一个进程，这就是多路复用，这种思想很类似一个 CPU 并发多个进程，所以也叫做时分多路复用。

我们熟悉的 select/poll/epoll 内核提供给用户态的多路复用系统调用，**进程可以通过一个系统调用函数从内核中获取多个事件**。

**select/poll/epoll 是如何获取网络事件的呢？**在获取事件时，先把所有连接（文件描述符）传给内核，再由内核返回产生了事件的连接，然后在用户态中再处理这些连接对应的请求即可。

select/poll/epoll 这是三个多路复用接口，都能实现 C10K 吗？接下来，我们分别说说它们。

**select/poll**
select 实现多路复用的方式是，将已连接的 Socket 都放到一个**文件描述符集合**，然后调用 select 函数将文件描述符集合**拷贝**到内核里，让内核来检查是否有网络事件产生，检查的方式很粗暴，就是通过**遍历**文件描述符集合的方式，当检查到有事件产生后，将此 Socket 标记为可读或可写， 接着再把整个文件描述符集合**拷贝**回用户态里，然后用户态还需要再通过**遍历**的方法找到可读或可写的 Socket，然后再对其处理。

所以，对于 select 这种方式，需要进行 **2 次「遍历」文件描述符集合**，一次是在内核态里，一个次是在用户态里 ，而且还会发生 **2 次「拷贝」文件描述符集合**，先从用户空间传入内核空间，由内核修改后，再传出到用户空间中。

select 使用固定长度的 BitsMap，表示文件描述符集合，而且所支持的文件描述符的个数是有限制的，在 Linux 系统中，由内核中的 FD_SETSIZE 限制， 默认最大值为 `1024`，只能监听 0~1023 的文件描述符。

poll 不再用 BitsMap 来存储所关注的文件描述符，取而代之用动态数组，以链表形式来组织，突破了 select 的文件描述符个数限制，当然还会受到系统文件描述符限制。

但是 poll 和 select 并没有太大的本质区别，**都是使用「线性结构」存储进程关注的 Socket 集合，因此都需要遍历文件描述符集合来找到可读或可写的 Socket，时间复杂度为 O(n)，而且也需要在用户态与内核态之间拷贝文件描述符集合**，这种方式随着并发数上来，性能的损耗会呈指数级增长。

**epoll**epoll 通过两个方面，很好解决了 select/poll 的问题。

*第一点*，epoll 在内核里使用**红黑树来跟踪进程所有待检测的文件描述字**，把需要监控的 socket 通过 `epoll_ctl()` 函数加入内核中的红黑树里，红黑树是个高效的数据结构，增删查一般时间复杂度是 `O(logn)`，通过对这棵黑红树进行操作，这样就不需要像 select/poll 每次操作时都传入整个 socket 集合，只需要传入一个待检测的 socket，减少了内核和用户空间大量的数据拷贝和内存分配。

*第二点*， epoll 使用事件驱动的机制，内核里**维护了一个链表来记录就绪事件**，当某个 socket 有事件发生时，通过回调函数内核会将其加入到这个就绪事件列表中，当用户调用 `epoll_wait()` 函数时，只会返回有事件发生的文件描述符的个数，不需要像 select/poll 那样轮询扫描整个 socket 集合，大大提高了检测的效率。epoll 的方式即使监听的 Socket 数量越多的时候，效率不会大幅度降低，能够同时监听的 Socket 的数目也非常的多了，上限就为系统定义的进程打开的最大文件描述符个数。因而，**epoll 被称为解决 C10K 问题的利器**。

插个题外话，网上文章不少说，`epoll_wait` 返回时，对于就绪的事件，epoll使用的是共享内存的方式，即用户态和内核态都指向了就绪链表，所以就避免了内存拷贝消耗。

这是错的！看过 epoll 内核源码的都知道，**压根就没有使用共享内存这个玩意**。你可以从下面这份代码看到， epoll_wait 实现的内核代码中调用了 `__put_user` 函数，这个函数就是将数据从内核拷贝到用户空间。epoll 支持两种事件触发模式，分别是**边缘触发（\*edge-triggered，ET\*）**和**水平触发（\*level-triggered，LT\*）**。

这两个术语还挺抽象的，其实它们的区别还是很好理解的。

- 使用边缘触发模式时，当被监控的 Socket 描述符上有可读事件发生时，**服务器端只会从 epoll_wait 中苏醒一次**，即使进程没有调用 read 函数从内核读取数据，也依然只苏醒一次，因此我们程序要保证一次性将内核缓冲区的数据读取完；
- 使用水平触发模式时，当被监控的 Socket 上有可读事件发生时，**服务器端不断地从 epoll_wait 中苏醒，直到内核缓冲区数据被 read 函数读完才结束**，目的是告诉我们有数据需要读取；

举个例子，你的快递被放到了一个快递箱里，如果快递箱只会通过短信通知你一次，即使你一直没有去取，它也不会再发送第二条短信提醒你，这个方式就是边缘触发；如果快递箱发现你的快递没有被取出，它就会不停地发短信通知你，直到你取出了快递，它才消停，这个就是水平触发的方式。

这就是两者的区别，水平触发的意思是只要满足事件的条件，比如内核中有数据需要读，就一直不断地把这个事件传递给用户；而边缘触发的意思是只有第一次满足条件的时候才触发，之后就不会再传递同样的事件了。

如果使用水平触发模式，当内核通知文件描述符可读写时，接下来还可以继续去检测它的状态，看它是否依然可读或可写。所以在收到通知后，没必要一次执行尽可能多的读写操作。

如果使用边缘触发模式，I/O 事件发生时只会通知一次，而且我们不知道到底能读写多少数据，所以在收到通知后应尽可能地读写数据，以免错失读写的机会。因此，我们会**循环**从文件描述符读写数据，那么如果文件描述符是阻塞的，没有数据可读写时，进程会阻塞在读写函数那里，程序就没办法继续往下执行。所以，**边缘触发模式一般和非阻塞 I/O 搭配使用**，程序会一直执行 I/O 操作，直到系统调用（如 `read` 和 `write`）返回错误，错误类型为 `EAGAIN` 或 `EWOULDBLOCK`。

一般来说，边缘触发的效率比水平触发的效率要高，因为边缘触发可以减少 epoll_wait 的系统调用次数，系统调用也是有一定的开销的的，毕竟也存在上下文的切换。select/poll 只有水平触发模式，epoll 默认的触发模式是水平触发，但是可以根据应用场景设置为边缘触发模式。

**另外，使用 I/O 多路复用时，最好搭配非阻塞 I/O 一起使用。简单点理解，就是多路复用 API 返回的事件并不一定可读写的**，如果使用阻塞 I/O， 那么在调用 read/write 时则会发生程序阻塞，因此最好搭配非阻塞 I/O，以便应对极少数的特殊情况。

**总结**
最基础的 TCP 的 Socket 编程，它是阻塞 I/O 模型，基本上只能一对一通信，那为了服务更多的客户端，我们需要改进网络 I/O 模型。

比较传统的方式是使用多进程/线程模型，每来一个客户端连接，就分配一个进程/线程，然后后续的读写都在对应的进程/线程，这种方式处理 100 个客户端没问题，但是当客户端增大到 10000  个时，10000 个进程/线程的调度、上下文切换以及它们占用的内存，都会成为瓶颈。

为了解决上面这个问题，就出现了 I/O 的多路复用，可以只在一个进程里处理多个文件的  I/O，Linux 下有三种提供 I/O 多路复用的 API，分别是：select、poll、epoll。

select 和 poll 并没有本质区别，它们内部都是使用「线性结构」来存储进程关注的 Socket 集合。

在使用的时候，首先需要把关注的 Socket 集合通过 select/poll 系统调用从用户态拷贝到内核态，然后由内核检测事件，当有网络事件产生时，内核需要遍历进程关注 Socket 集合，找到对应的 Socket，并设置其状态为可读/可写，然后把整个 Socket 集合从内核态拷贝到用户态，用户态还要继续遍历整个 Socket 集合找到可读/可写的 Socket，然后对其处理。

很明显发现，select 和 poll 的缺陷在于，当客户端越多，也就是 Socket 集合越大，Socket 集合的遍历和拷贝会带来很大的开销，因此也很难应对 C10K。

epoll 是解决 C10K 问题的利器，通过两个方面解决了 select/poll 的问题。

- epoll 在内核里使用「红黑树」来关注进程所有待检测的 Socket，红黑树是个高效的数据结构，增删查一般时间复杂度是 O(logn)，通过对这棵黑红树的管理，不需要像 select/poll 在每次操作时都传入整个 Socket 集合，减少了内核和用户空间大量的数据拷贝和内存分配。
- epoll 使用事件驱动的机制，内核里维护了一个「链表」来记录就绪事件，只将有事件发生的 Socket 集合传递给应用程序，不需要像 select/poll 那样轮询扫描整个集合（包含有和无事件的 Socket ），大大提高了检测的效率。

而且，epoll 支持边缘触发和水平触发的方式，而 select/poll 只支持水平触发，一般而言，边缘触发的方式会比水平触发的效率高。



## 16.RPC与HTTP对比

1）HTTP的本质首先你要明确 HTTP 是一个协议，是一个超文本传输协议。

它基于 TCP/IP 来传输文本、图片、视频、音频等。HTTP 不提供数据包的传输功能，也就是数据包从浏览器到服务端再来回的传输和它没关系。这是 TCP/IP 干的。那 HTTP 有啥用？我们来分析一波。我们上网要么就是获取一些信息来看，要么就是修改一些信息。比如你用浏览器刷微博就是获取信息，发微博就是修改信息。所以说浏览器需要告知服务器它需要什么，这次的请求是要获取哪些信息？发怎么样的微博。这就涉及到浏览器和服务器之间的通信交互。而交互就需要一种格式。像你我之间的谈话就用中文，你要突然换成俄语我听不懂那不就 GG 了。所以说 HTTP 它规定了一种格式，一种通信格式，大家都用这个格式来交谈。这样不论你是什么服务器、什么浏览器都能顺利的交流，减少交互的成本。就像全世界如果都讲中文，那我们不就不需要学英文了，那不就较少交互的成本了。不像现在我们还得学英文，不然就看不懂文档等等。万一之后俄语又起来了，咱还得对接俄文，这交互成本是不是就上来了。而网络世界还好，咱们现在的 Web 交互基本上就是 HTTP 了。其实 HTTP 协议的格式很像我们信封，有个固定的格式。所以 HTTP 就规定了请求先搞请求行、再搞请求报头、再搞请求体。响应就状态行、响应报头、响应体。HTTP的本质就是客户端和服务端约定好的一种通信格式。

HTTP 和 RPC 其实是两个维度的东西， HTTP 指的是通信协议。而 RPC 则是远程调用，其对应的是本地调用。RPC 的通信可以用 HTTP 协议，也可以自定义协议，是不做约束的。像之前的单体时代，我们的 service 调用就是自己实现的方法，是本地进程内的调用。

2）那为什么要有 RPC？

可能你常听到什么什么之间是 RPC 调用的，那你有没有想过为什么要 RPC， 我们直接 WebClient HTTP 调用不行么？其实 RPC 调用是因为服务的拆分，或者本身公司内部的多个服务之间的通信。服务的拆分独立部署，那服务间的调用就必然需要网络通信，用 WebClient 调用当然可行，但是比较麻烦。我们想即使服务被拆分了但是使用起来还是和之前本地调用一样方便。所以就出现了 RPC 框架，来屏蔽这些底层调用细节，使得我们编码上还是和之前本地调用相差不多。并且 HTTP 协议比较的冗余，RPC 都是内部调用所以不需要太考虑通用性，只要公司内部保持格式统一即可。所以可以做各种定制化的协议来使得通信更高效。所以公司内部服务的调用一般都用 RPC，而 HTTP 的优势在于通用，大家都认可这个协议。所以三方平台提供的接口都是通过 HTTP 协议调用的。所以现在知道为什么我们调用第三方都是 HTTP ，公司内部用 RPC 了吧？



## 17.优化的地方

-  **处理一个接口有多个类实现的情况** ：对服务分组，发布服务的时候增加一个 group 参数即可。
-  **集成 Spring 通过注解注册服务**
-  **集成 Spring 通过注解进行服务消费** 。参考： [PR#10](https://github.com/Snailclimb/guide-rpc-framework/pull/10)
-  **增加服务版本号** ：建议使用两位数字版本，如：1.0，通常在接口不兼容时版本号才需要升级。为什么要增加服务版本号？为后续不兼容升级提供可能，比如服务接口增加方法，或服务模型增加字段，可向后兼容，删除方法或删除字段，将不兼容，枚举类型新增字段也不兼容，需通过变更版本号升级。
-  **对 SPI 机制的运用**
-  **增加可配置比如序列化方式、注册中心的实现方式,避免硬编码** ：通过 API 配置，后续集成 Spring 的话建议使用配置文件的方式进行配置
-  客户端与服务端通信协议（数据包结构）重新设计，可以将原有的RpcRequest和 RpcReuqest对象作为消息体，然后增加如下字段（可以参考：《Netty 入门实战小册》和 Dubbo 框架对这块的设计）：
  - **魔数** ： 通常是 4 个字节。这个魔数主要是为了筛选来到服务端的数据包，有了这个魔数之后，服务端首先取出前面四个字节进行比对，能够在第一时间识别出这个数据包并非是遵循自定义协议的，也就是无效数据包，为了安全考虑可以直接关闭连接以节省资源。
  - **序列化器编号** ：标识序列化的方式，比如是使用 Java 自带的序列化，还是 json，kyro 等序列化方式。
  - **消息体长度** ： 运行时计算出来。
  - ......
-  **编写测试为重构代码提供信心**
-  **服务监控中心（类似dubbo admin）**

## 18. rpc的通信 

### 18.1 服务端

服务端在使用的时候，使用构造方法初始化服务端的ip地址、端口以及序列化器。

然后调用start()方法启动服务端，服务端这时会（调用scanServices()）对扫描使用了注解的服务并向Nacos注册服务。

> scanServices具体逻辑：
>
> 使用反射工具获取全部的Class( String mainClassName = ReflectUtil.getStackTrace();      Class栈，栈顶是启动类),并获取启动类的类对象：Class.forName(mainClassName)
> 检查启动类是否有@ServiceScan注解  startClass.isAnnotationPresent(WeRpcServiceScan.class)
> 根据@ScanService注解的value值扫描指定的包，找到使用@WeRpcService注解的类   clazz.isAnnotationPresent(WeRpcService.class)
> 获取注解的name值   clazz.getAnnotation(WeRpcService.class).name();
> 实例化该服务类  obj = clazz.newInstance()，如果name值不为空，那么就使用这个值作为服务的名称，如果为空，就获取这个类的接口名称作为服务名称
> 然后调用publishService(obj,serviceName)注册服务
> publishService()方法主要有两个作用：一个是使用本地注册表保存服务的实例，就是一个Map,用serviceName作为key,服务的实例作为值进行存储。(key去重)
> 另一个作用就是向Nacos注册中心注册服务，（NacosUtil中使用HashSet对serviceName去重） namingService.registerInstance(serviceName, address.getHostName(), address.getPort());



接下来：就是服务端处理网络连接的过程。

服务端首先会创建一个监听套接字，然后给这个套接字绑定一个ip和端口，这一步对应的方法就是bind(),之后就是调用listen()来监听端口，端口是和应用程序对应的，网卡收到一个数据包的时候后需要知道这个包是给哪个程序用的，当然一个应用程序可以监听多个端口。之后客户端发起连接内核会分配一个随机端口，然后tcp在经历三次握手成功后，客户端会创建一个套接字由connect()方法返回，而服务端的accept()方法也会返回一个套接字，之后双方都会基于这个套接字进行读写操作。所以服务端会维护两种类型的套接字，一种用于监听，另一种用于和客户端进行读写。

> **建立连接**
>
> linux内核中会维护两个队列，这两个队列的长度都是有限制且可以配置的，当客户端发起connect()请求后，服务端收到syn包后将该信息放入sync队列，之后客户端回复ack后从sync队列取出，放到accept队列，之后服务端调用accept()方法会从accept队列取出生成socket。
>
> 如果客户端发起sync请求，但是不回复ack，将导致sync队列满载，之后会拒接新的连接。如果客户端发起ack请求后，服务端一直不调用，或者调用accept队列太慢，将导致accept队列满载，accept队列满了则收到ack后无法从syn队列移出去，导致syn队列也会堆积，最终拒绝连接。所以服务端一般会将accept单独起一个线程执行，避免accept太慢导致数据丢弃。当然accept()方法也有阻塞和非阻塞两种，当accept队列为空的时候阻塞方法会一直等待，非阻塞方法会直接返回一个错误码。
>
> **发送消息**
>
> 连接建立好后，客户端和服务端都有一个socket套接字，双方都可以通过各自的套接字进行发送和接收消息，socket里面维护了两个队列，一个发送队列，一个接收队列。发送的时候数据在用户空间的内存中，当调用send()或者write()方法的时候，会将待发送的数据按照MSS进行拆分，然后将拆分好的数据包拷贝到内核空间的发送队列，这个队列里面存放的是所有已经发送的数据包，对应的数据结构就是sk_buff，每一个数据包也就是sk_buff都有一个序号，以及一个状态，只有当服务端返回ack的时候，才会把状态改为发送成功，并且会将这个ack报文的序号之前的报文都确认掉，如果长期没有确认，会重新调用tcp_push继续发送，如果发送队列慢了，则从用户空间拷贝到内核空间的操作就会阻塞，并触发清理队列中已确认发送成功的数据包。tcp层会将数据包加上ip头然后发给ip层处理，ip层将数据包加入到一个qdisc队列，网卡驱动程序检测到qdisc队列有数据就会调用DMA Engine将sk_buff拷贝到网卡并发送出去，网卡驱动通过ringbuffer来指向内核中的数据，所以qdisc的长度也会影响到网络发送的吞吐量。
>
> 关于mss分片：mtu是数据链路层的最大传输单元，一般为1500字节，而一个ip包的最大长度为65535，所以ip层在发送数据前会根据mtu分片，这样一个tcp包本来对应一个ip包，分片后将对应多个ip包，每个包都有一个ip头,在接收端需要等到所有的ip包到达后，才能确定这个tcp收到然后才发送ack，这种方式无疑是低效的，所以tcp层会尽量阻止ip层进行分片，他会在从用户空间拷贝的时候就会按照mtu进行拆分，将一个数据包拆分成多个数据包。但是链路中mtu是会改变的，为了完全避免ip层进行分片，可以在ip层设置一个df标记，如果一定要分片就慧慧一个icmp报文。
>
> 关于流控：
>
> - 滑动窗口：接收方返回的一个最大发送序号。这个不是报文大小，而是一个序号，接收方每次会返回一个下次报文发送的序号不要超过的值。这个值主要和接收方内部缓存大小有关。
> - 阻塞窗口：发送方根据网络拥堵情况，根据已经发送到网络但是还未确认的数据包的数量来计算。由于广域网络的复杂所以拥塞控制有一系列算法，如慢启动等。
> - nagle算法：为了避免机器发了大量的小数据包，nagle算法限制每次将多个小数据包达到一定大小后在发送。
>
> 由于tcp发送的时候会进行各种分片和合并，所以接收方会出现粘包现象，需要应用层进行处理。
>
> **消息接收**
>
> 当服务端网卡收到一个报文后，网卡驱动调用DMA engine将数据包通过ringbuffer拷贝到内核缓冲区中，拷贝成功后，发起中断通知中断处理程序，这时候ip层会处理该数据包，之后交给tcp层，最终到达tcp层的recv buffer（接收队列）,这时候就会返回ack给客户端，并没有等到客户端调用read将数据从内核拷贝到用户空间，所以应用层也应该有相关的确认机制。如果recv buffer设置的太小，或者应用层一直不来取，那么也将阻塞数据接收，从而影响到滑动窗口大小，导致吞吐量降低。tcp在收到数据包后会获取序号，并且看是否应该正好放入接收队列，如果此时收到一个大序号的报文，会将该报文缓存直到接收队列中之前的报文已经插入。
>
> 另外如果网卡支持多队列，可以将多个队列绑定到不同的cpu上，这样网卡收到报文后，不同的队列就会通过中断触发不同的cpu，从而可以提高吞吐量。
>
> **c10k问题**
>
> c10k问题是指怎么支持单机1万的并发请求，我们想到通过select的多路复用模式，用一个单独的线程去扫描需要监听的文件描述符，如果这些文件描述符里面有可读或者可写的就返回（tcp层在收到报文拷贝到内存后会修改这个文件描述符的状态），没有就阻塞，不过这种方式需要对文件描述符进行扫描，效率不高。而epoll方式采用红黑树去管理文件描述符，当文件可读或者可写的时候会通过一个回调函数通知用户进行具体的io操作。
>
> **TCP资源占用问题**
> 一个tcp连接需要：1，socket文件描述符；2，IP地址；3，端口；4，内存



项目里使用的通信方式，一种是java 原生的socket，另一种是Netty。他们直接最大的区别其实也就是BIO和NIO之间的区别。

服务端里加入心跳机制、编码器、解码器以及服务处理器。
长时间未检测到心跳即断开连接，尝试间隔5分钟的3次重连。
编码器、解码器参考前文
服务处理器参考前文

### 18.2 客户端

参考客户端调用流程







