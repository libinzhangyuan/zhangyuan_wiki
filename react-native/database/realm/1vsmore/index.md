[return](../index)

你想了解在React Native中使用Realm数据库时，1对多（One-to-Many）关系的Model定义方式、核心用法以及相关操作，这是Realm中最常用的关联关系之一。

## 一、核心概念与前置准备
### 1. 环境安装
首先确保在React Native项目中安装Realm：
```bash
# npm
npm install realm --save

# yarn
yarn add realm
```

### 2. 1对多关系核心逻辑
- 「一」的一方（主模型）：包含一个`list`类型的属性，指向「多」的一方（子模型）
- 「多」的一方（子模型）：可通过`linkingObjects`定义反向关联，实现从子模型找到主模型
- Realm所有写操作（增/删/改）必须在**事务（transaction）** 中执行

## 二、1对多Model定义示例
以「用户（User）- 任务（Task）」为例（一个用户有多个任务），完整定义如下：

```javascript
import Realm from 'realm';

// 1. 定义「多」的一方：Task（子模型）
class Task extends Realm.Object {}
Task.schema = {
  name: 'Task', // 模型唯一标识
  primaryKey: 'id', // 主键（建议设置）
  properties: {
    id: 'string', // 任务ID
    title: 'string', // 任务标题
    isCompleted: { type: 'bool', default: false }, // 任务状态，默认未完成
    userId: 'string', // 关联用户ID（外键，用于反向关联）
    // 反向关联：从Task找到所属的User（可选但推荐）
    user: {
      type: 'linkingObjects',
      objectType: 'User', // 关联的主模型名
      property: 'tasks' // 对应User模型中的list属性名
    }
  },
};

// 2. 定义「一」的一方：User（主模型）
class User extends Realm.Object {}
User.schema = {
  name: 'User',
  primaryKey: 'id',
  properties: {
    id: 'string', // 用户ID
    name: 'string', // 用户名
    age: 'int?', // 年龄（?表示可空）
    // 核心：1对多关系定义，list类型指向Task
    tasks: { type: 'list', objectType: 'Task' }
  },
};

// 3. 初始化Realm实例
const initRealm = async () => {
  const realm = await Realm.open({
    schema: [User, Task], // 注册所有模型
    schemaVersion: 1, // 版本号，模型变更时需递增
  });
  return realm;
};
```

### 关键解释
- `type: 'list'`：Realm专门用于1对多关系的属性类型，`objectType`必须与子模型的`name`一致
- `linkingObjects`：实现子模型反向查询主模型，返回的是数组（即使只有一个关联对象）
- 外键`userId`：用于关联User的主键，让数据关系更清晰，非Realm强制要求但建议设置

## 三、1对多关系的核心操作
以下是完整的增/查/改/删示例（基于上面定义的Model）：

```javascript
// 初始化Realm
const realm = await initRealm();

// 1. 新增：创建用户并关联多个任务
const createUserWithTasks = () => {
  realm.write(() => {
    // 创建两个任务
    const task1 = realm.create('Task', {
      id: 'task_001',
      title: '学习Realm 1对多关系',
      userId: 'user_001'
    });
    const task2 = realm.create('Task', {
      id: 'task_002',
      title: '完成RN项目开发',
      userId: 'user_001'
    });

    // 创建用户并关联任务
    realm.create('User', {
      id: 'user_001',
      name: '张三',
      age: 25,
      tasks: [task1, task2] // 关联任务列表
    });
  });
};

// 2. 查询：获取用户及其所有任务
const queryUserTasks = () => {
  // 根据主键查用户
  const user = realm.objectForPrimaryKey('User', 'user_001');
  if (user) {
    console.log('用户名：', user.name);
    console.log('任务数量：', user.tasks.length);
    // 遍历任务
    user.tasks.forEach(task => {
      console.log('任务：', task.title, '状态：', task.isCompleted);
    });

    // 反向查询：从任务找所属用户
    const task = realm.objectForPrimaryKey('Task', 'task_001');
    console.log('任务所属用户：', task.user[0].name); // linkingObjects返回数组
  }
};

// 3. 修改：给用户新增任务/修改任务状态
const updateData = () => {
  realm.write(() => {
    // 新增任务
    const newTask = realm.create('Task', {
      id: 'task_003',
      title: '新增任务',
      userId: 'user_001'
    });
    const user = realm.objectForPrimaryKey('User', 'user_001');
    user.tasks.push(newTask); // 向list添加新任务

    // 修改任务状态
    const task = realm.objectForPrimaryKey('Task', 'task_001');
    task.isCompleted = true;
  });
};

// 4. 删除：删除单个任务/删除用户（级联删除）
const deleteData = () => {
  realm.write(() => {
    // 删除单个任务（Realm自动同步用户的tasks列表）
    const taskToDelete = realm.objectForPrimaryKey('Task', 'task_002');
    realm.delete(taskToDelete);

    // 删除用户（需手动删除关联任务，Realm默认不级联）
    const userToDelete = realm.objectForPrimaryKey('User', 'user_001');
    realm.delete(userToDelete.tasks); // 先删任务
    realm.delete(userToDelete); // 再删用户
  });
};

// 调用示例
createUserWithTasks();
queryUserTasks();
updateData();
deleteData();
```

## 总结
1. Realm的1对多关系核心是在主模型中定义`{ type: 'list', objectType: '子模型名' }`属性；
2. 所有写操作必须包裹在`realm.write()`事务中执行，否则会报错；
3. 子模型可通过`linkingObjects`实现反向关联，Realm默认不支持级联删除，需手动处理关联数据的删除。