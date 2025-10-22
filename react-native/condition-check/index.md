[return](/react-native/index)

在 React Native 中，条件判断（条件渲染）是根据不同状态或数据动态显示不同 UI 的核心能力。由于 JSX 本质是 JavaScript 的语法扩展，因此可以直接在 JSX 中结合 JavaScript 逻辑实现条件判断。以下是常用的条件判断写法及示例：


### 1. 基本写法介绍

#### （1）if 语句（JSX 外部判断）
在 JSX 外部通过 `if` 语句判断条件，根据结果返回不同的 JSX 元素。适合逻辑较复杂的场景。

**示例**：根据用户是否登录显示不同内容
```javascript
import React from 'react';
import { Text, View } from 'react-native';

const UserGreeting = ({ isLoggedIn }) => {
  // 在 JSX 外部用 if 判断
  if (isLoggedIn) {
    return <Text>欢迎回来！</Text>;
  } else {
    return <Text>请先登录~</Text>;
  }
};

// 使用组件
const App = () => {
  return (
    <View>
      <UserGreeting isLoggedIn={true} />  {/* 返回 <Text>欢迎回来！</Text> */}
      <UserGreeting isLoggedIn={false} /> {/* 返回 <Text>请先登录~</Text> */}
    </View>
  );
};
```


#### （2）三元运算符（JSX 内部嵌入）
在 JSX 内部直接使用 `条件 ? 结果1 : 结果2` 语法，适合简单的二选一逻辑，可直接嵌入到 JSX 结构中。

**示例**：根据数量显示不同文本
```javascript
import React from 'react';
import { Text, View } from 'react-native';

const Cart = ({ itemCount }) => {
  return (
    <View>
      {/* 直接在 JSX 中用三元运算符 */}
      <Text>
        {itemCount === 0 
          ? '购物车是空的' 
          : `购物车有 ${itemCount} 件商品`}
      </Text>
    </View>
  );
};

// 使用组件
const App = () => {
  return (
    <View>
      <Cart itemCount={0} />   {/* 返回 <Text>购物车是空的</Text> */}
      <Cart itemCount={3} />   {/* 返回 <Text>购物车有 3 件商品</Text> */}
    </View>
  );
};
```


#### （3）逻辑与 &&（条件为真时渲染）
使用 `条件 && 元素` 语法，当条件为 `true` 时渲染后面的元素，为 `false` 时不渲染（相当于渲染 `null`）。适合“满足条件才显示，不满足就隐藏”的场景。

**示例**：有新消息时显示提示
```javascript
import React from 'react';
import { Text, View } from 'react-native';

const MessageTip = ({ hasNewMessage }) => {
  return (
    <View>
      {/* 条件为 true 时显示，false 时不显示 */}
      {hasNewMessage && <Text>您有新消息！</Text>}
    </View>
  );
};

// 使用组件
const App = () => {
  return (
    <View>
      <MessageTip hasNewMessage={true} />  {/* 返回 <Text>您有新消息！</Text> */}
      <MessageTip hasNewMessage={false} /> {/* 不渲染任何内容（返回 null） */}
    </View>
  );
};
```


#### （4）多条件判断（switch 或嵌套三元）
对于多个条件的场景，可使用 `switch` 语句（JSX 外部）或嵌套三元运算符（JSX 内部），但嵌套三元需注意可读性。

**示例**：根据用户等级显示不同标签
```javascript
import React from 'react';
import { Text, View } from 'react-native';

// 方法1：switch 语句（JSX 外部）
const LevelTag = ({ level }) => {
  let tagContent;
  switch (level) {
    case 'vip':
      tagContent = <Text style={{ color: 'red' }}>VIP用户</Text>;
      break;
    case 'member':
      tagContent = <Text style={{ color: 'blue' }}>普通会员</Text>;
      break;
    default:
      tagContent = <Text style={{ color: 'gray' }}>游客</Text>;
  }
  return <View>{tagContent}</View>;
};

// 方法2：嵌套三元（JSX 内部，适合简单多条件）
const LevelTag2 = ({ level }) => {
  return (
    <View>
      {level === 'vip' ? (
        <Text style={{ color: 'red' }}>VIP用户</Text>
      ) : level === 'member' ? (
        <Text style={{ color: 'blue' }}>普通会员</Text>
      ) : (
        <Text style={{ color: 'gray' }}>游客</Text>
      )}
    </View>
  );
};

// 使用组件
const App = () => {
  return (
    <View>
      <LevelTag level="vip" />      {/* 返回 <Text style={{color:'red'}}>VIP用户</Text> */}
      <LevelTag2 level="member" />  {/* 返回 <Text style={{color:'blue'}}>普通会员</Text> */}
    </View>
  );
};
```


#### （5）条件渲染组件（封装逻辑）
将条件判断逻辑封装到独立组件中，通过 props 传递条件，使代码更清晰（适合复杂条件）。

**示例**：根据加载状态显示不同组件
```javascript
import React from 'react';
import { Text, View, ActivityIndicator } from 'react-native';

// 封装条件渲染组件
const LoadingState = ({ isLoading, loadedContent, loadingText = '加载中...' }) => {
  if (isLoading) {
    return (
      <View>
        <ActivityIndicator size="small" />
        <Text>{loadingText}</Text>
      </View>
    );
  } else {
    return loadedContent; // 加载完成后显示的内容
  }
};

// 使用组件
const App = () => {
  const [loading, setLoading] = React.useState(true);

  // 模拟加载完成
  React.useEffect(() => {
    setTimeout(() => setLoading(false), 2000);
  }, []);

  return (
    <View>
      <LoadingState
        isLoading={loading}
        loadedContent={<Text>数据加载完成！</Text>}
      />
      {/* 加载中时返回：加载指示器 + "加载中..."；完成后返回："数据加载完成！" */}
    </View>
  );
};
```


### 2. 对比表
```
写法               | 适用场景                     | 优点                     | 缺点                     | 示例代码片段
-------------------|------------------------------|--------------------------|--------------------------|--------------------------
if 语句            | 逻辑复杂、多分支判断         | 可读性高，适合复杂逻辑   | 需在JSX外部处理，较繁琐  | if (a) { return <A /> } else { return <B /> }
三元运算符         | 简单二选一逻辑               | 可直接嵌入JSX，简洁      | 多分支嵌套时可读性差     | { a ? <A /> : <B /> }
逻辑与 &&          | 条件为真时才渲染某个元素     | 极简，适合简单显示/隐藏  | 条件为0、''等假值时可能意外隐藏 | { hasData && <List /> }
switch 语句        | 多分支（3个以上条件）        | 结构清晰，适合多条件     | 代码量稍多，需在JSX外部  | switch (type) { case 1: ... }
条件渲染组件       | 重复使用的复杂条件逻辑       | 复用性高，代码模块化     | 需额外定义组件           | <LoadingState isLoading={...} />
```


### 3. 注意事项
- **避免在 JSX 中直接写 if 语句**：JSX 中只能写表达式（expression），不能写语句（statement），因此 if 必须放在 JSX 外部或用三元/&& 替代。
- **警惕假值陷阱**：使用 `&&` 时，若条件是 0、'' 等“假值”，会渲染这些值（而非不渲染）。例如 `{ 0 && <Text>数量</Text> }` 会渲染 `0`，需注意判断条件的准确性。
- **优先拆分组件**：复杂条件逻辑建议封装到独立组件中，避免主组件代码臃肿。
- **空渲染处理**：不需要渲染时，可直接返回 `null`（React Native 会忽略 `null` 渲染）。


通过以上方法，可以灵活处理 React Native 中的各种条件判断场景，根据实际逻辑复杂度选择合适的写法即可。