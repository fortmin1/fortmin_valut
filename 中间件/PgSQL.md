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