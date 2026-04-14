
# 概念
除了使用批处理之外，Mybatis还为我们提供了一种更好的方式来处理这种问题，我们可以使用动态SQL来一次性生成一个批量操作的SQL语句，这里先介绍一下什么是动态SQL。

> 动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

简单来说，动态SQL在执行时可以进行各种条件判断以及循环拼接等操作，极大地提升了SQL语句编写的的灵活性。


## \<if/> 标签
我们先来看看条件判断，在编写SQL时，我们可以添加一些用于条件判断的标签到SQL语句中，比如我们希望在根据ID查询用户时，如果查询的ID大于3，那么必须同时要满足大于18岁这个条件，这看似是一个很奇怪的查询条件，但是在我们进入到公司后确实可能会遇到这些奇葩需求，此时动态SQL就能很轻松实现这个操作：

```xml
<select id="selectUserById" resultType="User">
    select * from user where id = #{id}
    <if test="id > 3">
        and age > 18
    </if>
</select>
```

这里我们使用`if`标签表示里面的内容会在判断条件满足时拼接到后面，如果不满足，那么就不拼接里面的内容到原本的SQL中，其中test属性就是我们需要填写的判断条件，它采用OGNL表达式进行编写，语法与Java比较相似，如果各位小伙伴比较感兴趣也可以前往：[https://commons.apache.org/dormant/commons-ognl/](https://commons.apache.org/dormant/commons-ognl/) 详细了解。

可以看到，当我们查询条件不同时，Mybatis会选择性拼接我们的SQL语句：

![QQ_1724052613799](https://oss.itbaima.cn/internal/markdown/2024/08/19/fSOXZdTjFR4oMgP.png)



## \<choose/>\<when/> \<otherwise/>
除了if操作之外，Mybatis还针对多分支情况提供了choose操作，它类似于Java中的switch语句，比如现在我们希望在查询用户时，ID等于1的必须同时要满足小于18岁，ID等于2的必须满足等于18岁，其他情况的必须满足大于18岁，我们可以像这样进行编写：

```xml
<select id="selectUserById" resultType="User">
    select * from user where id = #{id}
    <choose>
        <when test="id == 1">
             and age &lt;= 18
        </when>
        <when test="id == 2">
            and age = 18
        </when>
        <otherwise>
            and age > 18
        </otherwise>
    </choose>
</select>
```

注意在`when`中不允许使用`<`或是`>`这种模糊匹配的条件。

## <foreach/>
最后我们再来介绍一下`foreach`操作，它与Java中的for类似，可以实现批量操作，这非常适合处理我们前面说的批量执行SQL的问题：

```java
for (int i = 1; i <= 5; i++) {
    mapper.deleteUserById(i);
}
```

但是实际上这种情况完全可以简写为一个SQL语句：

```sql
DELETE FROM users WHERE id IN (1, 2, 3, 4, 5);
```

所以，现在我们使用foreach来完成它就很简单了：

```xml
<delete id="deleteUsers">
    delete from user where id in
    <foreach collection="list" item="item" index="index" open="(" separator="," close=")">
        #{item}
    </foreach>
</delete>
```

其中collection就是我们需要遍历的集合或是数组等任意可迭代对象，
item和index分别代表我们在foreach标签中使用每一个元素和下标的变量名称，
最后open和close用于控制起始和结束位置添加的符号，
separator用于控制分隔符，现在执行以下操作：

```java
session.delete("deleteUsers", List.of(1, 2, 3, 4, 5));
```

最后实际执行的SQL为：

![QQ_1724059653501](https://oss.itbaima.cn/internal/markdown/2024/08/19/rFP1oJNM56ZebA4.png)


# 例子

是不是感觉有点那味了？我们再来看一个例子，比如现在我们想要批量插入一些用户到数据库里面，原本Java应该这样写，但是这是一种极其不推荐的做法：

```java
TestMapper mapper = session.getMapper(TestMapper.class);
List<User> users = List.of(new User("小美", 17),
        new User("小张", 18),
        new User("小刘", 19));
for (User user : users) {
    mapper.insertUser(user);
}
```

实际上这种操作完全可以浓缩为一个SQL语句：

```sql
INSERT INTO user (name, age) VALUES ('小美', 17), ('小张', 18), ('小刘', 19);
```

那这时又可以直接使用咱们的动态SQL来完成操作了：

```xml
<insert id="insertAllUser">
    insert into user (name, age) values
    <foreach collection="list" item="user" separator=",">
        (#{user.name}, #{user.age})
    </foreach>
</insert>
```

![QQ_1724060416443](https://oss.itbaima.cn/internal/markdown/2024/08/19/DRAt7hbdZizqpSk.png)

通过使用动态SQL语句，我们基本上可以解决大部分的SQL查询和批量处理场景了。