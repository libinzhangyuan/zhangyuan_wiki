[return](../index)

# React Native Hooks 详解
React Hooks 是 React 16.8+ 引入的核心特性，允许**函数组件**使用原本只有 class 组件才能实现的功能（如状态管理、生命周期、 refs 引用等），无需编写复杂的 class 语法，让代码更简洁、逻辑更易复用。

## 一、核心 Hooks 详解
### 1. useState：局部状态管理
#### 作用
管理组件的局部状态（如数字、字符串、对象、数组等），返回当前状态和状态更新函数。

#### 语法
```javascript
const [state, setState] = useState(initialValue);
```
```
- `initialValue`：状态初始值（可直接传值，或函数返回值，函数仅执行一次）
- `state`：当前状态值
- `setState`：状态更新函数（异步执行）
```
#### 示例：计数器组件
```javascript
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

const Counter = () => {
  // 初始化计数为 0，返回状态 count 和更新函数 setCount
  const [count, setCount] = useState(0);
  // 初始化用户信息（对象类型状态）
  const [user, setUser] = useState({ name: '张三', age: 20 });

  // 依赖前一个状态的更新（推荐用函数形式）
  const increment = () => setCount(prev => prev + 1);
  // 更新对象类型状态（需合并原有属性）
  const updateUser = () => setUser(prev => ({ ...prev, age: prev.age + 1 }));

  return (
    <View style={styles.container}>
      <Text style={styles.text}>当前计数：{count}</Text>
      <Text style={styles.text}>用户名：{user.name}，年龄：{user.age}</Text>
      <Button title="计数+1" onPress={increment} />
      <Button title="年龄+1" onPress={updateUser} style={styles.button} />
      <Button title="重置" onPress={() => {
        setCount(0);
        setUser({ name: '张三', age: 20 });
      }} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { alignItems: 'center', marginTop: 50 },
  text: { fontSize: 18, marginBottom: 10 },
  button: { marginTop: 10 }
});

export default Counter;
```

#### 返回结果说明
```
| 返回项    | 类型       | 描述                                                                 |
| --------- | ---------- | -------------------------------------------------------------------- |
| state     | 任意类型   | 当前状态值，初始为 `initialValue`                                    |
| setState  | 函数       | 1. 接收新值（如 `0`、`count+1`）或函数（`prev => newVal`）；<br>2. 异步执行，更新后组件重新渲染；<br>3. 对象类型需手动合并原有属性 |
```
### 2. useEffect：副作用处理
#### 作用
处理组件的**副作用**（如网络请求、订阅、定时器、DOM 操作等），替代 class 组件的 `componentDidMount`、`componentDidUpdate`、`componentWillUnmount` 生命周期。

#### 语法
```javascript
useEffect(() => {
  // 副作用逻辑（如请求数据、启动定时器）
  return () => {
    // 清理逻辑（如取消订阅、清除定时器）
  };
}, [dependencies]); // 依赖数组（控制 effect 执行时机）
```

#### 示例 1：组件挂载时请求数据（类似 componentDidMount）
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator, StyleSheet } from 'react-native';

const DataFetcher = () => {
  const [todo, setTodo] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 副作用：发起网络请求
    const fetchTodo = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/todos/1');
        if (!response.ok) throw new Error('请求失败');
        const data = await response.json();
        setTodo(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchTodo();

    // 无清理逻辑（无需订阅/定时器）
  }, []); // 空依赖数组 → 仅组件挂载时执行一次

  if (loading) return <ActivityIndicator size="large" style={styles.loader} />;
  if (error) return <Text style={styles.error}>错误：{error}</Text>;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>任务详情</Text>
      <Text>ID：{todo.id}</Text>
      <Text>标题：{todo.title}</Text>
      <Text>状态：{todo.completed ? '已完成' : '未完成'}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { alignItems: 'center', marginTop: 50 },
  loader: { marginTop: 50 },
  error: { color: 'red', fontSize: 16, marginTop: 50 },
  title: { fontSize: 20, fontWeight: 'bold', marginBottom: 10 }
});

export default DataFetcher;
```

#### 示例 2：带清理逻辑的订阅（类似 componentWillUnmount）
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';

const SubscriptionDemo = () => {
  const [time, setTime] = useState(new Date().toLocaleTimeString());

  useEffect(() => {
    // 副作用：启动定时器（模拟订阅）
    const timer = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);

    // 清理逻辑：组件卸载前清除定时器
    return () => clearInterval(timer);
  }, []); // 空依赖 → 仅挂载时启动一次定时器

  return (
    <View style={styles.container}>
      <Text style={styles.time}>{time}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { alignItems: 'center', marginTop: 50 },
  time: { fontSize: 24 }
});

export default SubscriptionDemo;
```

#### 依赖数组用法说明
```
| 依赖数组形式       | 执行时机                                  | 对应 class 生命周期                |
| ------------------ | ----------------------------------------- | --------------------------------- |
| 不写依赖数组       | 组件挂载 + 每次更新都执行                  | componentDidMount + componentDidUpdate |
| 空数组 []          | 仅组件挂载时执行一次                      | componentDidMount                 |
| [a, b]（指定依赖） | 组件挂载 + a 或 b 变化时执行              | componentDidMount + 依赖更新时执行 |
```

### 3. useRef：引用与持久化变量
#### 作用
```
- 获取 React Native 组件/原生 DOM 元素的引用（如 TextInput、View）；
- 存储**持久化变量**（组件重新渲染时不会重置，且修改不会触发重新渲染）。
```

#### 语法
```javascript
const ref = useRef(initialValue);
```
```
- `initialValue`：初始值（可任意类型，如 `null`、`0`、组件实例）
- `ref.current`：访问/修改引用的值（核心属性）
```

#### 示例 1：获取 TextInput 引用（操作焦点）
```javascript
import React, { useRef } from 'react';
import { View, TextInput, Button, StyleSheet } from 'react-native';

const InputDemo = () => {
  // 创建 TextInput 引用
  const inputRef = useRef(null);

  // 触发输入框聚焦
  const focusInput = () => {
    inputRef.current.focus(); // 通过 ref.current 访问组件实例方法
  };

  return (
    <View style={styles.container}>
      <TextInput
        ref={inputRef} // 绑定 ref
        style={styles.input}
        placeholder="请输入内容"
        keyboardType="default"
      />
      <Button title="聚焦输入框" onPress={focusInput} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { padding: 20 },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 10, borderRadius: 5, marginBottom: 10 }
});

export default InputDemo;
```

#### 示例 2：存储持久化变量（避免重新渲染）
```javascript
import React, { useState, useRef } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

const PersistVarDemo = () => {
  const [count, setCount] = useState(0);
  // 存储点击次数（修改不会触发组件重新渲染）
  const clickCountRef = useRef(0);

  const handleClick = () => {
    clickCountRef.current += 1; // 修改 ref 值，不触发重新渲染
    setCount(count + 1);
    console.log('累计点击次数：', clickCountRef.current); // 控制台输出实时次数
  };

  return (
    <View style={styles.container}>
      <Text style={styles.text}>计数：{count}</Text>
      <Button title="点击" onPress={handleClick} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { alignItems: 'center', marginTop: 50 },
  text: { fontSize: 18, marginBottom: 10 }
});

export default PersistVarDemo;
```

#### 返回结果说明
```
| 返回项   | 类型       | 描述                                                                 |
| -------- | ---------- | -------------------------------------------------------------------- |
| ref      | 对象       | 固定结构 `{ current: initialValue }`                                 |
| ref.current | 任意类型 | 1. 绑定组件时：存储组件实例（可调用组件方法，如 `focus()`）；<br>2. 存储变量时：持久化数据，修改不触发渲染 |
```
### 4. 其他常用 Hooks
```
| Hook          | 作用                                  | 核心语法与示例                                                                 |
| ------------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| useContext    | 跨组件状态共享（避免 props 层层传递） | 语法：`const value = useContext(Context)`<br>示例：<br>```javascript<br>// 创建上下文<br>const ThemeContext = React.createContext('light');<br>// 子组件使用<br>const Child = () => {<br>  const theme = useContext(ThemeContext);<br>  return <Text>主题：{theme}</Text>;<br>}<br>``` |
| useReducer    | 复杂状态逻辑管理（替代 useState）     | 语法：`const [state, dispatch] = useReducer(reducer, initialState)`<br>核心：通过 `dispatch(action)` 触发状态更新 |
| useCallback   | 缓存函数（避免不必要的重新创建）     | 语法：`const fn = useCallback(() => {}, [dependencies])`<br>场景：传递给子组件的回调函数（配合 React.memo 优化） |
| useMemo       | 缓存计算结果（避免重复计算）          | 语法：`const result = useMemo(() => compute(), [dependencies])`<br>场景：复杂计算、大数据处理 |
| useLayoutEffect | 同步 DOM 操作（比 useEffect 早执行）  | 语法与 useEffect 一致，执行时机：组件渲染后、DOM 更新完成前（同步执行） |
```
## 二、Class 组件 vs Hooks 对比表
```
| 特性                | Class 组件                                  | Hooks 函数组件                              |
| ------------------- | ------------------------------------------- | ------------------------------------------- |
| 状态管理            | this.state + this.setState({ key: val })     | useState(initialValue) → [state, setState]   |
| 生命周期            | componentDidMount（挂载）                    | useEffect(() => {}, [])（挂载）             |
|                     | componentDidUpdate（更新）                   | useEffect(() => {}, [deps])（依赖更新）      |
|                     | componentWillUnmount（卸载）                 | useEffect(() => { return () => {} }, [])（清理） |
| 逻辑复用            | HOC（高阶组件）/ Render Props（渲染属性）    | 自定义 Hooks（以 use 开头，直接复用逻辑）    |
| 代码量              | 多（模板代码多，如 constructor、render）     | 少（无 this，逻辑集中，简洁直观）            |
| 状态更新依赖前状态  | this.setState(prev => ({ count: prev.count + 1 })) | setState(prev => prev + 1)                  |
| refs 引用           | this.refs.input 或 createRef()               | useRef(initialValue) → ref.current          |
```

## 三、自定义 Hooks 示例
自定义 Hooks 是 Hooks 的核心优势，用于**抽取可复用逻辑**，必须以 `use` 开头（React 强制规则）。

### 示例：useLocalStorage（本地存储同步 Hooks）
封装 `AsyncStorage` 逻辑，实现状态与本地存储自动同步：
```javascript
import React, { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

// 自定义 Hooks：同步本地存储与组件状态
const useLocalStorage = (key, initialValue) => {
  const [value, setValue] = useState(initialValue);
  const [loading, setLoading] = useState(true); // 加载状态

  // 挂载时读取本地存储
  useEffect(() => {
    const loadData = async () => {
      try {
        const storedVal = await AsyncStorage.getItem(key);
        if (storedVal) setValue(JSON.parse(storedVal)); // 解析 JSON
      } catch (err) {
        console.error('读取本地存储失败：', err);
      } finally {
        setLoading(false);
      }
    };
    loadData();
  }, [key]);

  // 状态变化时同步到本地存储
  useEffect(() => {
    if (!loading) {
      const saveData = async () => {
        try {
          await AsyncStorage.setItem(key, JSON.stringify(value));
        } catch (err) {
          console.error('保存本地存储失败：', err);
        }
      };
      saveData();
    }
  }, [key, value, loading]);

  return [value, setValue, loading]; // 返回状态、更新函数、加载状态
};

// 使用自定义 Hooks
const Profile = () => {
  const [username, setUsername, loading] = useLocalStorage('username', '');

  if (loading) return <Text style={{ marginTop: 50, textAlign: 'center' }}>加载中...</Text>;

  return (
    <View style={{ alignItems: 'center', marginTop: 50 }}>
      <Text>当前用户名：{username || '未设置'}</Text>
      <Button title="设置用户名" onPress={() => setUsername('React Native 开发者')} />
      <Button title="清空" onPress={() => setUsername('')} style={{ marginTop: 10 }} />
    </View>
  );
};

export default Profile;
```

#### 自定义 Hooks 返回结果
```
| 返回项    | 类型    | 描述                          |
| --------- | ------- | ----------------------------- |
| value     | 任意类型 | 本地存储中的值（初始为初始值） |
| setValue  | 函数    | 更新状态并同步到本地存储      |
| loading   | boolean | 本地存储读取状态（true 加载中）|
```
## 四、Hooks 使用规则
```
1. **只能在顶层调用**：不能在 if、for、嵌套函数、try/catch 中调用 Hooks；
2. **只能在 React 函数中调用**：只能在函数组件或自定义 Hooks 中使用，不能在普通 JS 函数中调用；
3. **自定义 Hooks 必须以 use 开头**：如 `useLocalStorage`、`useRequest`（便于 React 识别和 lint 检查）；
4. **依赖数组要完整**：useEffect/useCallback/useMemo 的依赖数组需包含所有用到的外部变量（避免闭包陷阱）。
```

## 五、总结
```
React Native Hooks 彻底改变了函数组件的能力边界，核心优势：
1. **简化代码**：消除 class 组件的 this 困扰和模板代码；
2. **逻辑复用**：通过自定义 Hooks 抽取通用逻辑（如请求、存储、订阅），无需 HOC/Render Props；
3. **性能优化**：useCallback/useMemo 可缓存函数和计算结果，减少不必要的渲染；
4. **易于维护**：逻辑集中在函数内，代码结构更清晰。
```
Hooks 已成为 React Native 开发的首选方案，与原生组件、第三方库完全兼容，是必备的核心技能。