
# DML方法
前面我们介绍了查询操作，我们接着来看修改相关操作。Mybatis为我们的DML操作提供了几个预设方法：

```java
int insert(String statement);
int insert(String statement, Object parameter);
int update(String statement);
int update(String statement, Object parameter);
int delete(String statement);
int delete(String statement, Object parameter);
```

可以看到，这些方法默认情况下返回的结果都是`int`类型的，这与我们之前JDBC中是一样的，它代表执行SQL后受影响的行数。


## insert
我们来尝试编写一个插入操作，Mybatis为我们提供的插入操作非常快捷，我们可以直接让一个User对象作为参数传入，即可在配置中直接解析其属性到insert语句中，这里需要用到insert标签：

```xml
<insert id="addUser" parameterType="com.test.entity.User">
    insert into user (name, age) values (#{name}, #{age})
</insert>
```

这里我们将parameterType类型设置为我们的实体类型，这样下面在使用`#{name}`时Mybatis就会自动调用类中对应的Get方法来获取结果，不过，即使这里不指定具体类型，Mybatis也能完成自动推断，非常智能。

### 插入时自增主键的获取
有些时候，我们的数据插入后使用的是一个自增主键ID，那么这个自增的主键值我们该如何获取到呢？Mybatis为我们提供了一些参数用于处理这种问题：

```xml
<insert id="addUser" parameterType="com.test.entity.User" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
    insert into user (name, age) values (#{name}, #{age})
</insert>
```

这里`useGeneratedKeys`设置为`true`表示我们希望获取数据库生成的键，`keyProperty`设置为User类中的需要获取自增结果的属性名，`keyColumn`为数据库中自增的字段名称，但是一般情况下不需要手动设置，但是某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，必须设置。

这样我们就可以获取到自增后的值了，接着我们什么都不需要做，Mybatis会在查询完后自动为我们的User对象赋值：

![QQ_1723824700173](https://oss.itbaima.cn/internal/markdown/2024/08/17/OUWu97HmpPNyB1K.png)

是不是感觉很方便？和之前一样，我们也可以直接将其绑定到一个接口上：

```java
public interface TestMapper {
    int addUser(User user);
}
```

注意返回类型必须是int或是long这类数字类型，表示生效的行数，然后这里我们传入的参数直接写成对应的类型即可。



## update
我们接着来看修改操作，比如要根据ID修改用户的年龄：

```xml
<update id="setUserAgeById">
    update user set age = #{age} where id = #{id}
</update>
```

```java
int setUserAgeById(User user);
```

这里的参数我们依然选择使用User，和之前insert一样，Mybatis会从传入的对象中自动获取需要的参数，当然我们也可以将此方法设计为两个参数的形式：

```java
int setUserAgeById(@Param("age") int age, @Param("id") int id);
```


## delete
删除操作则更为简单，假设我们要根据用户的id进行数据的删除：


```xml
<delete id="deleteUserById">
    delete from user where id = #{id}
</delete>
```

这些操作相比查询操作来说非常简单就可以实现，这里就不多做介绍了。