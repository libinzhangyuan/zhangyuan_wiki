[返回](/postgre-sql/knowledge/index)

* [bool_or bool_and](bool-or-bool-and/index)


# PostgreSQL 中的聚合函数

聚合函数是 SQL 中用于对一组值执行计算并返回单个值的函数，在数据分析、报表生成和数据汇总中非常有用。PostgreSQL 提供了丰富的聚合函数集，包括标准函数和高级函数。

## 常用聚合函数

### 基础聚合函数

1. **COUNT()** - 计算行数
   ```sql
   SELECT COUNT(*) FROM employees; -- 计算总行数
   SELECT COUNT(department) FROM employees; -- 计算非NULL值的数量
   ```

2. **SUM()** - 计算总和
   ```sql
   SELECT SUM(salary) FROM employees;
   ```

3. **AVG()** - 计算平均值
   ```sql
   SELECT AVG(salary) FROM employees;
   ```

4. **MIN()** - 查找最小值
   ```sql
   SELECT MIN(hire_date) FROM employees;
   ```

5. **MAX()** - 查找最大值
   ```sql
   SELECT MAX(salary) FROM employees;
   ```

### 高级聚合函数

6. **STRING_AGG()** - 字符串连接
   ```sql
   SELECT department, STRING_AGG(name, ', ') 
   FROM employees 
   GROUP BY department;
   ```

7. **ARRAY_AGG()** - 将值聚合成数组
   ```sql
   SELECT department, ARRAY_AGG(name) 
   FROM employees 
   GROUP BY department;
   ```

8. **JSON_AGG()** - 将值聚合成JSON数组
   ```sql
   SELECT department, JSON_AGG(json_build_object('name', name, 'id', id))
   FROM employees 
   GROUP BY department;
   ```

9. **PERCENTILE_CONT()** - 连续百分位数
   ```sql
   SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) 
   FROM employees;
   ```

10. **PERCENTILE_DISC()** - 离散百分位数
    ```sql
    SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) 
    FROM employees;
    ```

## 统计聚合函数

11. **STDDEV()** - 标准差
    ```sql
    SELECT STDDEV(salary) FROM employees;
    ```

12. **VARIANCE()** - 方差
    ```sql
    SELECT VARIANCE(salary) FROM employees;
    ```

13. **CORR()** - 相关系数
    ```sql
    SELECT CORR(salary, bonus) FROM employees;
    ```

14. **REGR_** 系列函数 - 线性回归函数
    ```sql
    SELECT REGR_SLOPE(y, x), REGR_INTERCEPT(y, x) FROM data;
    ```

## 布尔聚合函数

15. **BOOL_AND()** - 所有值为真时返回真
    ```sql
    SELECT BOOL_AND(is_active) FROM users WHERE department = 'IT';
    ```

16. **BOOL_OR()** - 任意值为真时返回真
    ```sql
    SELECT BOOL_OR(is_admin) FROM users;
    ```

## 自定义聚合函数

PostgreSQL 允许创建自定义聚合函数：
```sql
CREATE AGGREGATE my_agg (input_type) (
    SFUNC = state_func,
    STYPE = state_type,
    FINALFUNC = final_func,
    INITCOND = 'initial_state'
);
```

## 使用技巧

- 可以与 GROUP BY 结合使用进行分组聚合
- 使用 FILTER 子句进行条件聚合：
  ```sql
  SELECT 
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE salary > 5000) AS high_earners
  FROM employees;
  ```
- 使用 DISTINCT 去重聚合：
  ```sql
  SELECT COUNT(DISTINCT department) FROM employees;
  ```

聚合函数是 PostgreSQL 数据分析的核心功能，熟练掌握这些函数可以极大地提高数据处理的效率和灵活性。