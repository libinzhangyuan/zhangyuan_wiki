[返回](/postgre-sql/knowledge/index)
# PostgreSQL 中的 ROLLUP 子句

ROLLUP 是 GROUP BY 子句的一个扩展，它生成从最详细级别到总计的多层次分组聚合结果。它是 GROUPING SETS 的一种特殊形式，特别适合用于生成层次化的汇总报表。

## 基本语法

```sql
SELECT column1, column2, ..., aggregate_function(column)
FROM table_name
GROUP BY ROLLUP (column1, column2, ...);
```

## 功能说明

ROLLUP 会按照从右向左的顺序逐步移除分组列，生成不同层次的分组：

1. 首先按所有列分组（最详细级别）
2. 然后移除最右边的列进行分组
3. 重复此过程直到移除所有列（生成总计行）

例如，`ROLLUP (a, b, c)` 等价于：
```sql
GROUPING SETS (
    (a, b, c),  -- 最详细级别
    (a, b),     -- 移除c
    (a),        -- 移除b,c
    ()          -- 总计行
)
```

## 实际示例

假设有一个销售数据表，我们想分析不同地区、不同产品的销售总额：

```sql
SELECT region, product, SUM(sales_amount) AS total_sales
FROM sales
GROUP BY ROLLUP (region, product);
```
```
这将生成：
1. 每个地区+产品组合的详细销售额
2. 每个地区的销售额小计（产品为NULL）
3. 所有地区的销售总额（地区和产品都为NULL）
```
## 与 GROUPING SETS 的区别

- ROLLUP 生成的是有层次结构的分组（从详细到总计）
- GROUPING SETS 可以指定任意组合的分组方式
- ROLLUP (a,b,c) 相当于特定模式的 GROUPING SETS

## 实际应用场景

- 生成带有小计和总计的财务报表
- 制作多级汇总的管理报表
- 数据仓库中的OLAP分析

## 注意事项

- 结果中NULL值可能是原始数据中的NULL，也可能是ROLLUP生成的小计行
- 可以使用GROUPING函数区分这两种NULL
- ROLLUP中列的顺序会影响汇总的层次结构