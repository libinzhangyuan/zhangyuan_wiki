[返回](/java/doc/mybatis/index)

### mybatis-plus-join-plugin 多表查寻

```markdown
# Dynamic Join 插件与 MyBatis-Plus 多表查询方案

**Dynamic Join** 是一个专门为 MyBatis-Plus 提供动态多表查询支持的插件，简化了复杂的多表连接查询操作。这个插件可以让你通过简单的注解或配置，动态生成 SQL 语句，完成复杂的联表查询，而不必手动编写大量的 SQL 语句。

## 1. 插件简介

Dynamic Join 插件为 MyBatis-Plus 提供了一种更简洁、高效的多表查询方案。它的目标是让开发者在实体类中通过简单的注解来实现多表查询，不用在 XML 文件或 Java 代码中手动编写复杂的 SQL。

### 功能特点

1. **多表关联查询**：通过注解配置，实现多表的 `LEFT JOIN`、`INNER JOIN`、`RIGHT JOIN` 等。
2. **动态条件过滤**：支持动态条件查询，只有在需要时才会生成相应的 `JOIN` 语句。
3. **简化 SQL 构建**：自动生成多表查询的 SQL，避免手写 SQL 的繁琐操作。
4. **与 MyBatis-Plus 无缝集成**：结合 MyBatis-Plus 提供的 CRUD 功能，动态实现多表联查。

## 2. 使用步骤

### 2.1 引入依赖

首先，你需要在项目中引入 Dynamic Join 插件的依赖。通常可以通过 Maven 或 Gradle 来添加。

#### Maven 依赖

```xml
<dependency>
    <groupId>com.github.yulichang</groupId>
    <artifactId>mybatis-plus-join</artifactId>
    <version>最新版本号</version>
</dependency>
```

#### Gradle 依赖

```gradle
implementation 'com.github.yulichang:mybatis-plus-join:最新版本号'
```

### 2.2 配置实体类

Dynamic Join 插件的使用非常简单，你只需要在实体类中通过注解来定义表之间的关系。

假设你有两个表：`User` 和 `Department`，分别对应两个实体类。

#### 实体类 `User`

```java
import com.github.yulichang.annotation.JoinTable;

public class User {

    private Long id;
    private String name;
    private Long deptId;

    @JoinTable(target = Department.class, 
               left = "dept_id", 
               right = "id")
    private Department department;

    // getters and setters
}
```

#### 实体类 `Department`

```java
public class Department {

    private Long id;
    private String deptName;

    // getters and setters
}
```

在 `User` 类中，使用 `@JoinTable` 注解指定了与 `Department` 表的关联关系。这里的 `left` 参数指的是 `User` 表中的 `dept_id` 字段，`right` 参数指的是 `Department` 表中的 `id` 字段。

### 2.3 定义 Mapper 接口

然后，你需要定义一个 Mapper 接口，使用 Dynamic Join 提供的方法来进行多表查询。

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.github.yulichang.base.MPJBaseMapper;
import com.github.yulichang.query.MPJQueryWrapper;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends MPJBaseMapper<User> {

}
```

这里的 `MPJBaseMapper` 是 Dynamic Join 提供的基础 Mapper，它继承自 MyBatis-Plus 的 `BaseMapper`。

### 2.4 进行查询

接下来，你可以在服务层使用 `MPJQueryWrapper` 来动态构建多表查询。

```java
import com.github.yulichang.query.MPJLambdaWrapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public List<User> getUsersWithDepartment() {
        MPJLambdaWrapper<User> wrapper = new MPJLambdaWrapper<>();
        wrapper.selectAll(User.class)
               .select(Department::getDeptName)
               .leftJoin(Department.class, Department::getId, User::getDeptId);

        return userMapper.selectJoinList(User.class, wrapper);
    }
}
```

在上述代码中，我们使用了 `MPJLambdaWrapper` 来动态构建查询条件。`selectJoinList` 方法会根据指定的条件，生成相应的多表查询 SQL。

### 2.5 SQL 输出示例

使用 Dynamic Join 进行查询后，生成的 SQL 类似于：

```sql
SELECT u.id, u.name, d.dept_name 
FROM user u
LEFT JOIN department d ON u.dept_id = d.id
```

这将根据你定义的查询条件自动生成所需的 SQL，并返回结果。

## 3. Dynamic Join 常用注解

- **`@JoinTable`**：用于定义实体类中的关联关系，指定 `left` 和 `right` 字段。
- **`@LeftJoin`、`@InnerJoin`、`@RightJoin`**：用于指定不同的连接方式（左连接、内连接、右连接）。

## 4. 总结

Dynamic Join 插件为 MyBatis-Plus 提供了一种简单高效的多表查询解决方案，通过注解的方式大大简化了多表关联查询的实现，减少了手写 SQL 的工作量。它非常适合在复杂的多表查询场景中使用，尤其是在表关系较为明确的情况下。

你可以通过引入该插件，实现高效且灵活的多表查询，提升项目的开发效率。
```

这个 Markdown 文件结构清晰，并通过代码块展示了如何使用 Dynamic Join 插件进行多表查询的步骤。