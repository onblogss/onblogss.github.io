---
layout: post
title: PowerMock单元测试踩坑与总结
date: 2019-01-03 11:06:00
author: 薛师兄
tags: Mock
---
## 1.Mock是什么？

通过提供定制的类加载器以及一些字节码篡改技巧的应用，PowerMock 现了对静态方法、构造方法、私有方法以及 Final 方法的模拟支持，对静态初始化过程的移除等强大的功能。

## 2.为什么要用PowerMock？

举个例子：当测试单机应用的时候，直接写Junit单元测试即可，但当涉及到多个服务时，你写好了你的服务，其它服务尚未完成，这时候就需要模拟调用远程服务，也就需要Mock。


## 3.Mock的流程

简单来说，模拟测试一共分为4步：数据准备、打桩（Mock）、执行、验证。

数据准备阶段可以为Mock阶段的准备期望值、参数等数据，执行mock对象的方法后，最后进行验证与判断。

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(PrivatePartialMockingExample.class)
public class PowerMockTest {
    @Test
    public void demoPrivateMethodMocking() throws Exception {
        //数据准备
        final String expected = "TEST VALUE";

        PrivatePartialMockingExample underTest = PowerMockito.spy(new PrivatePartialMockingExample());

        // mock || 打桩
//        PowerMockito.when(underTest, nameOfMethodToMock, input).thenReturn(expected);
        PowerMockito.doReturn(expected).when(underTest).methodToTest();

        // 执行
        String toTest = underTest.methodToTest();

        // 验证
        // doReturn 设置不会执行when()后的method()方法，依旧会获得和期望值一样的结果
        assertEquals(expected, toTest);
    }
}

public class PrivatePartialMockingExample {
    public String methodToTest() {
        System.out.println("public Method");
        return methodToMock("input");
    }

    private String methodToMock(String input) {
        System.out.println("private Method");
        return "REAL VALUE = " + input;
    }
}
```

## 4.Mock的细节

### @PrepareForTest(IdGenerator.class)  

告诉PowerMock为测试准备某些类,放在测试类和单独的测试方法。支持通配符 @PrepareForTest("com.mypackage.*")

### mock和spy

- 使用Mock生成的类，所有方法都不是真实的方法，而且返回值都是NULL。

- 使用Spy生成的类，所有方法都是真实方法，返回值都是和真实方法一样的。

### when和doReturn

when(dao.getOrder()).thenReturn("returened by mock "); 

doReturn(expected).when(underTest).methodToTest();

使用when去设置模拟返回值时，它里面的方法（dao.getOrder()）会先执行一次。

使用doReturn去设置的话，就不会产生上面的问题，因为有when来进行控制要模拟的方法，所以不会执行原来的方法。

### doNothing()和verify()

doNothing() 用于模拟void方法,其实不做任何事。

Mockito.verify() 验证某些方法使用了几次（默认1次），否定则抛出异常

```java
@Test
public void getTotalEmployee() {
    EmployeeService service = PowerMockito.mock(EmployeeService.class);

    //doNothing() 用于执行void方法,不做任何事
    PowerMockito.doNothing().when(service).getTotalEmployee();

    service.getTotalEmployee();

    //验证某些方法使用了几次（默认1次），否定则抛出异常
    Mockito.verify(service,Mockito.times(1)).getTotalEmployee();
}
```

### whenNew和withArguments

whenNew 模拟new行为，并不会真的创建对象，withArguments 传入构造函数的参数，无参使用 withNoArguments()

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(DirectoryStructure.class)
public class DirectoryStructureTest {
    @Test
    public void createDirectoryStructureWhenPathDoesntExist() throws Exception {
        final String directoryPath = "mocked path";

        File directoryMock = PowerMockito.mock(File.class);

        // 这就是如何告诉PowerMockito模拟新文件的构造。
        PowerMockito.whenNew(File.class).withArguments(directoryPath).thenReturn(directoryMock);

        
        //验证某个行为发生过几次（默认1次）
  		PowerMockito.verifyNew(File.class,times(0)).withArguments(directoryPath);

    }
}
```

### Arguments Matcher

一个作为参数的接口类，可以应对“根据不同参数返回不同值”的场景。


```java
PowerMockito.when(demoService.findNameById(Mockito.argThat(new ArgumentMatcher<String>() {
     @Override
     public boolean matches(String s) {
         if (s.equals("Jerry")) {
             return true;
         } else {
             return false;
         }
     }
 }))).thenReturn("Marry");
 System.out.println(demoService.findNameById("Jerry")); //返回Marry
 System.out.println(demoService.findNameById("NotJerry")); //返回null
```


### Answer interface

一个作为参数的接口类，可以应对“根据不同参数返回不同值”的场景。更强大。

```java
PowerMockito.when(demoService.findNameById(Mockito.anyString())).then(new Answer<Object>() {
    @Override
    public Object answer(InvocationOnMock invocationOnMock) {
        Object[] arguments = invocationOnMock.getArguments();
        String arg = (String) arguments[0];
        if (arg.equals("Jerry")){
            return "Marry";
        }else {
            return "Not";
        }
    }
});
System.out.println(demoService.findNameById("Jerry")); //返回Marry
System.out.println(demoService.findNameById("NotJerry")); //返回Not
```

### 任意数

```java
ArgumentMatchers.anyLong()
```

示例：

```java
PowerMockito.doReturn(application).when(Application).getById(ArgumentMatchers.anyLong());
```

注意：不可以出现

```java
PowerMockito.doReturn(application).when(Application).getById(ArgumentMatchers.anyLong(), object);
```

上面的这种形式，要么全用虚拟参数，要么不用。而且，ArgumentMatchers只适用于此处，不能在其它地方使用。

## 5.MockMVC、PowerMock集成Spring

首先看我的依赖，对于mockito-core的2.8.55版本我一直无法进行Maven下载，无奈手动下载导包。

```xml
<properties>
    <java.version>1.8</java.version>
    <powermock.version>1.7.1</powermock.version>
    <mock.version>2.8.55</mock.version>
</properties>

<dependencies>
    <!-- mockito-core 手动安装 -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>${mock.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-module-junit4</artifactId>
        <version>${powermock.version}</version>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-api-mockito2</artifactId>
        <version>${powermock.version}</version>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-module-testng</artifactId>
        <version>${powermock.version}</version>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-classloading-xstream</artifactId>
        <version>${powermock.version}</version>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-module-junit4-rule</artifactId>
        <version>${powermock.version}</version>
    </dependency>
    <dependency>
        <groupId>org.powermock</groupId>
        <artifactId>powermock-api-support</artifactId>
        <version>${powermock.version}</version>
    </dependency>

    <!--web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencies>
```

假如我的Controller层有这么一个方法

```java
@RestController
public class DemoController {
    @Autowired
    DemoService demoService;

    @RequestMapping("/demo")
    public User mapping(){
        return demoService.create();
    }

}
```

他的DemoService方法是下面这个简单的例子

```java
@Service
public class DemoService {

    public User create(){
        return new User("service",1);
    }

}
```

我只想测试这个接口，那么我可以这么写测试用例

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockdemoApplication.class)
public class MockdemoApplicationTests{

    @Autowired
    private MockMvc mockMvc;


    @Test
    public void contextLoads() throws Exception {
        MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get("/demo")).andReturn();
        String content = result.getResponse().getContentAsString();
        System.out.println(content);
    }

}
```

关于MockMvc需要我们手动注入，我写了一个Bean专门做注入，这个Bean实现了ApplicationListener接口，原因后面会讲到。

```java
@Configuration
public class AutoPowerMock implements ApplicationListener<ContextRefreshedEvent> {

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Bean
    public MockMvc getMockMvc() {
        return MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }
    
  }
```

这样执行我的测试用例，是可以完全成功的！

但需要考虑的是，如果在应用中需要消费其他服务的API。由于我依赖的服务并不由我所在的项目组维护（对方可能接口中途会发生变化，甚至，有时候可能并未启动）。集成测试成本略高，故而需要Mock测试。

对于任意一个类的Mock，开始我们已经讲过，那对于Spring的Bean的Mock，就需要多费点心思。

考虑到@Autowired注解会把Bean注入到测试类中，而且在程序运行时，调用的也都是Spring容器的Bean，所以一个简单的思路就是：**把Spring容器中的Bean替换成我们Mock后的Bean**。

这就用到上面的对ApplicationListener的实现类了，它的onApplicationEvent方法会在Bean加载完后调用，在调用时我们手动对Bean进行偷天换日，完整代码如下

```java
@Configuration
public class AutoPowerMock implements ApplicationListener<ContextRefreshedEvent> {

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Bean
    public MockMvc getMockMvc() {
        return MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        //这是需要Mock掉的Bean
        Class<?> value = DemoService.class;
        //获取BeanFactory
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) webApplicationContext.getAutowireCapableBeanFactory();
        //Mock
        Object mock = PowerMockito.mock(value);
        //替换
        String[] beanNames = beanFactory.getBeanNamesForType(value);
        for (String bean : beanNames) {
            beanFactory.removeBeanDefinition(bean);
            beanFactory.registerSingleton(bean, mock);
        }
    }

}
```

### 学习资源

[IBM的PowerMock教程](https://www.ibm.com/developerworks/cn/java/j-lo-powermock/index.html)

[Spring Boot、Dubbo项目Mock测试踩坑与总结](http://www.itmuch.com/dubbo/spring-boot-dubbo-mock/)

[SpringMVC测试mockMVC](https://www.cnblogs.com/lyy-2016/p/6122144.html)
