
# 安装

前面我们给大家介绍了依赖的导入方式和各种作用域，我们接着来看如何在其他项目中引入我们自己编写的Maven项目作为依赖使用。这里我们创建一个用于测试的简单项目：

我们项目的pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.test</groupId>
    <artifactId>TestMaven</artifactId>
    <version>1.0-SNAPSHOT</version>

    ...

</project>
```
我的项目的具体代码
```java
public class TestUtils {
    public static void test() {
        System.out.println("抛开事实不谈，你们就没有一点错吗？");
    }
}
```

接着我们点击右上角的Maven选项，然后执行`install`或直接在命令行中输入`mvn install`来安装我们自己的项目到本地Maven仓库中。

接着我们就可以在需要使用此项目作为依赖的其他项目中使用它了，只需要填写和这边一样的坐标：
其他项目的pom.xml
```xml
<dependency>
    <groupId>com.test</groupId>
    <artifactId>TestMaven</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

接着可以在项目中直接使用了

```java
public static void main(String[] args) {
    TestUtils.test();
}
```


# 可选
简单来说，当你在项目 A 中将某个依赖标注为 `optional=true` 时，这个依赖就变成了“可选”的，它**不会被自动传递**给依赖项目 A 的下游项目。

假设有三个项目：**项目 C $\rightarrow$ 项目 B $\rightarrow$ 项目 A**（C 依赖 B，B 依赖 A）。

- **如果不设置（默认情况）**：
    当项目 B 依赖项目 A 时，项目 C 会通过**依赖传递**自动获得项目 A。
- **如果设置 `<optional>true</optional>`**：
    1. **编译阶段**：项目 B 在编译时依然可以使用项目 A。
    2. **传递性失效**：项目 C 在依赖项目 B 时，**不会**自动下载和引入项目 A。
    3. **打包结果**：如果项目 B 打包，它可能包含 A；但项目 C 打包时，绝对不会包含 A。

## 设置为可选后，该怎么导入可选依赖

我该如何将可选依赖导入？

由于 `optional` 阻断了自动传递，如果你在项目 C 中确实需要使用那个被标注为可选的依赖（项目 A），你必须在项目 C 的 `pom.xml` 中**手动显式声明**它。
导入步骤：

1. **在下游项目（项目 C）中添加依赖**：
    你不能指望通过引入项目 B 来带入项目 A，你必须直接写出项目 A 的坐标。

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>project-b</artifactId>
        <version>1.0.0</version>
    </dependency>

    <dependency>
        <groupId>com.example</groupId>
        <artifactId>project-a</artifactId>
        <version>1.0.0</version>
        </dependency>
</dependencies>
```


# 排除

现在我们可以让使用此项目作为依赖的项目不使用可选依赖，
但是如果别人的项目中没有将我们不希望的依赖作为可选依赖，这就导致我们还是会连带引入这些依赖，
这个时候我们就可以通过排除依赖来防止添加不必要的依赖，只需添加`exclusion`标签即可：

```xml
<dependency>
    <groupId>com.test</groupId>
    <artifactId>TestMaven</artifactId>
    <version>1.1-SNAPSHOT</version>
    <exclusions>
        <exclusion>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
          	<!--  可以不指定版本号，只需要组名和项目名称  -->
        </exclusion>
    </exclusions>
</dependency>
```

此时我们通过这种方式手动排除了Test项目中包含的MyBatis依赖，这样项目中就不会包含此依赖了。
