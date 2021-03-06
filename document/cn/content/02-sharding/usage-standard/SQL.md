+++
toc = true
title = "SQL"
weight = 2
+++

## 全局支持项

### DQL

#### SELECT主语句

```sql
SELECT select_expr [, select_expr ...] FROM table_reference [, table_reference ...]
[WHERE where_condition] 
[GROUP BY {col_name | position} [ASC | DESC]] 
[ORDER BY {col_name | position} [ASC | DESC], ...] 
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

#### select_expr

```sql
* | 
COLUMN_NAME [AS] [alias] | 
(MAX | MIN | SUM | AVG)(COLUMN_NAME | alias) [AS] [alias] | 
COUNT(* | COLUMN_NAME | alias) [AS] [alias]
```

#### table_reference

```sql
tbl_name [AS] alias] [index_hint_list] | 
table_reference ([INNER] | {LEFT|RIGHT} [OUTER]) JOIN table_factor [JOIN ON conditional_expr | USING (column_list)] | 
```

## 全局不支持项

### 有限支持子查询
Sharding-JDBC除了分页子查询的支持之外(详情请参考[分页](/02-sharding/subquery/))，也支持同等模式的子查询。无论嵌套多少层，Sharding-JDBC都可以解析至第一个包含数据表的子查询，一旦在下层嵌套中再次找到包含数据表的子查询将直接抛出解析异常。

例如，以下子查询可以支持：

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o)
```

以下子查询不支持：

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o WHERE o.id IN (SELECT id FROM t_order WHERE status = ?))
```

简单来说，通过子查询进行非功能需求，在大部分情况下是可以支持的。比如分页、统计总数等；而通过子查询实现业务查询当前并不能支持。

由于归并的限制，子查询中包含聚合函数目前无法支持。

### 不支持包含冗余括号的SQL

### 不支持OR

### 不支持CASE WHEN


#### 示例

### DQL

| SQL                                      | 无条件支持 | 必要条件 |
| ---------------------------------------- | ----- | ---- |
| SELECT * FROM tbl_name                   | 是     |      |
| SELECT * FROM tbl_name WHERE col1 = val1 ORDER BY col2 DESC LIMIT limit | 是     |      |
| SELECT COUNT(*), SUM(col1), MIN(col1), MAX(col1), AVG(col1) FROM tbl_name WHERE col1 = val1 | 是     |      |
| SELECT COUNT(col1) FROM tbl_name WHERE col2 = val2 GROUP BY col1 ORDER BY col3 DESC LIMIT offset, limit | 是     |      |

### DML

| SQL                                      | 无条件支持 | 必要条件        |
| ---------------------------------------- | ----- | ----------- |
| INSERT INTO tbl_name (col1, col2,...) VALUES (val1, val2,....) | 否     | 插入列需要包含分片键  |
| INSERT INTO tbl_name VALUES (val1, val2,....) | 否     | 通过Hint注入分片键 |
| UPDATE tbl_name SET col1 = val1 WHERE col2 = val2 | 是     |             |
| DELETE FROM tbl_name WHERE col1 = val1   | 是     |             |

### DDL

| SQL                                      | 无条件支持 | 必要条件                    |
| ---------------------------------------- | ----- | ----------------------- |
| CREATE TABLE tbl_name (col1 int,...)     | 是     |                         |
| ALTER TABLE tbl_name ADD col1 varchar(10) | 是     |                         |
| DROP TABLE tbl_name                      | 是     |                         |
| TRUNCATE TABLE tbl_name                  | 是     |                         |
| CREATE INDEX idx_name ON tbl_name        | 是     |                         |
| DROP INDEX idx_name ON tbl_name          | 是     |                         |
| DROP INDEX idx_name                      | 是     | tableRule中配置logic-index |

## 不支持的SQL

| SQL                                      |
| ---------------------------------------- |
| INSERT INTO tbl_name (col1, col2, ...) VALUES (val1, val2,....), (val3, val4,....) |
| INSERT INTO tbl_name (col1, col2, ...) SELECT col1, col2, ... FROM tbl_name WHERE col3 = val3 |
| INSERT INTO tbl_name SET col1 = val1     |
| SELECT DISTINCT * FROM tbl_name WHERE column1 = value1 |
| SELECT * FROM tbl_name WHERE column1 = value1 OR column1 = value2 |
| SELECT COUNT(col1) as count_alias FROM tbl_name GROUP BY col1 HAVING count_alias > val1 |
| SELECT * FROM tbl_name1 UNION SELECT * FROM tbl_name2 |
| SELECT * FROM tbl_name1 UNION ALL SELECT * FROM tbl_name2 |
| SELECT * FROM tbl_name1 WHERE (val1=?) AND (val1=?) |
