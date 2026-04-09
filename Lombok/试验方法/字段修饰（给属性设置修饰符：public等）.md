
### 默认字段修饰

默认字段修饰可以为类中字段快速生成对应的修饰符，避免我们手动进行编写。只需添加`@FieldDefaults`即可：

```java
@FieldDefaults(level = AccessLevel.PRIVATE)
public class Account {
    int id;
    String name;
    int age;
}
```

这样生成的代码中所有字段就自动变成private了：

```java
public class Account {
    private int id;
    private String name;
    private int age;
  	...
}
```

这个注解比较简单，无需多说。