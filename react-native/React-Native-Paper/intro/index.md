[return](/react-native/React-Native-Paper/index)


React Native Paper 是一款**遵循 Material Design 规范**的 React Native UI 组件库，核心价值是为跨平台（iOS/Android）RN 应用提供开箱即用、风格统一且可定制的组件，解决原生 RN 开发中 UI 样式碎片化、设计规范落地难的问题，尤其适合快速搭建企业级或品牌化应用。



### 一、核心特性
```
React Native Paper 的优势集中在“规范统一”“易用性”和“扩展性”上，具体包括以下几点：
1. **严格遵循 Material Design 3**：组件样式、交互逻辑（如按钮点击反馈、弹窗动画）完全对齐 Google 官方 Material Design 3 规范，无需手动适配设计细节。
2. **跨平台一致性**：同一套代码在 iOS 和 Android 上呈现统一的 UI 风格，避免因平台原生组件差异导致的视觉割裂（如按钮圆角、输入框样式）。
3. **灵活的主题定制**：支持全局主题（主色、次色、背景色、字体）自定义，可快速匹配品牌视觉，且支持浅色/深色模式一键切换。
4. **组件丰富且实用**：涵盖 50+ 常用组件，包括基础组件（Button、TextInput、Card）、导航组件（AppBar、Drawer）、表单组件（Checkbox、RadioButton）、反馈组件（Snackbar、Dialog）等，满足大多数场景需求。
5. **无障碍支持**：内置无障碍属性（如屏幕阅读器标签、焦点管理），符合 WCAG 标准，降低企业级应用的无障碍适配成本。
6. **TypeScript 原生支持**：所有组件均提供完整的 TypeScript 类型定义，开发时可获得自动补全和类型校验，减少 runtime 错误。
```

### 二、安装与基础配置
React Native Paper 依赖 `react-native-vector-icons` 处理图标，需两步完成安装和配置，适用于 Expo 和裸 RN 项目。

#### 1. 安装依赖
```bash
# 1. 安装核心库
npm install react-native-paper
# 或 yarn
yarn add react-native-paper

# 2. 安装依赖的图标库（必装）
npm install react-native-vector-icons
# 或 yarn
yarn add react-native-vector-icons
```

#### 2. 基础配置（全局主题 Provider）
需在应用入口（如 `App.js`）包裹 `PaperProvider`，这是实现全局“uni style”的核心，所有子组件将自动继承主题样式。

```javascript
import React from 'react';
import { SafeAreaProvider } from 'react-native-safe-area-context'; // 可选，适配安全区域
import { PaperProvider, DefaultTheme } from 'react-native-paper';
import HomeScreen from './HomeScreen';

// 1. 自定义主题（可选，覆盖默认值）
const CustomTheme = {
  ...DefaultTheme,
  colors: {
    ...DefaultTheme.colors,
    primary: '#2196F3', // 品牌主色（替换为自己的主色）
    secondary: '#FFC107', // 品牌次色
    background: '#F9F9F9', // 页面背景色
    surface: '#FFFFFF', // 卡片、输入框等表面色
    error: '#F44336', // 错误色
  },
  fonts: {
    ...DefaultTheme.fonts,
    regular: {
      fontFamily: 'Roboto-Regular', // 自定义字体（需提前配置字体文件）
    },
  },
};

// 2. 应用入口：用 PaperProvider 包裹所有组件
const App = () => {
  return (
    <SafeAreaProvider>
      <PaperProvider theme={CustomTheme}>
        <HomeScreen />
      </PaperProvider>
    </SafeAreaProvider>
  );
};

export default App;
```

**返回效果**：所有使用 React Native Paper 的组件（如 Button、Card）将自动应用 `CustomTheme` 的颜色和字体，无需在每个组件内重复设置样式。


### 三、常用组件实战示例
选取 4 个最高频使用的组件，展示其基础用法和“统一风格”特性。

#### 1. Button（按钮）
支持 `contained`（填充）、`outlined`（描边）、`text`（文本）三种变体，自动继承主题色。

```javascript
import React from 'react';
import { View, StyleSheet } from 'react-native';
import { Button } from 'react-native-paper';

const MyButtons = () => {
  return (
    <View style={styles.container}>
      {/* 填充按钮（默认，用主题 primary 色） */}
      <Button mode="contained" onPress={() => console.log('点击填充按钮')}>
        确认提交
      </Button>

      {/* 描边按钮（边框用 primary 色，背景透明） */}
      <Button mode="outlined" onPress={() => console.log('点击描边按钮')}>
        取消操作
      </Button>

      {/* 文本按钮（无背景/边框，文字用 primary 色） */}
      <Button mode="text" onPress={() => console.log('点击文本按钮')}>
        查看详情
      </Button>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    gap: 12, // 按钮间距
    padding: 16,
  },
});

export default MyButtons;
```

**返回效果**：三个按钮分别为蓝色填充、蓝色描边、蓝色文本（继承 CustomTheme 的 primary 色），间距统一，点击时有 Material Design 特有的波纹反馈。


#### 2. Card（卡片）
用于包裹信息块（如商品卡片、列表项），自带阴影、圆角，结构清晰（支持标题、内容、行动区）。

```javascript
import React from 'react';
import { View, StyleSheet } from 'react-native';
import { Card, Title, Paragraph, Button } from 'react-native-paper';

const ProductCard = () => {
  return (
    <View style={styles.container}>
      <Card>
        {/* 卡片内容区 */}
        <Card.Content>
          <Title>无线蓝牙耳机</Title>
          <Paragraph>降噪模式 | 续航24小时 | 防水IPX5</Paragraph>
          <Paragraph style={styles.price}>¥299</Paragraph>
        </Card.Content>

        {/* 卡片行动区（底部按钮） */}
        <Card.Actions>
          <Button>加入购物车</Button>
          <Button mode="contained">立即购买</Button>
        </Card.Actions>
      </Card>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
  },
  price: {
    color: '#F44336', // 错误色（继承主题 error 色也可）
    fontSize: 16,
    marginTop: 8,
  },
});

export default ProductCard;
```

**返回效果**：白色卡片（继承主题 surface 色）带轻微阴影，标题粗体、内容常规，底部两个按钮对齐，整体符合 Material Design 卡片规范。


#### 3. TextInput（输入框）
支持标签、错误提示、图标、密码隐藏等功能，自带下划线或边框样式，避免原生输入框的样式差异。

```javascript
import React, { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import { TextInput, HelperText } from 'react-native-paper';

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [emailError, setEmailError] = useState(false);

  // 验证邮箱格式
  const validateEmail = () => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    setEmailError(!regex.test(email) && email !== '');
  };

  return (
    <View style={styles.container}>
      {/* 邮箱输入框 */}
      <TextInput
        label="邮箱"
        value={email}
        onChangeText={setEmail}
        onBlur={validateEmail} // 失焦时验证
        keyboardType="email-address"
        style={styles.input}
        left={<TextInput.Icon icon="email" />} // 左侧图标
      />
      {/* 错误提示（验证失败时显示） */}
      <HelperText type="error" visible={emailError}>
        请输入有效的邮箱地址
      </HelperText>

      {/* 密码输入框 */}
      <TextInput
        label="密码"
        value={password}
        onChangeText={setPassword}
        secureTextEntry // 密码隐藏
        style={styles.input}
        left={<TextInput.Icon icon="lock" />}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
  },
  input: {
    marginBottom: 8,
  },
});

export default LoginForm;
```

**返回效果**：输入框带灰色标签（聚焦时上浮并变为 primary 色），左侧有图标，邮箱输入错误时显示红色提示文字，iOS 和 Android 样式完全一致。


#### 4. Snackbar（轻量反馈）
用于显示短期提示（如“操作成功”“网络错误”），自动消失，不阻塞用户操作，比原生 Alert 更友好。

```javascript
import React, { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import { Button, Snackbar } from 'react-native-paper';

const SnackbarExample = () => {
  const [visible, setVisible] = useState(false);

  return (
    <View style={styles.container}>
      <Button mode="contained" onPress={() => setVisible(true)}>
        显示提示
      </Button>

      {/* Snackbar：底部弹出，3秒后自动消失 */}
      <Snackbar
        visible={visible}
        onDismiss={() => setVisible(false)}
        duration={3000} // 显示时长（毫秒）
        action={{
          label: '撤销',
          onPress: () => console.log('点击撤销'),
        }}
      >
        操作成功！
      </Snackbar>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
    justifyContent: 'center',
    flex: 1,
  },
});

export default SnackbarExample;
```

**返回效果**：点击按钮后，底部弹出黑色半透明提示框，显示“操作成功！”，右侧有“撤销”按钮，3秒后自动隐藏，交互流畅。


### 四、进阶用法：主题定制与扩展
#### 1. 全局主题覆盖
除了基础颜色，还可定制字体、圆角、阴影等全局样式，例如修改所有组件的圆角大小：
```javascript
const CustomTheme = {
  ...DefaultTheme,
  roundness: 12, // 全局组件圆角（默认8）
  colors: {
    ...DefaultTheme.colors,
    primary: '#6200EE', // 紫色主色
  },
  fonts: {
    ...DefaultTheme.fonts,
    medium: {
      fontFamily: 'MyCustomFont-Medium', // 自定义字体
      fontSize: 15,
    },
  },
};
```

#### 2. 局部主题覆盖
若需某个组件脱离全局主题，可使用 `Provider` 嵌套覆盖：
```javascript
import { Provider as Paper局部Provider } from 'react-native-paper';

const LocalThemedButton = () => {
  const localTheme = {
    ...DefaultTheme,
    colors: { primary: '#FF9800' }, // 局部橙色按钮
  };

  return (
    <Paper局部Provider theme={localTheme}>
      <Button mode="contained">局部橙色按钮</Button>
    </Paper局部Provider>
  );
};
```


### 五、优缺点对比
```
| 维度                | 优点                                  | 缺点                                  |
|---------------------|---------------------------------------|---------------------------------------|
| 设计规范            | 严格遵循 Material Design 3，风格统一  | 非 Material Design 项目需额外定制样式  |
| 跨平台一致性        | iOS/Android 视觉、交互完全一致        | 相比原生组件，包体积略大（约 500KB）  |
| 易用性              | 开箱即用，无需手动适配基础样式        | 高度定制化场景（如特殊动画）需二次开发|
| 生态与维护          | 由 Callstack（RN 核心贡献团队）维护，更新频繁 | 依赖 `react-native-vector-icons`，需额外配置 |
| 无障碍与TypeScript  | 内置无障碍支持，完整 TypeScript 类型  | 部分冷门组件（如 Calendar）功能较简单 |
```

### 六、适用场景
- 适合**快速搭建企业级应用**（如管理后台、办公APP），需统一设计风格且开发周期短；
- 适合**遵循 Material Design 规范**的项目，无需从零开发组件；
- 适合**跨平台一致性要求高**的应用，避免 iOS/Android 样式割裂。


要不要我帮你整理一份**React Native Paper 实战代码模板**？模板包含登录页（TextInput + Button）、商品列表页（Card + FlatList）、个人中心页（AppBar + List），可直接复制到项目中快速启动开发。