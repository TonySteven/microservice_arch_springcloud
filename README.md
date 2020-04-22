# Fenix's BookStore后端：以SpringCloud微服务实现


<p align="center">
  <a href="https://icyfenix.cn" target="_blank">
    <img width="180" src="https://raw.githubusercontent.com/fenixsoft/awesome-fenix/master/.vuepress/public/images/logo-color.png" alt="logo">
  </a>
</p>
<p align="center">
    <a href="https://icyfenix.cn"  style="display:inline-block"><img src="https://raw.githubusercontent.com/fenixsoft/awesome-fenix/master/.vuepress/public/images/Release-v1.svg"></a>
    <a href="https://travis-ci.com/fenixsoft/microservice_arch_springcloud" target="_blank"  style="display:inline-block"><img src="https://travis-ci.com/fenixsoft/microservice_arch_springcloud.svg?branch=master" alt="Travis-CI"></a>
    <a href='https://coveralls.io/github/fenixsoft/microservice_arch_springcloud?branch=master'><img src='https://coveralls.io/repos/github/fenixsoft/microservice_arch_springcloud/badge.svg?branch=master'  target="_blank"  style="display:inline-block" alt='Coverage Status' /></a>
    <a href="https://www.apache.org/licenses/LICENSE-2.0"  target="_blank" style="display:inline-block"><img src="https://raw.githubusercontent.com/fenixsoft/awesome-fenix/master/.vuepress/public/images/License-Apache.svg" alt="License"></a>
<a href="https://creativecommons.org/licenses/by/4.0/"  target="_blank" style="display:inline-block"><img src="https://raw.githubusercontent.com/fenixsoft/awesome-fenix/master/.vuepress/public/images/DocLicense-CC-red.svg" alt="Document License"></a>
    <a href="https://icyfenix.cn/introduction/about-me.html" target="_blank" style="display:inline-block"><img src="https://raw.githubusercontent.com/fenixsoft/awesome-fenix/master/.vuepress/public/images/Author-IcyFenix-blue.svg" alt="About Author"></a>
</p>


如果你此时并不曾了解过什么是“The Fenix Project”，建议先阅读<a href="https://icyfenix.cn/introduction/about-the-fenix-project.html">这部分内容</a>。

至少到目前，基于Spring Cloud的微服务解决方案仍是以Java为运行平台的微服务中，使用者数量最多的一个分支。这个结果即是Java在服务端应用中长久积累的深厚基础的体现，也是Spring在Java应用中统治性的地位的体现。Spring Cloud令现存数量极为庞大的、基于Spring和SpringBoot的单体系统，得以平滑地迁移到微服务架构中，令这些系统的大部分代码都能够无需或少量修改即可保留重用。微服务兴起的早期，Spring Cloud就集成了[Netflix OSS](https://netflix.github.io/)（Netflix闭源后也有着对应的替代组件）所开发的体系化的微服务套件，几乎透明地解决了在微服务环境中必然会面临的服务发现、远程调用、负载均衡、集中配置等基础问题。

不过，笔者自己并不太认同Spring Cloud Netflix这种以应用代码去解决基础设施功能问题的“解题思路”，以笔者的观点看来，这既是容器化、原生化的微服务基础设施完全成熟之前必然会出现的应用形态，同时也决定了这是微服务进化过程中必然会被替代的过渡形态。无论笔者的看法如何，基于Spring Cloud Netflix的微服务在当前是主流，直至未来不算短的一段时间内仍会是主流，并且以应用服务的视角，自顶向下观察基础设施在微服务中面临的需求和问题，也便于利用我们熟悉的Java代码来解释分析，有利于对微服务的整体思想的深入理解，所以将它作为我们了解的第一种微服务架构风格是十分适合的。

## 运行程序

以下几种途径，可以运行程序，浏览最终的效果：

- 通过Docker容器方式运行：<br/>微服务涉及到多个容器的协作，通过link单独运行容器已经被Docker官方声明为不提倡的方式，所以在工程中提供了专门的配置，以便使用docker-compose来运行：

  ```bash
  # 下载docker-compose配置文件
  $ curl -O https://raw.githubusercontent.com/fenixsoft/microservice_arch_springcloud/master/docker-compose.yml
  
  # 启动服务
  $ docker-compose up
  ```

  然后在浏览器访问：[http://localhost:8080](http://localhost:8080)

- 通过Git上的源码，以Maven编译、运行：<br/>由于笔者已经在配置文件中设置好了各个微服务的默认的地址和端口号，以便于本地调试。如果要在同一台机运行这些服务，并且每个微服务都只启动一个实例的话，那不加任何配置、参数即可正常以Maven编译、以Jar包形式运行。由于各个微服务需要从配置中心里获取具体的参数信息，因此唯一的要求只是“配置中心”的微服务必须作为第一个启动的服务进程，对其他的启动顺序则没有更多要求了。具体的操作过程如下所示：

  ``` bash
  # 克隆获取源码
  $ git clone https://github.com/fenixsoft/microservice_arch_springcloud.git
  
  # 进入工程根目录
  $ cd microservice_arch_springcloud
  
  # 编译打包（方式1）
  # 采用Maven Wrapper，此方式只需要机器安装有JDK 8或以上版本即可，无需包括Maven在内的其他任何依赖
  # 克隆后你可能需要使用chmod给mvnw赋予执行权限，如在Windows下应使用mvnw.cmd package代替以下命令
  $ ./mvnw package
  
  # 编译打包（方式2）
  # 直接采用Maven，由于国内访问Apache Maven的分发地址和中央仓库速度感人
  # 采用Maven Wrapper有可能长时间无响应，如你机器已安装了Maven，建议使用如下命令
  $ mvn package
  
  # 工程将编译出七个SpringBoot Jar
  # 启动服务需要运行以下七个微服务组件
  # 配置中心微服务：localhost:8888
  $ java -jar ./bookstore-microservices-platform-configuration/target/bookstore-microservice-platform-configuration-1.0.0-SNAPSHOT.jar
  # 服务发现微服务：localhost:8761
  $ java -jar ./bookstore-microservices-platform-registry/target/bookstore-microservices-platform-registry-1.0.0-SNAPSHOT.jar
  # 服务网关微服务：localhost:8080
  $ java -jar ./bookstore-microservices-platform-gateway/target/bookstore-microservices-platform-gateway-1.0.0-SNAPSHOT.jar
  # 安全认证微服务：localhost:8301
  $ java -jar ./bookstore-microservices-domain-security/target/bookstore-microservices-domain-security-1.0.0-SNAPSHOT.jar
  # 用户信息微服务：localhost:8401
  $ java -jar ./bookstore-microservices-domain-account/target/bookstore-microservices-domain-account-1.0.0-SNAPSHOT.jar
  # 商品仓库微服务：localhost:8501
  $ java -jar ./bookstore-microservices-domain-warehouse/target/bookstore-microservices-domain-warehouse-1.0.0-SNAPSHOT.jar
  # 商品交易微服务：localhost:8601
  $ java -jar ./bookstore-microservices-domain-payment/target/bookstore-microservices-domain-payment-1.0.0-SNAPSHOT.jar
  ```

  由于在命令行启动多个服务、通过容器实现各服务隔离、扩展等都较繁琐，笔者提供了一个docker-compose.dev.yml文件，便于开发期调试使用：

  ```bash
  # 使用Maven编译出JAR包后，可使用以下命令直接在本地构建镜像运行
  $ docker-compose -f docker-compose.dev.yml up
  ```

  以上两种本地运行的方式可任选其一，服务全部启动后，在浏览器访问：[http://localhost:8080](http://localhost:8080)<br/>

- 通过Git上的源码，在IDE环境中运行：

  - 以IntelliJ IDEA为例，Git克隆本项目后，在File -> Open菜单选择本项目所在的目录，或者pom.xml文件，以Maven方式导入工程。
  - 待Maven自动安装依赖后，即可在IDE或者Maven面板中编译全部子模块的程序。
  - 本工程下面八个模块，其中除bookstore-microservices-library-infrastructure外，其余均是SpringBoot工程，将这七个工程的Application类加入到IDEA的Run Dashboard面板中。
  - 在Run Dashboard中先启动“bookstore-microservices-platform-configuration”微服务，然后可一次性启动其余六个子模块的微服务。

- 配置与横向扩展<br/>工程中预留了一些的环境变量，便于配置和扩展，譬如，对于热点模块，往往需要启动多个微服务扩容，此时需要调整每个服务的端口号。预留的这类环境变量包括：

  ```bash
  # 修改配置中心的主机和端口，默认为localhost:8888
  CONFIG_HOST
  CONFIG_PORT
  
  # 修改服务发现的主机和端口，默认为localhost:8761
  REGISTRY_HOST
  REGISTRY_PORT
  
  # 修改认证中心的主机和端口，默认为localhost:8301
  AUTH_HOST
  AUTH_PORT
  
  # 修改当前微服务的端口号
  # 譬如，你打算在一台机器上扩容四个支付微服务以应对促销活动的流量高峰
  # 可将它们的端口设置为8601（默认）、8602、8603、8604等
  # 真实环境中，它们可能是在不同的物理机、容器环境下，这时扩容可无需调整端口
  PORT
  
  # SpringBoot所采用Profile配置文件，默认为default
  # 譬如，服务默认使用HSQLDB的内存模式作为数据库，如需调整为MySQL，可将此环境变量调整为mysql
  # 因为笔者默认预置了名为applicaiton-mysql.yml的配置，以及HSQLDB和MySQL的数据库脚本
  # 如果你需要支持其他数据库、修改程序中其他的配置信息，可以在代码中自行加入另外的初始化脚本
  PROFILES
  
  # Java虚拟机运行参数，默认为空
  JAVA_OPTS
  ```
  
## 技术组件

Fenix's BookStore采用基于SpringCloud微服务架构，微服务部分主要采用了Netflix OSS组件进行支持，它们包括：

- **配置中心**：默认采用[Spring Cloud Config](https://spring.io/projects/spring-cloud-config)，亦可使用[Spring Cloud Consul](https://spring.io/projects/spring-cloud-consul)、[Spring Cloud Alibaba Nacos](https://spring.io/projects/spring-cloud-alibaba)代替。
- **服务发现**：默认采用[Netflix Eureka](https://github.com/Netflix/eureka)，亦可使用[Spring Cloud Consul](https://spring.io/projects/spring-cloud-consul)、[Spring Cloud Zookeeper](https://spring.io/projects/spring-cloud-zookeeper)、[etcd](https://github.com/etcd-io/etcd)等代替。
- **服务网关**：默认采用[Netflix Zuul](https://github.com/Netflix/zuul)，亦可使用[Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)代替。
- **服务熔断**：默认采用[Netflix Hystrix](https://github.com/Netflix/Hystrix)，亦可使用[Sentinel](https://github.com/alibaba/Sentinel)、[Resilience4j](https://github.com/resilience4j/resilience4j)代替。
- **进程内负载均衡**：默认采用[Netfilix Ribbon](https://github.com/Netflix/ribbon)，亦可使用[Spring Cloud Loadbalancer](https://spring.io/guides/gs/spring-cloud-loadbalancer/)代替。
- **声明式HTTP客户端**：默认采用[Spring Cloud OpenFeign](https://spring.io/projects/spring-cloud-openfeign)。这个并没有代替的必要，访问远程服务而已，使用RestTemplete或者更底层的[OkHTTP](https://square.github.io/okhttp/)、[HTTPClient](https://hc.apache.org/httpcomponents-client-ga/)也能完成，多写点代码罢了。

尽管Netflix套件的使用人数很多，但由于Spring Cloud Netflix已进入维护模式，所以笔者均列出了上述组件的代替品。这些组件几乎都是声明式的，这保证了替代它们的成本相当低，只需要更换注解，修改配置，无需改动代码。你在阅读源码时也会发现，三个“platform”开头的服务，基本上没有任何实际代码的存在。

其他与微服务无关的技术组件（REST服务、安全、数据访问，等等），笔者已在<a href="https://icyfenix.cn/exploration/projects/monolithic_arch_springboot.html#技术组件">Fenix's BookStore单体架构</a>中介绍过，在此不再重复。


## 协议

- 本文档代码部分采用[Apache 2.0协议](https://www.apache.org/licenses/LICENSE-2.0)进行许可。遵循许可的前提下，你可以自由地对代码进行修改，再发布，可以将代码用作商业用途。但要求你：
  - **署名**：在原有代码和衍生代码中，保留原作者署名及代码来源信息。
  - **保留许可证**：在原有代码和衍生代码中，保留Apache 2.0协议文件。
  
- 本作品文档部分采用[知识共享署名 4.0 国际许可协议](http://creativecommons.org/licenses/by/4.0/)进行许可。 遵循许可的前提下，你可以自由地共享，包括在任何媒介上以任何形式复制、发行本作品，亦可以自由地演绎、修改、转换或以本作品为基础进行二次创作。但要求你：
  - **署名**：应在使用本文档的全部或部分内容时候，注明原作者及来源信息。
  - **非商业性使用**：不得用于商业出版或其他任何带有商业性质的行为。如需商业使用，请联系作者。
  - **相同方式共享的条件**：在本文档基础上演绎、修改的作品，应当继续以知识共享署名 4.0国际许可协议进行许可。