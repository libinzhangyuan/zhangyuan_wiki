[return](/react-native/index)
```
在 React/React Native 中，“兜底渲染（fallback rendering）”是指
当主内容无法正常显示（如数据为空、加载中、发生错误等）时，显示备用内容的机制。
除了 `||` 运算符和 `<Suspense fallback={...}>`，还有多种常用方案，适用于不同场景。
以下是详细介绍：
```


### || 可能的陷阱与注意事项
`||` 的陷阱同样与“假值”相关，需注意：**左侧若为“真值但非UI元素”，会直接渲染该值**，导致意外结果。


#### 陷阱示例：左侧是真值但非UI元素
```javascript
import { Text, View } from 'react-native';

const ScoreDisplay = ({ score }) => {
  return (
    <View>
      {/* 预期：score 存在则显示分数，否则显示 "未评分" */}
      {/* 问题：当 score = 0 时（0 是假值），会显示 "未评分"（可能符合预期）；但当 score = 5 时（5 是真值），会直接渲染 5 */}
      <Text>分数：{score || '未评分'}</Text>
    </View>
  );
};

// 使用场景
<ScoreDisplay score={5} /> 
// 渲染结果：<Text>分数：5</Text>（看似正常，但如果 score 是其他类型可能有问题）

<ScoreDisplay score={0} /> 
// 渲染结果：<Text>分数：未评分</Text>（若业务中 0 是有效分数，这会不符合预期）
```

**问题分析**：  
- 当 `score = 0` 时，`0` 是假值，`||` 返回右侧的 `'未评分'`，但如果业务中“0分是有效分数”，这会错误隐藏真实分数；  
- 当 `score` 是其他真值（如 `5`、`'A'`），会直接渲染该值，若该值不是预期的UI文本，可能导致显示异常。  


#### 如何避免陷阱？
- **明确判断“有效内容”**：对可能为 `0`、`''` 等“有效假值”的场景，用显式条件判断（如 `score !== undefined`）替代 `||`；  
  ```javascript
  // 修复上述示例（允许 0 分正常显示）
  <Text>分数：{score !== undefined ? score : '未评分'}</Text>
  ```
- **确保左侧是“预期渲染的UI内容”**：若左侧是变量，需保证其为“需要显示的内容”或“假值”，避免非UI真值（如数字、布尔值）直接参与渲染。  




### 1. 三元运算符（Ternary Operator）
**核心逻辑**：通过显式条件判断，决定渲染主内容还是兜底内容，语法为 `条件 ? 主内容 : 兜底内容`。  
**适用场景**：需要明确判断“主内容是否有效”的场景（如数据是否加载完成、是否为空等），尤其适合处理“有效假值”（如 `0`、`''` 等业务上有意义的假值）。

**示例**：列表数据为空时显示兜底提示  
```javascript
import { Text, View, FlatList } from 'react-native';

const DataList = ({ data }) => {
  // 显式判断数据是否为空（即使 data 是 []，也能正确兜底）
  return (
    <View>
      {data.length > 0 ? (
        <FlatList 
          data={data} 
          renderItem={({ item }) => <Text>{item.name}</Text>} 
        />
      ) : (
        <Text>暂无数据（兜底内容）</Text> // 兜底渲染
      )}
    </View>
  );
};
```

**优势**：逻辑清晰，可避免 `||` 对“有效假值”的误判（如 `0` 作为有效数据时，`0 || '兜底'` 会错误显示兜底，而三元 `0 !== undefined ? 0 : '兜底'` 可正确处理）。


### 2. 条件变量/函数（Conditional Variables/Functions）
**核心逻辑**：在 JSX 外部通过 `if/else` 或函数定义主内容和兜底内容，再在 JSX 中引用。  
**适用场景**：兜底逻辑复杂（如需要多条件判断、处理多种异常状态）时，避免 JSX 内代码臃肿。

**示例**：处理加载、空数据、错误三种状态的兜底  
```javascript
import { Text, View, ActivityIndicator } from 'react-native';

const DataDisplay = ({ isLoading, data, error }) => {
  // 在 JSX 外部定义渲染内容
  let content;
  if (isLoading) {
    content = <ActivityIndicator size="large" />; // 加载中兜底
  } else if (error) {
    content = <Text>加载失败：{error.message}</Text>; // 错误兜底
  } else if (data.length === 0) {
    content = <Text>暂无数据</Text>; // 空数据兜底
  } else {
    content = <Text>数据：{data.join(',')}</Text>; // 主内容
  }

  return <View>{content}</View>;
};
```

**优势**：逻辑拆分清晰，适合多状态兜底（加载中、错误、空数据等），可读性高。


### 3. 错误边界（Error Boundary）
**核心逻辑**：通过 React 组件捕获子组件抛出的错误，并渲染兜底 UI（而非崩溃整个应用）。  
**适用场景**：处理运行时错误（如组件渲染报错、数据解析错误等）的兜底，是 React 官方推荐的错误处理方案。

**示例**：捕获子组件错误并显示兜底  
```javascript
import React from 'react';
import { Text, View } from 'react-native';

// 定义错误边界组件
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true }; // 发生错误时更新状态
  }

  render() {
    if (this.state.hasError) {
      // 错误时的兜底渲染
      return <Text>抱歉，内容加载出错了（兜底）</Text>;
    }
    return this.props.children; // 正常时渲染子组件（主内容）
  }
}

// 使用错误边界
const RiskyComponent = ({ data }) => {
  // 模拟可能报错的逻辑（如访问 undefined 的属性）
  return <Text>{data.name}</Text>;
};

const App = () => {
  return (
    <View>
      <ErrorBoundary>
        <RiskyComponent data={null} /> {/* data 为 null 时会报错 */}
      </ErrorBoundary>
    </View>
  );
};
// 渲染结果：<Text>抱歉，内容加载出错了（兜底）</Text>
```

**注意**：错误边界只能捕获子组件的渲染错误、生命周期错误等，无法捕获异步代码（如 `setTimeout`）或事件处理函数中的错误。


### 4. 高阶组件（HOC，Higher-Order Component）
**核心逻辑**：封装兜底渲染逻辑为高阶组件，通过复用该组件为多个主组件添加兜底能力。  
**适用场景**：多个组件需要相同的兜底逻辑（如统一的加载中、空数据提示）时，提高代码复用性。

**示例**：创建带加载和空数据兜底的 HOC  
```javascript
import { Text, View, ActivityIndicator } from 'react-native';

// 定义高阶组件：为目标组件添加兜底逻辑
const withFallback = (WrappedComponent) => {
  return ({ isLoading, data, ...props }) => {
    if (isLoading) {
      return <ActivityIndicator size="small" />; // 加载中兜底
    }
    if (!data || data.length === 0) {
      return <Text>暂无数据（统一兜底）</Text>; // 空数据兜底
    }
    return <WrappedComponent data={data} {...props} />; // 主内容
  };
};

// 目标组件（主内容）
const UserList = ({ data }) => (
  <View>
    {data.map(user => <Text key={user.id}>{user.name}</Text>)}
  </View>
);

// 用 HOC 包装目标组件，获得兜底能力
const UserListWithFallback = withFallback(UserList);

// 使用
const App = () => {
  return (
    <View>
      <UserListWithFallback isLoading={false} data={[]} /> 
      {/* 渲染结果：<Text>暂无数据（统一兜底）</Text> */}
    </View>
  );
};
```


### 5. Render Props 模式
**核心逻辑**：通过组件的 props 传递一个函数，该函数返回主内容或兜底内容，实现逻辑复用。  
**适用场景**：需要在组件间共享兜底逻辑，但高阶组件可能导致“包装地狱”时，可作为替代方案。

**示例**：用 Render Props 实现加载状态兜底  
```javascript
import { Text, View, ActivityIndicator } from 'react-native';

// 定义 Render Props 组件
const LoadingFallback = ({ isLoading, renderContent }) => {
  if (isLoading) {
    return <ActivityIndicator size="large" />; // 加载中兜底
  }
  return renderContent(); // 渲染主内容
};

// 使用
const DataComponent = ({ data }) => {
  return (
    <LoadingFallback
      isLoading={data === null} // 数据为 null 时视为加载中
      renderContent={() => <Text>数据：{data}</Text>} // 主内容
    />
  );
};

// 场景1：data 为 null（加载中）
<DataComponent data={null} /> 
// 渲染结果：<ActivityIndicator ... />（兜底）

// 场景2：data 加载完成
<DataComponent data="Hello" /> 
// 渲染结果：<Text>数据：Hello</Text>（主内容）
```


### 6. 总结：各种兜底渲染方案对比
```
方案               | 核心语法/逻辑                          | 适用场景                                  | 优势                                  | 局限性
-------------------|----------------------------------------|-------------------------------------------|---------------------------------------|---------------------------------------
|| 运算符          | { 主内容 || 兜底内容 }                 | 简单场景，主内容为假值时兜底              | 极简，一行代码搞定                    | 对有效假值（0、''）不友好，易误判
<Suspense>         | <Suspense fallback={...}>{主内容}</Suspense> | 配合 React 并发模式，处理代码分割/数据预加载 | 官方推荐，支持异步加载的优雅兜底      | 仅用于 Suspense 支持的异步场景（如 lazy 组件）
三元运算符        | 条件 ? 主内容 : 兜底内容               | 需显式判断的场景，支持有效假值            | 逻辑清晰，无假值误判风险              | 多分支时嵌套可读性差
条件变量/函数      | 外部用 if/else 定义 content，JSX 中引用 | 多状态兜底（加载、空数据、错误等）        | 逻辑拆分清晰，适合复杂场景            | 代码量稍多
错误边界          | 类组件捕获子组件错误，渲染兜底          | 处理运行时错误（如渲染报错）              | 防止应用崩溃，官方推荐错误处理方案    | 无法捕获异步/事件处理中的错误
高阶组件（HOC）    | withFallback(WrappedComponent)         | 多个组件共享相同兜底逻辑                  | 复用性高，一次封装多处使用            | 可能导致组件层级过深（包装地狱）
Render Props       | <Fallback renderContent={() => 主内容} /> | 共享兜底逻辑，替代 HOC 避免包装地狱        | 灵活，避免层级问题                    | 语法稍繁琐，需传递函数
```


### 选择建议
- 简单的“值为空时兜底”：优先用 `||`（注意有效假值风险）或三元运算符；  
- 异步加载（如懒加载组件）：必须用 `<Suspense fallback={...}>`；  
- 多状态（加载中、空数据、错误）：用条件变量/函数或 HOC；  
- 运行时错误处理：强制用错误边界；  
- 复用兜底逻辑：根据组件层级选择 HOC 或 Render Props。  

根据具体场景选择合适的方案，可在保证功能的同时兼顾代码可读性和复用性。