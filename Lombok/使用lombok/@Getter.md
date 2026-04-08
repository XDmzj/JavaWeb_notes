# Getter源代码
我们从类**属性**相关注解开始介绍，首先是`@Getter`，它用于自动生成Getter方法，定义如下：

```java
@Target({ElementType.FIELD, ElementType.TYPE})   //此注解可以添加在字段或是类型上
@Retention(RetentionPolicy.SOURCE)
public @interface Getter {
    AccessLevel value() default AccessLevel.PUBLIC;  //自动生成的Getter的访问权限级别
    AnyAnnotation[] onMethod() default {};  //用于添加额外的注解
    boolean lazy() default false;   //懒加载功能
    ...
}
```

# 用法

它最简单的用法，就是直接添加到类上或是字段上：

```java
@Getter   //添加到类上时，将为类中所有字段添加Getter方法
public class Account {
    private int id;
    @Getter  //当添加到字段上时，仅对此字段生效
    private String name;
    private int age;
}
```

假设我们这里将`@Getter`编写在类上，那么生成得到的代码为：

```java
import lombok.Generated;

public class Account {
    private int id;
    private String name;
    private int age;

    public Account() {}

    @Generated
    public int getId() {   //自动为所有字段生成了对应的Getter方法
        return this.id;
    }

    ...省略
}
```

是不是感觉非常方便？而且使用起来也很灵活。注意它存在一定的命名规则，如果该字段名为`foo`，则将其直接按照字段名称命名为`getFoo`，但是注意，如果字段的类型为`boolean`，则会命名`isFoo`，这是比较特殊的地方。

## Getter属性
### AccessLevel value()

我们接着来看Getter注解的其他属性，首先是访问权限，默认情况下为public，但是有时候可能我们只希望生成一个private的get方法，此时我们可以对其进行修改：

- PUBLIC - 对应public关键字
- PACKAGE - 相当于不添加任何访问权限关键字
- PRIVATE - 对应private关键字
- PROTECTED - 对应protected关键字
- MODULE - 仅限模块内使用，与PACKAGE类似，相当于不添加任何访问权限关键字
- NONE - 表示不生成对应的方法，这很适合对类中不需要生成的字段进行排除

这里我们尝试将其更改为：

```java
@Getter(AccessLevel.PRIVATE)   //为所有字段生成private的Getter方法
public class Account {
    private int id;
    @Getter(AccessLevel.NONE)   //不为name生成Getter方法，字段上的注解优先级更高
    private String name;
    private int age;
}
```

得到的结果就是：

```java
public class Account {
    private int id;
    private String name;
    private int age;

    public Account() {
    }

    private int getId() {   //得到的就是private的Getter方法
        return this.id;
    }

    ...
}
```

### onMethod

我们接着来看它的`onMethod`属性，这个属性用于添加一些额外的注解到生成的方法上，比如我们要为Getter方法添加一个额外的`@Deprecated`表示它不推荐使用，那么：

```java
@Getter
public class Account {
    private int id;
    @Getter(onMethod_ = { @Deprecated })
    private String name;
    private int age;
}
```

此时得到的代码为：

```java
public class Account {
    ...

    /** @deprecated */
    @Deprecated   //由Lombok额外添加的注解
    public String getName() {
        return this.name;
    }
}
```

### lazy

最后我们再来看看它的`lazy`属性，这是用于控制懒加载

> 懒加载就是在一开始的时候此字段没有值，当我们需要的时候再将值添加到此处。

只不过它有一些要求，我们的字段必须是private且final的：

```java
public class Account {
    @Getter(lazy = true)
    private final String name = "你干嘛";
}
```

生成的代码如下

```java
public class Account {
  	//这里会自动将我们的字段修改为AtomicReference原子类型，以防止多线程环境下出现的问题
    private final AtomicReference<Object> name = new AtomicReference();

    ...

    //当我们调用getName才会去初始化字段的值，为了保证初始化只进行一次，整个过程与懒汉式单例模式一致
    public String getName() {
        Object $value = this.name.get();
        if ($value == null) {   //判断值是否为null，如果是则需要进行懒初始化
            synchronized(this.name) {    //对我们的字段加锁，保证同时只能进一个
                $value = this.name.get();
                if ($value == null) {    //再次进行一次判断，因为有可能其他线程后进入
                    String actualValue = "你干嘛";
                    $value = "你干嘛" == null ? this.name : "你干嘛";
                    this.name.set($value);
                }
            }
        }
				//返回得到的结果
        return (String)($value == this.name ? null : $value);
    }
}
```

有关原子类相关知识点，可以在JUC篇视频教程中进行学习，有关单例模式相关知识点，可以在Java设计模式篇视频教程中学习，这里不再赘述。我们作为使用者来说，只需要知道懒加载其实就是将字段的值延迟赋值给它了。比如下面这种场景就很适合：

```java
public class Account {
    @Getter(lazy = true)
    private final String name = initValue();

    private String initValue() {
        System.out.println("我不希望在对象创建时就执行");
        return "666";
    }
}
```

至此，有关`@Getter`注解相关的内容我们就介绍完毕