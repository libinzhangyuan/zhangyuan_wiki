[return](../index)

我们来详细介绍一下 React Native 中的**普通函数组件**（Functional Component）。

普通函数组件是 React Native 中定义 UI 组件的一种基础方式，它是一个以 JavaScript 函数形式存在的组件，核心作用是接收输入的 `props` 并返回用于描述 UI 结构的 JSX。

### 一、什么是普通函数组件？

**定义**：
普通函数组件是一个纯函数，它接收一个 `props` 对象作为参数，并返回一个 React 元素（通常是 JSX 语法编写）。

**核心特点**：

*   **简洁直观**：代码结构简单，易于阅读和编写。
*   **无状态**：在引入 Hooks 之前，函数组件本身不能拥有自己的状态（State），也无法访问生命周期方法。它的输出完全由输入的 `props` 决定。
*   **易于测试**：由于其纯函数的特性（相同的输入总是返回相同的输出），函数组件非常容易进行单元测试。

### 二、基本语法与示例

#### 1. 最简单的函数组件

这是一个最基础的函数组件，它不接收任何 `props`，只返回一段固定的文本。

```
// Greeting.js
import React from 'react';
import { Text, View } from 'react-native';

const Greeting = () => {
  return (
    <View style={{ alignItems: 'center', marginTop: 50 }}>
      <Text>Hello, React Native!</Text>
    </View>
  );
};

export default Greeting;
```

**代码解析**：

*   `const Greeting = () => { ... }`：我们定义了一个名为 `Greeting` 的箭头函数。
*   `return (...)`：函数返回了一个 JSX 结构，描述了组件的 UI。这里是一个 `View` 容器包裹着一个 `Text` 组件。
*   `export default Greeting;`：将这个组件导出，以便在其他文件中可以导入和使用它。

**如何使用它**：

你可以在你的主组件（如 `App.js`）中像使用 HTML 标签一样使用它。

```
// App.js
import React from 'react';
import { View } from 'react-native';
import Greeting from './Greeting'; // 导入我们创建的组件

const App = () => {
  return (
    <View style={{ flex: 1 }}>
      <Greeting /> {/* 使用组件 */}
    </View>
  );
};

export default App;
```

**返回结果**：
当你运行应用时，屏幕上会显示：
```
Hello, React Native!
```

#### 2. 接收 Props 的函数组件

组件的强大之处在于其可复用性，而 `props` 是实现组件复用的关键。`props` 是父组件传递给子组件的数据。

```
// UserProfile.js
import React from 'react';
import { Text, View, Image } from 'react-native';

// 函数接收一个名为 props 的参数
const UserProfile = (props) => {
  return (
    <View style={{ alignItems: 'center', padding: 20, borderBottomWidth: 1, borderBottomColor: '#eee' }}>
      <Image
        source={{ uri: props.avatarUrl }}
        style={{ width: 100, height: 100, borderRadius: 50 }}
      />
      <Text style={{ fontSize: 20, fontWeight: 'bold', marginTop: 10 }}>{props.name}</Text>
      <Text style={{ color: '#666' }}>{props.bio}</Text>
    </View>
  );
};

export default UserProfile;
```

**代码解析**：

*   `const UserProfile = (props) => { ... }`：函数现在接收一个 `props` 对象。
*   `props.name`, `props.avatarUrl`, `props.bio`：我们从 `props` 对象中获取父组件传递过来的数据，并将其渲染到 UI 中。

**如何使用它**：

在使用时，你可以通过属性的方式向组件传递数据。

```
// App.js
import React from 'react';
import { View, ScrollView } from 'react-native';
import UserProfile from './UserProfile';

const App = () => {
  // 模拟用户数据
  const user1 = {
    name: '张三',
    avatarUrl: 'https://randomuser.me/api/portraits/men/32.jpg',
    bio: ' React Native 开发者'
  };

  const user2 = {
    name: '李四',
    avatarUrl: 'https://randomuser.me/api/portraits/women/44.jpg',
    bio: '喜欢探索新技术'
  };

  return (
    <ScrollView>
      {/* 传递 props 给 UserProfile 组件 */}
      <UserProfile name={user1.name} avatarUrl={user1.avatarUrl} bio={user1.bio} />
      <UserProfile name={user2.name} avatarUrl={user2.avatarUrl} bio={user2.bio} />
    </ScrollView>
  );
};

export default App;
```

**返回结果**：
屏幕上会显示两个用户卡片，每个卡片都包含了从 `App` 组件传递过去的不同用户信息（头像、姓名、简介）。

#### 3. Props 解构赋值（更简洁的写法）

为了让代码更简洁，我们通常会使用 ES6 的解构赋值语法来直接获取 `props` 中的属性。

```
// UserProfile.js - 解构赋值版本
import React from 'react';
import { Text, View, Image } from 'react-native';

// 直接在函数参数中解构 props
const UserProfile = ({ name, avatarUrl, bio }) => {
  return (
    <View style={{ alignItems: 'center', padding: 20 }}>
      <Image source={{ uri: avatarUrl }} style={{ width: 100, height: 100, borderRadius: 50 }} />
      <Text style={{ fontSize: 20, fontWeight: 'bold', marginTop: 10 }}>{name}</Text>
      <Text style={{ color: '#666' }}>{bio}</Text>
    </View>
  );
};

export default UserProfile;
```

这种写法与之前的效果完全相同，但代码更清晰，避免了重复书写 `props.`。

### 三、普通函数组件 vs 类组件

在 React Hooks 出现之前，函数组件（无状态）和类组件（有状态）有明确的分工。虽然 Hooks 让函数组件也能拥有状态和生命周期特性，但了解它们的原始区别仍然很重要。
```
| 特性 | 普通函数组件 | 类组件 |
| :--- | :--- | :--- |
| **定义方式** | 使用 JavaScript 函数定义 | 继承 `React.Component` 或 `React.PureComponent` 的 ES6 类 |
| **状态 (State)** | 无（Hooks 出现后可通过 `useState` 拥有） | 有，通过 `this.state` 和 `this.setState()` 管理 |
| **生命周期** | 无（Hooks 出现后可通过 `useEffect` 模拟） | 有，如 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` 等 |
| **`this` 关键字** | 不使用 `this` | 需要使用 `this` 来访问 `props`、`state` 和方法 |
| **代码简洁性** | 更简洁，代码量少 | 相对冗长，有固定的模板代码（如 `constructor`） |
| **性能** | 通常性能更好，因为不需要创建类实例和绑定 `this` | 性能开销稍大 |
```
**示例：等价的类组件**

上面的 `UserProfile` 函数组件如果用类组件来写，会是这样：

```
// UserProfileClass.js
import React, { Component } from 'react';
import { Text, View, Image } from 'react-native';

class UserProfileClass extends Component {
  render() {
    // 通过 this.props 访问属性
    const { name, avatarUrl, bio } = this.props;
    return (
      <View style={{ alignItems: 'center', padding: 20 }}>
        <Image source={{ uri: avatarUrl }} style={{ width: 100, height: 100, borderRadius: 50 }} />
        <Text style={{ fontSize: 20, fontWeight: 'bold', marginTop: 10 }}>{name}</Text>
        <Text style={{ color: '#666' }}>{bio}</Text>
      </View>
    );
  }
}

export default UserProfileClass;
```

### 总结

普通函数组件是 React Native 中构建 UI 的基石。它以其简洁、易于理解和测试的特性而受到青睐。

*   **核心思想**：接收 `props`，返回 JSX。
*   **主要用途**：展示数据（Presentational Components）。
*   **现代发展**：随着 React Hooks 的引入，函数组件的能力得到了极大增强，现在几乎可以完全替代类组件来构建复杂的、有状态的应用。

在实际开发中，**普通函数组件**是你定义 UI 组件的首选方式。当你需要处理状态或副作用时，再引入相应的 Hooks 即可。