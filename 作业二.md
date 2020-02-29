Tomcat是一个http服务器，也是一个Servlet容器。Tomcat有两个非常重要的功能。一个是负责对外交流，另一个是负责内部处理



![image-20200229234037417](C:\Users\T\AppData\Roaming\Typora\typora-user-images\image-20200229234037417.png)



所以Tomcat的设计有两个核心的组件，一个是《对外交流》的 连接器组件Connector 另一个是《内部处理》的 容器组件 Container。

Tomcat 连接器组件名称是 Coyote  。它主要的处理任务是：

1，它处理Socket连接。主要封装了底层的网络通信（Socket请求及响应处理）

2，负责网络字节流与Request和Response对象的转化。Coyote 将Socket 输⼊转换封装为 Request 对象，进⼀步封装后交由Catalina 容器进⾏处理，处理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写⼊输出流。所以 Coyote 使Catalina 容器（容器组件）与具体的请求协议及IO操作⽅式完全解耦

3，Coyote 负责的是具体协议（应⽤层）和IO（传输层）相关内容  



连接器Coyote的内部结构

![image-20200229233201284](C:\Users\T\AppData\Roaming\Typora\typora-user-images\image-20200229233201284.png)



| **组件**        | **作⽤描述**                                                 |
| --------------- | ------------------------------------------------------------ |
| EndPoint        | EndPoint 是 Coyote  通信端点，即通信监听的接⼝，是具体Socket接收和发送处理器，是对传输层的抽象，因此EndPoint⽤来实现TCP/IP协议的 |
| Processor       | Processor 是Coyote  协议处理接⼝ ，如果说EndPoint是⽤来实现TCP/IP协议的，那么Processor⽤来实现HTTP协议，Processor接收来⾃EndPoint的Socket，读取字节流解析成Tomcat Request和Response对象，并通过Adapter将其提交到容器处理，Processor是对应⽤层协议的抽象 |
| ProtocolHandler | Coyote 协议接⼝， 通过Endpoint 和 Processor  ， 实现针对具体协议的处理能⼒。Tomcat  按照协议和I/O 提供了6个实现类 ： AjpNioProtocol  ， AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ， Http11Nio2Protocol ，Http11AprProtocol |
| Adapter         | 由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了⾃⼰的Request类来封装这些请求信息。ProtocolHandler接⼝负责解析请求并⽣成Tomcat Request类。但是这个Request对象不是标准的ServletRequest，不能⽤Tomcat Request作为参数来调⽤容器。Tomcat设计者的解决⽅案是引  ⼊CoyoteAdapter，这是适配器模式的经典运⽤，连接器调⽤CoyoteAdapter  的 Sevice ⽅ 法 ， 传 ⼊ 的 是 Tomcat  Request 对 象 ， CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调⽤容器 |



Servlet容器Catalina的具体结构

![image-20200229232627730](C:\Users\T\AppData\Roaming\Typora\typora-user-images\image-20200229232627730.png)

其实，可以认为整个Tomcat就是⼀个Catalina实例，Tomcat 启动的时候会初始化这个实例，Catalina 实例通过加载server.xml完成其他实例的创建，创建并管理⼀个Server，Server创建并管理多个服务，   每个服务⼜可以有多个Connector和⼀个Container。

⼀个Catalina实例（容器）

⼀个 Server实例（容器） 多个Service实例（容器）

每⼀个Service实例下可以有多个Connector实例和⼀个Container实例

| 组件      | 作用说明                                                     |
| --------- | ------------------------------------------------------------ |
| Catalina  | 负责解析Tomcat的配置⽂件（server.xml） , 以此来创建服务器Server组件并进⾏管理 |
| Server    | 服务器表示整个Catalina   Servlet容器以及其它组件，负责组装并启动Servlaet引擎,Tomcat连接器。Server通过实现Lifecycle接⼝，提供了⼀种优雅的启动和关闭整个系统的⽅式 |
| Service   | 服务是Server内部的组件，⼀个Server包含多个Service。它将若⼲个Connector组件绑定到⼀个Container |
| Container | 负责处理⽤户的servlet请求，并返回对象给web⽤户的模块         |

Container组件的具体结构

| 组件    | 作用说明                                                     |
| ------- | ------------------------------------------------------------ |
| Engine  | 表示整个Catalina的Servlet引擎，⽤来管理多个虚拟站点，⼀个Service最多只能有⼀个Engine，  但是⼀个引擎可包含多个Host |
| Host    | 代表⼀个虚拟主机，或者说⼀个站点，可以给Tomcat配置多个虚拟主机地址，⽽⼀个虚拟主机下   可包含多个Context |
| Context | 表示⼀个Web应⽤程序， ⼀个Web应⽤可包含多个Wrapper           |
| Wrapper | 表示⼀个Servlet，Wrapper 作为容器中的最底层，不能包含⼦容器  |

PS：以上组件的配置其实就体现在核心配置文件conf/server.xml中。