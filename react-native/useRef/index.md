[return](../index)

# React Native 中 useRef 详解

## 一、什么是 useRef？
`useRef` 是 React 提供的核心 Hook 之一，用于创建一个**可变的 ref 对象**。该对象包含唯一的 `current` 属性，可用于存储任意类型的数据（如 DOM/组件实例、计时器 ID、前一次状态等），且修改 `current` 属性**不会触发组件重渲染**。

### 基本语法
```
import { useRef } from 'react';

// 初始化 ref，initialValue 为初始值（可选，默认 undefined）
const refObj = useRef(initialValue);

// 读写 current 属性
refObj.current = 新值; // 修改
const 数据 = refObj.current; // 读取
```

## 二、核心特点
1. **不触发重渲染**：修改 `ref.current` 不会导致组件重新渲染，适合存储不影响 UI 的数据。
2. **数据持久化**：ref 对象在组件的整个生命周期内保持不变（除非手动替换），不会因组件重渲染而重置。
3. **多用途容器**：可存储任意类型数据，不仅限于 DOM 元素。
4. **避免闭包陷阱**：在定时器、事件处理函数等闭包中，能直接获取最新的 `current` 值（无需依赖状态依赖数组）。

## 三、常见使用场景（带代码+返回结果）

### 场景 1：访问原生组件/ DOM 实例
React Native 中，可通过 `useRef` 绑定原生组件（如 `TextInput`、`ScrollView`），直接调用组件的原生方法（如获取焦点、滚动到指定位置）。

#### 示例：TextInput 自动获取焦点
```
import React, { useRef } from 'react';
import { View, TextInput, Button, StyleSheet } from 'react-native';

const InputFocusDemo = () => {
  // 创建 ref 绑定 TextInput 组件
  const inputRef = useRef(null);

  // 点击按钮触发输入框聚焦
  const handleFocus = () => {
    // 通过 current 访问组件实例，调用 focus() 原生方法
    inputRef.current?.focus();
  };

  return (
    <View style={styles.container}>
      <TextInput
        ref={inputRef} // 绑定 ref
        style={styles.input}
        placeholder="点击按钮获取焦点"
        placeholderTextColor="#999"
      />
      <Button title="获取输入框焦点" onPress={handleFocus} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { padding: 20, gap: 15 },
  input: { borderWidth: 1, borderColor: '#ddd', borderRadius: 8, padding: 12 }
});

export default InputFocusDemo;
```

#### 返回结果
- 初始状态：输入框显示占位文字，无光标。
- 点击“获取输入框焦点”按钮后：输入框自动激活，光标显示在输入框内，键盘弹出（若在真机/模拟器中）。

### 场景 2：保存持久化数据（不触发重渲染）
当需要存储“不影响 UI，但需要跨渲染周期保留”的数据时（如计时器 ID、前一次状态），`useRef` 是最佳选择（比 `useState` 更高效，无需触发重渲染）。

#### 示例：计时器（保存 interval ID）
```
import React, { useRef, useEffect, useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

const TimerDemo = () => {
  const [count, setCount] = useState(0);
  // 用 ref 保存计时器 ID，避免重渲染时丢失
  const intervalRef = useRef(null);

  // 启动计时器
  const startTimer = () => {
    // 避免重复启动（判断 current 是否为空）
    if (!intervalRef.current) {
      intervalRef.current = setInterval(() => {
        setCount(prev => prev + 1); // 函数式更新获取最新状态
      }, 1000);
    }
  };

  // 停止计时器
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null; // 重置 ref
    }
  };

  // 组件卸载时清除计时器（避免内存泄漏）
  useEffect(() => {
    return () => stopTimer();
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.count}>当前计数：{count}</Text>
      <View style={styles.buttonGroup}>
        <Button title="启动计时器" onPress={startTimer} />
        <Button title="停止计时器" onPress={stopTimer} color="#ff4444" />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { padding: 20, alignItems: 'center', gap: 20 },
  count: { fontSize: 20 },
  buttonGroup: { flexDirection: 'row', gap: 15 }
});

export default TimerDemo;
```

#### 返回结果
- 点击“启动计时器”：`count` 每秒增加 1，`intervalRef.current` 存储计时器 ID（如 `123`，数字类型）。
- 点击“停止计时器”：计时器停止，`intervalRef.current` 重置为 `null`。
- 组件重渲染（`count` 变化）时：`intervalRef` 的引用和 `current` 值保持不变，不会导致重复启动计时器。
- 组件卸载时：自动调用 `stopTimer()`，清除计时器（避免内存泄漏）。

### 场景 3：访问自定义组件的暴露方法
默认情况下，`useRef` 无法直接访问自定义组件的内部状态/方法，需配合 `forwardRef`（转发 ref）和 `useImperativeHandle`（暴露指定方法）实现。

#### 示例：父组件调用子组件方法
```
import React, { useRef, forwardRef, useImperativeHandle, useState } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';

// 子组件：用 forwardRef 转发 ref，useImperativeHandle 暴露方法
const CustomCounter = forwardRef((props, ref) => {
  const [count, setCount] = useState(0);

  // 暴露给父组件的方法（仅暴露需要的方法，隐藏内部实现）
  useImperativeHandle(ref, () => ({
    reset: () => setCount(0), // 重置计数
    addFive: () => setCount(prev => prev + 5) // 每次加 5
  }), []); // 依赖空数组，方法不会因重渲染变化

  return <Text style={styles.counter}>子组件计数：{count}</Text>;
});

// 父组件：通过 ref 调用子组件暴露的方法
const ParentDemo = () => {
  const counterRef = useRef(null);

  return (
    <View style={styles.container}>
      <CustomCounter ref={counterRef} />
      <View style={styles.buttonGroup}>
        <Button
          title="重置子组件计数"
          onPress={() => counterRef.current?.reset()}
        />
        <Button
          title="子组件+5"
          onPress={() => counterRef.current?.addFive()}
          color="#2196f3"
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { padding: 20, gap: 20 },
  counter: { fontSize: 18, textAlign: 'center' },
  buttonGroup: { flexDirection: 'row', gap: 15, justifyContent: 'center' }
});

export default ParentDemo;
```

#### 返回结果
- 点击“重置子组件计数”：子组件 `count` 变为 0（调用 `reset` 方法）。
- 点击“子组件+5”：子组件 `count` 每次增加 5（调用 `addFive` 方法）。
- 父组件无法直接访问子组件的 `count` 状态，仅能调用 `useImperativeHandle` 暴露的方法（符合封装原则）。

## 四、useRef vs useState 对比表
```
| 对比维度                | useRef                          | useState                        |
| ----------------------- | ------------------------------- | ------------------------------- |
| 更新是否触发重渲染      | 不触发（修改 current 无影响）   | 触发（setState 后组件重渲染）    |
| 数据访问方式            | 直接读写 ref.current            | 读：直接访问变量；写：通过 setter 方法（如 setXxx） |
| 适用场景                | 1. 访问 DOM/组件实例 2. 保存不影响 UI 的持久化数据 3. 避免闭包陷阱 | 1. 保存影响 UI 的状态 2. 触发组件重渲染更新界面 |
| 数据持久化              | 组件生命周期内保持（除非手动修改 current） | 组件生命周期内保持，setter 会替换旧值 |
| 闭包中的表现            | 始终获取最新的 current 值       | 捕获当前渲染周期的状态值，需用函数式更新（如 prev => prev + 1）获取最新值 |
| 初始值类型              | 支持任意类型（如 null、数字、对象等） | 支持任意类型（同 useRef）        |
```

## 五、注意事项
1. **避免过度使用**：若数据需要驱动 UI 变化，优先使用 `useState`；仅当数据不影响 UI 时，再用 `useRef`。
2. **自定义组件需转发 ref**：直接给自定义组件绑定 `ref` 会报错，必须配合 `forwardRef` 转发。
3. **避免暴露过多内部方法**：`useImperativeHandle` 应仅暴露必要的方法，避免破坏组件封装性。
4. **清理副作用**：若用 `useRef` 保存计时器、订阅等，需在组件卸载时清理（如 `clearInterval`），避免内存泄漏。

## 总结
`useRef` 是 React Native 中处理“可变且不触发重渲染”数据的核心工具，核心价值在于**持久化存储**和**访问实例**。它弥补了 `useState` 触发重渲染的短板，适用于计时器 ID、DOM/组件实例、前一次状态等场景，是优化组件性能、解决闭包陷阱的重要手段。