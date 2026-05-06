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
# SQL
## DDL
## DML
## DQL
## DCL
