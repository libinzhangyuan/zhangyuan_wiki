# Java try-with-resources 详解

try-with-resources 是 Java 7 引入的一种语法结构，用于简化资源管理，能够自动关闭实现了 AutoCloseable 或 Closeable 接口的资源。

## 基本语法

```java
try (资源类型 变量名 = new 资源类型()) {
    // 使用资源的代码
} catch (异常类型 e) {
    // 异常处理
}
```

## 核心特点

1. **自动资源管理**：在 try 代码块执行完毕后，资源会自动关闭，无需手动调用 close() 方法。

2. **支持多资源**：可以同时管理多个资源，用分号隔开。

```java
try (InputStream in = new FileInputStream("input.txt");
     OutputStream out = new FileOutputStream("output.txt")) {
    // 使用这两个资源
}
```

3. **关闭顺序**：资源按照声明的相反顺序关闭（后声明的先关闭）。

## 使用要求

要在 try-with-resources 中使用的类必须：
- 实现 `AutoCloseable` 接口（Java 7 引入）
- 或者实现 `Closeable` 接口（现在扩展了 AutoCloseable）

## 优势

- **减少样板代码**：不需要显式编写 finally 块来关闭资源
- **更可靠**：即使发生异常，资源也会被正确关闭
- **代码更清晰**：资源管理逻辑更加直观

## 示例代码

```java
import java.io.*;

public class TryWithResources示例 {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // br 在这里会自动关闭
    }
}
```

## 异常处理机制

如果 try 块和自动关闭都抛出异常：
- try 块的异常会被抛出
- 关闭时的异常会被抑制（可以通过 Throwable.getSuppressed() 获取）

## Java 7 之前的等价写法

没有 try-with-resources 时的传统写法：

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("test.txt"));
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

try-with-resources 使代码更加简洁且不易出错，是处理需要关闭的资源时的首选方式。