[return](../index)

在React Native中，`useMemo` 是React提供的一个Hook，用于**缓存计算结果**，避免在组件每次渲染时重复执行“昂贵”（耗时）的计算逻辑，从而优化性能。


### 基本语法
```
const 缓存的结果 = useMemo(
  () => {
    // 这里是需要缓存结果的计算逻辑（通常是耗时操作）
    return 计算结果;
  },
  [依赖项数组] // 当依赖项变化时，才会重新执行计算逻辑；否则返回缓存的结果
);
```


### 用法示例
假设我们有一个“计算两个数乘积”的函数，当输入的数字不变时，不需要重复计算，此时可以用`useMemo`缓存结果。

#### 不使用`useMemo`（每次渲染都重新计算）
```
import { useState } from 'react';
import { View, Text, Button } from 'react-native';

const Example = () => {
  const [a, setA] = useState(2);
  const [b, setB] = useState(3);

  // 每次组件渲染时，都会重新执行这个计算
  const multiply = () => {
    console.log('重新计算乘积');
    return a * b;
  };
  const result = multiply();

  return (
    <View>
      <Text>乘积结果: {result}</Text>
      <Button title="改变a" onPress={() => setA(a + 1)} />
      <Button title="改变b" onPress={() => setB(b + 1)} />
    </View>
  );
};

export default Example;
```
**返回结果**：  
每次点击按钮（改变`a`或`b`），或组件因其他原因重新渲染时，控制台都会打印`重新计算乘积`，即使`a`和`b`没有变化。


#### 使用`useMemo`（仅依赖项变化时计算）
```
import { useState, useMemo } from 'react';
import { View, Text, Button } from 'react-native';

const Example = () => {
  const [a, setA] = useState(2);
  const [b, setB] = useState(3);

  // 用useMemo缓存计算结果，仅当a或b变化时才重新计算
  const result = useMemo(() => {
    console.log('重新计算乘积');
    return a * b;
  }, [a, b]); // 依赖项：a和b

  return (
    <View>
      <Text>乘积结果: {result}</Text>
      <Button title="改变a" onPress={() => setA(a + 1)} />
      <Button title="改变b" onPress={() => setB(b + 1)} />
    </View>
  );
};

export default Example;
```
**返回结果**：  
只有当`a`或`b`的值发生变化时，控制台才会打印`重新计算乘积`；若组件因其他原因（如父组件传参不变时的重渲染）重新渲染，`result`会直接使用缓存的结果，不会重新计算。


### 何时使用`useMemo`？
- 计算逻辑**耗时**（如大量数据循环、复杂数学运算）；
- 计算结果被用于子组件的`props`或`useEffect`的依赖项，需要避免因“无意义的结果变化”导致子组件重渲染或`useEffect`重复执行。


### 对比表
```
场景                  是否使用useMemo       计算执行时机                  性能表现
--------------------- -------------------- ----------------------------- --------------------------
计算逻辑简单（如a+b）   不使用                每次渲染都执行                性能影响可忽略
计算逻辑简单（如a+b）   使用                  仅依赖项变化时执行            可能增加额外缓存开销（不推荐）
计算逻辑复杂（如大数据处理） 不使用          每次渲染都执行                可能导致卡顿、性能下降
计算逻辑复杂（如大数据处理） 使用            仅依赖项变化时执行            减少重复计算，提升性能
```