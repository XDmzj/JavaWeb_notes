# 登录功能
在生活中，很多网站都支持用户名和密码登录，这也是一种安全机制，防止别人非法访问我们自己的账户。尤其是一些银行相关的网站，如果不设置密码就可以直接访问我们的账户，那随便来个人都可以转走我们的钱了。

通过一个密码验证，就可以很好地解决这个问题，在进行各种操作之前，需要使用账号密码登录，验证是你之后，才可以开始，这也是现在最主流的方式。

实际上想要设计一个这样的系统也很简单，同样是设计一张用户表，只不过这次我们需要带上用户自己设定的密码，并且用户的名字或是ID必须是唯一的，否则无法区分：

![QQ_1722413094272](https://oss.itbaima.cn/internal/markdown/2024/07/31/IbmOj4HJUsytQfa.png)

接着我们设计一个对应的实体类：

```java
public record Account (int id, String name, String password){
    public boolean verifyPassword(String password) {
        return this.password.equals(password);
    }
}
```

接着我们就可以从控制台接受数据并校验了：

```java
try (Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/web_study", "test", "123456");
     Statement statement = connection.createStatement();
     Scanner scanner = new Scanner(System.in)){
    System.out.print("请输入用户名: ");
    String username = scanner.nextLine();
    System.out.print("请输入密码: ");
    String password = scanner.nextLine();
    ResultSet set = statement.executeQuery("select * from account where name = '" + username + "'and password = '" + password + "'");
    if(set.next()) {
        Account account = new Account(set.getInt(1),
                set.getString(2), set.getString(3));
        System.out.println(account + " 登录成功");
    } else {
        System.out.println("用户名或密码错误");
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

![QQ_1722415319102](https://oss.itbaima.cn/internal/markdown/2024/07/31/18tfCopcKGgL5In.png)

这样我们就实现了一个最基本的用户名密码登录系统了，用户可以通过自己输入用户名和密码来登陆。



# SQL注入攻击

但是，实际上这样的系统存在一些问题，如果我们输入一些不太正常的内容：

```sh
请输入用户名: test
请输入密码: 1111' or 1=1; -- 
```

此时，我们的密码输入的是一个不太常规的内容：`1111' or 1=1; --` ，居然登录成功了，各位小伙伴可以想象一下如果这样的字符串拼接到我们的SQL语句中，会变成什么样呢？

```sql
select * from account where name = 'test' and password = '1111' or 1=1; --;'
```

由于在SQL中`--`为注释，所以最终执行的SQL就变成了这样：

```sql
select * from account where name = 'test' and password = '1111' or 1=1;
```

我们发现此时的SQL语句已经完全脱离我们想要的意思了，最后多出来了一个`or 1=1`，由于1=1一定为真，此时or后面的条件直接被判定为真了，导致后面的密码判定条件失效。

因此，如果允许这样的数据插入，那么我们原有的SQL语句结构就遭到了破坏，使得用户能够随意登陆别人的账号，所以我们可能需要限制用户的输入来防止用户输入一些SQL语句关键字，但是SQL关键字非常多，这并不是解决问题的最好办法，那该怎么办呢。