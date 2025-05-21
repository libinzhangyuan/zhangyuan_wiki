[返回](/postgre-sql/knowledge/index)

# PostgreSQL 中的 GROUP BY CUBE

GROUP BY CUBE 是 PostgreSQL 中一种强大的分组操作扩展，它属于 GROUP BY 子句的高级功能之一，用于生成多维度的数据汇总。

## 基本概念

CUBE 会为指定的列集合生成所有可能的分组组合，包括：
- 每个单独列的分组
- 所有列组合的分组
- 全局总计（空分组）

## 语法

```sql
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY CUBE (column1, column2);
```

## 示例

假设有一个销售数据表 sales(product, region, year, amount)：

```sql
SELECT product, region, year, SUM(amount) as total
FROM sales
GROUP BY CUBE (product, region, year);
```
```
这个查询会生成：
1. 按 product, region, year 三列分组的结果
2. 按 product, region 两列分组的结果
3. 按 product, year 两列分组的结果
4. 按 region, year 两列分组的结果
5. 按 product 单独分组的结果
6. 按 region 单独分组的结果
7. 按 year 单独分组的结果
8. 全局总计（不按任何列分组）
```
## 实际应用

CUBE 特别适合用于：
- 生成数据透视表
- 多维数据分析
- 商业智能报表
- 快速查看数据的各种汇总维度

## 与 ROLLUP 的区别

CUBE 会生成所有可能的分组组合，而 ROLLUP 则生成层次化的分组（从最详细到最汇总）。

## 注意事项

- CUBE 生成的分组数量是指数列的2^n次方（n是指定的列数）
- 对于大量列，可能会产生非常大的结果集
- PostgreSQL 9.5 及以上版本支持此功能

CUBE 是数据分析中非常有用的工具，可以一次性获取数据的多种维度汇总视图。