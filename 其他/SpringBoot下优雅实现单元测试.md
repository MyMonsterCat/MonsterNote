Hello，这里是Monster

又是一个周末，最近正在研究Springboot下如何进行测试用例的编写，以及如何完成至少70%测试用例的覆盖。

经过一番学（zhe）习（mo），对于测试用例这块已经有了大致的了解，总结了一些心得和教程供大家参考

> 此为基础理论，后续会有进阶与真实项目实战

## 为什么要使用 mock

### 容器启动需花费时间

现在的java项目几乎离不开spring框架，而其最为著名的就是IOC，所有的bean用容器来管理，所以这给我们单元测试带来一个问题，如果要对bean做单元测试，就需要启动容器，那么带来的时间的开销将会很大。而Mockito给我门带来了一系列的解决方法，让我们可以轻松的对bean 进行测试。

> 不得不吐槽一下，改一下就要启动一次重新测试，涉及数据库的操作需要进行删数据，在没接触mock之前测试太痛苦了

### 特定情况很难测试

比如传入null，抛出异常测试，数据库指定返回某条数据等等，有的很容易通过造数据实现，但是来回切换工具是不是有点太麻烦了。

## mock是什么

Mock 可以理解为创建一个虚假的对象，或者说模拟出一个对象，在测试环境中用来替换掉真实的对象，以达到我们可以：

- 验证该对象的某些方法的调用情况，调用了多少次，参数是多少
- 给这个对象的行为做一个定义，来指定返回结果或者指定特定的动作

### 如何使用mock

在java maven环境中，推荐使用方式一，不用关心版本冲突

### 方式一：使用Spring全家桶

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/>
    </parent>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
```

### 方式二：单独引入

```xml
    <!-- junit5 -->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.8.2</version>
      <scope>test</scope>
    </dependency>

    <!-- mockito -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>4.6.1</version>
      <scope>test</scope>
    </dependency>

    <!-- mockito 的junit5适配器 -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-junit-jupiter</artifactId>
      <version>4.6.1</version>
      <scope>test</scope>
    </dependency>
```

## Mockito 的正确使用方式

### 第一步：mock一个对象

在使用某个方法或者某个类时（比如XxxserviceImpl），由于我们不需要启动Spring容器，而真正执行的时候，需要用到XxxserviceImpl，所以需要模拟一个虚假的XxxserviceImpl，让我们在执行测试的时候可以使用到XxxserviceImpl

#### 方式一：使用Mockito.mock方法

mock 方法来自 `org.mockito.Mock`，它表示可以 mock 一个对象或者是接口。

```java
public static <T> T mock(Class<T> classToMock)
// classToMock：待 mock 对象的 class 类。
// T ：返回 mock 出来的类  
```

```java
//实例：使用 mock 方法 mock 一个Random类
Random random = Mockito.mock(Random.class);
```

#### 方式二：使用@Mock 注解

@Mock注解 等效于 Mockito.mock

***注意：mock 注解需要搭配 MockitoAnnotations.openMocks(testClass) 方法一起使用。***

```java
@Mock
private Random random;

@Test
void check() {
  	// 一定要加这行，意思为开启mock
    MockitoAnnotations.openMocks(this);
    // 下面两行的意思请继续往下看第二步
    Mockito.when(random.nextInt()).thenReturn(100);
    Assertions.assertEquals(100, random.nextInt());
}
```

#### 方式三：使用@Spy注解

被 spy 的对象会走真实的方法，而 mock 对象不会

```java
@Spy
private CheckAuthorityImpl checkAuthority;

@Test
void check() {
    // 一定要加这行，意思为开启mock
    MockitoAnnotations.openMocks(this);
    // checkAuthority中add方法： return a+b
    int res = checkAuthority.add(1, 2);
  	// 3==0，结果为:true
    Assertions.assertEquals(3, res);
}
```

而如果我们使用@Mock注解

***注意：当使用 mock 对象时，如果不对其行为进行定义，则 mock 对象方法的返回值为返回类型的默认值。***

```java
@Mock
private CheckAuthorityImpl checkAuthority;

@Test
void check() {
    // 一定要加这行，意思为开启mock
  	MockitoAnnotations.openMocks(this);
  	// 不会去调用 checkAuthority中add方法，会返回类型默认值，即0
    int res = checkAuthority.add(1, 2);
  	// 3!=0，结果为:false
    Assertions.assertEquals(3, res);
}
```

#### 方式四：使用Mockito.spy方法

同@Spy注解一样，是调用真实的方法。

它跟Mockito.mock方法的区别在于spy() 方法的参数是对象实例，mock 的参数是 class

```java
@Test
void check() {
    CheckAuthorityImpl checkAuthority = Mockito.spy(new CheckAuthorityImpl());

    CheckAuthorityImpl checkAuthority1 = Mockito.mock(CheckAuthorityImpl.class);
 }
```

### 第二步：给 Mock 对象打桩

打桩可以理解为：为 mock 对象规定一系列行为，使其按照我们的要求来执行具体的操作。在 Mockito 中，常用的打桩方法为

| 方法                      | 含义                                    |
| ------------------------- | --------------------------------------- |
| when().thenReturn()       | Mock 对象在触发指定行为后返回指定值     |
| when().thenThrow()        | Mock 对象在触发指定行为后抛出指定异常   |
| when().doCallRealMethod() | Mock 对象在触发指定行为后调用真实的方法 |

```java
// thenReturn() 代码示例
@Test
void check() {
    Random random = Mockito.mock(Random.class, "test");
    // 意思为：当执行random.nextInt()时返回100
    Mockito.when(random.nextInt()).thenReturn(100);
    Assertions.assertEquals(100, random.nextInt());
}
```

### 第三步：对 Mock 出来的对象进行行为验证和结果断言

#### 行为验证

验证是校验待验证的对象是否发生过某些行为。Mockito 中验证的方法是：verify。

```java
@Test
void check() {
    Random random = Mockito.mock(Random.class, "test");
    System.out.println(random.nextInt());
    // Verify 配合 time() 方法，可以校验某些操作发生的次数。
    Mockito.verify(random,Mockito.times(2)).nextInt();
}
```

#### 结果断言

判断结果是不是服务预期。断言使用到的类是 Assertions.

```java
Random random = Mockito.mock(Random.class, "test");
Assertions.assertEquals(100, random.nextInt());
//输出结果：
//org.opentest4j.AssertionFailedError: 
//Expected :100
//Actual   :0
```



## 常用的其他方法或注解

### mockStatic

使用 `mockStatic()` 方法来 mock静态方法的所属类，此方法返回一个具有作用域的模拟对象。

```java
@Test
void range() {
    MockedStatic<StaticUtils> utilities = Mockito.mockStatic(StaticUtils.class);
    utilities.when(() -> StaticUtils.range(2, 6)).thenReturn(Arrays.asList(10, 11, 12));
    Assertions.assertTrue(StaticUtils.range(2, 6).contains(10));
}
@Test
void name() {
    MockedStatic<StaticUtils> utilities = Mockito.mockStatic(StaticUtils.class);
    utilities.when(StaticUtils::name).thenReturn("bilibili");
    Assertions.assertEquals("1", StaticUtils.	name());
}
```

### @BeforeEach 

在执行测试前需要执行的操作

```java
@Mock
private Random random;

@BeforeEach
void setUp() {
    System.out.println("----测试开始----");
}

@Test
void check() {
    MockitoAnnotations.openMocks(this);
    Mockito.when(random.nextInt()).thenReturn(100);
    Assertions.assertEquals(100, random.nextInt());
}
```

### @AfterEach

在执行完测试后需要执行的操作

```java
@Mock
private Random random;

@Test
void check() {
    MockitoAnnotations.openMocks(this);
    Mockito.when(random.nextInt()).thenReturn(100);
    Assertions.assertEquals(100, random.nextInt());
}

@AfterEach
void after() {
    System.out.println("----测试结束----");
}
```

### @InjectMocks

创建一个实例，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中。

```java
@InjectMocks
@Spy
private DBSourceService testService;
@Mock
private JdbcTemplate jdbcTemplate;
```

在上面的代码中，DBSourceService中涉及到了数据库操作，所以用到了JdbcTemplate，即在容器DBSourceService中用到了容器JdbcTemplate，使用@InjectMocks便可以将JdbcTemplate注入到DBSourceService中！**这非常重要！！**

## 总结

### IDEA中生成测试的通用步骤

在需要生成测试的类中：单击鼠标右键 --> 生成 --> 测试 --> 选择junit5和需要测试的方法 --> 确定

![](https://teamblog-1254331889.cos.ap-guangzhou.myqcloud.com/monster/202207031612453.png)

### 通用测试模板

```java
    @InjectMocks
    @Spy
    private XxxService XxxService;
    @Mock
    private XXXMapper XxxMapper;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void listUsers() {
        // 模拟Service调用
        Mockito.when(XxxService.listUsers())..thenCallRealMethod();
        // 模拟Mapper查询数据库
        Mockito.when(XxxMapper.queryForList(name)).thenReturn(new ArrayList());
        // 调用+断言
        Assertions.assertEquals(testService.listUsers().size(),0);
    }

```



> 未完待续...



**参考文章**

[单元测试如何提升代码覆盖率](https://www.bilibili.com/video/BV1g94y1Z79d?spm_id_from=333.880.my_history.page.click&vd_source=04bbf5cbf6b70328b6f7302873cbe54a)  |  [官方文档](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)  | [Java编程技巧之单元测试用例编写流程](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247503385&idx=1&sn=cb98877a669669b6af6fbd95fb310e10&chksm=e92af316de5d7a00c704f852560ee99d57fc702e545cc3dd02e97f0c6675992f5701b47a64e6&scene=178&cur_album_id=1538305828238262273#rd)
