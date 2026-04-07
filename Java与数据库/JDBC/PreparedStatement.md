# 优势

我们发现，如果单纯地使用Statement来执行SQL命令，会存在严重的SQL注入攻击漏洞！而这种问题，我们可以使用PreparedStatement来解决。

> `PreparedStatement` 在执行之前会被预编译，因此如果同样的 SQL 查询多次执行，性能会更好。数据库只需编译一次，而不是每次都编译，并且`PreparedStatement` 使用参数化查询，自动对输入进行转义，从而有效防止 SQL 注入攻击，通过使用参数，SQL 语句更加清晰，将值与查询逻辑分开，使得代码更易于维护和理解。

它相比我们之前使用的Statement来说，不仅性能更好，还更加安全。


# 写法

我们还是以之前的用户登录为例，这次我们换成更加安全的PreparedStatement来操作：

```java
PreparedStatement statement = connection.prepareStatement("select * from account where name = ? and password = ?")
```

我们需要提前给到PreparedStatement一个SQL语句，并且使用`?`作为占位符，它会预编译一个SQL语句，通过直接将我们的内容进行替换的方式来填写数据。

现在我们修改一下之前的写法：
```java
try (Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/web_study", "test", "123456");
     PreparedStatement statement = connection.prepareStatement("select * from account where name = ? and password = ?");
     Scanner scanner = new Scanner(System.in)){
    System.out.print("请输入用户名: ");
    statement.setString(1, scanner.nextLine());  //我们需要手动调用set方法填入参数到对应位置上去
    System.out.print("请输入密码: ");
    statement.setString(2, scanner.nextLine());
    ResultSet set = statement.executeQuery();
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

执行之前的SQL注入攻击案例：

![QQ_1722416161535](https://oss.itbaima.cn/internal/markdown/2024/07/31/kCcFwoa18Si2ysD.png)

我们发现，此时无法再被破解了。我们来看看实际执行的SQL语句是什么，这里直接打印Statement对象：

```java
System.out.println(statement);
```

实际执行的SQL语句如下：

```sql
select * from account where name = 'test' and password = '1111\' or 1=1; --'
```

此时我们发现，我们输入的恶意内容中单引号被转义了，也就是说它现在仅仅代表的是单引号这个字符，而真正用作囊括内容的是后面的那个单引号，也就是PreparedStatement自动将可能会产生歧义的地方进行了处理，保证了我们输入的内容是什么，那么这里填入的一定是什么。这样就可以防止语义被改变，从根源上解决SQL注入攻击。