[return](../index)


我们来详细介绍一下 React Native 中非常重要的一个概念：`React.FC`。

`React.FC` 是 `React.FunctionComponent` 的缩写，它是 React 中定义**函数式组件**的一种类型别名（尤其在 TypeScript 中非常常用）。它本质上是一个函数，接收 `props` 对象作为参数，并返回 React 元素（JSX）。

### 1. 核心定义与作用
#### 定义：
```
type React.FC<P = {}> = (props: P & { children?: React.ReactNode }) => React.ReactElement | null;
```
- `<P = {}>`：泛型参数，指定组件的 `props` 类型（默认是空对象）。
- `props: P & { children?: React.ReactNode }`：组件接收的参数，包含：
  - 自定义 `props`（类型由 `P` 定义）；
  - 内置的 `children`（可选，类型为 `React.ReactNode`，支持文本、元素、数组等）。
- 返回值：`React.ReactElement | null`，即组件渲染的 JSX 结构（或 `null` 表示不渲染）。

#### 核心作用：
- 定义可复用的 UI 组件（函数形式）；
- 接收外部传入的 `props` 实现组件个性化；
- 内部可通过 React Hooks（如 `useState`、`useEffect`）管理状态和副作用。


### 2. 基础使用示例（TSX）
#### 示例 1：无自定义 props 的简单组件
```
// 定义一个无 props 的函数式组件
const HelloWorld: React.FC = () => {
  return <Text>Hello, React Native!</Text>;
};

// 使用组件（无需传参）
const App: React.FC = () => {
  return (
    <View style={{ padding: 20 }}>
      <HelloWorld />
    </View>
  );
};
```
**返回结果**：
```markdown
渲染出一个包含文本 "Hello, React Native!" 的视图，文本位于屏幕上方，有 20px 内边距。
```

#### 示例 2：带自定义 props 的组件
```
// 定义 props 类型接口
interface GreetingProps {
  name: string;       // 必传参数（字符串类型）
  age?: number;       // 可选参数（数字类型）
  onGreet: () => void; // 回调函数（无参数，无返回值）
}

// 定义带 props 的函数式组件
const Greeting: React.FC<GreetingProps> = (props) => {
  const { name, age, onGreet, children } = props; // 解构 props

  return (
    <View style={{ margin: 10 }}>
      <Text>Hello, {name}!</Text>
      {age && <Text>You are {age} years old.</Text>} {/* 条件渲染 */}
      <Button title="Greet Me" onPress={onGreet} />
      {children} {/* 渲染子元素 */}
    </View>
  );
};

// 使用组件（传入 props）
const App: React.FC = () => {
  const handleGreet = () => {
    alert("Greeted successfully!");
  };

  return (
    <View style={{ padding: 20 }}>
      <Greeting 
        name="Alice" 
        age={25} 
        onGreet={handleGreet}
      >
        <Text style={{ color: 'gray' }}>This is a child element.</Text>
      </Greeting>
    </View>
  );
};
```
**返回结果**：
```markdown
1. 一个包含以下内容的视图（外边距 10px）：
   - 文本 "Hello, Alice!"；
   - 文本 "You are 25 years old."（因 age 存在而显示）；
   - 一个标题为 "Greet Me" 的按钮，点击后弹出 "Greeted successfully!"；
   - 灰色文本 "This is a child element."（作为 children 传入）。
2. 整个组件位于屏幕上方，有 20px 内边距。
```


### 3. React.FC 与普通函数组件的对比
在 TypeScript 中，有两种定义函数式组件的方式：`React.FC` 和普通函数。两者核心功能一致，但 `React.FC` 有一些额外优势。
```
| 特性                | React.FC                          | 普通函数组件                          |
|---------------------|-----------------------------------|---------------------------------------|
| **类型定义**        | 显式指定 props 类型（泛型参数）   | 需要手动定义 props 类型作为参数       |
| **children 支持**   | 自动包含 `children?: React.ReactNode` | 需手动在 props 中添加 children 类型   |
| **默认 props**      | 支持 `Component.defaultProps`（但不推荐，建议用默认参数） | 直接使用函数参数默认值（更简洁）      |
| **返回值类型**      | 自动推断为 `React.ReactElement | null` | 需手动指定返回值类型（或依赖 TS 推断）|
| **代码简洁性**      | 更简洁（减少重复类型定义）        | 稍繁琐（需手动处理 children 等）      |
```
#### 对比示例：
```
// 1. React.FC 方式
interface MyComponentProps {
  title: string;
}

const MyComponent: React.FC<MyComponentProps> = ({ title, children }) => {
  return <View><Text>{title}</Text>{children}</View>;
};

// 2. 普通函数组件方式
interface MyComponentProps {
  title: string;
  children?: React.ReactNode; // 需手动添加 children 类型
}

const MyComponent = ({ title, children }: MyComponentProps): React.ReactElement => {
  return <View><Text>{title}</Text>{children}</View>;
};
```


### 4. 关键注意事项
1. **泛型约束**：`React.FC<P>` 中的 `P` 必须是对象类型（默认是空对象 `{}`），不能是基本类型（如 `string`、`number`）。
   ```
   // 错误：P 不能是 string
   const MyComponent: React.FC<string> = (str) => <Text>{str}</Text>;

   // 正确：P 是对象类型
   interface MyComponentProps {
     value: string;
   }
   const MyComponent: React.FC<MyComponentProps> = ({ value }) => <Text>{value}</Text>;
   ```

2. **返回值限制**：组件必须返回 `React.ReactElement`（JSX）或 `null`，不能返回 `undefined` 或其他类型。
   ```
   // 错误：返回 undefined
   const MyComponent: React.FC = () => {
     // 无 return 语句，默认返回 undefined
   };

   // 正确：返回 null（不渲染）
   const MyComponent: React.FC = () => {
     return null;
   };
   ```

3. **Hooks 支持**：`React.FC` 内部可以正常使用所有 React Hooks（`useState`、`useEffect`、`useRef` 等），这是函数式组件的核心优势。
   ```
   const Counter: React.FC = () => {
     const [count, setCount] = useState(0); // 使用 useState 管理状态

     return (
       <View>
         <Text>Count: {count}</Text>
         <Button title="Increment" onPress={() => setCount(count + 1)} />
       </View>
     );
   };
   ```

4. **默认 Props**：虽然 `React.FC` 支持 `defaultProps`，但在 TypeScript 中更推荐使用**函数参数默认值**（类型推断更友好）。
   ```
   interface MyComponentProps {
     name: string;
     age?: number;
   }

   // 推荐：使用参数默认值
   const MyComponent: React.FC<MyComponentProps> = ({ 
     name, 
     age = 18 // 默认值 18
   }) => {
     return <Text>{name} ({age})</Text>;
   };

   // 使用时无需传 age
   <MyComponent name="Bob" /> // 渲染 "Bob (18)"
   ```


### 总结
`React.FC` 是 React Native（尤其 TypeScript 项目）中定义函数式组件的标准方式，核心优势：
- 类型安全：通过泛型严格约束 `props` 类型；
- 简洁高效：自动包含 `children`，减少重复代码；
-  Hooks 兼容：完美支持 React Hooks 生态。

在实际开发中，绝大多数 UI 组件都会用 `React.FC` 来定义，是 React Native 开发的基础核心概念。