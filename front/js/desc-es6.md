[返回](/front/js/index)

ES6+ 引入了许多强大的特性，极大地提升了 JavaScript 的开发效率和代码质量。以下是你提到的几个核心特性的详细介绍：


### 1. 箭头函数（Arrow Functions）
箭头函数是函数的简洁写法，具有以下特点：
- 语法简洁，使用 `=>` 定义函数
- 没有自己的 `this`，继承自外层作用域
- 不能用作构造函数，无法使用 `new` 调用
- 没有 `arguments` 对象

```javascript
// 传统函数
const add = function(a, b) {
  return a + b;
};

// 箭头函数等价写法
const add = (a, b) => a + b;

// 带函数体的箭头函数
const greet = (name) => {
  console.log(`Hello, ${name}!`);
  return name;
};
```


### 2. 解构赋值（Destructuring Assignment）
允许从数组或对象中提取值，并赋值给变量，使代码更简洁。

**数组解构**：
```javascript
const [a, b, ...rest] = [1, 2, 3, 4, 5];
console.log(a); // 1
console.log(b); // 2
console.log(rest); // [3,4,5]
```

**对象解构**：
```javascript
const user = { name: 'Alice', age: 30, city: 'New York' };
const { name, age } = user;
console.log(name); // 'Alice'
console.log(age); // 30

// 重命名变量
const { city: userCity } = user;
console.log(userCity); // 'New York'
```


### 3. let/const
引入块级作用域的变量声明方式，替代了 `var`：

- `let`：声明可重新赋值的变量，具有块级作用域
- `const`：声明不可重新赋值的常量（但对象/数组的属性可修改），同样具有块级作用域

```javascript
if (true) {
  let x = 10; // 块级作用域，外部无法访问
  const y = 20; // 常量，不能重新赋值
  // y = 30; // 报错
}

// console.log(x); // 报错，x未定义
```


### 4. Promise
用于处理异步操作，解决了回调地狱问题，有三种状态：`pending`（进行中）、`fulfilled`（成功）、`rejected`（失败）。

```javascript
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) {
        resolve('数据获取成功'); // 成功时调用
      } else {
        reject('数据获取失败'); // 失败时调用
      }
    }, 1000);
  });
};

// 使用 Promise
fetchData()
  .then(data => console.log(data)) // 成功回调
  .catch(error => console.error(error)) // 失败回调
  .finally(() => console.log('操作完成')); // 无论成功失败都会执行
```


### 5. async/await
建立在 Promise 之上的语法糖，使异步代码看起来像同步代码，更易读。

- `async` 声明异步函数，返回一个 Promise
- `await` 只能在 `async` 函数中使用，等待 Promise 完成

```javascript
// 结合上面的 fetchData 函数
const processData = async () => {
  try {
    const data = await fetchData(); // 等待 Promise 完成
    console.log(data);
    return data;
  } catch (error) {
    console.error(error);
  } finally {
    console.log('处理完成');
  }
};

processData();
```


### 6. 模块（import/export）
实现了 JavaScript 的模块化，允许将代码分割到不同文件，通过 `import`/`export` 共享功能。

**导出（export）**：
```javascript
// utils.js
export const sum = (a, b) => a + b;

export const multiply = (a, b) => a * b;

// 默认导出（一个模块只能有一个）
export default function greet(name) {
  return `Hello, ${name}`;
}
```

**导入（import）**：
```javascript
// main.js
import greet, { sum, multiply } from './utils.js';

console.log(greet('Bob')); // 'Hello, Bob'
console.log(sum(2, 3)); // 5
console.log(multiply(2, 3)); // 6
```

使用模块时，需要在 HTML 中添加 `type="module"`：
```html
<script type="module" src="main.js"></script>
```


这些特性共同构成了现代 JavaScript 开发的基础，大幅提升了代码的可读性、可维护性和开发效率。