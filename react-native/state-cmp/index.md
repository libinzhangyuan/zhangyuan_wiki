[return](../index)


### 1. `useContext` + `useState`/`useReducer`  
**核心作用**：跨层级共享状态，解决“props 透传”问题（状态无需经过多层无关组件传递）。  
通过 `createContext` 创建上下文容器，父组件用 `Provider` 注入状态，任意层级子组件可通过 `useContext` 直接获取。


#### 适用场景  
- 状态需在多层级组件中共享（如全局主题、用户登录状态）。  
- 状态逻辑简单，或结合 `useReducer` 处理中等复杂度逻辑。


#### 优点  
- 无需手动逐层传递 props，简化深层组件通信。  
- 比全局状态库更轻量，无额外依赖，适合中小型应用。


#### 缺点  
- 上下文更新时，所有消费该上下文的组件都会重渲染，可能导致性能浪费。  
- 不支持复杂状态逻辑（如多模块联动、异步请求）。


#### 示例代码  
```
import { createContext, useContext, useState } from 'react';
import { View, Text, Button } from 'react-native';

// 1. 创建上下文
const ThemeContext = createContext();

// 2. 父组件提供状态
const App = () => {
  const [theme, setTheme] = useState('light'); // 浅色主题默认值

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <ParentComponent />
    </ThemeContext.Provider>
  );
};

// 3. 中间组件（无需传递 props）
const ParentComponent = () => <ChildComponent />;

// 4. 深层子组件消费状态
const ChildComponent = () => {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <View style={{ padding: 20 }}>
      <Text>当前主题：{theme}</Text>
      <Button 
        title="切换深色主题" 
        onPress={() => setTheme(prev => prev === 'light' ? 'dark' : 'light')} 
        style={{ marginTop: 10 }}
      />
    </View>
  );
};

export default App;
```

**返回结果**：  
- 子组件 `ChildComponent` 无需通过 `ParentComponent` 传递 props，直接获取 `theme` 状态和 `setTheme` 方法。  
- 点击“切换深色主题”按钮后，全局主题状态同步更新，所有使用 `ThemeContext` 的组件会自动重新渲染，显示最新主题。


### 2. `useReducer`  
**核心作用**：增强 `useState`，集中管理组件内复杂状态逻辑（如多状态关联、多条件更新）。  
通过定义 `reducer` 函数统一处理状态变更，用 `dispatch` 触发不同 `action` 来修改状态。


#### 适用场景  
- 组件内状态逻辑复杂（如购物车：添加商品、删除商品、清空购物车）。  
- 状态更新依赖前一个状态，且更新逻辑分散（如表单多字段校验）。


#### 优点  
- 状态更新逻辑集中在 `reducer` 中，便于维护和单元测试。  
- 通过 `action` 类型区分更新意图，代码逻辑更清晰（如 `ADD_ITEM`、`CLEAR_CART`）。


#### 缺点  
- 比 `useState` 语法复杂，简单状态使用会增加冗余代码。  
- 跨层级传递时，仍需结合 `useContext` 或 props，无法单独实现全局共享。


#### 示例代码  
```
import { useReducer, View, Text, Button, StyleSheet } from 'react-native';

// 1. 定义 reducer 函数（集中处理状态更新）
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      return [...state, action.payload]; // 添加商品
    case 'CLEAR_CART':
      return []; // 清空购物车
    default:
      return state;
  }
};

const ShoppingCart = () => {
  // 2. 初始化状态和 dispatch 方法（初始状态为空数组）
  const [cart, dispatch] = useReducer(cartReducer, []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>购物车</Text>
      <Text style={styles.goodsText}>
        商品列表：{cart.length > 0 ? cart.join('、') : '暂无商品'}
      </Text>
      
      <Button 
        title="添加苹果" 
        onPress={() => dispatch({ type: 'ADD_ITEM', payload: '苹果' })} 
        style={styles.button}
      />
      
      <Button 
        title="清空购物车" 
        onPress={() => dispatch({ type: 'CLEAR_CART' })} 
        style={styles.button}
      />
    </View>
  );
};

// 样式定义
const styles = StyleSheet.create({
  container: { padding: 20 },
  title: { fontSize: 18, fontWeight: 'bold', marginBottom: 10 },
  goodsText: { marginBottom: 15 },
  button: { marginBottom: 10 }
});

export default ShoppingCart;
```

**返回结果**：  
- 点击“添加苹果”按钮，`dispatch` 触发 `ADD_ITEM` 类型的 action，`reducer` 向购物车数组中添加“苹果”，页面同步显示“商品列表：苹果”。  
- 点击“清空购物车”按钮，`dispatch` 触发 `CLEAR_CART` 类型的 action，购物车数组重置为空，页面显示“商品列表：暂无商品”。


### 3. 全局状态管理库(如 Redux、Zustand、Jotai）)

以 Zustand 为例 <br><br>
**核心作用**：管理应用级全局状态（如用户信息、多页面共享的配置），支持复杂逻辑（异步请求、多模块联动）。  
相比传统 Redux 更轻量，API 简洁，无需繁琐的 `reducer` 和 `action` 定义。


#### 适用场景  
- 大型应用，状态需在多个页面/模块间共享（如电商的购物车、全局用户信息）。  
- 状态逻辑包含异步操作（如登录请求、数据加载）。


#### 优点  
- 状态全局可访问，无需通过 props 或 `useContext` 传递。  
- 支持异步逻辑，API 简洁，学习成本低。  
- 可精准控制组件重渲染（只更新使用目标状态的组件）。


#### 缺点  
- 小型应用使用会增加不必要的复杂度。  
- 多模块状态管理时，需手动划分状态边界，避免命名冲突。


#### 示例代码  
```
// 1. 安装依赖：npm install zustand
import { create } from 'zustand';
import { View, Text, Button, StyleSheet } from 'react-native';

// 2. 创建全局状态存储
const useUserStore = create((set) => ({
  username: '访客', // 初始状态：访客
  isLogin: false,
  // 登录方法（异步示例）
  login: async (name) => {
    // 模拟接口请求（2秒延迟）
    await new Promise(resolve => setTimeout(resolve, 2000));
    set({ username: name, isLogin: true });
  },
  // 退出登录方法
  logout: () => set({ username: '访客', isLogin: false })
}));

// 3. 组件1：显示用户信息（头部组件）
const Header = () => {
  const { username, isLogin } = useUserStore();
  
  return (
    <View style={styles.header}>
      <Text style={styles.userText}>
        {isLogin ? `欢迎，${username}` : '请登录'}
      </Text>
    </View>
  );
};

// 4. 组件2：处理登录/退出（登录页面组件）
const LoginPage = () => {
  const { login, logout, isLogin } = useUserStore();
  
  return (
    <View style={styles.loginContainer}>
      {!isLogin ? (
        <Button title="登录（模拟2秒请求）" onPress={() => login('张三')} />
      ) : (
        <Button title="退出登录" onPress={logout} />
      )}
    </View>
  );
};

// 5. 根组件
const App = () => {
  return (
    <View style={styles.container}>
      <Header />
      <LoginPage />
    </View>
  );
};

// 样式定义
const styles = StyleSheet.create({
  container: { flex: 1 },
  header: { padding: 20, backgroundColor: '#f5f5f5' },
  userText: { fontSize: 16 },
  loginContainer: { padding: 20 }
});

export default App;
```

**返回结果**：  
- 初始状态下，`Header` 显示“请登录”，`LoginPage` 显示“登录（模拟2秒请求）”按钮。  
- 点击登录按钮后，等待2秒（模拟接口请求），`username` 更新为“张三”，`isLogin` 变为 `true`，`Header` 显示“欢迎，张三”，`LoginPage` 显示“退出登录”按钮。  
- 点击退出登录按钮，状态重置为初始值，页面同步更新。


### 四种状态管理方案对比表  
```
方案                  适用场景                          优点                                  缺点
-------------------  --------------------------------  ------------------------------------  ------------------------------------
useState + Props     父子/近邻组件、简单状态（如计数器）  语法简单，符合单向数据流，无学习成本      深层组件传递繁琐（props透传），复杂状态难维护
useContext + useState 跨层级共享、简单状态（如主题）      避免透传，轻量无依赖                    上下文更新时，所有消费组件均重渲染，无异步支持
useReducer           组件内复杂状态（如购物车）          状态逻辑集中，便于测试和维护            语法复杂，跨层级需结合useContext，无全局共享能力
Zustand（全局库）    大型应用、全局状态（如用户信息）    全局可访问，支持异步，精准控制重渲染      小型应用冗余，多模块需手动划分状态边界
```


要不要我帮你整理一份 **React Native 状态管理方案选型指南**？里面会包含不同场景下的具体决策步骤和代码模板，方便你快速套用。