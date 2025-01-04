

在 Spring Boot 中使用 `MongoRepository` 时，`@Query` 注解可以用来定义自定义的查询语法。Spring Data MongoDB 使用 MongoDB 的查询语言来查询数据，`@Query` 注解使你能够执行更复杂的查询。

以下是如何在 `MongoRepository` 中使用 `@Query` 的示例：

### 1. 定义实体类 (Model)

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "users")
public class User {

    @Id
    private String id;
    private String name;
    private int age;

    // Getter 和 Setter 方法
}
```

### 2. 创建 MongoRepository 接口

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

import java.util.List;

public interface UserRepository extends MongoRepository<User, String> {

    // 使用 @Query 注解进行自定义查询
    @Query("{ 'name' : ?0 }") // 查询 name 等于传入参数的文档
    List<User> findByName(String name);

    // 使用 @Query 注解查询 age 大于传入值的所有用户
    @Query("{ 'age' : { $gt: ?0 } }")
    List<User> findByAgeGreaterThan(int age);

    // 使用 @Query 注解查询 name 等于某个值并且 age 大于某个值的用户
    @Query("{ 'name' : ?0, 'age' : { $gt: ?1 } }")
    List<User> findByNameAndAgeGreaterThan(String name, int age);

    // 使用 @Query 注解进行投影查询，只返回 name 字段
    @Query("{ 'name' : ?0 }")
    List<String> findNameByName(String name);
}
```

### 3. 使用自定义查询方法

在你的服务层或控制器中，你可以使用这些方法来执行查询：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> getUsersByName(String name) {
        return userRepository.findByName(name);
    }

    public List<User> getUsersByAgeGreaterThan(int age) {
        return userRepository.findByAgeGreaterThan(age);
    }

    public List<User> getUsersByNameAndAge(String name, int age) {
        return userRepository.findByNameAndAgeGreaterThan(name, age);
    }

    public List<String> getUserNamesByName(String name) {
        return userRepository.findNameByName(name);
    }
}
```

### 4. 使用复杂查询和运算符

你还可以使用 MongoDB 查询操作符，比如 `$gt`, `$lt`, `$in`, `$and` 等来进行更复杂的查询。例如：

```java
// 查询年龄大于等于 18 且名字是 "John" 的所有用户
@Query("{ 'age' : { $gte: 18 }, 'name' : 'John' }")
List<User> findAdultsNamedJohn();

// 查询年龄在 20 到 30 之间的用户
@Query("{ 'age' : { $gte: 20, $lte: 30 } }")
List<User> findUsersInAgeRange();
```

### 5. 使用 @Query 与动态查询（参数传递）

如果你需要根据多个条件动态生成查询，可以使用 Spring Data MongoDB 的查询参数：

```java
@Query("{ 'name' : ?0, 'age' : { $gte: ?1 } }")
List<User> findByNameAndAgeGreaterThan(String name, int age);
```

通过以上方式，你可以使用 `@Query` 注解灵活地定义 MongoDB 查询，执行复杂的数据筛选操作。

### 注意事项

1. **占位符：** 在 `@Query` 中，使用 `?0`, `?1` 等来表示方法参数的位置。
2. **MongoDB 查询语法：** `@Query` 注解支持 MongoDB 的 JSON 查询语法，确保了解 MongoDB 查询的基本语法（如运算符、嵌套查询等）。
3. **性能考虑：** 对于复杂的查询，尤其是包含多层嵌套或者使用 `$or` 等条件时，确保查询能够高效执行，避免性能瓶颈。