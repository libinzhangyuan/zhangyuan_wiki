[返回](/postgre-sql/knowledge/index)

## RETURNING 子句

PostgreSQL 的 RETURNING 子句是一个非常实用的功能，它允许在 INSERT、UPDATE 或 DELETE 操作后返回被修改的数据。

### INSERT 与 RETURNING

```sql
INSERT INTO products (name, price, stock) 
VALUES ('笔记本电脑', 5999, 100)
RETURNING id, name, price;
```

可能的返回结果：
```
 id |     name      | price 
----+---------------+-------
 10 | 笔记本电脑   |  5999
```

### UPDATE 与 RETURNING

```sql
UPDATE products 
SET price = price * 0.9 
WHERE stock > 50
RETURNING id, name, price;
```

可能的返回结果：
```
 id |     name      | price 
----+---------------+-------
 10 | 笔记本电脑   | 5399.1
 12 | 智能手机     | 3599.1
```

### DELETE 与 RETURNING

```sql
DELETE FROM products 
WHERE stock = 0
RETURNING id, name;
```

可能的返回结果：
```
 id |     name      
----+---------------
 15 | 旧款手机    
```
