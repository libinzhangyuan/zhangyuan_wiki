[return](../index)

Props 是 React 中组件间传递数据的核心机制，全称是 “Properties”，本质是父组件传递给子组件的只读参数，用于实现组件间的通信和数据共享。


### 一、核心概念与关键特性
```
Props 的核心作用是“父传子”，让子组件能根据父组件传递的数据动态渲染，其关键特性决定了使用规则：
1. **只读性（不可修改）**：子组件不能直接修改 props 的值，若需修改，需由父组件更新传递的数据源（如父组件的 state）。直接修改 props 会触发 React 报错。
2. **单向数据流**：数据只能从父组件流向子组件，若子组件需影响父组件，需通过 props 传递“回调函数”间接触发父组件的状态更新。
3. **传递任意类型**：props 可传递基本类型（字符串、数字、布尔值）、复杂类型（对象、数组）、函数，甚至是 React 组件。
```

### 二、基本使用流程
Props 的使用分为“父组件传递”和“子组件接收”两步，以下以**函数组件**（React 主流写法）为例说明：

#### 1. 父组件：传递 Props
父组件在使用子组件时，通过“属性名=值”的形式传递数据，类似 HTML 标签的属性。
```
// 父组件 App
import React from 'react';
import Child from './Child'; // 导入子组件

const App = () => {
  // 1. 定义要传递给子组件的数据
  const articleTitle = "React Props 详解";
  const articleAuthor = { name: "小明", age: 28 };
  const handleLike = () => alert("文章被点赞了！"); // 要传递的函数

  // 2. 向子组件传递 props（属性名可自定义）
  return (
    <div>
      <h1>父组件</h1>
      <Child 
        title={articleTitle} // 传递基本类型（字符串）
        author={articleAuthor} // 传递复杂类型（对象）
        onLike={handleLike} // 传递函数（回调）
        isShowAuthor={true} // 传递布尔值
      />
    </div>
  );
};

export default App;
```

#### 2. 子组件：接收并使用 Props
子组件通过“函数参数”接收 props 对象，直接解构或通过对象属性使用传递的数据。
```
// 子组件 Child
import React from 'react';

// 方式1：直接接收 props 对象，通过 props.xxx 使用
// const Child = (props) => {
//   return <h2>{props.title}</h2>;
// };

// 方式2：解构 props（更简洁，推荐）
const Child = ({ title, author, onLike, isShowAuthor }) => {
  return (
    <div className="article">
      <h2>{title}</h2> {/* 使用基本类型 props */}
      {/* 条件渲染：根据 isShowAuthor 决定是否显示作者信息 */}
      {isShowAuthor && (
        <p>作者：{author.name}（{author.age}岁）</p> {/* 使用复杂类型 props */}
      )}
      <button onClick={onLike}>点赞</button> {/* 使用函数 props（触发父组件逻辑） */}
    </div>
  );
};

export default Child;
```

**返回结果**（组件渲染后）：
```html
<div>
  <h1>父组件</h1>
  <div class="article">
    <h2>React Props 详解</h2>
    <p>作者：小明（28岁）</p>
    <button>点赞</button>
  </div>
</div>
```


### 三、进阶用法：默认值与类型校验
为避免 props 未传递时出现“undefined”错误，或确保传递的数据类型正确，需配置默认值和类型校验：

#### 1. 设置 Props 默认值
当父组件未传递某个 props 时，子组件可通过“函数参数默认值”或 `defaultProps`（类组件常用）设置默认值。
```
// 子组件：通过函数参数设置默认值（推荐函数组件使用）
const Child = ({ 
  title = "默认文章标题", // 未传递 title 时使用默认值
  isShowAuthor = false // 未传递 isShowAuthor 时默认隐藏
}) => {
  return (
    <div>
      <h2>{title}</h2>
      {isShowAuthor && <p>作者信息</p>}
    </div>
  );
};

// 父组件：未传递 title 和 isShowAuthor
const App = () => <Child />;
```

**返回结果**（未传递 props 时）：
```html
<div>
  <h2>默认文章标题</h2>
  <!-- isShowAuthor 为 false，不渲染作者信息 -->
</div>
```

#### 2. Props 类型校验
通过 `prop-types` 库（需额外安装）或 TypeScript 约束 props 的类型，避免传递错误类型的数据（如把数字传给需要字符串的 props）。
```
// 1. 安装依赖：npm install prop-types
// 2. 子组件中使用 PropTypes 校验
import React from 'react';
import PropTypes from 'prop-types';

const Child = ({ title, author, onLike }) => {
  return (
    <div>
      <h2>{title}</h2>
      <p>{author.name}</p>
      <button onClick={onLike}>点赞</button>
    </div>
  );
};

// 定义类型校验规则
Child.propTypes = {
  title: PropTypes.string.isRequired, // title 必须是字符串，且必填
  author: PropTypes.shape({ // author 必须是包含 name 属性的对象
    name: PropTypes.string.isRequired
  }).isRequired,
  onLike: PropTypes.func.isRequired // onLike 必须是函数，且必填
};

export default Child;
```
- 若父组件传递的 `title` 是数字（如 `title={123}`），控制台会触发警告，提示“Invalid prop `title` of type `number` supplied to `Child`, expected `string`”。


### 四、常见使用场景示例
Props 可适配不同业务场景，以下是 3 个典型场景的完整示例：

#### 场景 1：传递数组并渲染列表
父组件传递数组，子组件接收后用 `map` 渲染列表：
```
// 父组件：传递文章列表数组
const App = () => {
  const articles = [
    { id: 1, title: "Props 基础" },
    { id: 2, title: "Props 类型校验" }
  ];
  return <ArticleList list={articles} />;
};

// 子组件：接收数组并渲染列表
const ArticleList = ({ list }) => {
  return (
    <ul>
      {list.map(article => (
        <li key={article.id}>{article.title}</li> // key 为列表渲染必填项
      ))}
    </ul>
  );
};
```
**返回结果**：
```html
<ul>
  <li>Props 基础</li>
  <li>Props 类型校验</li>
</ul>
```

#### 场景 2：传递函数实现子传父通信
子组件通过调用 props 传递的函数，将数据传递给父组件：
```
// 父组件：定义状态和回调函数
const App = () => {
  const [inputValue, setInputValue] = React.useState("");

  // 子组件触发的回调函数，接收子组件传递的值
  const handleInputChange = (value) => {
    setInputValue(value);
  };

  return (
    <div>
      <p>父组件接收的值：{inputValue}</p>
      <Input onValueChange={handleInputChange} />
    </div>
  );
};

// 子组件：输入框变化时调用父组件的函数
const Input = ({ onValueChange }) => {
  const handleChange = (e) => {
    onValueChange(e.target.value); // 向父组件传递输入框的值
  };
  return <input type="text" onChange={handleChange} />;
};
```
**返回结果**：
- 当在输入框中输入“hello”时，父组件会实时显示“父组件接收的值：hello”。

#### 场景 3：传递组件（组件组合）
Props 可传递自定义组件，实现组件的灵活组合：
```
// 父组件：传递 Button 组件给 Card 组件
const App = () => {
  const CustomButton = () => <button>自定义按钮</button>;
  return (
    <Card 
      title="带按钮的卡片" 
      actionComponent={<CustomButton />} // 传递组件
    />
  );
};

// 子组件：接收并渲染传递的组件
const Card = ({ title, actionComponent }) => {
  return (
    <div style={{ border: "1px solid #000", padding: "10px" }}>
      <h3>{title}</h3>
      <div style={{ textAlign: "right" }}>
        {actionComponent} {/* 渲染父组件传递的按钮 */}
      </div>
    </div>
  );
};
```
**返回结果**：
```html
<div style="border: 1px solid #000; padding: 10px;">
  <h3>带按钮的卡片</h3>
  <div style="textAlign: right;">
    <button>自定义按钮</button>
  </div>
</div>
```


### 五、Props 与 State 的核心区别
Props 和 State 是 React 中管理数据的两大核心，容易混淆，以下是关键对比：

```
| 对比维度         | Props                                  | State                                  |
| ---------------- | -------------------------------------- | -------------------------------------- |
| 核心作用         | 组件间传递数据（父传子）               | 管理组件内部的可变状态                 |
| 可变性           | 只读，子组件不可修改                   | 可修改，通过 setState（类组件）或 useState（函数组件）更新 |
| 数据流向         | 单向（父 → 子）                        | 组件内部私有，不对外暴露               |
| 触发更新         | 父组件传递的 props 变化时，子组件重新渲染 | 自身状态更新时，组件重新渲染           |
| 使用场景         | 子组件需依赖父组件数据时               | 组件需自身维护动态数据时（如输入框值、弹窗显隐） |
```


要不要我帮你整理一份 **Props 实用代码手册**，包含所有场景的可复制代码、类型校验模板和避坑指南，方便你在项目中直接参考使用？