[back](../Index)

### find findOne
```
And: db.inventory.find( { status: "A", qty: { $lt: 30 } } )
Or: db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )

匹配算子 Query Operators
db.inventory.find( { qty: { $eq: 20 } } )
db.inventory.find( { qty: { $gt: 20 } } )
db.inventory.find( { qty: { $gte: 20 } } )
db.inventory.find( { qty: { $lt: 20 } } )
db.inventory.find( { qty: { $lte: 20 } } )
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
db.inventory.find( { qty: { $ne: 20 } } )
db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )    # not in

 $eq $gt $gte $in $lt $lte $ne $nin
```


### insert

### update
```
update_one/update_many # 更新指定属性
replace_one # 完整更换

$inc $set $unset 
数组
 $push 
 $push with $ne
 $pop
 $addToSet
 $addToSet with $each
 $pull
数组中定位修改： 下标方式： db.blog.update({"post":post_id},{"$inc": {"comments.0.votes":1}}
               定位符方式：db.blog.update({"comments.author": "John"}, {"$set": "comments.$.author": "Jim"})

 $upset
 $currentDate: { lastModified: true } # 设置最后修改时间 == updated_at

```

### index
```
db.person.ensureIndex({"age":1})  “1”：表示按照age进行升序，“-1”：表示按照age进行降序
```