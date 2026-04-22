


# 其他属性
除了三个基本的属性用于定位坐标外，依赖还可以添加以下属性：

- **type**：依赖的类型，对于项目坐标定义的packaging。大部分情况下，该元素不必声明，其默认值为jar
- **scope**：依赖的范围（作用域，着重讲解）
- **optional**：标记依赖是否可选
- **exclusions**：用来排除传递性依赖（一个项目有可能依赖于其他项目，就像我们的项目，如果别人要用我们的项目作为依赖，那么就需要一起下载我们项目的依赖，如Lombok）


## scope
我们着重来讲解一下`scope`属性，它决定了依赖的作用域范围：

- **compile** ：默认的依赖有效范围，如果在定义依赖关系的时候，没有明确指定依赖有效范围的话，则默认采用该依赖有效范围，此范围表示在编译、运行、测试时均有效。
- **provided** ：仅在编译、测试时有效，但是在运行时无效，也就是说，项目在运行时，不需要此依赖，比如我们上面的Lombok，我们只需要在编译阶段使用它，编译完成后，实际上已经转换为对应的代码了，因此Lombok不需要在项目运行时也存在。
- **runtime** ：在运行、测试时有效，但是在编译代码时无效。比如JDBC驱动就是典型的只需要运行时使用，因为JDBC驱动由数据库厂商开发，我们使用的始终是JDK中提供的接口，不需要直接使用特定驱动中的类或是方法，因此只需在运行时包含即可。
- **test** ：只在测试时有效，例如：JUnit框架，我们一般只会在测试阶段使用JUnit，而实际项目运行时，我们就用不到测试了，所以这个选项非常适合测试相关的框架。

### 不同作用域详解
#### 1. `compile` (编译作用域)

这是 **默认值**。如果你不写 `scope`，Maven 就会认为是 `compile`。

- **特点**：在所有阶段都有效，并且会随着项目一起打包发布。
    
- **传递性**：如果 A 依赖 B（compile），B 依赖 C（compile），那么 A 也会依赖 C。
    

#### 2. `provided` (已提供作用域)

这意味着你预期 **JDK 或者容器（如 Tomcat）会在运行时提供这个依赖**。

- **典型场景**：`servlet-api`。你在写代码时需要它，但打包成 `war` 包后，Tomcat 内部已经自带了这个 jar 文件夹，如果你的包里再带一个，就会产生版本冲突。
    
- **打包**：不会被打入最终的 `jar/war` 包。
    

#### 3. `runtime` (运行时作用域)

编译时不需要（代码里不直接调用），但程序跑起来后需要。

- **典型场景**：JDBC 驱动实现。代码里我们通常只用 `java.sql.Connection`（这是 JDK 的标准接口），只有在连接数据库时才需要具体的驱动。
    

#### 4. `test` (测试作用域)

只在单元测试（`src/test/java`）阶段有效。

- **典型场景**：`JUnit`。项目正常上线运行是不需要测试框架的，所以它不会被打包。
    

#### 5. `system` (系统作用域)

和 `provided` 类似，但它不从仓库找，而是通过 `<systemPath>` 指定本地磁盘的绝对路径。

- **注意**：不推荐使用，因为换台电脑可能路径就失效了，导致构建失败

### 导入测试

#### 导入JUnit

这里我们来测试一下JUnit，我们可以在网站上搜索JUnit的依赖，我们这里导入最新的JUnit5作为依赖：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

我们所有的测试用例全部编写到Maven项目给我们划分的test目录下，位于此目录下的内容不会在最后被打包到项目中，只用作开发阶段测试使用

```java
public class MainTest {

    @Test
    public void test(){
        System.out.println("测试");
      	//Assert在JUnit5时名称发生了变化Assertions
        Assertions.assertArrayEquals(new int[]{1, 2, 3}, new int[]{1, 2});
    }
}
```

因此，一般仅用作测试的依赖如JUnit只保留在测试中即可，

#### 导入JDBC和Mybatis
那么现在我们再来添加JDBC和Mybatis的依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.27</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>
```

我们发现，Maven还给我们提供了一个`resource`目标，我们可以将一些静态资源，比如配置文件，放入到这个文件夹中，项目在打包时会将资源文件夹中文件一起打包的Jar中，比如我们在这里编写一个Mybatis的配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="cacheEnabled" value="true"/>
        <setting name="logImpl" value="JDK_LOGGING" />
    </settings>
    <!-- 需要在environments的上方 -->
    <typeAliases>
        <package name="com.test.entity"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/web_study"/>
                <property name="username" value="test"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper class="com.test.mapper.TestMapper"/>
    </mappers>
</configuration>
```

现在我们创建一下测试用例，顺便带大家回顾一下JUnit5的使用：

```java
public class MainTest {

    //因为配置文件位于内部，我们需要使用Resources类的getResourceAsStream来获取内部的资源文件
    private static SqlSessionFactory factory;

    //在JUnit5中@Before被废弃，它被细分了：
    @BeforeAll // 一次性开启所有测试案例只会执行一次 (方法必须是static)
    // @BeforeEach 一次性开启所有测试案例每个案例开始之前都会执行一次
    @SneakyThrows
    public static void before(){
        factory = new SqlSessionFactoryBuilder()
                .build(Resources.getResourceAsStream("mybatis.xml"));
    }


    @DisplayName("Mybatis数据库测试")  //自定义测试名称
    @RepeatedTest(3)  //自动执行多次测试
    public void test(){
        try (SqlSession sqlSession = factory.openSession(true)){
            TestMapper testMapper = sqlSession.getMapper(TestMapper.class);
            System.out.println(testMapper.getStudentBySid(1));
        }
    }
}
```


那么就有人提问了，如果我需要的依赖没有上传的远程仓库，而是只有一个Jar怎么办呢？我们可以使用第四种作用域：

- **system**：作用域和provided是一样的，但是它不是从远程仓库获取，而是直接导入本地Jar包。

```xml
<dependency>
     <groupId>javax.jntm</groupId>
     <artifactId>lbwnb</artifactId>
     <version>2.0</version>
     <scope>system</scope>
     <systemPath>C://学习资料/4K高清无码/test.jar</systemPath>
</dependency>
```

比如上面的例子，如果scope为system，那么我们需要添加一个systemPath来指定jar文件的位置，这里就不再演示了。