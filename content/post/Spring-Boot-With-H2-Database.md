
+++
title = "Spring Boot 与 H2 数据库"
date = "2024-01-14T14:18:41"
draft = false
tags = []
categories = ['Persistence', 'Spring Boot']
+++

**1. 概述**
---------



在本教程中，我们将探索将 H2 与 Spring Boot 结合使用。就像其他数据库一样，Spring Boot 生态系统对其有完整的内在支持。



**2. 依赖关系**
-----------



让我们从[h2](https://mvnrepository.com/artifact/com.h2databasea:h2)和[spring-boot-starter-data-jpa](https://mvnrepository.com/search?q=/spring-boot-starter-data-jpag:org.springframework.boot) 依赖项开始：



```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

**3. 数据库配置**
------------



默认情况下，Spring Boot 将应用程序配置为使用用户名**sa****和空密码****连接到内存存储**。


但是，我们可以通过将以下属性添加到application.properties文件来更改这些参数：



```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

或者，我们也可以使用 YAML 进行应用程序的数据库配置，方法是在application.yaml文件中添加相应的属性：




```
spring:
  datasource:
    url: jdbc:h2:mem:mydb
    username: sa
    password: password
    driverClassName: org.h2.Driver
  jpa:
    spring.jpa.database-platform: org.hibernate.dialect.H2Dialect
```

根据设计，内存数据库是易失性的，并会导致应用程序重新启动后数据丢失。


我们可以通过使用基于文件的存储来改变这种行为。为此，我们需要更新spring.datasource.url属性：



```
spring.datasource.url=jdbc:h2:file:/data/demo
```

同样，在application.yaml中，我们可以为基于文件的存储添加相同的属性：



```
spring:
  datasource:
    url: jdbc:h2:file:/data/demo
```

数据库还可以[以其他模式运行](http://www.h2database.com/html/features.html#connection_modes)。


**4. 数据库操作**
------------



在 Spring Boot 中使用 H2执行[CRUD 操作](https://www.baeldung.com/cs/crud-operations)与其他 SQL 数据库相同，我们在[Spring Persistence](https://www.baeldung.com/persistence-with-spring-series)系列中的教程很好地涵盖了这一点。



### 4.1. 数据源初始化



我们可以使用基本的 SQL 脚本来初始化数据库。为了演示这一点，我们在src/main/resources目录下添加一个data.sql文件：



```
INSERT INTO countries (id, name) VALUES (1, 'USA');
INSERT INTO countries (id, name) VALUES (2, 'France');
INSERT INTO countries (id, name) VALUES (3, 'Brazil');
INSERT INTO countries (id, name) VALUES (4, 'Italy');
INSERT INTO countries (id, name) VALUES (5, 'Canada');
```

在这里，脚本使用一些示例数据填充我们架构中的国家/地区表。


Spring Boot 将自动获取此文件并针对嵌入式内存数据库（例如我们配置的 H2 实例）运行它。这是**为测试或初始化目的植入数据库的好方法**。


我们可以通过将spring.sql.init.mode属性设置为never来禁用此默认行为。此外，还可以配置[多个SQL文件来加载初始数据。](https://www.baeldung.com/spring-boot-sql-import-files#spring-jdbc-support)


我们关于[加载初始数据的](https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)文章更详细地介绍了这个主题。



### 4.2. Hibernate 和data.sql



默认情况下，**data.sql脚本在 Hibernate 初始化之前执行**。这使得基于脚本的初始化与其他数据库迁移工具（例如[Flyway](https://www.baeldung.com/database-migrations-with-flyway)和[Liquibase ）](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)保持一致。当我们每次重新创建 Hibernate 生成的模式时，我们需要设置一个附加属性：



```
spring.jpa.defer-datasource-initialization=true
```

这**会修改默认的 Spring Boot 行为并在 Hibernate 生成架构后填充数据**。此外，我们还可以在使用data.sql填充之前使用schema.sql脚本来构建 Hibernate 生成的模式。但是，不建议混合使用不同的模式生成机制。


**5. 访问H2控制台**
--------------



H2 数据库有一个嵌入式 GUI 控制台，用于浏览数据库内容并运行 SQL 查询。默认情况下，Spring 中未启用 H2 控制台。


要启用它，我们需要将以下属性添加到application.properties：



```
spring.h2.console.enabled=true
```

如果我们使用 YAML 配置，我们需要将该属性添加到application.yaml：



```
spring:
  h2:
    console.enabled: true
```

然后，启动应用程序后，我们可以导航到http://localhost:8080/h2-console，这将向我们显示一个登录页面。


在登录页面上，我们将提供与application.properties中使用的相同凭据：


[![h2 控制台 - 登录](https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.21.34-PM-e1555173105246-1024x496.png)](https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.21.34-PM-e1555173105246-1024x496.png)
连接后，我们将看到一个综合网页，其中列出了页面左侧的所有表以及用于运行 SQL 查询的文本框：



[![h2控制台-SQL语句](https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.25.16-PM.png)](https://www.baeldung.com/wp-content/uploads/2019/04/Screenshot-2019-04-13-at-5.25.16-PM.png)
Web 控制台具有自动完成功能，可以建议 SQL 关键字。事实上，控制台是轻量级的，因此可以方便地直观地检查数据库或直接执行原始 SQL。


此外，我们可以通过在项目的application.properties中使用我们想要的值指定以下属性来进一步配置控制台：



```
spring.h2.console.path=/h2-console
spring.h2.console.settings.trace=false
spring.h2.console.settings.web-allow-others=false
```

同样，当使用 YAML 配置时，我们可以将上述属性添加为：



```
spring:
  h2:
    console:
      path: /h2-console
      settings.trace: false
      settings.web-allow-others: false
```

在上面的代码片段中，我们将控制台路径设置为/h2-console，它与正在运行的应用程序的地址和端口相关。因此，如果我们的应用程序在http://localhost:9001上运行，则控制台将在http://localhost:9001/h2-console上可用。


此外，我们将spring.h2.console.settings.trace设置为false以防止跟踪输出，并且我们还可以通过设置spring来禁用远程访问。h2.console.settings.web-allow-others为false。


**六，结论**
--------



H2数据库与Spring Boot完全兼容。我们已经了解了如何配置它以及如何使用 H2 控制台来管理我们正在运行的数据库。


[完整的源代码可在GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2)上获取。



    