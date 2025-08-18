[return](/front/js/index)

"use strict" 是 JavaScript 中的严格模式声明，它允许开发者在更严格的条件下运行代码，有助于捕获常见的编码错误并提高代码安全性。

### 严格模式的主要特点：

1. **变量必须声明后才能使用**
   ```javascript
   "use strict";
   x = 10; // 报错：x is not defined
   ```

2. **禁止删除变量或函数**
   ```javascript
   "use strict";
   var x = 10;
   delete x; // 报错
   ```

3. **禁止使用八进制语法**
   ```javascript
   "use strict";
   var x = 010; // 报错
   ```

4. **禁止使用 with 语句**
   ```javascript
   "use strict";
   with (Math) { x = cos(2) }; // 报错
   ```

5. **函数参数不能重名**
   ```javascript
   "use strict";
   function func(a, a) {}; // 报错
   ```

6. **禁止 this 指向全局对象**
   ```javascript
   "use strict";
   function func() {
     console.log(this); // undefined
   }
   func();
   ```

7. **禁止在函数内部声明 eval 或 arguments 变量**
   ```javascript
   "use strict";
   function func() {
     var eval = 10; // 报错
   }
   ```

### 使用方式：
- 在脚本文件顶部添加，作用于整个脚本：
  ```javascript
  "use strict";
  // 整个脚本都处于严格模式
  ```

- 在函数内部添加，仅作用于该函数：
  ```javascript
  function strictFunc() {
    "use strict";
    // 此函数内为严格模式
  }
  ```

严格模式有助于写出更安全、更规范的 JavaScript 代码，现代 JavaScript 开发中推荐使用。它能提前发现潜在问题，减少代码运行时的意外行为。