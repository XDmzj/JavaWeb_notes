[[MyBatis的依赖关系.canvas]]

# 查询操作
配置mybatis-config.xml
配置Mappers
java程序中创建Factory、Session、返回结果对象
java执行，得到结果

前面大概熟悉了一下Mybatis的配置流程，这一节我们来详细介绍一下如何配置查询操作。由于`SqlSessionFactory`一般只需要创建一次，因此我们可以创建一个工具类来集中创建`SqlSession`，这样会更加方便一些：


## 创建工具类
```java
public class MybatisUtil {

    //在类加载时就进行创建
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(new FileInputStream("mybatis-config.xml"));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取一个新的会话
     * @param autoCommit 是否开启自动提交（跟JDBC是一样的，如果不自动提交，则会变成事务操作）
     * @return SqlSession对象
     */
    public static SqlSession openSession(boolean autoCommit){
        return sqlSessionFactory.openSession(autoCommit);
    }
}
```


现在我们只需要在main方法中直接使用工具类就能快速创建一个新的会话，然后查询结果了：

```java
try(SqlSession sqlSession = MybatisUtil.openSession(true)) {
    List<User> users = sqlSession.selectList("selectUser");
    users.forEach(System.out::println);
}
```

## 例：根据id查询
查询操作在XML配置中使用一个select标签进行囊括，这里我们通过一个例子来介绍一下最基本的查询需要配置的参数，假设我们现在需要编写一个根据ID查询用户的操作，首先我们需要指定它的id，建议把id名称起的有代表性一点：

```xml
<select id="selectUserById">  
</select>
```

### 参数问题
接着是我们需要进行查询的参数，这里我们需要根据用户ID查询，那么传入的参数就是一个int类型的参数，参数也可以是字符串类型的，类型名称：

1. 如果是基本类型，需要使用`_int`这样前面添加下划线。
2. 如果是JDK内置的包装类型或是其他类型，可以直接使用其名称，比如`String`、`int`（Integer的缩写）、`Long`
3. 如果是自己编写的类型，需要完整的包名+类名才可以。

当然，如果各位小伙伴觉得非常麻烦，我们也可以直接不填这个属性，Mybatis会自动判断：

```xml
<select id="selectUserById" parameterType="int">
</select>
```


#### xml中sql语句该怎么写
接下来就是编写我们的SQL语句了，由于这里我们需要通过一个参数来查询，所以需要填入一个占位符，通过使用`#{xxx}`或是`${xxx}`来填入我们给定的属性，名称我们先随便起一个：

```xml
<select id="selectUserById" parameterType="int">
    select * from user where id = #{id}
</select>
```

实际上Mybatis也是通过`PreparedStatement`首先进行一次预编译，来有效地防止SQL注入问题，但是如果使用`${xxx}`就不再是通过预编译，而是直接传值，因此对于常见的一些查询参数，我们一般都使用`#{xxx}`来进行操作保证安全性。

### 指定结果类型

最后我们查询到结果后，一般都是将其转换为对应的实体类对象，所以说这里我们之间填写之前建好的实体类名称，使用resultType属性来指定：

```xml
<select id="selectUserById" parameterType="int" resultType="com.test.User">
    select * from user where id = #{id}
</select>
```

当然，如果你觉得像这样每次都要写一个完整的类名太累了，也可以为它起个别名，我们只需要在Mybatis的配置文件中进行编写即可：

```xml
<typeAliases>
    <typeAlias type="com.test.User" alias="User"/>
</typeAliases>
```

也可以直接扫描整个包下的所有实体类，自动起别名，默认情况下别名就是类的名称：

```xml
<typeAliases>
    <package name="com.test.entity"/>
</typeAliases>
```
### 结果
这样，SQL语句映射配置我们就编写好了，接着就是Java这边进行调用了：

```java
//这里我们填写刚刚的id，然后将我们的参数填写到后面
User user = session.selectOne("selectUserById", 1);
System.out.println(user);
```

这样就可以成功查询到ID为1的用户信息了：

![QQ_1723625949253](https://oss.itbaima.cn/internal/markdown/2024/08/14/iQcgRtofhPlDU7T.png)




## resultMap
可以看到Mybatis直接省去了我们之前使用JDBC读取ResultSet的部分，直接转换为对应的实体类，当然，如果你不需要转换为实体类，Mybatis也为我们提供了多种转换方案，比如转换为一个Map对象：

```java
//使用Map类型变量进行接受，Key为String类型，Value为Object类型
Map<String, Object> user = session.selectOne("selectUserById", 1);
System.out.println(user);
```

我们可以尝试接着来写一个同时查询ID和年龄的查询操作，为了方便这里我们就不写parameterType属性了：

```xml
<select id="selectUserByIdAndAge" resultType="hashmap">
    select * from user where id = #{id} and age = #{age}
</select>
```

因为这里需要多个参数，我们可以使用一个Map或是具有同样参数的实体类来传递，显然Map用起来更便捷一些，注意key的名称需要与我们编写的SQL语句中占位符一致：

```java
User user = session.selectOne("selectUserByIdAndAge", Map.of("id", 1, "age", 18));
System.out.println(user);
```


## 实体类中定义的属性名称和数据库中的名称不一样

我们接着来看下面这种情况，实体类中定义的属性名称和我们数据库中的名称似乎有点不太一样，这会导致Mybatis自动处理出现问题：

```java
@Data
public class User {
    int uid;
    String username;但是数据库中是name
    int age;
}
```

运行后发现，Mybatis虽然可以查询到对应的记录，但是转换的实体类数据并没有被添加上去，这是因为数据库字段名称与类中字段名称不匹配导致的，我们可以手动配一个resultMap来解决这种问题，直接在Mapper中添加：

```xml
<select id="selectUserByIdAndAge" resultMap="test"> 
								resultMap意思是返回的result与这里设定的Map进行映射
								具体映射关系写在下面
    select * from user where id = #{id} and age = #{age}
</select>
<resultMap id="test" type="com.test.User">User是原来的resultType中的实体类
  	<!-- 因为id为主键，这里也可以使用<id>标签，有助于提高性能 -->
    <result column="id" property="uid"/>
    <result column="name" property="username"/>
</resultMap>
```

这里我们在resultMap标签中配置了一些result标签，每一个result标签都可以配置数据库字段和类属性的对应关系，这样Mybatis就可以按照我们的配置来正确找到对应的位置并赋值了，没有手动配置的字段会按照之前默认的方式进行赋值。配置完成后，最终只需要将resultType改为resultMap并指定对应id即可，然后就能够正确查询了。



##selectMap
我们接着来看一个比较特殊的选择方法`selectMap`，它可以将查询结果以一个Map的形式表示，只不过这和我们之前说的Map不太一样，它返回的Map是使用我们想要的属性作为Key，然后得到的结果作为Value的Map，它适用于单个数据查询或是多行数据查询：

```java
//最后一个参数为我们希望作为key的属性
Map<String, User> user = session.selectMap("selectUserById", 1, "id");
```

此时得到的结果就是：

![QQ_1723711033070](https://oss.itbaima.cn/internal/markdown/2024/08/15/skVRAyvo8KqjxE7.png)

可以看到这个Map中确实使用的是id作为Key，然后查询得到的实体对象作为Value。

## selectCursor
还有一个比较特殊的选择操作是`selectCursor`，它可以得到一个`Cursor`对象，同样是用于列表查询的，只不过使用起来和我们之前JDBC中的ResultSet比较类似，也是通过迭代器的形式去进行数据的读取，官方解释它主要用于惰性获取数据，提高性能：

```java
public interface Cursor<T> extends Closeable, Iterable<T> { ... }
```

可以看到它本身是实现了Iterable接口的，表明它可以获取迭代器或是直接使用foreach来遍历：

```java
Cursor<User> cursor = session.selectCursor("selectUsers");
for (User user : cursor) {
    System.out.println(user);
}
```

只不过这种方式在大部分请情况下还是用的比较少，我们主要还是以`selectOne`和`selectList`为主。


## 普通的select
最后还有一个普通的`select`方法，它支持我们使用Lambda的形式进行查询结果的处理：

```java
session.select("selectUsers", context -> {  //使用ResultHandler来处理结果
    System.out.println(context.getResultObject());
});
```

结果会自动进行遍历并依次执行我们传入的Lambda表达式。