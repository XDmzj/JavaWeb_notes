


我们可以看到在IDEA右上角Maven板块中，每个Maven项目都有一个生命周期，实际上这些是Maven的一些插件，每个插件都有各自的功能，比如：

- `clean`命令，执行后会清理整个`target`文件夹，在之后编写Springboot项目时可以解决一些缓存没更新的问题。
- `validate`命令可以验证项目的可用性。
- `compile`命令可以将项目编译为.class文件。
- `install`命令可以将当前项目安装到本地仓库，以供其他项目导入作为依赖使用
- `verify`命令可以按顺序执行每个默认生命周期阶段（`validate`，`compile`，`package`等）


## clean
比如`clean`命令会自动清理`target`目录下的所有内容：

![image-20241119113903101](https://oss.itbaima.cn/internal/markdown/2024/11/19/dH8mpF3CQ2Yf69k.png)

所有的命令在执行完成之后都会显示BUILD SUCCESS，否则就是在执行过程中出现了什么错误。


## test
除了上述介绍的几种命令外，我们还可以通过使用`test`命令，一键测试所有位于test目录下的测试案例，但是请注意默认的`test`命令有以下要求：

- 测试类的名称必须是以`Test`结尾，比如`MainTest`
- 测试方法上必须标注`@Test`注解或是其他标记JUnit测试案例的注解

```java
public class MainTest {

    @Test
    public void test() {
        System.out.println("我是测试");
    }
}
```

![image-20241119161523176](https://oss.itbaima.cn/internal/markdown/2024/11/19/F9McUmyIqoVZX1T.png)


## package

我们接着来看`package`命令，它用于将我们的项目打包为jar文件，以供其他项目作为依赖引入，或是作为一个可执行的Java应用程序运行。

我们可以直接点击`package`来进行打包操作。注意，在使用`package`命令打包之前也会自动执行一次`test`命令，来保证项目能够正常运行，当测试出现问题时，打包将无法完成，我们也可以手动跳过，选择`执行Maven目标`来手动执行Maven命令，输入`mvn package -Dmaven.test.skip=true` 来以跳过测试的方式进行打包。

![image-20241119162039936](https://oss.itbaima.cn/internal/markdown/2024/11/19/Hca5MzbeWtNkFU6.png)

接着在target目录下会出现我们打包完成的jar包，在JavaSE中我们就给大家介绍过，一个jar包实际上就是对我们生成的字节码文件进行的压缩打包，因此，我们也可以使用常见的压缩工具打开jar包查看其内部文件。

![image-20241119162344760](https://oss.itbaima.cn/internal/markdown/2024/11/19/WdgNQJArRwfxpyY.png)

此时jar包中已经包含了我们项目中编写的类了，可以直接被其他项目导入使用。


### 问题
当然，以上方式存在一定的问题，比如这里并没有包含项目中用到的一些其他依赖，如果我们需要打包一个可执行文件，那么我不仅需要将自己编写的类打包到Jar中，同时还需要将依赖也一并打包到Jar中，因为我们使用了别人为我们提供的框架，自然也需要运行别人的代码，我们需要使用另一个插件来实现一起打包：


```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>com.test.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

导入插件后，我们可以重新进行一次打包任务，等待打包完成即可得到我们的Jar文件，此时会出现两个文件，其中一个是之前的正常打包得到的jar文件，还有一个就是包含了所有依赖以及配置了主类的jar文件。

我们只需要执行`java -jar`命令即可运行打包好的Java程序：

![image-20241119162858366](https://oss.itbaima.cn/internal/markdown/2024/11/19/hxT1lJDQgBGOSHf.png)
### 多模块下，父项目的`<packing>`负责打包

我们之前还讲解了多模块项目，那么多模块下父项目存在一个`packing`打包类型标签，所有的父级项目的`packing`都为`pom`，`packing`默认是`jar`类型，如果不作配置，maven会将该项目打成jar包：

```xml
<packaging>pom</packaging>
```

作为父级项目，还有一个重要的属性，那就是modules，通过modules标签将项目的所有子项目引用进来，在`build`父级项目时，会根据子模块的相互依赖关系整理一个`build`顺序，然后依次`build`直到所有任务都完成。