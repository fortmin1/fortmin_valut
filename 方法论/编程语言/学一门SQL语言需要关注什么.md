# 一、基础语法（入门，但不值钱）

包括：

- `SELECT / INSERT / UPDATE / DELETE`
- `WHERE / GROUP BY / ORDER BY / HAVING`
- `JOIN（INNER / LEFT / RIGHT）`
- 子查询

👉 目标：

> 能写出正确 SQL

⚠️ 现实：

- 面试几乎不考语法
- 工作中语法随查随用

---

# 二、数据模型（非常关键）

SQL 的本质是：**操作数据模型**

要掌握：

- 表设计（字段、类型）
- 主键 / 外键
- 一对多 / 多对多关系
- 范式（1NF / 2NF / 3NF）
- 反范式设计（冗余字段）

👉 举个你熟悉的场景：

- 用户表
- 角色表
- 用户角色关联表

👉 这一层目标：

> 看业务 → 能设计表结构

---

# 三、索引与查询优化（核心能力🔥）

这是 SQL 水平的分水岭

必须掌握：

- B+树索引结构
- 聚簇索引 vs 二级索引
- 联合索引（最左前缀原则）
- 覆盖索引
- 回表

👉 一个典型知识点：  
为什么这个 SQL 慢：

SELECT * FROM order   
WHERE user_id = 100   
ORDER BY create_time;

👉 本质是：

- 有没有索引？
- 索引是否命中？
- 是否排序？

👉 这一层目标：

> 能解释 SQL 为什么快 / 慢

---

# 四、执行计划（必须会🔥）

不会看执行计划 = 不会 SQL

重点：

- `EXPLAIN`
- key（用没用索引）
- rows（扫描行数）
- type（ALL / index / range / ref / const）
- extra（Using index / Using filesort）

👉 这一层目标：

> 能分析 SQL 执行路径

---

# 五、事务与锁（高频面试点🔥）

这是后端工程师必备

必须理解：

## 1️⃣ 事务四大特性（ACID）

- 原子性
- 一致性
- 隔离性
- 持久性

## 2️⃣ 隔离级别

- Read Uncommitted
- Read Committed
- Repeatable Read（MySQL 默认）
- Serializable

## 3️⃣ 并发问题

- 脏读
- 不可重复读
- 幻读

## 4️⃣ 锁机制（重点）

- 行锁 / 表锁
- 间隙锁（Gap Lock）
- Next-Key Lock

👉 比如你之前问的：

UPDATE org_nodes   
SET node_path = REPLACE(node_path, '/5/', '/12/')  
WHERE tenant_id=1001 AND node_path LIKE '%/5/%';

👉 本质问题：

- 是否走索引
- 锁范围多大（可能锁很多行）

👉 这一层目标：

> 能解释并发问题 & 锁行为

---

# 六、SQL 性能问题（实战能力🔥）

必须能解决：

### 常见问题：

- 深分页（LIMIT offset 很慢）
- JOIN 性能差
- COUNT(*) 慢
- LIKE '%xxx%' 不走索引

👉 你应该会：

- 用子查询优化分页
- 用覆盖索引
- 用分库分表

---

# 七、数据库原理（进阶）

这一层让你成为“高级工程师”

包括：

- InnoDB 存储结构（页 16KB）
- MVCC（多版本并发控制）
- undo log / redo log / binlog
- Buffer Pool

👉 举个你之前问过的点：

- B+树页结构
- 为什么 JOIN 成本高

👉 这一层目标：

> 理解数据库内部机制

---

# 八、工程实践（最重要）

现实中你用的是：

👉 **SQL + 业务 + ORM（MyBatis / JPA）**

你要掌握：

- SQL 在代码中的使用
- MyBatis 映射
- 动态 SQL
- 慢 SQL 日志分析
- SQL 规范（命名、索引设计）

---

# 一个更本质的总结

学 SQL，其实就是掌握这 4 件事：

### 1️⃣ 数据如何组织

（表结构 / 关系模型）

### 2️⃣ 数据如何查找

（索引 / 执行计划）

### 3️⃣ 并发如何控制

（事务 / 锁 / MVCC）

### 4️⃣ 如何更快

（优化 / 架构 / 分库分表）