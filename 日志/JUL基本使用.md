

# 日志的级别

 日志的打印并不是简单的输出，有些时候我们可以会打印一些比较重要的日志信息，或是一些非常紧急的日志信息，根据不同类型的信息进行划分，日志一般分为7个级别，详细信息我们可以在Level类中查看：

```java
public class Level implements java.io.Serializable {
	...
  
    //出现严重故障的消息级别，值为1000，也是可用的日志级别中最大的
    public static final Level SEVERE = new Level("SEVERE",1000, defaultBundle);
    //存在潜在问题的消息级别，比如边充电边打电话就是个危险操作，虽然手机爆炸的概率很小，但是还是会有人警告你最好别这样做，这是日志级别中倒数第二大的
    public static final Level WARNING = new Level("WARNING", 900, defaultBundle);
    //所有常规提示日志信息都以INFO级别进行打印
    public static final Level INFO = new Level("INFO", 800, defaultBundle);
  	//以下日志级别依次降低，不太常用
    public static final Level CONFIG = new Level("CONFIG", 700, defaultBundle);
    public static final Level FINE = new Level("FINE", 500, defaultBundle);
    public static final Level FINER = new Level("FINER", 400, defaultBundle);
    public static final Level FINEST = new Level("FINEST", 300, defaultBundle);
 
  ...
}

```


## info()
之前通过`info`方法直接输出的结果就是使用的默认级别的日志，实际上每个级别都有一个对应的方法用于打印：

```java
public static void main(String[] args) {
    Logger logger = Logger.getLogger(Main.class.getName());
    logger.severe("severe");  //最高日志级别
    logger.warning("warning");
    logger.info("info"); //默认日志级别
    logger.config("config");
    logger.fine("fine");
    logger.finer("finer");
    logger.finest("finest");   //最低日志级别
}
```


## log()
当然，如果需要更加灵活地控制日志级别，我们也可以通过`log`方法来主动设定该条日志的输出级别：


```java
Logger logger = Logger.getLogger(Main.class.getName());
logger.log(Level.SEVERE, "严重的错误", new NullPointerException("祝你明天就遇到我"));
logger.log(Level.WARNING, "警告的内容");
logger.log(Level.INFO, "普通的信息");
logger.log(Level.CONFIG, "级别低于普通信息");
```

不过，可能会有小伙伴发现，某些级别的日志，并没有在控制台打印。这其实因为Logger默认情况下只会打印INFO级别以上的日志，而以下的日志则会直接省略，我们可以通过配置来进行调整，只不过调整日志打印级别比较麻烦，需要经过后续的学习我们再来讨论这个问题。