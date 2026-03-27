# 单表查询

单表查询是最简单的一种查询，我们只需要在一张表中去查找数据即可，通过使用`select`语句来进行单表查询：

```sql
-- 指定查询某一列数据
SELECT 列名[,列名] FROM 表名
-- 会以别名显示此列
SELECT 列名 别名 FROM 表名
-- 查询所有的列数据
SELECT * FROM 表名
-- 只查询不重复的值
SELECT DISTINCT 列名 FROM 表名
```

我们也可以添加`where`字句来限定查询目标：

```sql
SELECT * FROM 表名 WHERE 条件
```

## 常用查询条件

- 一般的比较运算符，包括=、>、<、>=、<=、!=等。
- 是否在集合中：in、not in
- 字符模糊匹配：like，not like
- 多重条件连接查询：and、or、not

我们来尝试使用一下上面这几种条件。

## 排序查询

我们可以通过`order by`来将查询结果进行排序：

sql复制代码

```sql
SELECT * FROM 表名 WHERE 条件 ORDER BY 列名 ASC|DESC
```

使用ASC表示升序排序，使用DESC表示降序排序，默认为升序。

我们也可以可以同时添加多个排序：

```sql
SELECT * FROM 表名 WHERE 条件 ORDER BY 列名1 ASC|DESC, 列名2 ASC|DESC
```

这样会先按照列名1进行排序，每组列名1相同的数据再按照列名2排序。

## 聚集函数

聚集函数一般用作统计，包括：

- `count([distinct]*)`统计所有的行数（distinct表示去重再统计，下同）
- `count([distinct]列名)`统计某列的值总和
- `sum([distinct]列名)`求一列的和（注意必须是数字类型的）
- `avg([distinct]列名)`求一列的平均值（注意必须是数字类型）
- `max([distinct]列名)`求一列的最大值
- `min([distinct]列名)`求一列的最小值

#### 基础用法
统计全校老师的平均工资：

```
SELECT AVG(salary) AS avg_salary FROM teacher;
```

#### 结合 DISTINCT（去重统计）

统计学校里一共有多少个不同的姓氏：

```
SELECT COUNT(DISTINCT tname) FROM teacher;
```

#### 结合 GROUP BY（分组统计）

这是聚集函数最强大的地方。比如：统计每个性别的老师分别有多少人？

```
SELECT sex, COUNT(*) 
FROM teacher 
GROUP BY sex;
```

## 分组和分页查询


### group by  和 having
通过使用`group by`来对查询结果进行分组，它需要结合聚合函数一起使用：

```sql
SELECT sum(*) FROM 表名 WHERE 条件 GROUP BY 列名
```

我们还可以添加`having`来限制分组条件：

```sql
SELECT sum(*) FROM 表名 WHERE 条件 GROUP BY 列名 HAVING 约束条件
```

- **WHERE（第一道关卡）**：在 **分组前** 过滤。它对着原始表的每一行进行检查。如果某行不符合条件，直接踢掉，不参与后面的统计。
- **HAVING（第二道关卡）**：在 **分组后** 过滤。它对着聚合后的“组”进行检查。如果某个组的统计结果（如平均分、总数）不达标，把整个组踢掉。

### limit

我们可以通过`limit`来限制查询的数量，只取前n个结果：
基础语法

`LIMIT` 通常放在 SQL 语句的最末尾（甚至在 `ORDER BY` 之后）。

#### 只取前 N 条记录

```
SELECT * FROM teacher 
ORDER BY salary DESC 
LIMIT 5; -- 获取工资最高的前 5 名老师
```

#### 分页取值（两个参数

语法：`LIMIT offset, count`

- **offset**：偏移量，表示从第几条记录之后开始取（从 0 开始计数）。
    
- **count**：获取多少条记录。

```
-- 获取第 11 到 20 条记录（即第二页，每页 10 条）
SELECT * FROM teacher 
LIMIT 10, 10;
```


# 多表查询
## 不同表连接（笛卡尔积
多表查询是同时查询的两个或两个以上的表，多表查询会提通过连接转换为单表查询。

```sql
SELECT * FROM 表1, 表2
```

直接这样查询会得到两张表的笛卡尔积，也就是每一项数据和另一张表的每一项数据都结合一次，会产生庞大的数据。

```sql
SELECT * FROM 表1, 表2 WHERE 条件
```

这样，只会从笛卡尔积的结果中得到满足条件的数据。

**注意：** 如果两个表中都带有此属性吗，需要添加表名前缀来指明是哪一个表的数据。

## 自身连接查询

自身连接，就是将表本身和表进行笛卡尔积计算，得到结果，但是由于表名相同，因此要先起一个别名：


```sql
SELECT * FROM 表名 别名1, 表名 别名2
```

其实自身连接查询和前面的是一样的，只是连接对象变成自己和自己了。

## 外连接查询（连接->某一项元素相同
### 概念
外连接就是专门用于联合查询情景的，比如现在有一个存储所有用户的表，还有一张用户详细信息的表，我希望将这两张表结合到一起来查看完整的数据，我们就可以通过使用外连接来进行查询，外连接有三种方式：

- 通过使用`inner join`进行内连接，只会返回两个表满足条件的交集部分：

![在这里插入图片描述](https://oss.itbaima.cn/internal/markdown/2023/03/06/EDihOjuk8X2t3KW.png)

- 通过使用`left join`进行左连接，不仅会返回两个表满足条件的交集部分，也会返回左边表中的全部数据，而在右表中缺失的数据会使用`null`来代替（右连接`right join`同理，只是反过来而已，这里就不再介绍了）：

![在这里插入图片描述](https://oss.itbaima.cn/internal/markdown/2023/03/06/1VYXgZjiBhRrudy.png)

### 举例

**内连接（Inner Join）**：只有当 A 表和 B 表的关联字段完全匹配时，才会显示结果。
**外连接** ：它会保留其中一张表的所有记录，即使另一张表没有对应的数据，也会用 `NULL` 来填充。

---
 **左外连接 (LEFT JOIN / LEFT OUTER JOIN) —— 最常用**

**逻辑**：包含**左表**的所有行。如果右表没有匹配，右表部分显示 `NULL`。

- **场景**：查询“所有老师及其负责的班级”。即使某个新老师还没分到班级，他的名字也得列出来。

```
SELECT t.tname, c.cname
FROM teacher t
LEFT JOIN classes c ON t.tid = c.tid;
```
 
 **右外连接 (RIGHT JOIN / RIGHT OUTER JOIN)**

**逻辑**：包含**右表**的所有行。如果左表没有匹配，左表部分显示 `NULL`。

- **注意**：它和 `LEFT JOIN` 其实是镜像关系。`A RIGHT JOIN B` 等价于 `B LEFT JOIN A`。在实际开发中，为了代码可读性，大家习惯统一使用 `LEFT JOIN`。

---

内连接 vs 外连接 的直观对比

假设 `teacher` 表有“张三”、“李四”，`classes` 表只有“张三”带的“一班”。

|**连接类型**|**结果内容**|**缺失数据的处理**|
|---|---|---|
|**Inner Join**|只有张三|李四被无情丢弃|
|**Left Join**|张三（一班）、李四（NULL）|**保留李四**，班级填 NULL|

---

外连接的“妙用”：查找“差集”

在外连接的基础上配合 `WHERE ... IS NULL`，你可以轻松找到那些“落单”的数据。

**场景：找出哪些老师目前“闲着”（没带任何班级）？**

```
SELECT t.tname
FROM teacher t
LEFT JOIN classes c ON t.tid = c.tid
WHERE c.cid IS NULL; -- 关键点：找出右表匹配不上的记录
```

这种写法在数据清洗和逻辑校验（比如查出没下单的用户、没选课的学生）中极其高效。


## 嵌套查询

根据子查询返回的结果形式，我们可以将其分为以下三类：

 **标量子查询（返回单一值）**
子查询只返回一个数字或一个字符串。
- **场景**：查询工资高于“张三”的所有老师。

```
SELECT tname, salary 
FROM teacher 
WHERE salary > (SELECT salary FROM teacher WHERE tname = '张三');
```


 **列子查询（返回一列多行）**
子查询返回一组值，通常配合 `IN`、`ANY`、`ALL` 使用。
- **场景**：查询所有带过“研究生课程”的老师姓名。

```
SELECT tname 
FROM teacher 
WHERE tid IN (SELECT tid FROM classes WHERE cname LIKE '%研究生%');
```


 **行子查询（返回一行多列）**
- **场景**：查询和“王教授”性别相同且职称也相同的老师。

```
SELECT tname 
FROM teacher 
WHERE (sex, job_title) = (SELECT sex, job_title FROM teacher WHERE tname = '王教授');
```
