# 1. 什么是 JDBC？（定义）

**JDBC (Java Database Connectivity)** 是 Java 官方提供的一套**标准接口**（API）。它的存在是为了让 Java 程序能够以统一的方式连接并操作各种不同的数据库（如 MySQL、Oracle、PostgreSQL 等）。

# 2. 生活化比喻：万能转换插头

想象你要去不同国家旅游（不同类型的数据库），每个国家的插座形状都不一样。你不需要为每个国家买一个专门的电器，你只需要带一个**“万能转换插头”**（JDBC）。

- **电器**：你的 Java 程序。
    
- **万能插座接口**：JDBC 接口。
    
- **特定国家的适配器**：数据库驱动（Driver）。
    
- **墙上的插座**：数据库。
    

# 3. JDBC 架构图解

理解 JDBC 的分层结构是快速回忆的关键：

- **Java Application**：你的代码。
    
- **JDBC API**：Java 定义的一套“规则”（全是接口）。
    
- **DriverManager**：管理各种数据库驱动的“管理员”。
    
- **Database Driver**：数据库厂商（如 MySQL）提供的实现类（翻译官）。
    

---

# 4. 笔记核心：JDBC 编程 6 步走（建议直接抄录）

这是开发中最经典的流程，只要记住这 6 个动词，就能回忆起整段代码：

1. **加载驱动 (Load)**：告诉 Java 你要用哪种数据库。
    
    - `Class.forName("com.mysql.cj.jdbc.Driver");`
        
2. **建立连接 (Connect)**：通过账号密码连接到具体的数据库。
    
    - `DriverManager.getConnection(url, user, password);`
        
3. **创建语句 (Statement)**：准备好要执行的 SQL 纸条。
    
    - `connection.createStatement();`
        
4. **执行 SQL (Execute)**：把纸条递给数据库并运行。
    
    - `statement.executeQuery(sql);` 或 `executeUpdate(sql);`
        
5. **处理结果 (Process)**：如果查到了数据，用 `ResultSet` 读出来。
    
    - `while(resultSet.next()) { ... }`
        
6. **释放资源 (Close)**：用完后必须关门（倒序关闭：Result -> Statement -> Connection）。
    
    - `close();`
        

---

# 5. 为什么现在很少直接写 JDBC？

在实际的项目开发（如你可能接触到的 Spring Boot 项目）中，我们通常使用 **MyBatis** 或 **Spring Data JPA**。

**为什么要记住这个差异点：**

- **JDBC** 是底层地基，手动挡，控制力强但代码冗长。
    
- **MyBatis/Hibernate** 是在 JDBC 之上盖的精装房，自动挡，开发效率高。
    
- **重点**：无论框架怎么变，底层运行的永远是 JDBC。