# 核心概念
# 数据类型
PostgreSQL 以其丰富的数据类型著称，以下是开发中最常用的几类：
数值类型
- **`INTEGER`****/****`INT`**: 4 字节整数，范围约±21±21 亿。
- **`BIGINT`**: 8 字节整数，用于大 ID 或大数据量计数。
- **`NUMERIC(p, s)`****/****`DECIMAL`**: 精确的小数，用于金额。pp是总位数，ss 是小数点后的位数。
- **`SERIAL`****/****`BIGSERIAL`**: 自增整数（实际上是封装了`SEQUENCE`）。
字符类型
- **`VARCHAR(n)`**: 变长字符串，有长度限制。
- **`TEXT`**: 变长字符串，无长度限制（PostgreSQL 推荐直接用这个，除非有业务硬性长度约束）。
- **`CHAR(n)`**: 定长字符串，不足部分补空格。
日期/时间类型
- **`TIMESTAMP`**: 日期和时间。
- **`TIMESTAMPTZ`**: 带时区的日期和时间（**生产环境推荐使用**，避免时区混乱）。
- **`DATE`**: 仅日期。
- **`INTERVAL`**: 时间间隔（如`1 day 2 hours`）。
- **`tsrange`**: 对应`timestamp without time zone` 的范围。
- **`tstzrange`**: 对应`timestamp with time zone`的范围（**推荐用于跨时区业务**）。
- **`daterange`**: 对应`date` 的范围。
特色/高级类型
- **`BOOLEAN`**:`TRUE`,`FALSE`, 或`NULL`。
- **`JSONB`**: 二进制存储的 JSON 数据，支持索引，性能极佳。
- **`UUID`**: 通用唯一识别码。
- **`ARRAY`**: 数组类型，例如`TEXT[]` 可以存储标签列表。
- `INET`：ip类型
# 常用函数
## 字符函数
- **`CONCAT(a, b)`****/****`||`**：拼接字符串。
- **`UPPER()`****/****`LOWER()`**：转换大小写。
- **`SUBSTR(string, start, len)`**：截取字符串。
- **`REPLACE(string, from, to)`**：替换内容。
- **`COALESCE(value, default)`**：**【超级常用】** 如果字段是 NULL，就给它一个默认值。
## 数值函数
- **`ROUND(numeric, 2)`**：四舍五入。
- **`CEIL()`****/****`FLOOR()`**：向上/向下取整。
- **`ABS()`**：取绝对值。
## 日期函数
- **`NOW()`**：获取当前完整时间。
- **`CURRENT_DATE`**：获取当前日期。
- **`AGE(timestamp)`**：计算时间差。
- **`EXTRACT(field FROM source)`**：提取年、月、日、小时。
- - **`TO_CHAR()`**：将时间格式化为字符串。
```SQL
-- 变成类似 2026-04-14 14:30:00 的格式
 SELECT TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS') FROM file_details;
```
## 聚合函数
- **`CASE WHEN`**：SQL 里的`if-else`。
```SQL
-- 给文件分等级
 SELECT file_name, 
 CASE
  WHEN file_size > 1024*1024*100 THEN '大文件'
  WHEN file_size > 1024*1024*10 THEN '中文件' 
  ELSE '小文件' 
  END AS file_level 
  FROM files;
```
- **`STRING_AGG(field, delimiter)`**：**【PG特色】** 把多行结果合并成一行字符串。
```SQL
-- 把某个用户的所有标签拼成一个逗号分隔的字符串 
SELECT uploader_ip, STRING_AGG(tags::text, ' | ') FROM file_details GROUP BY uploader_ip;
```
OR
```SQL
SELECT uploader_ip, 
-- 先把所有数组合并成一个大数组，再转为字符串 
array_to_string(array_agg(unnest_tags), ', ') FROM ( SELECT uploader_ip, unnest(tags) AS unnest_tags FROM file_details ) t GROUP BY uploader_ip;
```
# SQL
## DDL
## DML
## DQL
## DCL
# 权限体系
## 角色
pg中user是role的子集，底层相同，区别是user可以登录，role之间可以继承，继承是复制行为。
```SQL
-- 两者完全等价 
CREATE USER app_user;
 CREATE ROLE app_user LOGIN; 
 -- 无登录权限的组角色 CREATE ROLE app_group;
```
## Schema
schema是数据库内部的**逻辑命名空间**
**search path关键匹配规则**
用户查询表时不写schema，按照search path匹配
```SQL
-- 查看当前检索路径
 SHOW search_path;
  -- 给用户设置默认schema
   ALTER ROLE app_user SET search_path = biz, public;
```
## 三、分层权限：角色控制四层权限（从粗到细）

### 层级 1：数据库级权限（连接、创建 schema）
控制角色能不能进库、能不能新建 schema：

|权限|作用|
|---|---|
|CONNECT|允许角色登录连接该数据库|
|CREATE|允许在库内新建 Schema|

```
GRANT CONNECT, CREATE ON DATABASE test_db TO app_group;
```

### 层级 2：Schema 专属权限（Schema 层核心）

Schema 自身的权限，**不控制表数据，只控制能不能进文件夹、新建对象**

```
GRANT USAGE, CREATE ON SCHEMA biz TO app_group;
```

1. `USAGE`（必选）：允许角色查看 schema 内所有对象、访问内部表 / 视图；无 USAGE 则 schema 内所有对象完全不可见；
2. `CREATE`：允许角色在该 schema 下新建表、视图、函数等对象；
3. 回收示例：

```
REVOKE CREATE ON SCHEMA biz FROM app_group;
```

> 关键区分：Schema 权限只管「能不能进入这个文件夹、能不能新建文件」，**不控制表里数据读写**。

### 层级 3：表 / 视图 / 函数 对象级权限（单表 CRUD）

Schema 有 USAGE 后，再单独授予每张表的操作权限，控制整表读写：

```
-- 给组授予biz下所有表查询、新增、修改、删除
GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA biz TO app_group;
-- 未来新建的表自动授权
ALTER DEFAULT PRIVILEGES IN SCHEMA biz GRANT SELECT,INSERT,UPDATE,DELETE TO app_group;
```

常用表权限：

- `SELECT`：查询表数据
- `INSERT`：插入数据
- `UPDATE`：更新数据
- `DELETE`：删除数据
- `TRUNCATE`：清空表
- `REFERENCES`：外键引用
- `TRIGGER`：创建触发器

### 层级 4：字段（列）级细粒度权限（精确到单个字段）

PostgreSQL 支持**列粒度独立权限**，同一表不同字段授予不同角色，优先级高于表权限。

#### 语法规则
```
GRANT 权限(字段1,字段2) ON 表 TO 角色;
```

仅支持 `SELECT / INSERT / UPDATE` 三类列权限（DELETE 无列概念，不能按列控制删除）
#### 示例 1：只能查询姓名，不能查手机号、身份证

sql

```
-- 允许查询name，禁止查询phone/id_card
GRANT SELECT(name) ON biz.user_info TO read_user;
```

此时执行 `SELECT * FROM biz.user_info` 会报错，只能指定 `name` 字段查询。

#### 示例 2：仅允许修改姓名，不能修改金额
```
GRANT UPDATE(name) ON biz.account TO op_user;
```

执行 `UPDATE set amount=100` 会权限拒绝，只能更新 name。

#### 示例 3：插入时仅能填写基础字段，敏感字段禁止写入
```
GRANT INSERT(id,name) ON biz.user_info TO reg_user;
```

插入语句必须只提供 id、name，携带 phone 则报错。

#### 补充规则：表权限与列权限叠加逻辑

1. 若授予**整表 SELECT**：所有列自动可读；
2. 若仅授予**部分列 SELECT**：仅指定列可读，其余列无权限；
3. 先授予整表权限，再回收单列权限：
```
GRANT SELECT ON biz.user_info TO app_user;
REVOKE SELECT(phone,id_card) ON biz.user_info FROM app_user;
```

效果：除 phone、id_card 外，其余字段均可查询。

## 四、角色 + Schema 完整配合流程（实战流程）

### 场景：业务库 biz_db，schema biz，区分管理员、普通操作员、只读账号

#### 1. 创建分组角色（无登录，统一管理权限）

```
-- 分组角色，承载所有权限，不直接登录
CREATE ROLE biz_admin_group;
CREATE ROLE biz_op_group;
CREATE ROLE biz_read_group;
```

#### 2. 创建可登录用户，关联对应分组（继承权限）
```
CREATE USER admin1 LOGIN PASSWORD '123456';
CREATE USER op1 LOGIN PASSWORD '123456';
CREATE USER read1 LOGIN PASSWORD '123456';

-- 用户继承组权限
GRANT biz_admin_group TO admin1;
GRANT biz_op_group TO op1;
GRANT biz_read_group TO read1;

-- 设置默认schema，登录不用手写biz.表名
ALTER ROLE admin1 SET search_path = biz, public;
ALTER ROLE op1 SET search_path = biz, public;
ALTER ROLE read1 SET search_path = biz, public;
```

#### 3. 数据库层权限
```
GRANT CONNECT ON DATABASE biz_db TO biz_admin_group, biz_op_group, biz_read_group;
-- 管理员允许新建schema
GRANT CREATE ON DATABASE biz_db TO biz_admin_group;
```

#### 4. Schema 层权限（核心配合点）
```
-- 管理员：可进入schema、新建表
GRANT USAGE, CREATE ON SCHEMA biz TO biz_admin_group;
-- 操作员/只读：仅能进入schema，不能新建对象
GRANT USAGE ON SCHEMA biz TO biz_op_group, biz_read_group;
```

> 若不给 `USAGE`，后续所有表权限全部失效，角色看不到 schema 内任何表。

#### 5. 表级权限分配
```
-- 管理员：全部操作权限
GRANT ALL ON ALL TABLES IN SCHEMA biz TO biz_admin_group;
-- 操作员：增删改查
GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA biz TO biz_op_group;
-- 只读：仅查询
GRANT SELECT ON ALL TABLES IN SCHEMA biz TO biz_read_group;
```

#### 6. 列级精细化控制（敏感字段隔离）
```
-- 只读账号禁止查看身份证、手机号
REVOKE SELECT(phone,id_card) ON biz.user_info FROM biz_read_group;
-- 操作员不能修改账户余额
REVOKE UPDATE(balance) ON biz.account FROM biz_op_group;
```

## 五、关键区分：角色管身份权限，Schema 管对象隔离

### 1. 角色 (Role / 用户) 负责什么？

1. 身份标识：登录账号、密码、连接数据库；
2. 权限承载：所有层级权限（库 /schema/ 表 / 列）都授予给角色；
3. 权限复用：通过继承批量分配权限，不用逐个给用户授权；
4. 全局配置：`search_path`、会话参数、资源限制。

### 2. Schema 负责什么？

1. 命名隔离：区分多业务模块，避免表名冲突；
2. 权限边界：统一控制一组表的访问入口（USAGE 权限）；
3. 批量授权：支持 `ALL TABLES IN SCHEMA` 批量给内部所有表授权；
4. 默认权限模板：`ALTER DEFAULT PRIVILEGES` 控制该 schema 未来新建对象的权限。

### 3. 两者依赖关系

1. 没有角色，无法授予任何权限；
2. 没有 Schema 的 USAGE 权限，即便有表 / 列权限也无法访问；
3. 完整访问链路：
    
    `角色CONNECT数据库 → Schema拥有USAGE → 表拥有对应CRUD权限 → 字段拥有单列权限`