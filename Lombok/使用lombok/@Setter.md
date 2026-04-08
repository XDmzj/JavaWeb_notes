# 使用

我们接着来看`@Setter`注解，它与`@Getter`非常相似，用于生成字段对应的Setter方法：

```java
public class Account {
    @Setter
    private String name;
}
```

得到结果为：

```java
public class Account {
    private String name;

   	...
      
    public void setName(String name) {  //自动生成一个setter方法
        this.name = name;
    }
}
```

可以看到它同样会根据字段名称来生成Setter方法，其他参数与`@Getter`用法一致，这里就不重复介绍了。

## onParam

唯一一个不一样的参数为`onParam`，它可以在形参上的额外添加的自定义注解。

## 手动写Setter/Getter会导致lombok不生效

最后需要注意的是，如果我们手动编写了对应字段的Getter或是Setter方法（按照上述命名规则进行判断）那么Lombok提供的注解将不会生效，也不会覆盖我们自己编写的方法：

```java
public class Account {
    @Setter
    private String name;

    public void setName(int name) {   //即使上面添加Setter注解，这里也不会被覆盖，但是仅限于同名不同参的情况
        System.out.println("我是自定义的");
        this.name = name;
    }
}
```


	### @Tolerate
如果出现同名不同参数的情况导致误判，我们也可以使用`@Tolerate`注解使Lombok忽略它的存在，继续生成。
```java
public class Account {
    @Setter
    private String name;
	@Tolerate
    public void setName(int name) {   
		加了Tolerat之后，lombok就会将这个方法忽略，然后正常生成Setter
        System.out.println("我是自定义的");
        this.name = name;
    }
}

```

结果：
![[Pasted image 20260408112443.png]]