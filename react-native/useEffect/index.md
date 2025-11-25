[return](../index)
好的，我们来系统地梳理一下 React Native 中 `useEffect` Hook 的相关知识。

---

## 1. 什么是 `useEffect`

`useEffect` 是 React 提供的一个核心 Hook，它用于在**函数组件中处理副作用**。

**副作用**指的是那些不直接属于组件渲染逻辑的操作，例如：

*   **数据获取** (如调用 API)
*   **订阅事件** (如监听网络状态、设备传感器)
*   **手动操作 DOM**
*   **设置定时器或间隔器**
*   **日志记录**

在 `useEffect` 出现之前，这些操作通常在类组件的生命周期方法中完成，比如 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount`。`useEffect` 将这三个生命周期函数的功能整合到了一个 API 中，让代码更简洁、更易于理解。

### 基本语法

```
import React, { useEffect } from 'react';

function MyComponent() {
  // 第一个参数是一个回调函数，包含了要执行的副作用逻辑
  // 这个函数可以返回一个清理函数（可选）
  useEffect(() => {
    // 副作用逻辑...

    // 清理函数（可选）
    return () => {
      // 在组件卸载或 effect 重新执行前清理副作用
      // 例如：取消订阅、清除定时器等
    };
  }, [
    // 第二个参数是一个依赖数组（可选）
    // 数组中的元素是 effect 所依赖的值
    // 只有当这些依赖项发生变化时，effect 才会重新执行
  ]);

  return <div>Component Content</div>;
}
```

---

## 2. `useEffect` 的依赖数组

依赖数组是 `useEffect` 中最关键也最容易出错的部分。它决定了 `useEffect` 何时执行。

### 场景一：不提供依赖数组

```
useEffect(() => {
  console.log('Effect ran');
});
```

*   **行为**：每次组件渲染后（包括初始渲染和每次更新后），这个 effect 都会执行。
*   **类比**：同时扮演了 `componentDidMount` 和 `componentDidUpdate` 的角色。
*   **注意**：这种用法很容易导致性能问题或无限循环，需要谨慎使用。

### 场景二：提供一个空依赖数组

```
useEffect(() => {
  console.log('Effect ran once after initial render');
  
  return () => {
    console.log('Cleanup before component unmount');
  };
}, []); // 空数组
```

*   **行为**：effect **只在组件挂载后执行一次**。返回的清理函数**只在组件卸载前执行一次**。
*   **类比**：`componentDidMount` (执行) 和 `componentWillUnmount` (清理)。
*   **常见用途**：初始化数据获取、设置一次性的事件监听器。

### 场景三：提供包含依赖项的数组

```
const [count, setCount] = useState(0);

useEffect(() => {
  console.log(`Count has changed to: ${count}`);
  
  // 假设我们有一个基于 count 的订阅
  const subscription = someEventSource.subscribe(count);

  return () => {
    // 清理上一次的订阅
    subscription.unsubscribe();
  };
}, [count]); // 依赖 count
```

*   **行为**：
    1.  在**初始渲染后**执行一次。
    2.  在**每次渲染后**，如果 `count` 的值与上一次渲染不同，则执行 effect。
    3.  在 effect **重新执行前**，会先执行上一次 effect 返回的清理函数。
    4.  在**组件卸载前**，会执行最后一次 effect 返回的清理函数。
*   **类比**：结合了 `componentDidMount`、`componentDidUpdate` (针对特定 props/state) 和 `componentWillUnmount` 的逻辑。
*   **常见用途**：当某个值变化时，需要重新计算、重新订阅或重新获取数据的场景。

**黄金法则**：确保依赖数组包含了 effect 中使用的**所有**组件内的变量（props、state、函数等）。否则，你可能会遇到“ stale closure”（过时的闭包）问题，即 effect 中访问到的是变量的旧值。

---

## 3. React Native 中的常见用法示例

### 示例 1：组件挂载时获取数据 (API Call)

这是最常见的用法。在组件首次加载时从服务器获取数据。

```
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';

const DataList = () => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const jsonData = await response.json();
        setData(jsonData); // 更新状态
      } catch (err) {
        setError(err.message); // 处理错误
      } finally {
        setLoading(false); // 无论成功失败，都停止加载
      }
    };

    fetchData();
  }, []); // 空依赖数组，只执行一次

  if (loading) return <ActivityIndicator size="large" color="#0000ff" />;
  if (error) return <Text>Error: {error}</Text>;

  return (
    <FlatList
      data={data}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => <Text>{item.title}</Text>}
    />
  );
};

export default DataList;
```
```
**返回结果**：
*   组件挂载后，会发起网络请求。
*   在请求期间，屏幕上会显示一个加载指示器 (`ActivityIndicator`)。
*   请求成功后，`FlatList` 会显示从 API 获取的数据列表。
*   如果请求失败，会显示错误信息。
```
---

### 示例 2：监听设备网络状态

在 React Native 中，经常需要根据网络状态来调整 UI，比如显示“无网络”提示。

```
import React, { useState, useEffect } from 'react';
import { View, Text, NetInfo } from 'react-native';

const NetworkStatus = () => {
  const [isConnected, setIsConnected] = useState(true);

  useEffect(() => {
    // 定义一个监听网络状态变化的函数
    const handleNetworkChange = (state) => {
      setIsConnected(state.isConnected);
    };

    // 订阅网络状态变化事件
    const subscription = NetInfo.addEventListener(handleNetworkChange);

    // 清理函数：取消订阅
    return () => {
      subscription.remove();
    };
  }, []); // 空依赖数组

  return (
    <View>
      <Text>
        {isConnected ? '网络连接正常' : '无网络连接，请检查设置'}
      </Text>
    </View>
  );
};

export default NetworkStatus;
```
**注意**：`NetInfo` 需要从 `@react-native-community/netinfo` 安装和导入（React Native 0.60+）。
```
**返回结果**：
*   组件挂载后，立即订阅网络状态变化。
*   当设备网络连接或断开时，`handleNetworkChange` 会被调用，`isConnected` 状态会更新，
UI 也会相应地显示不同的文本。
*   当组件卸载时，`useEffect` 的清理函数会移除事件监听，防止内存泄漏。
```
---

### 示例 3：根据 props 变化更新数据

假设你有一个详情页，当传入的 `itemId` props 变化时，需要重新获取对应的数据。

```
import React, { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

const ItemDetail = ({ itemId }) => {
  const [item, setItem] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 每次 itemId 变化时，都重新设置 loading 状态
    setLoading(true);

    const fetchItem = async () => {
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${itemId}`);
        const jsonData = await response.json();
        setItem(jsonData);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };

    fetchItem();
  }, [itemId]); // 依赖 itemId

  if (loading) return <ActivityIndicator size="large" />;
  if (!item) return <Text>Item not found</Text>;

  return (
    <View>
      <Text style={{ fontSize: 20 }}>{item.title}</Text>
      <Text>{item.body}</Text>
    </View>
  );
};

export default ItemDetail;
```

**返回结果**：
*   当 `ItemDetail` 组件首次渲染且 `itemId` 为 `1` 时，它会加载并显示 ID 为 1 的帖子。
*   如果父组件传递的 `itemId` 变为 `2`，`useEffect` 会检测到依赖变化，再次执行 `fetchItem` 函数，加载并显示 ID 为 2 的帖子。在此期间，会显示加载指示器。

---

## 4. 总结与最佳实践
```
| 特性 | 描述 |
| :--- | :--- |
| **核心功能** | 在函数组件中处理副作用 |
| **执行时机** | 组件渲染**之后**执行 |
| **依赖数组** | 控制 effect 的执行时机，是性能优化和避免 bug 的关键 |
| **清理机制** | 通过返回一个函数，可以在组件卸载或 effect 重新执行前清理资源，防止内存泄漏 |
```
### 最佳实践：

1.  **明确依赖**：始终为 `useEffect` 提供正确的依赖数组。这是避免不必要渲染和 `stale closure` 问题的关键。
2.  **拆分 Effects**：将不同目的的副作用拆分成多个独立的 `useEffect`。例如，一个用于数据获取，另一个用于事件监听。这会让代码更清晰。
3.  **清理副作用**：对于订阅、定时器、事件监听等，务必在清理函数中取消或清除它们。
4.  **避免无限循环**：如果你在 `useEffect` 中更新了一个 state，而这个 state 又在依赖数组里，就可能导致无限循环。确保更新逻辑是有条件的。
5.  **使用 `useCallback`/`useMemo`**：如果 `useEffect` 的依赖是一个函数或一个复杂对象，考虑使用 `useCallback` 或 `useMemo` 来记忆它们，以避免不必要的 effect 触发。

`useEffect` 是 React Native 开发中不可或缺的工具，掌握它的使用方法对于编写健壮、高效的组件至关重要。

