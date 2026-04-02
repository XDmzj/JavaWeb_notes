
在 Java 中使用 JDBC 执行 SQL 语句，主要依赖 `Statement` 或 `PreparedStatement` 对象。根据 SQL 类型的不同，调用的方法和**返回值**也有很大区别。

我们可以将其分为两大类：**查询（Query）** 和 **更新（Update）**。

---

# 1. 执行查询语句 (`SELECT`)

对于查询语句，我们使用 `executeQuery()` 方法。它就像是一个“信使”，去数据库里把数据取回来并打包给你。

## 代码示例

```java
String sql = "SELECT id, name, email FROM users";
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    
    // 执行查询
    ResultSet rs = pstmt.executeQuery();
    
    // 处理结果集
    while (rs.next()) {
        System.out.println("ID: " + rs.getInt("id") + ", Name: " + rs.getString("name"));
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 返回值：`ResultSet`

- **本质**：一个指向数据库结果集的数据表。
    
- **特点**：它维护了一个光标（Cursor），初始指向第一行之前。通过调用 `rs.next()`，你可以逐行读取数据。
    
- **注意**：使用完后必须关闭（通常放在 `try-with-resources` 中自动处理）。

---

# 2. 执行更新语句 (`INSERT`, `UPDATE`, `DELETE`)

对于改变数据库内容的 DML 语句，我们使用 `executeUpdate()` 方法。这更像是一个“工头”，干完活后告诉你进度。

## 代码示例


```java
String insertSql = "INSERT INTO users (name, email) VALUES (?, ?)";
String updateSql = "UPDATE users SET email = ? WHERE name = ?";

try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement pstmt = conn.prepareStatement(insertSql)) {
    
    pstmt.setString(1, "Gemini");
    pstmt.setString(2, "gemini@example.com");
    
    // 执行更新
    int rows = pstmt.executeUpdate();
    System.out.println("成功插入了 " + rows + " 行数据。");
    
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 返回值：`int`

- **本质**：受影响的行数（Affected Rows）。
    
- **场景含义**：
    
    - 执行 `INSERT`：返回插入了几行。
        
    - 执行 `UPDATE`：返回修改了几行。
        
    - 执行 `DELETE`：返回删除了几行。
        
    - **如果是 0**：意味着 SQL 执行成功了，但没有一行数据符合条件被修改。
        

---
---

# 3. execute()的核心机制

`execute()` 方法可以执行任何 SQL 语句。它的返回值并不是数据本身，而是一个 **`boolean`**，用来告诉你“接下来该去哪里拿结果”。

### 返回值的含义：

- **返回 `true`**：表示执行的是 **查询语句**（如 `SELECT`）。你需要调用 `getResultSet()` 来获取结果集。
    
- **返回 `false`**：表示执行的是 **更新语句**（如 `INSERT`, `UPDATE`, `DELETE`）或 DDL（如 `CREATE`）。此时没有结果集，你可以调用 `getUpdateCount()` 获取受影响的行数（如果是 DDL 则返回 -1）。
## 代码示例：处理不确定的 SQL

当你编写一个通用的数据库工具类，或者允许用户直接输入 SQL 脚本时，`execute()` 就派上用场了：

```java
String sql = "用户输入的任意SQL"; 
try (Statement stmt = conn.createStatement()) {
    
    boolean isResultSet = stmt.execute(sql);
    
    if (isResultSet) {
        // 如果是查询，获取结果集
        ResultSet rs = stmt.getResultSet();
        while (rs.next()) {
            // 处理数据...
        }
    } else {
        // 如果是更新，获取影响行数
        int count = stmt.getUpdateCount();
        if (count != -1) {
            System.out.println("操作成功，影响行数: " + count);
        } else {
            System.out.println("执行的是 DDL 语句或没有返回计数");
        }
    }
}
```


# 核心差异总结

| **方法**                | **适用 SQL**                   | **返回类型**    | **返回值含义**                                          |
| --------------------- | ---------------------------- | ----------- | -------------------------------------------------- |
| **`executeQuery()`**  | `SELECT`                     | `ResultSet` | 包含查询结果的数据集合。                                       |
| **`executeUpdate()`** | `INSERT`, `UPDATE`, `DELETE` | `int`       | 本次操作影响的记录行数。                                       |
| **`execute()`**       | 任何 SQL (通常用于动态 SQL)          | `boolean`   | **true**: 返回了 ResultSet；**false**: 返回的是受影响行数或没有结果。 |