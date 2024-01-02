
+++
title = "Spring Boot 中的测试"
date = "2024-01-01T10:05:10"
draft = false
tags = []
categories = ['Spring Boot', 'Testing']
+++

**1. 概述**
---------



在本教程中，我们将了解如何**使用 Spring Boot 中的框架支持编写测试。**我们将介绍可以独立运行的单元测试以及在执行测试之前引导 Spring 上下文的集成测试。


如果您是 Spring Boot 新手，请查看我们的[Spring Boot 简介](https://www.baeldung.com/spring-boot-start)。



**2. 项目设置**
-----------



我们将在本文中使用的应用程序是一个 API，它提供对员工资源的一些基本操作。这是一个典型的分层架构——API调用从Controller到Service再到Persistence层进行处理。


**3.Maven依赖**
-------------



让我们首先添加我们的测试依赖项：



```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

spring [-boot-starter-test](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22)是主要依赖项，包含我们测试所需的大部分元素。




H2 [DB](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22)是我们的内存数据库。它消除了为测试目的配置和启动实际数据库的需要。


### 3.1. J单元4



从 Spring Boot 2.4 开始，JUnit 5 的老式引擎已从spring-boot-starter-test中删除。如果我们仍然想使用 JUnit 4 编写测试，我们需要添加以下 Maven 依赖项：



```
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**4.使用@SpringBootTest进行集成测试**
-----------------------------



顾名思义，集成测试侧重于集成应用程序的不同层。这也意味着不涉及任何嘲笑。


**理想情况下，我们应该将集成测试与单元测试分开，并且不应与单元测试一起运行。**我们可以通过使用不同的配置文件来仅运行集成测试来做到这一点。这样做的几个原因可能是集成测试非常耗时，并且可能需要实际的数据库来执行。


然而在本文中，我们不会关注这一点，而是使用内存中的 H2 持久存储。




集成测试需要启动一个容器来执行测试用例。因此，为此需要一些额外的设置——所有这些在 Spring Boot 中都很容易：



```
@RunWith(SpringRunner.class)
@SpringBootTest(
 webEnvironment = SpringBootTest.WebEnvironment.MOCK,
 classes = Application.class)
@AutoConfigureMockMvc
@TestPropertySource(
 locations = "classpath:application-integrationtest.properties")
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private EmployeeRepository repository;

    // write test cases here
}
```

**当我们需要引导整个容器时，@SpringBootTest注释非常有用。**该注释通过创建将在我们的测试中使用的ApplicationContext来工作。


我们可以使用@SpringBootTest的webEnvironment属性来配置我们的运行环境；我们在这里使用WebEnvironment.MOCK，以便容器可以在模拟 servlet 环境中运行。


接下来，@TestPropertySource注释帮助配置特定于我们的测试的属性文件的位置。请注意，使用@TestPropertySource加载的属性文件将覆盖现有的application.properties文件。


application-integrationtest.properties包含配置持久性存储的详细信息：





```
spring.datasource.url = jdbc:h2:mem:test
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect
```

如果我们想针对 MySQL 运行集成测试，我们可以更改属性文件中的上述值。


集成测试的测试用例可能看起来与控制器层单元测试类似：



```
@Test
public void givenEmployees\_whenGetEmployees\_thenStatus200()
  throws Exception {

    createTestEmployee("bob");

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(content()
      .contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$[0].name", is("bob")));
}
```

与控制器层单元测试的区别在于，这里没有任何内容被模拟，并且将执行端到端的场景。


5. 使用@TestConfiguration测试配置
---------------------------



正如我们在上一节中看到的，用@SpringBootTest注释的测试将引导完整的应用程序上下文，这意味着我们可以将组件扫描拾取的任何 bean @Autowire到我们的测试中：



```
@RunWith(SpringRunner.class)
@SpringBootTest
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // class code ...
}

```

但是，我们可能希望避免引导真实的应用程序上下文，而是使用特殊的测试配置。我们可以通过 @TestConfiguration注释来实现这一点。有两种使用注释的方法。在同一测试类中的静态内部类上，我们想要@Autowire bean：



```
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
        @Bean
        public EmployeeService employeeService() {
            return new EmployeeService() {
                // implement methods
            };
        }
    }

    @Autowired
    private EmployeeService employeeService;
}
```

或者，我们可以创建一个单独的测试配置类：



```
@TestConfiguration
public class EmployeeServiceImplTestContextConfiguration {
    
    @Bean
    public EmployeeService employeeService() {
        return new EmployeeService() { 
            // implement methods 
        };
    }
}
```

用@TestConfiguration注释的配置类 被排除在组件扫描之外，因此我们需要在每个想要@Autowire它的测试中显式导入它。我们可以使用 @Import 注释来做到这一点：



```
@RunWith(SpringRunner.class)
@Import(EmployeeServiceImplTestContextConfiguration.class)
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // remaining class code
}
```

**6.使用@MockBean进行模拟**
---------------------



我们的服务层代码依赖于我们的存储库：




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_1", slotId: "baeldung\_incontent\_1" });
 


```
@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Override
    public Employee getEmployeeByName(String name) {
        return employeeRepository.findByName(name);
    }
}
```

然而，要测试Service层，我们不需要了解或关心持久层是如何实现的。理想情况下，我们应该能够编写和测试我们的服务层代码，而无需连接完整的持久层。


为了实现这一点，**我们可以使用 Spring Boot Test 提供的模拟支持。**


我们先看一下测试类骨架：



```
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
 
        @Bean
        public EmployeeService employeeService() {
            return new EmployeeServiceImpl();
        }
    }

    @Autowired
    private EmployeeService employeeService;

    @MockBean
    private EmployeeRepository employeeRepository;

    // write test cases here
}
```

要检查Service类，我们需要创建Service类的一个实例并将其用作@Bean，以便我们可以在测试类中@Autowire它。我们可以使用@TestConfiguration注释来实现此配置。


这里另一个有趣的事情是@MockBean的使用。它为EmployeeRepository[创建一个 Mock](https://www.baeldung.com/mockito-mock-methods)，可用于绕过对实际EmployeeRepository 的调用：



```
@Before
public void setUp() {
    Employee alex = new Employee("alex");

    Mockito.when(employeeRepository.findByName(alex.getName()))
      .thenReturn(alex);
}
```

设置完成后，测试用例会更简单：



```
@Test
public void whenValidName\_thenEmployeeShouldBeFound() {
    String name = "alex";
    Employee found = employeeService.getEmployeeByName(name);
 
     assertThat(found.getName())
      .isEqualTo(name);
 }
```

**7. 使用@DataJpaTest进行集成测试**
---------------------------



我们将使用一个名为Employee 的实体，该实体有一个id和一个名称作为其属性：



```
@Entity
@Table(name = "person")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Size(min = 3, max = 20)
    private String name;

    // standard getters and setters, constructors
}
```

这是我们使用 Spring Data JPA 的存储库：




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_2", slotId: "baeldung\_incontent\_2" });
 


```
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

    public Employee findByName(String name);

}
```

这就是持久层代码。现在让我们开始编写我们的测试类。


首先，让我们创建测试类的骨架：



```
@RunWith(SpringRunner.class)
@DataJpaTest
public class EmployeeRepositoryIntegrationTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    // write test cases here

}
```

@RunWith(SpringRunner.class)提供了 Spring Boot 测试功能和 JUnit 之间的桥梁。每当我们在 JUnit 测试中使用任何 Spring Boot 测试功能时，都需要此注释。


@DataJpaTest提供了测试持久层所需的一些标准设置：


* 配置H2，一个内存数据库
* 设置 Hibernate、Spring Data 和DataSource
* 执行@EntityScan
* 打开 SQL 日志记录


为了执行数据库操作，我们需要数据库中已有一些记录。要设置此数据，我们可以使用TestEntityManager。


**Spring Boot TestEntityManager 是标准 JPA EntityManager的替代品，它提供了编写测试时常用的方法。**


EmployeeRepository是我们要测试的组件。


现在让我们编写第一个测试用例：





```
@Test
public void whenFindByName\_thenReturnEmployee() {
    // given
    Employee alex = new Employee("alex");
    entityManager.persist(alex);
    entityManager.flush();

    // when
    Employee found = employeeRepository.findByName(alex.getName());

    // then
    assertThat(found.getName())
      .isEqualTo(alex.getName());
}
```

在上面的测试中，我们使用 TestEntityManager在数据库中插入一个Employee并通过按名称查找 API 读取它。


assertThat (…)部分来自[Assertj 库](https://www.baeldung.com/introduction-to-assertj)，它与 Spring Boot 捆绑在一起。


**8.使用@WebMvcTest进行单元测试**
-------------------------



我们的Controller依赖于Service层；为了简单起见，我们只包含一个方法：



```
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    @Autowired
    private EmployeeService employeeService;

    @GetMapping("/employees")
    public List<Employee> getAllEmployees() {
        return employeeService.getAllEmployees();
    }
}
```

由于我们只关注控制器代码，因此很自然地为我们的单元测试模拟服务层代码：



```
@RunWith(SpringRunner.class)
@WebMvcTest(EmployeeRestController.class)
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private EmployeeService service;

    // write test cases here
}
```

**要测试控制器，我们可以使用@WebMvcTest。它将为我们的单元测试自动配置 Spring MVC 基础设施。**


在大多数情况下，@ WebMvcTest将仅限于引导单个控制器。我们还可以将它与@MockBean一起使用，为任何所需的依赖项提供模拟实现。


@WebMvcTest还自动配置MockMvc，它提供了一种强大的方法来轻松测试 MVC 控制器，而无需启动完整的 HTTP 服务器。


话虽如此，让我们编写测试用例：




 freestar.config.enabled\_slots.push({ placementName: "baeldung\_incontent\_4", slotId: "baeldung\_incontent\_4" });
 


```
@Test
public void givenEmployees\_whenGetEmployees\_thenReturnJsonArray()
  throws Exception {
    
    Employee alex = new Employee("alex");

    List<Employee> allEmployees = Arrays.asList(alex);

    given(service.getAllEmployees()).willReturn(allEmployees);

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$", hasSize(1)))
      .andExpect(jsonPath("$[0].name", is(alex.getName())));
}
```

get (…)方法调用可以替换为与 HTTP 动词对应的其他方法，例如put()、post()等。请注意，我们还在请求中设置内容类型。


MockMvc很灵活，我们可以使用它创建任何请求。


9. 自动配置测试
---------



Spring Boot 自动配置注释的惊人功能之一是它有助于加载完整应用程序的部分内容并测试代码库的特定层。


除了上面提到的注解之外，这里还列出了一些广泛使用的注解：


* @WebF luxTest：我们可以使用 @WebFluxTest注释来测试Spring WebFlux控制器。它通常与@MockBean一起使用 ，为所需的依赖项提供模拟实现。


* @JdbcTest：我们可以使用@JdbcTest注释来测试JPA应用程序，但它适用于仅需要数据源的测试。该注释配置内存中的嵌入式数据库和 JdbcTemplate 。


* @JooqTest：为了测试 jOOQ 相关的测试，我们可以使用 @JooqTest注解，它配置一个 DSLContext。


* @DataMongoTest：为了测试 MongoDB 应用程序，@DataMongoTest是一个有用的注释。默认情况下，如果驱动程序可通过依赖项使用，它会配置内存中的嵌入式 MongoDB、配置 MongoTemplate 、扫描@Document类并配置 Spring Data MongoDB 存储库。


* @DataRedisTest使测试 Redis 应用程序变得更加容易。它默认扫描@RedisHash类并配置 Spring Data Redis 存储库。
* @DataLdapTest 配置内存中嵌入式LDAP（如果可用），配置LdapTemplate，扫描@Entry类，并默认配置 Spring Data LDAP存储库。


* @RestClientTest：我们通常使用@RestClientTest注释来测试REST客户端。它自动配置不同的依赖项，例如 Jackson、GSON 和 Jsonb 支持；配置RestTemplateBuilder；并默认添加对MockRestServiceServer 的支持。
* @JsonTest：仅使用测试 JSON 序列化所需的那些 bean 初始化 Spring 应用程序上下文。


[您可以在我们的文章优化 Spring 集成测试](https://www.baeldung.com/spring-tests)中阅读有关这些注释以及如何进一步优化集成测试的更多信息。


**10. 结论**
----------



在本文中，我们深入研究了 Spring Boot 中的测试支持，并展示了如何高效地编写单元测试。


本文的完整源代码可以[在 GitHub 上找到](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)。源代码包含更多示例和各种测试用例。


如果您想继续学习测试，我们有与[集成测试](https://www.baeldung.com/integration-testing-in-spring)、[优化 Spring 集成测试](https://www.baeldung.com/spring-tests)和[JUnit 5 中的单元测试](https://www.baeldung.com/junit-5)相关的单独文章。





    