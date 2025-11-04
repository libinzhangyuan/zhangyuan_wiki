[return](/react-native/index)


在 React Native 中，`createContext`、`useContext` 和 `Provider` 是一套用于**跨组件传递数据**的核心方案，无需通过“props 逐层传递”（即“prop drilling”），能高效实现全局或局部状态共享。


### 1. createContext：创建上下文容器
`createContext` 是 React 提供的函数，用于创建一个“上下文容器”。这个容器本身不存储数据，仅定义数据的结构（或设置默认值），为后续传递和使用数据搭建桥梁。

#### 核心作用
- 定义上下文的“数据模板”，指定可传递的数据类型（如对象、字符串、函数等）。
- 生成一对关联的组件：`Context.Provider`（用于提供数据）和 `Context.Consumer`（早期获取数据的方式，现多被 `useContext` 替代）。

#### 代码示例
```
import React, { createContext } from 'react';
import { View, Text } from 'react-native';

// 1. 创建上下文，设置默认值（仅在没有匹配的 Provider 时生效）
const ThemeContext = createContext({
  color: '#333', // 文本颜色默认值
  backgroundColor: '#fff' // 背景色默认值
});

// 2. 直接使用默认值（无 Provider 时）
const DemoComponent = () => {
  // 此处暂未用 useContext，先看上下文创建效果
  return (
    <View style={{ padding: 20 }}>
      <Text>上下文已创建，默认主题为浅色</Text>
    </View>
  );
};

export default DemoComponent;
```

#### 返回结果
组件渲染后，屏幕显示文本“上下文已创建，默认主题为浅色”，此时上下文的默认值已定义，但未被主动传递和修改。


### 2. Provider：提供上下文数据
`Provider` 是 `createContext` 生成的上下文对象的属性（如 `ThemeContext.Provider`），它是一个 React 组件，用于**向下传递上下文的具体数据**。

#### 核心作用
- 作为“数据源头”，通过 `value` 属性将数据注入到其包裹的整个组件树中。
- 只要组件在 `Provider` 包裹范围内，无论层级多深，都能通过 `useContext` 获取 `value` 中的数据。

#### 代码示例
```
import React, { createContext } from 'react';
import { View, Text, Button } from 'react-native';

// 1. 先创建上下文
const ThemeContext = createContext();

// 2. 子组件（层级较深，需获取主题数据）
const ChildComponent = () => {
  // 后续用 useContext 获取数据，此处先定义组件结构
  return (
    <View style={{ marginVertical: 10, padding: 15, borderRadius: 8 }}>
      <Text>我是子组件，主题由 Provider 控制</Text>
    </View>
  );
};

// 3. 父组件（用 Provider 包裹子组件，传递数据）
const ParentComponent = () => {
  // 定义要传递的主题数据
  const darkTheme = {
    color: '#fff',
    backgroundColor: '#1a1a1a'
  };

  return (
    // Provider 包裹子组件，通过 value 传递深色主题
    <ThemeContext.Provider value={darkTheme}>
      <View style={{ flex: 1, padding: 20 }}>
        <Text style={{ color: darkTheme.color }}>父组件：当前为深色主题</Text>
        <ChildComponent /> {/* 子组件可获取 Provider 传递的主题 */}
      </View>
    </ThemeContext.Provider>
  );
};

export default ParentComponent;
```

#### 返回结果
组件渲染后，父组件文本为白色（`color: #fff`），背景为深灰色（`#1a1a1a`）；子组件已被 `Provider` 包裹，后续可通过 `useContext` 直接获取深色主题数据，无需手动传递 props。


### 3. useContext：读取上下文数据
`useContext` 是 React 的 Hook 函数，用于在**子组件中快速读取 Provider 传递的上下文数据**，替代了早期繁琐的 `Context.Consumer` 嵌套写法。

#### 核心作用
- 接收 `createContext` 生成的上下文对象（如 `ThemeContext`），直接返回该上下文对应的 `Provider` 传递的 `value` 数据。
- 当 `Provider` 的 `value` 发生变化时，使用 `useContext` 的组件会自动重新渲染，获取最新数据。

#### 代码示例（结合 Provider 完整流程）
```jsx
import React, { createContext, useContext, useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

// 1. 1. 创建上下文（可设置默认值，此处省略）
const ThemeContext = createContext();

// 2. 子组件：用 useContext 获取主题数据
const ChildComponent = () => {
  // 关键：通过 useContext 读取上下文数据
  const theme = useContext(ThemeContext);

  return (
    <View style={[styles.childContainer, { backgroundColor: theme.backgroundColor }]}>
      <Text style={{ color: theme.color }}>子组件文本（来自上下文）</Text>
    </View>
  );
};

// 3. 父组件：用 Provider 提供数据，并用状态控制主题切换
const App = () => {
  // 用 useState 管理主题状态（浅色/深色）
  const [isDark, setIsDark] = useState(false);

  // 根据 isDark 生成主题数据
  const theme = isDark
    ? { color: '#fff', backgroundColor: '#1a1a1a' } // 深色
    : { color: '#333', backgroundColor: '#fff' }; // 浅色

  return (
    <ThemeContext.Provider value={theme}>
      <View style={[styles.container, { backgroundColor: theme.backgroundColor }]}>
        <Text style={{ color: theme.color, fontSize: 18, marginBottom: 20 }}>
          当前主题：{isDark ? '深色' : '浅色'}
        </Text>
        {/* 按钮切换主题（修改状态，间接更新 Provider 的 value） */}
        <Button
          title={isDark ? '切换为浅色主题' : '切换为深色主题'}
          onPress={() => setIsDark(!isDark)}
          color={isDark ? '#fff' : '#1a1a1a'}
        />
        <ChildComponent /> {/* 子组件自动同步主题变化 */}
      </View>
    </ThemeContext.Provider>
  );
};

// 样式定义
const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center'
  },
  childContainer: {
    padding: 15,
    borderRadius: 8,
    marginTop: 20
  }
});

export default App;
```

#### 返回结果
- 初始渲染：显示“当前主题：浅色”，父组件和子组件背景为白色，文本为深灰色。
- 点击按钮：状态 `isDark` 切换，`Provider` 的 `value` 同步更新为深色主题，父组件和子组件自动重新渲染，背景变为深灰色，文本变为白色。


### 三者核心对比表
```
| 概念          | 定义                                                                 | 核心作用                                  | 使用场景                                  |
|---------------|----------------------------------------------------------------------|-------------------------------------------|-------------------------------------------|
| createContext | React 函数，用于创建“上下文容器”                                     | 定义上下文结构，生成 Provider/Consumer    | 初始化上下文，指定数据默认格式（如主题、用户信息） |
| Provider      | createContext 生成的组件，属于上下文对象的属性                        | 通过 value 向下传递数据到组件树            | 作为数据源头，包裹需要共享数据的组件范围        |
| useContext    | React Hook 函数，用于读取上下文数据                                   | 快速获取 Provider 传递的 value，自动响应更新 | 子组件中读取共享数据（如主题、权限、全局状态）  |
```


### 关键注意事项
1. **Provider 必须包裹使用范围**：只有在 `Provider` 包裹的组件树内，`useContext` 才能获取到数据；若超出范围，会使用 `createContext` 设置的默认值（无默认值则为 `undefined`）。
2. **避免不必要的重渲染**：若 `Provider` 的 `value` 是动态对象（如 `{ color: '#fff' }`），每次父组件渲染都会生成新对象，导致所有使用 `useContext` 的子组件重新渲染。建议用 `useMemo` 缓存 `value`，优化性能。
3. **不建议多层嵌套 Provider**：除非有明确的局部数据隔离需求（如“主题 Provider”+“用户 Provider”），否则过多嵌套会增加代码复杂度，可考虑合并上下文数据。


要不要我帮你整理一份**完整的 React Native 上下文使用示例代码文件**？包含主题切换、用户信息共享两个实际场景，可直接复制到项目中运行测试。