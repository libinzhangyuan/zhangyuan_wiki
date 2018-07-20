[back](../Index)

### find findOne
```
匹配算子 Query Operators
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
db.inventory.find( { status: "A", qty: { $lt: 30 } } 
db.inventory.find( { qty: { $ne: 20 } } )
db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )

 $eq $gt $gte $in $lt $lte $ne $nin
```

### insert

### update
```
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

```
