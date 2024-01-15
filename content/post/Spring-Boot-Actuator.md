
+++
title = "Spring Boot Actuator"
date = "2024-01-14T16:48:11"
draft = false
tags = []
categories = ['Spring Boot']
+++

**1. 概述**
---------



在这篇文章中，我们介绍了 Spring Boot Actuator。**我们将首先介绍基础知识，然后详细讨论 Spring Boot 2.x 与 1.x 中的可用功能。**


我们将学习如何利用反应式编程模型在 Spring Boot 2.x 和 WebFlux 中使用、配置和扩展此监控工具。然后，我们将讨论如何使用 Boot 1.x 执行相同的操作。


Spring Boot Actuator 自 2014 年 4 月起随第一个 Spring Boot 版本一起推出。


随着[Spring Boot 2 的发布](https://www.baeldung.com/new-spring-boot-2)，Actuator 进行了重新设计，并添加了新的令人兴奋的端点。


我们将本指南分为三个主要部分：



* [什么是Actuator？](#understanding-actuator)
* [Spring Boot 2.x Actuator](#boot-2x-actuator)
* [Spring Boot 1.x Actuator](#boot-1x-actuator)



**2. 什么是Actuator？**
--------------



从本质上讲，Actuator 为我们的应用程序带来了生产就绪的功能。


**由于这种依赖关系，监控我们的应用程序、收集指标以及了解流量或数据库状态变得微不足道。**


该库的主要好处是我们可以获得生产级工具，而无需自己实际实现这些功能。


执行器主要公开**有关正在运行的应用程序的操作信息**- 运行状况、指标、信息、转储、环境等。它使用 HTTP 端点或 JMX beans 使我们能够与其交互。


一旦这种依赖关系出现在类路径上，我们就可以使用几个开箱即用的端点。与大多数 Spring 模块一样，我们可以通过多种方式轻松配置或扩展它。



### **2.1. 入门**



我们需要将 spring-boot-actuator 依赖项添加到包管理器中以启用 Spring Boot Actuator。


在 Maven 中：



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

请注意，无论 Boot 版本如何，这仍然有效，因为版本是在 Spring Boot 物料清单 (BOM) 中指定的。


**3.Spring Boot 2.x Actuator**
------------------------



在 2.x 中，执行器保留了其基本意图，但简化了其模型、扩展了其功能并合并了更好的默认值。


首先，这个版本变得与技术无关。它还通过将其与应用程序合并来简化其安全模型。



在各种变化中，重要的是要记住其中一些变化正在破坏。这包括 HTTP 请求和响应以及 Java API。


最后，最新版本现在支持 CRUD 模型，而不是旧的读/写模型。


### **3.1. 技术支持**



在第二个主要版本中，Actuator 现在与技术无关，而在 1.x 中，它与 MVC 相关，因此与 Servlet API 相关。


在 2.x 中，Actuator 将其模型定义为可插拔和可扩展，而不依赖于 MVC。


**因此，通过这个新模型，我们可以利用 MVC 和 WebFlux 作为底层 Web 技术。**


此外，可以通过实施正确的适配器来添加即将推出的技术。


最后，JMX 仍然支持公开端点，无需任何额外代码。


### **3.2. 重要变化**



与以前的版本不同，**Actuator 禁用了大多数端点。**



因此，默认情况下唯一可用的两个是/health和/info。


如果我们想启用所有这些，我们可以设置management.endpoints.web.exposure.include=*。或者，我们可以列出应启用的端点。


**此外，Actuator 现在与常规应用程序安全规则共享安全配置，从而极大地简化了安全模型。**


因此，如果我们在项目中使用 Spring Security，那么我们将必须调整执行器安全规则以允许执行器端点。


我们只需为/actuator/**添加一个条目：



```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll()
      .anyExchange().authenticated()
      .and().build();
}
```

[我们可以在全新的 Actuator 官方文档](https://docs.spring.io/spring-boot/docs/2.0.x/actuator-api/html/)中找到更多详细信息。


**此外，默认情况下，所有 Actuator 端点现在都放置在 /actuator 路径下。**


与之前的版本相同，我们可以使用新的属性management.endpoints.web.base-path调整此路径。




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_2", slotId: "baeldung\_incontent\_2" });
 

### **3.3. 预定义端点**



让我们看一下一些可用的端点，其中大部分已经在 1.x 中可用。


此外，**还添加了一些端点，删除了一些端点，并重组了一些端点**：


* /auditevents列出与安全审核相关的事件，例如用户登录/注销。此外，我们还可以按主体或类型在其他字段中进行过滤。
* /beans返回BeanFactory中所有可用的 beans 。与/auditevents不同，它不支持过滤。
* /conditions以前称为 / autoconfig，围绕自动配置构建条件报告。
* /configprops允许我们获取所有@ConfigurationProperties bean。
* /env返回当前环境属性。此外，我们可以检索单个属性。
* /flyway提供有关 Flyway 数据库迁移的详细信息。
* /health总结了我们应用程序的健康状况。
* /heapdump从我们的应用程序使用的 JVM 构建并返回堆转储。
* /info返回一般信息。它可能是自定义数据、构建信息或有关最新提交的详细信息。
* /liquibase 的行为类似于/flyway，但适用于 Liquibase。
* /logfile返回普通应用程序日志。
* /loggers使我们能够查询和修改应用程序的日志记录级别。
* /metrics详细说明了我们应用程序的指标。这可能包括通用指标和自定义指标。
* /prometheus返回与前一个类似的指标，但格式设置为与 Prometheus 服务器配合使用。
* /scheduledtasks提供有关应用程序中每个计划任务的详细信息。
* /sessions列出了 HTTP 会话，假设我们使用的是 Spring Session。
* /shutdown执行应用程序的正常关闭。
* /threaddump转储底层 JVM 的线程信息。


### 3.4. Actuator端点的超媒体



Spring Boot 添加了一个发现端点，该端点返回所有可用执行器端点的链接。这将有助于发现执行器端点及其相应的 URL。


**默认情况下，可以通过/actuator端点访问此发现端点 。**


因此，如果我们向此 URL 发送 GET请求，它将返回各个端点的执行器链接：



```json
{
  "\_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "features-arg0": {
      "href": "http://localhost:8080/actuator/features/{arg0}",
      "templated": true
    },
    "features": {
      "href": "http://localhost:8080/actuator/features",
      "templated": false
    },
    "beans": {
      "href": "http://localhost:8080/actuator/beans",
      "templated": false
    },
    "caches-cache": {
      "href": "http://localhost:8080/actuator/caches/{cache}",
      "templated": true
    },
    // truncated
}
```

如上所示，  /actuator端点报告\_links字段下所有可用的执行器端点。


**此外，如果我们配置自定义管理基本路径，那么我们应该使用该基本路径作为发现 URL。**


例如，如果我们将 management.endpoints.web.base-path设置为/mgmt，我们应该请求/mgmt端点来查看链接列表。



非常有趣的是，当管理基本路径设置为/时，发现端点将被禁用，以防止与其他映射发生冲突的可能性。


### **3.5. 健康指标**



就像之前的版本一样，我们可以轻松添加自定义指标。与其他 API 不同，创建自定义健康端点的抽象保持不变。但是，**添加了****一个新接口ReactiveHealthIndicator来实现反应式运行状况检查。**


让我们看一下一个简单的自定义反应式健康检查：



```java
@Component
public class DownstreamServiceHealthIndicator implements ReactiveHealthIndicator {

    @Override
    public Mono<Health> health() {
        return checkDownstreamServiceHealth().onErrorResume(
          ex -> Mono.just(new Health.Builder().down(ex).build())
        );
    }

    private Mono<Health> checkDownstreamServiceHealth() {
        // we could use WebClient to check health reactively
        return Mono.just(new Health.Builder().up().build());
    }
}
```

**健康指标的一个方便功能是我们可以将它们聚合为层次结构的一部分。**


因此，按照前面的示例，我们可以将所有下游服务分组到下游服务类别下。只要每个嵌套服务都可以访问，这个类别就会是健康的。


请查看我们关于[健康指标的](https://www.baeldung.com/spring-boot-health-indicators)文章，进行更深入的了解。


### 3.6. 健康团体



**从 Spring Boot 2.2 开始，我们可以将健康指标组织成[组](https://github.com/spring-projects/spring-boot/blob/c3aa494ba32b8271ea19dd041327441b27ddc319/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthEndpointGroups.java#L30)，并对所有组成员应用相同的配置。**


例如，我们可以 通过将其添加到application.properties来创建一个名为custom的健康组：




```
management.endpoint.health.group.custom.include=diskSpace,ping
```

这样，自定义组包含 磁盘空间和 ping运行状况指示器。


现在，如果我们调用/actuator/health端点，它将在 JSON 响应中告诉我们新的健康组：



```java
{"status":"UP","groups":["custom"]}
```

**通过健康分组，我们可以看到一些健康指标的汇总结果。**


在这种情况下，如果我们向/actuator/health/custom发送请求 ，则：



```java
{"status":"UP"}
```

我们可以通过application.properties配置该组以显示更多详细信息 ：



```
management.endpoint.health.group.custom.show-components=always
management.endpoint.health.group.custom.show-details=always
```

现在，如果我们向/actuator/health/custom发送相同的请求，我们将看到更多详细信息：



```java
{
  "status": "UP",
  "components": {
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 499963170816,
        "free": 91300069376,
        "threshold": 10485760
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

也可以仅为授权用户显示这些详细信息：



```
management.endpoint.health.group.custom.show-components=when_authorized
management.endpoint.health.group.custom.show-details=when_authorized
```

**我们还可以有一个自定义状态映射。**



例如，它可以返回 207 状态代码，而不是 HTTP 200 OK 响应：



```
management.endpoint.health.group.custom.status.http-mapping.up=207
```

如果自定义组状态为 UP，我们会告诉 Spring Boot 返回 207 HTTP 状态代码。


### **3.7. Spring Boot 2 中的指标**



**在 Spring Boot 2.0 中，内部指标被 Micrometer 支持取代**，因此我们可以期待重大变化。如果我们的应用程序使用GaugeService或CounterService等指标服务，它们将不再可用。


相反，我们应该直接与[Micrometer](https://www.baeldung.com/micrometer)交互。在 Spring Boot 2.0 中，我们将为我们自动配置一个MeterRegistry类型的 bean。


此外，Micrometer 现在是 Actuator 依赖项的一部分，因此只要 Actuator 依赖项位于类路径中，我们就可以继续进行。


此外，我们将从/metrics端点获得全新的响应：



```java
{
  "names": [
    "jvm.gc.pause",
    "jvm.buffer.memory.used",
    "jvm.memory.used",
    "jvm.buffer.count",
    // ...
  ]
}
```

正如我们所看到的，没有像 1.x 中那样的实际指标。


要获取特定指标的实际值，我们现在可以导航到所需的指标，例如/actuator/metrics/jvm.gc.pause，并获得详细的响应：




```java
{
  "name": "jvm.gc.pause",
  "measurements": [
    {
      "statistic": "Count",
      "value": 3.0
    },
    {
      "statistic": "TotalTime",
      "value": 7.9E7
    },
    {
      "statistic": "Max",
      "value": 7.9E7
    }
  ],
  "availableTags": [
    {
      "tag": "cause",
      "values": [
        "Metadata GC Threshold",
        "Allocation Failure"
      ]
    },
    {
      "tag": "action",
      "values": [
        "end of minor GC",
        "end of major GC"
      ]
    }
  ]
}
```

现在，指标更加全面，包括不同的值和一些相关的元数据。


### **3.8. 自定义/info端点**



/info端点保持不变。**和以前一样，我们可以使用相应的 Maven 或 Gradle 依赖项添加 git 详细信息**：



```xml
<dependency>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
</dependency>
```

同样，**我们还可以使用 Maven 或 Gradle 插件包含构建信息，包括名称、组和版本**：



```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### **3.9. 创建自定义端点**



正如我们之前指出的，我们可以创建自定义端点。然而，Spring Boot 2 重新设计了如何实现这一点，以支持新的技术不可知范式。


**让我们创建一个 Actuator 端点来查询、启用和禁用应用程序中的功能标志**：



```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {

    private Map<String, Feature> features = new ConcurrentHashMap<>();

    @ReadOperation
    public Map<String, Feature> features() {
        return features;
    }

    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }

    @WriteOperation
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }

    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }

    public static class Feature {
        private Boolean enabled;

        // [...] getters and setters 
    }

}
```

为了获得端点，我们需要一个 bean。在我们的示例中，我们使用@Component来实现此目的。另外，我们需要用@Endpoint来装饰这个 bean 。


我们端点的路径由@Endpoint的id参数确定。在我们的例子中，它将请求路由到/actuator/features。


准备好后，我们可以开始使用以下命令定义操作：



* @ReadOperation：它将映射到 HTTP GET。
* @WriteOperation：它将映射到 HTTP POST。
* @DeleteOperation：它将映射到 HTTP DELETE。


当我们使用应用程序中的先前端点运行应用程序时，Spring Boot 将注册它。


验证这一点的快速方法是检查日志：



```
[...].WebFluxEndpointHandlerMapping: Mapped "{[/actuator/features/{name}],
  methods=[GET],
  produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features],
  methods=[GET],
  produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
  methods=[POST],
  consumes=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
[...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
  methods=[DELETE]}"[...]
```

在之前的日志中，我们可以看到 WebFlux 如何公开我们的新端点。如果我们切换到 MVC，它会简单地委托该技术，而无需更改任何代码。


此外，对于这种新方法，我们需要牢记一些重要的注意事项：


* 与 MVC 没有依赖关系。
* 之前作为方法出现的所有元数据（敏感的、启用的……）都不再存在。但是，我们可以使用@Endpoint(id = “features”, enableByDefault = false)启用或禁用端点。
* 与 1.x 不同，不再需要扩展给定的接口。
* 与旧的读/写模型相反，我们现在可以使用@DeleteOperation定义DELETE操作。


### **3.10. 扩展现有端点**



假设我们想要确保应用程序的生产实例永远不是快照版本。


我们决定通过更改返回此信息的 Actuator 端点的 HTTP 状态代码（即/info ）来实现此目的。如果我们的应用程序碰巧是SNAPSHOT，我们会得到不同的HTTP状态代码。


**我们可以使用@EndpointExtension注释** 或其更具体的专业化@EndpointWebExtension或@EndpointJmxExtension**轻松扩展预定义端点的行为**：



```java
@Component
@EndpointWebExtension(endpoint = InfoEndpoint.class)
public class InfoWebEndpointExtension {

    private InfoEndpoint delegate;

    // standard constructor

    @ReadOperation
    public WebEndpointResponse<Map> info() {
        Map<String, Object> info = this.delegate.info();
        Integer status = getStatus(info);
        return new WebEndpointResponse<>(info, status);
    }

    private Integer getStatus(Map<String, Object> info) {
        // return 5xx if this is a snapshot
        return 200;
    }
}
```

### **3.11. 启用所有端点**



**为了使用 HTTP 访问执行器端点，我们需要启用并公开它们。**




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_7", slotId: "baeldung\_incontent\_8" });
 

默认情况下，启用除/shutdown之外的所有端点。默认情况下仅公开/health和/info端点。


我们需要添加以下配置来公开所有端点：



```java
management.endpoints.web.exposure.include=*
```

要显式启用特定端点（例如/shutdown），我们使用：



```java
management.endpoint.shutdown.enabled=true
```

要公开除一个端点（例如/loggers）之外的所有已启用端点，我们使用：



```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=loggers
```

**4. Spring Boot 1.x执行器**
-------------------------



在 1.x 中，Actuator 遵循读/写模型，这意味着我们可以从中读取或写入。


例如，我们可以检索应用程序的指标或运行状况。或者，我们可以优雅地终止我们的应用程序或更改我们的日志配置。


为了使其正常工作，Actuator 需要 Spring MVC 通过 HTTP 公开其端点。不支持其他技术。


### **4.1. 端点**



**在 1.x 中，Actuator 带来了自己的安全模型。它利用 Spring Security 构造，但需要独立于应用程序的其余部分进行配置。**




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_7", slotId: "baeldung\_incontent\_9" });
 

此外，大多数端点都是敏感的——这意味着它们不完全公开，或者大多数信息将被省略——而少数端点则不是，例如/info。


以下是 Boot 提供的一些最常见的开箱即用端点：


* /health显示应用程序运行状况信息（ 通过未经身份验证的连接访问时的简单状态或经过身份验证时的完整消息详细信息）；默认情况下它不敏感。
* /info显示任意应用程序信息；默认情况下它不敏感。
* /metrics显示当前应用程序的指标信息；默认情况下它是敏感的。
* /trace显示跟踪信息（默认情况下，最后几个 HTTP 请求）。


[我们可以在官方文档中](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)找到现有端点的完整列表。


### **4.2. 配置现有端点**



我们可以使用格式端点.[端点名称].[要自定义的属性]自定义每个端点的属性。


三个属性可用：


* id：通过 HTTP 访问此端点
* enabled：如果为true，则可以访问；否则不
* 敏感：如果为 true，则需要授权才能通过 HTTP 显示关键信息


例如，添加以下属性将自定义 / beans端点：



```
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
endpoints.beans.enabled=true
```

### **4.3. /健康端点**



**/health端点用于检查正在运行的应用程序的运行状况或状态。**


它通常通过监控软件来执行，如果正在运行的实例由于其他原因（例如，数据库的连接问题、磁盘空间不足等）出现故障或运行状况不佳，则会向我们发出警报。




默认情况下，未经授权的用户只能在通过 HTTP 访问时看到状态信息：



```
{
    "status" : "UP"
}

```

此运行状况信息是从实现应用程序上下文中配置的HealthIndicator接口的所有 bean 收集的。


HealthIndicator返回的一些信息本质上是敏感的，但我们可以配置endpoints.health.sensitive=false来公开更详细的信息，例如磁盘空间、消息代理连接、自定义检查等。


请注意，这仅适用于 1.5.0 以下的 Spring Boot 版本。对于1.5.0及更高版本，我们还应该通过设置management.security.enabled=false来禁用安全性，以防止未经授权的访问。


我们还可以**实现自己的自定义运行状况指示器**，它可以收集特定于应用程序的任何类型的自定义运行状况数据，并通过/health端点自动公开它：



```java
@Component("myHealthCheck")
public class HealthCheck implements HealthIndicator {
 
    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down()
              .withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }
    
    public int check() {
    	// Our logic to check health
    	return 0;
    }
}

```

输出如下所示：



```
{
    "status" : "DOWN",
    "myHealthCheck" : {
        "status" : "DOWN",
        "Error Code" : 1
     },
     "diskSpace" : {
         "status" : "UP",
         "free" : 209047318528,
         "threshold" : 10485760
     }
}
```

### **4.4. /信息端点**



我们还可以自定义/info端点显示的数据：



```
info.app.name=Spring Sample Application
info.app.description=This is my first spring boot application
info.app.version=1.0.0
```

以及示例输出：




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_7", slotId: "baeldung\_incontent\_11" });
 


```
{
    "app" : {
        "version" : "1.0.0",
        "description" : "This is my first spring boot application",
        "name" : "Spring Sample Application"
    }
}
```

### **4.5. /指标端点**



**指标端点发布有关操作系统和 JVM 以及应用程序级指标的信息。**启用后，我们可以获得内存、堆、处理器、线程、加载的类、卸载的类和线程池等信息，以及一些 HTTP 指标。


该端点的输出如下所示：



```
{
    "mem" : 193024,
    "mem.free" : 87693,
    "processors" : 4,
    "instance.uptime" : 305027,
    "uptime" : 307077,
    "systemload.average" : 0.11,
    "heap.committed" : 193024,
    "heap.init" : 124928,
    "heap.used" : 105330,
    "heap" : 1764352,
    "threads.peak" : 22,
    "threads.daemon" : 19,
    "threads" : 22,
    "classes" : 5819,
    "classes.loaded" : 5819,
    "classes.unloaded" : 0,
    "gc.ps_scavenge.count" : 7,
    "gc.ps_scavenge.time" : 54,
    "gc.ps_marksweep.count" : 1,
    "gc.ps_marksweep.time" : 44,
    "httpsessions.max" : -1,
    "httpsessions.active" : 0,
    "counter.status.200.root" : 1,
    "gauge.response.root" : 37.0
}

```

**为了收集自定义指标，我们支持仪表（数据的单值快照）和计数器，即递增/递减指标。**


让我们在/metrics端点中实现我们自己的自定义指标。


我们将自定义登录流程来记录成功和失败的登录尝试：



```java
@Service
public class LoginServiceImpl {

    private final CounterService counterService;
    
    public LoginServiceImpl(CounterService counterService) {
        this.counterService = counterService;
    }
	
    public boolean login(String userName, char[] password) {
        boolean success;
        if (userName.equals("admin") && "secret".toCharArray().equals(password)) {
            counterService.increment("counter.login.success");
            success = true;
        }
        else {
            counterService.increment("counter.login.failure");
            success = false;
        }
        return success;
    }
}
```

输出可能如下所示：



```
{
    ...
    "counter.login.success" : 105,
    "counter.login.failure" : 12,
    ...
}

```

请注意，登录尝试和其他安全相关事件在执行器中可以作为审计事件直接使用。


### **4.6. 创建新端点**



**除了使用 Spring Boot 提供的现有端点之外，我们还可以创建一个全新的端点。**


首先，我们需要让新端点实现Endpoint<T>接口：



```java
@Component
public class CustomEndpoint implements Endpoint<List<String>> {
    
    @Override
    public String getId() {
        return "customEndpoint";
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean isSensitive() {
        return true;
    }

    @Override
    public List<String> invoke() {
        // Custom logic to build the output
        List<String> messages = new ArrayList<String>();
        messages.add("This is message 1");
        messages.add("This is message 2");
        return messages;
    }
}
```

为了访问这个新端点，需要使用它的id来映射它。换句话说，我们可以通过点击/customEndpoint来练习它。


输出：



```
[ "This is message 1", "This is message 2" ]
```

### **4.7. 进一步定制**



出于安全目的，我们可能选择通过非标准端口公开执行器端点 - management.port属性可以轻松用于配置它。


另外，正如我们已经提到的，在 1.x 中。Actuator 基于 Spring Security 配置自己的安全模型，但独立于应用程序的其余部分。


因此，我们可以更改management.address属性来限制可以通过网络访问端点的位置：



```
#port used to expose actuator
management.port=8081 

#CIDR allowed to hit actuator
management.address=127.0.0.1 

#Whether security should be enabled or disabled altogether
management.security.enabled=false
```

此外，默认情况下，除/info之外的所有内置端点都是敏感的。


如果应用程序使用 Spring Security，我们可以通过在application.properties文件中定义默认安全属性（用户名、密码和角色）来保护这些端点：



```
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER
```

**5. 结论**
---------



在这篇文章中，我们讨论了 Spring Boot Actuator。我们首先定义 Actuator 的含义以及它对我们的作用。


接下来，我们重点关注当前 Spring Boot 2.x 版本的 Actuator，讨论如何使用它、调整它和扩展它。我们还讨论了在这个新迭代中可以发现的重要安全更改。我们讨论了一些流行的端点以及它们的变化。


然后，我们讨论了早期 Spring Boot 1 版本中的 Actuator。


最后，我们演示了如何定制和扩展Actuator。


与往常一样，本文中使用的代码可以在 GitHub 上找到[Spring Boot 2.x](https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-reactive-3)和[Spring Boot 1.x](https://github.com/eugenp/tutorials/tree/master/spring-4)的代码。



    