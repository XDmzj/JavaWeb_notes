### 强化Getter和Setter

虽然Lombok已经为我们提供了`@Getter`和`@Setter`注解，方便我们快速生成对应的get和set方法，但是我们还可以使其变得更加强大。Lombok为我们提供了一个`@Accessors`注解，用于配置lombok如何生成和查找getters和setters。

比如现在非常流行的链式设置属性，set方法直接返回当前对象，这样就可以一直set下去，很爽：

```java
public static void main(String[] args) {
    Account account = new Account();
    account.setId(1).setAge(18).setName("小明");
}
```


## chain
要实现这种方式也非常简单，我们只需要添加`@Accessors`注解即可：

```java
@Accessors(chain = true)  //将chain设置为true开启链式
@ToString
@Setter
public class Account {
    int id;
    String name;
    int age;
}
```


## fluent
我们接着来看看`@Accessors`的其他属性，首先是`fluent`，它可以直接去掉Getter或Setter的前缀，直接使用字段名称作为方法名称：


```java
public static void main(String[] args) {
    Account account = new Account();
    account.id(1).age(18).name("小明");    //大幅度简化代码
}
```

注意，开启`fluent`后默认也会启用`chain`属性。


## makefinal
我们接着来看`makeFinal`属性，它用于将所有生成的方法设置为final，防止子类进行修改：

```java
public final void setId(int id) {
    this.id = id;
}
```



## prefix（没什么用）
最后还有一个`prefix`属性，这个功能比较特殊，它可以在你有一些特殊命名的情况下使用，比如：

```java
public class Account {
    int userId;   //在命名时，有些人总爱添加点前缀
    String userName;
    int userAge;
}
```

这种情况下生成出来的Getter也会按照这样进行命名，非常不好用：

```java
public void setUserId(int userId) {
    this.userId = userId;
}
```

通过对`@Accessors`添加`prefix`属性，可以指示在生成Getter或Setter时去掉前缀，比如：

```java
@Accessors(prefix = "user")
@ToString
@Setter
public class Account {
    int userId;
    String userName;
    int userAge;
}
```

这样生成的方法就会直接去掉对应的前缀了。只不过，一旦设置了前缀，那么所有不是以此前缀开头的字段，会直接不生成对应的方法。如果各位小伙伴觉得`@Accessors`加到类上直接作用于全部字段，控制得不是很灵活，我们也可以将其单独放到某个字段上进行控制：

```java
@ToString
@Setter
public class Account {
    int userId;
    @Accessors(prefix = "user")
    String userName;
    int userAge;
}
```