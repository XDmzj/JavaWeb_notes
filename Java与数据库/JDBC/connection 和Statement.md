# Connection (连接) —— “通讯管道”

`Connection` 对象代表了应用程序与数据库之间的**物理会话**。在发送任何 SQL 指令之前，必须先建立连接。

- **主要职责：**
    
    - **身份验证：** 负责处理用户名、密码和数据库 URL。
        
    - **会话管理：** 控制事务（Transaction），比如设置 `setAutoCommit(false)` 来手动提交或回滚。
        
    - **资源创建：** 它是 `Statement` 对象的工厂，所有的 SQL 执行对象都必须通过 `Connection` 创建。
        
- **类比：** 想象成一条连接你家和商场的**专属隧道**。没有这条隧道，你无法运送任何货物。
    

# Statement (语句) —— “运输工具”

`Statement` 对象用于在已建立的连接上**执行 SQL 语句**并返回结果。

- **主要职责：**
    
    - **执行命令：** 发送 `SELECT`、`INSERT`、`UPDATE` 等 SQL 指令。
        
    - **承载结果：** 执行查询后，它会返回一个 `ResultSet`（结果集）。
        
- **主要类型：**
    
    1. **Statement：** 用于执行简单的、不带参数的静态 SQL。
        
    2. **PreparedStatement：** （最常用）预编译 SQL，支持占位符（`?`），能有效防止 **SQL 注入**，性能也更好。
        
    3. **CallableStatement：** 用于调用数据库中的存储过程。
        
- **类比：** 想象成在隧道里行驶的**货车**。SQL 语句就是货车上的货物，负责送往数据库并把结果拉回来。
    

---

# 核心区别对比

|**特性**|**Connection (连接)**|**Statement (语句)**|
|---|---|---|
|**层级**|顶层容器，代表会话。|属于 Connection，代表操作。|
|**生命周期**|较长，通常在应用运行或业务逻辑开始时开启。|较短，执行完 SQL 或业务块后即可关闭。|
|**主要功能**|管理连接状态、事务控制（Commit/Rollback）。|执行 SQL 增删改查。|
|**创建方式**|通过 `DriverManager.getConnection()`。|通过 `connection.createStatement()`。|
|**对应关系**|**1 个 Connection** 可以创建 **多个 Statement**。|**1 个 Statement** 必须依附于 **1 个 Connection**。|

---

# 代码示例 (Java JDBC)


```java
// 1. 建立连接 (Connection)
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    
    // 2. 创建执行对象 (Statement/PreparedStatement)
    String sql = "SELECT name FROM users WHERE id = ?";
    try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
        pstmt.setInt(1, 101); // 设置参数
        
        // 3. 执行并获取结果
        ResultSet rs = pstmt.executeQuery();
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
} // 自动关闭资源
```

### 总结

- **Connection** 是你与数据库的**连接协议和通道**。
    
- **Statement** 是你在这个通道上**发送指令的载体**。