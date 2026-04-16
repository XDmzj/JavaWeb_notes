

# 方法一：修改配置文件（最简单，推荐）

如果你只是想调整默认 `SimpleFormatter` 输出的内容顺序或样式，不需要写 Java 代码，直接在 `logging.properties` 文件中设置即可。

从 Java 7 开始，`SimpleFormatter` 支持通过 `java.util.logging.SimpleFormatter.format` 属性来自定义。

**在 `.properties` 文件中添加：**

```
# 格式说明：
# %1: 日期时间 (Date)
# %2: 源方法 (Source)
# %3: 日志名称 (Logger name)
# %4: 日志级别 (Level)
# %5: 日志消息 (Message)
# %6: 异常堆栈 (Thrown)

java.util.logging.SimpleFormatter.format=[%1$tF %1$tT] [%4$-7s] %5$s %n
```

**输出效果：** `[2023-10-27 10:15:30] [INFO ] 这是一个自定义格式的消息`

---

# 方法二：编写自定义 Formatter 类（高度自由）

如果你需要更复杂的逻辑（比如把日志转成 JSON 格式，或者根据特定条件加颜色），你需要自己写一个类。

## 1. 创建自定义类

你需要继承 `java.util.logging.Formatter` 并重写 `format` 方法。

```java
import java.util.logging.Formatter;
import java.util.logging.LogRecord;
import java.text.SimpleDateFormat;
import java.util.Date;

public class MyCustomFormatter extends Formatter {
    // 定义时间格式
    private final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Override
    public String format(LogRecord record) {
        StringBuilder sb = new StringBuilder();
        
        // 添加时间
        sb.append(dateFormat.format(new Date(record.getMillis())));
        
        // 添加级别
        sb.append(" [").append(record.getLevel()).append("] ");
        
        // 添加类名和方法名
        sb.append(record.getSourceClassName()).append(".").append(record.getSourceMethodName());
        
        // 添加消息内容
        sb.append(" - ").append(record.getMessage());
        
        // 换行
        sb.append(System.lineSeparator());
        
        return sb.toString();
    }
}
```


`Formatter` 必须设置在 `Handler` 上，然后将这个 `Handler` 添加给 `Logger`。

以下是完整的逻辑代码：

### 核心代码实现

```java
import java.util.logging.*;

public class LogTest {
    public static void main(String[] args) {
         1. 获取或创建一个 Logger
        Logger logger = Logger.getLogger("myCustomLogger");

         2. 关键步骤：禁用父级 Handler
         默认情况下，Logger 会把日志向上传递给父级（Root Logger），
	     Root Logger 默认自带一个 ConsoleHandler。如果不禁用，
         你的日志会以“你的格式”和“默认格式”各输出一次。
         
        logger.setUseParentHandlers(false);

         3. 创建一个处理器（这里以控制台输出为例）
        ConsoleHandler handler = new ConsoleHandler();

         4. 【核心点】给 Handler 设置自定义的格式化器
         假设 MyCustomFormatter 是你上一条回复中写的那个类
        handler.setFormatter(new MyCustomFormatter());

         5. 将配置好的 Handler 添加到 Logger 中
        logger.addHandler(handler);

         6. 测试输出
        logger.info("这条日志将使用我自定义的格式输出！");
        logger.severe("这是严重错误消息。");
    }
}
```