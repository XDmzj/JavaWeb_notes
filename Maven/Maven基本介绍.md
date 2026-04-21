


Maven 翻译为"专家"、"内行"，是 Apache 下的一个纯 Java 开发的开源项目。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。Maven 曾是 Jakarta 项目的子项目，现为由 Apache 软件基金会主持的独立 Apache 项目。

通过Maven，可以帮助我们做：

- 项目的自动构建，包括代码的编译、测试、打包、安装、部署等操作。
- 依赖管理，项目使用到哪些依赖，可以快速完成导入，不需要手动导入jar包。

Maven也需要安装环境，但是IDEA已经自带了Maven环境，因此我们不需要再去进行额外的环境安装（无IDEA也能使用Maven，但是配置过程很麻烦，并且我们现在使用的都是IDEA的集成开发环境，所以这里就不讲解Maven命令行操作了）我们直接创建一个新的Maven项目即可。

# Maven项目结构

我们之前使用的都是最原始的Java项目目录格式，其中`src`目录直接包含我们的包以及对应的代码：

这是IDEA为我们提供的一种非常高效简洁的项目目录格式，虽然它用起来非常的简单方便，但是在管理依赖上，确实比较麻烦，我们得手动将我们需要的依赖以jar包的形式导入，光寻找这些jar包就得花费很多时间，并且不同的jar包还会依赖更多jar包，就像下崽一样，所以对于大型项目来说，这并不是一个很好的使用方式。

而Maven就很好地解决了这个问题，我们可以先来看一下，一个Maven项目和我们普通的项目有什么区别：

![[Pasted image 20260421210911.png]]

其中src目录下存放我们的源代码和测试代码，分别位于main和test目录下，而test和main目录下又具有java、resources目录，它们分别用于存放Java源代码、静态资源（如配置文件、图片等）、很多JavaWeb项目可能还会用到webapp目录。



# pom.xml配置文件

而下面的pom.xml则是Maven的核心配置，也是整个项目的所有依赖、插件、以及各种配置的集合，它也是使用XML格式编写的，一个标准的pom配置长这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itbaima</groupId>
    <artifactId>BookManage</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

</project>
```

我们可以看到，Maven的配置文件是以`project`为根节点，而`modelVersion`定义了当前模型的版本，一般是4.0.0，我们不用去修改。


## GAV
`groupId`、`artifactId`、`version`这三个元素合在一起，用于唯一区别每个项目，别人如果需要将我们编写的代码作为依赖，那么就必须通过这三个元素来定位我们的项目，我们称为一个项目的基本坐标，所有的项目一般都有自己的Maven坐标，因此我们通过Maven导入其他的依赖只需要填写这三个基本元素就可以了，无需再下载Jar文件，而是Maven自动帮助我们下载依赖并导入：

GroupID：项目组 / 组织的唯一标识
- **代表什么**：这通常是项目所属的组织、公司或团队。
- **命名规范**：通常采用**反向域名**的形式。
    - _例如：`com.google`、`org.apache`、`com.tencent.pay`。_
- **逻辑比喻**：就像是**姓氏**。它告诉别人这个插件或 Jar 包是谁家生产的

ArtifactID：项目的唯一标识（模块名）
- **代表什么**：这是该组织下的具体项目名称或模块名称。
- **命名规范**：通常使用小写字母和连字符。
    - _例如：`guava`、`commons-lang3`、`spring-boot-starter-web`。_
- **逻辑比喻**：就像是**名字**。在同一个“姓氏”（GroupID）下，名字必须唯一。

Version：项目的版本号
- **代表什么**：当前项目的具体版本状态。
- **常见标识**：
    - `1.0.0-SNAPSHOT`：快照版，表示还在开发中，内容可能会随时变动。
    - `1.0.0-RELEASE`：发行版，表示代码已定型，不再修改。
- **逻辑比喻**：就像是**辈分或修订号**。即使名字一样，1.0 版和 2.0 版的内容可能完全不同。

实际案例分析

假设你在 `pom.xml` 中引入了 Spring Boot 的 Web 启动器：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.5</version>
</dependency>
```

**这个坐标在后台干了什么？**

1. Maven 会去你的本地仓库（或中央仓库）寻找路径。
2. 它会把点号（`.`）转换成文件夹斜杠（`/`）。
3. 最终寻找的路径是：`org/springframework/boot/spring-boot-starter-web/2.7.5/`。
4. 在该目录下，它会找到一个名为 `spring-boot-starter-web-2.7.5.jar` 的文件。
5. 
`properties`中一般都是一些变量和选项的配置，我们这里指定了JDK的源代码和编译版本为17，同时下面的源代码编码格式为UTF-8，无需进行修改。