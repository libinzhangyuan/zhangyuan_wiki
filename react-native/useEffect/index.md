[return](../index)

我们来详细解释一下如何让 `useEffect` 在**每次渲染后**都执行。

要实现“每次渲染后”都执行 `effect`，非常简单：**不要提供第二个参数（依赖数组）**。

### 代码示例

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';

function Counter() {
  const [count, setCount] = useState(0);
  const [message, setMessage] = useState("Hello");

  // 不提供依赖数组
  useEffect(() => {
    console.log('Effect ran after every render');
    console.log(`Current count: ${count}, Current message: ${message}`);
  }); // <-- 注意这里没有 []

  return (
    <View>
      <Text>Count: {count}</Text>
      <Text>Message: {message}</Text>
      <Button title="Increment Count" onPress={() => setCount(count + 1)} />
      <Button title="Change Message" onPress={() => setMessage("World")} />
    </View>
  );
}
```

### 运行结果分析

1.  **初始渲染后**:
    *   组件第一次渲染，`count` 是 `0`，`message` 是 `"Hello"`。
    *   `useEffect` 会执行，控制台输出：
        ```
        Effect ran after every render
        Current count: 0, Current message: Hello
        ```

2.  **点击 "Increment Count" 后**:
    *   `setCount` 被调用，`count` 变为 `1`。
    *   组件重新渲染。
    *   `useEffect` 再次执行，控制台输出：
        ```
        Effect ran after every render
        Current count: 1, Current message: Hello
        ```

3.  **点击 "Change Message" 后**:
    *   `setMessage` 被调用，`message` 变为 `"World"`。
    *   组件重新渲染。
    *   `useEffect` 再次执行，控制台输出：
        ```
        Effect ran after every render
        Current count: 1, Current message: World
        ```

**结论**：正如你所见，每当组件的状态（`count` 或 `message`）发生变化导致组件重新渲染时，这个 `useEffect` 都会在渲染之后立即执行。

---

### 对比：三种依赖数组场景

为了让你更清晰地理解，我们再次对比一下 `useEffect` 依赖数组的三种主要用法：
```
| 依赖数组写法 | 执行时机 | 适用场景 |
| :--- | :--- | :--- |
| **不提供**<br>`useEffect(() => { ... })` | **每次渲染后**<br>（初始渲染 + 每次更新后） | 需要在每次组件更新后都执行的操作，例如：<br>- 同步某些数据到第三方库<br>- 每次渲染后都需要更新的DOM操作<br>- 调试日志 |
| **空数组**<br>`useEffect(() => { ... }, [])` | **仅在初始渲染后执行一次** | 组件的初始化操作，例如：<br>- 一次性的数据获取<br>- 设置只需要订阅一次的事件监听器 |
| **包含依赖项**<br>`useEffect(() => { ... }, [count])` | **初始渲染后 + 依赖项变化后** | 当特定值变化时才需要执行的操作，例如：<br>- 根据 `count` 的变化重新计算某个值<br>- 当 `userId` 变化时重新获取该用户的数据 |
```
### 总结

*   如果你需要一个 `effect` 在**每次渲染后**都立即执行，**不要给 `useEffect` 传递第二个参数**（依赖数组）。
*   这种方式虽然简单，但请谨慎使用。因为它会在每次渲染后都运行，可能会引发性能问题，特别是在渲染频繁的组件中。通常，我们更推荐使用带有明确依赖项的 `useEffect`，以确保它只在必要时执行。