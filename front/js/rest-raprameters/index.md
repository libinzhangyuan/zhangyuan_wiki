[return](/front/js/index)

在 JavaScript 中，剩余参数（Rest Parameters）语法 `...args` 允许我们将一个不定数量的参数表示为一个数组，这在处理函数参数时非常有用。

### 基本语法
```javascript
function func(...args) {
  // args 是一个包含所有传入参数的数组
  console.log(args);
}
```

### 主要特点
1. **收集剩余参数**：当函数参数数量不确定时，`...` 会收集所有剩余参数到一个数组中
2. **必须是最后一个参数**：剩余参数必须是函数的最后一个参数
3. **与 arguments 对象的区别**：
   - 剩余参数是真正的数组，可以直接使用数组方法
   - `arguments` 是类数组对象，需要转换才能使用数组方法

### 使用示例

1. **处理任意数量的参数**：
```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(10, 20, 30, 40)); // 100
```

2. **与普通参数结合使用**：
```javascript
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(' and ')}!`;
}

console.log(greet('Hello', 'Alice', 'Bob')); // "Hello, Alice and Bob!"
console.log(greet('Hi', 'Charlie')); // "Hi, Charlie!"
```

3. **在箭头函数中使用**：
```javascript
const multiply = (...nums) => nums.reduce((product, num) => product * num, 1);
console.log(multiply(2, 3, 4)); // 24
```

4. **解构配合剩余参数**：
```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [2, 3, 4, 5]
```

剩余参数语法让处理可变数量的参数变得更加简洁和灵活，是现代 JavaScript 中常用的特性之一。