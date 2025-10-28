[return](/react-native/index)

在 TSX（TypeScript + JSX）语法中，`&` 符号的含义需要分两种场景来看：**值层面**和**类型层面**，二者用途完全不同，且都不直接用于“对象合并”（值层面的对象合并仍沿用 JavaScript 的语法）。


### 1. 值层面：`&` 仍是按位与运算符
在 TSX 中，当 `&` 用于**值（变量、表达式）** 时，作用和 JavaScript 完全一致，即**按位与运算符**，仅用于数值的二进制位运算，与对象合并无关。

示例：
```tsx
const a = 5; // 二进制 0101
const b = 3; // 二进制 0011

// 按位与运算
const result = a & b; // 1（二进制 0001）

// 对对象使用会报错（TypeScript 会提示类型错误）
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const wrong = obj1 & obj2; // 报错：Object 类型不能用于按位运算
```


### 2. 类型层面：`&` 是交叉类型（Intersection Type）运算符
在 TypeScript（包括 TSX）的**类型定义**中，`&` 是**交叉类型**的运算符，用于将多个类型“合并”为一个新类型，新类型包含所有原类型的属性和方法。

这是**类型系统层面的合并**，而非运行时对象的合并。

#### 语法：
```typescript
type TypeA = { a: number };
type TypeB = { b: string };
// 交叉类型：同时包含 TypeA 和 TypeB 的属性
type TypeC = TypeA & TypeB; // { a: number; b: string }
```

#### 示例（TSX 中使用）：
```tsx
// 定义两个接口类型
interface User {
  name: string;
}
interface Age {
  age: number;
}

// 交叉类型：合并 User 和 Age
type UserWithAge = User & Age; // { name: string; age: number }

// 使用交叉类型
const user: UserWithAge = {
  name: "Alice",
  age: 20, // 必须同时包含两个类型的属性
};
```

#### 特点：
- 交叉类型是“并且”的关系（`A & B` 表示“既是 A 类型，又是 B 类型”）；
- 若合并的类型有同名属性，且属性类型不兼容，会产生 `never` 类型（无法赋值）：
  ```typescript
  type A = { x: number };
  type B = { x: string };
  type C = A & B; // { x: never }（number 和 string 无交集）
  const c: C = { x: ??? }; // 无法赋值，因为 x 必须同时是 number 和 string
  ```


### 3. TSX 中对象（值）的合并语法
如果需要在 TSX 中合并**运行时的对象**（值层面），语法和 JavaScript 完全一致，仍使用：
- 扩展运算符（`...`）
- `Object.assign()`

示例：
```tsx
const obj1 = { a: 1, nested: { x: 10 } };
const obj2 = { b: 2, nested: { y: 20 } };

// 浅合并
const merged = { ...obj1, ...obj2 };
// merged 类型自动推断为：{ a: number; b: number; nested: { y: number } }
// （nested 被 obj2 覆盖，因为是浅合并）

// 深合并仍需手动实现或用库（如 lodash.merge）
import _ from 'lodash';
const deepMerged = _.merge({}, obj1, obj2);
// deepMerged.nested: { x: 10, y: 20 }
```


### 总结
- 在 TSX 中，`&` 在**值层面**是按位与运算符（同 JavaScript），不用于对象合并；
- 在**类型层面**，`&` 是交叉类型运算符，用于合并类型（仅影响类型检查，不涉及运行时）；
- 实际合并对象（值）仍使用扩展运算符（`...`）或 `Object.assign()`，与 JavaScript 一致。




<br><br><br>

在 JavaScript 中，“哈希”通常指对象（Object），对象的合并是常见操作。以下是几种常用的对象合并语法和方法，以及它们的特点：


### 1. `Object.assign()` 方法
`Object.assign(target, ...sources)` 用于将一个或多个源对象的可枚举属性复制到目标对象，返回合并后的目标对象。

**语法：**
```javascript
const merged = Object.assign(target, source1, source2, ...);
```

**示例：**
```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

// 合并到新对象（避免修改原对象）
const merged = Object.assign({}, obj1, obj2);
console.log(merged); // { a: 1, b: 3, c: 4 }（obj2的b覆盖obj1的b）
```

**特点：**
- 若有同名属性，后面的源对象属性会覆盖前面的；
- 是**浅合并**（若属性值为引用类型，仅复制引用，修改会相互影响）；
- 第一个参数是目标对象，若想不修改原对象，可传入空对象 `{}` 作为目标。


### 2. 扩展运算符（`...`）
ES6 引入的扩展运算符可用于“展开”对象属性，通过字面量语法实现合并，更简洁。

**语法：**
```javascript
const merged = { ...source1, ...source2, ...sourceN };
```

**示例：**
```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 3, c: 4 }（同assign，后面对象覆盖前面）
```

**特点：**
- 与 `Object.assign()` 行为一致，也是**浅合并**；
- 语法更简洁，适合合并少量对象；
- 不会修改原对象，始终返回新对象。


### 3. 处理嵌套对象（深合并）
上面两种方法都是浅合并，若对象包含嵌套对象（引用类型属性），需要深合并（递归合并嵌套属性）。

#### 手动实现简单深合并：
```javascript
function deepMerge(target, ...sources) {
  if (!sources.length) return target;
  const source = sources.shift();

  if (isObject(target) && isObject(source)) {
    for (const key in source) {
      if (isObject(source[key])) {
        if (!target[key]) Object.assign(target, { [key]: {} });
        deepMerge(target[key], source[key]); // 递归合并嵌套对象
      } else {
        Object.assign(target, { [key]: source[key] });
      }
    }
  }

  return deepMerge(target, ...sources);
}

// 辅助函数：判断是否为对象（排除null）
function isObject(item) {
  return item && typeof item === 'object' && !Array.isArray(item);
}

// 示例
const obj1 = { a: 1, nested: { x: 10 } };
const obj2 = { b: 2, nested: { y: 20 } };
const merged = deepMerge({}, obj1, obj2);
console.log(merged); // { a: 1, b: 2, nested: { x: 10, y: 20 } }
```

#### 使用库（推荐）：
实际开发中，深合并逻辑较复杂（如处理数组、特殊类型等），建议使用成熟库：
- Lodash 的 `_.merge()`（深合并）：
  ```javascript
  import _ from 'lodash';
  const merged = _.merge(obj1, obj2);
  ```


### 总结
- 简单浅合并：优先用扩展运算符 `{ ...a, ...b }` 或 `Object.assign()`；
- 深合并：使用 Lodash 的 `_.merge()` 或手动实现递归逻辑；
- 注意：合并时同名属性会被后面的对象覆盖，引用类型属性需关注浅/深合并的区别。


