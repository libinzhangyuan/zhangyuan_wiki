

### 在数据库服务器上dump数据
```
rails db
nbd52+

非高峰时间执行:
nohup mysqldump -hsoul-en-db -unbddog -pnbd5+ soul_english | gzip > soul_english-`date +%Y-%m-%d`.sql.gz &



# http://stackoverflow.com/questions/104612/run-mysqldump-without-locking-tables
```


### 下载到需要导入数据库的机器
```
cd /home/zhangyuan/1/
tar -xjvf soul_english.sql.tar.bz2
```

### do 导入数据库
```
# 在你本地测试机上!!!!!!!
mysql -h soul-en-db -u nbddog -p
nbd5+

use soul_english;
source /home/zhangyuan/1/soul_english.sql 
```