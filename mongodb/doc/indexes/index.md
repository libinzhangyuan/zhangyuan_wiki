```
创建
db.createIndex({title: 1})

# 查看索引
db.getIndexes()

# 查看索引键
db.getIndexKeys()

# 查看索引占用空间
db.books.totalIndexSize()
db.books.totalIndexSize(1) # 更详细

# 删除索引
db.collection.dropIndex("books")



```