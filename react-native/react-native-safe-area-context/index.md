[return](/react-native/index)

```
install:
npm i react-native-safe-area-context


```


```
最佳实践:
用 SafeAreaProvider 包裹根组件；
通过 useSafeAreaInsets（推荐）或 SafeAreaView 灵活适配安全区域。

<SafeAreaProvider>
    <SafeAreaView 
      style={{ flex: 1, backgroundColor: '#fff' }}
      edges={['top', 'right', 'bottom', 'left']} // 只对顶部和底部应用安全区域
    >
    <text>11111</text>
  </SafeAreaView>

</SafeAreaProvider>
```


`react-native-safe-area-context` 是 React Native 生态中用于处理**安全区域适配**的核心库，主要解决不同设备（如 iPhone 刘海屏、底部 Home Indicator、Android 异形屏等）中内容被系统元素（如状态栏、刘海、底部导航条）遮挡的问题。它通过提供安全区域的边界值（Insets），帮助开发者精准调整 UI 布局。


### 一、核心作用
安全区域指设备屏幕中不会被系统组件（如刘海、底部 Home 键指示器）遮挡的区域。该库的核心功能是：  
- 跨平台（iOS/Android）获取安全区域的边界值（`top`/`bottom`/`left`/`right`，单位为 dp）；  
- 提供组件和 Hook 让开发者基于安全区域值灵活调整布局。  


### 二、安装
使用 npm 或 yarn 安装：  
```bash
# npm
npm install react-native-safe-area-context

# yarn
yarn add react-native-safe-area-context
```

**额外步骤**：  
- iOS 需通过 CocoaPods 安装依赖：进入 `ios` 目录执行 `pod install`；  
- Android 无需额外配置（自动链接）。  


### 三、核心组件与 API
该库的核心能力通过以下组件和 Hook 实现：  


#### 1. `SafeAreaProvider`（必须）
作用：作为根组件提供安全区域数据，所有需要获取安全区域值的子组件必须包裹在它内部。  

**用法**：在应用入口（如 `App.js`）包裹整个应用：  
```jsx
import { SafeAreaProvider } from 'react-native-safe-area-context';

function App() {
  return (
    <SafeAreaProvider>
      {/* 你的应用内容 */}
      <MainScreen />
    </SafeAreaProvider>
  );
}
```

> 注意：必须确保 `SafeAreaProvider` 是根组件之一，否则子组件无法获取安全区域数据。


#### 2. `useSafeAreaInsets`（推荐）
作用：React Hook，在函数组件中直接获取安全区域的边界值（`insets`）。  

返回值 `insets` 是一个对象，包含四个方向的安全区域距离：  
- `insets.top`：顶部安全区域高度（如 iPhone 刘海+状态栏高度）；  
- `insets.bottom`：底部安全区域高度（如 iPhone 底部 Home Indicator 高度）；  
- `insets.left`/`insets.right`：左右安全区域宽度（通常用于平板或旋转屏幕场景）。  


**示例**：调整组件边距以适配安全区域  
```jsx
import { View, Text, StyleSheet } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function HomeScreen() {
  // 获取安全区域值
  const insets = useSafeAreaInsets();

  return (
    <View 
      style={[
        styles.container,
        {
          // 顶部留出安全区域（避免被刘海/状态栏遮挡）
          paddingTop: insets.top,
          // 底部留出安全区域（避免被 Home Indicator 遮挡）
          paddingBottom: insets.bottom,
        }
      ]}
    >
      <Text>内容不会被系统元素遮挡</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
});
```


#### 3. `SafeAreaView`
作用：内置的容器组件，自动为指定边缘应用安全区域的内边距（无需手动计算 `insets`）。  

**属性**：  
- `edges`：数组，指定需要适配安全区域的边缘（默认 `['top', 'right', 'bottom', 'left']`）；  
- 支持所有 `View` 的属性（如 `style`、`onLayout` 等）。  


**示例**：仅适配顶部和底部安全区域  
```jsx
import { SafeAreaView, Text } from 'react-native-safe-area-context';

function ProfileScreen() {
  return (
    <SafeAreaView 
      style={{ flex: 1, backgroundColor: '#fff' }}
      edges={['top', 'bottom']} // 只对顶部和底部应用安全区域
    >
      <Text>顶部和底部会自动留出安全区域</Text>
    </SafeAreaView>
  );
}
```


#### 4. `SafeAreaConsumer`（类组件兼容）
作用：用于类组件（或不支持 Hook 的场景），通过 render props 方式获取 `insets`。  

**示例**：  
```jsx
import { SafeAreaConsumer } from 'react-native-safe-area-context';
import { View, Text } from 'react-native';

class SettingsScreen extends React.Component {
  render() {
    return (
      <SafeAreaConsumer>
        {(insets) => (
          <View style={{ paddingTop: insets.top }}>
            <Text>类组件中使用安全区域</Text>
          </View>
        )}
      </SafeAreaConsumer>
    );
  }
}
```


### 四、常见场景与注意事项
1. **设备旋转适配**：当设备旋转（如横屏）时，`SafeAreaProvider` 会自动更新 `insets` 值，布局会实时适配。  

2. **嵌套使用**：`SafeAreaProvider` 可以嵌套（如在模态框中单独设置），子 Provider 会继承父 Provider 的数据，也可单独配置。  

3. **与导航库配合**：如果使用 `react-navigation`，建议将 `SafeAreaProvider` 放在导航容器外层，确保所有页面都能获取安全区域数据：  
   ```jsx
   <SafeAreaProvider>
     <NavigationContainer>{/* 导航页面 */}</NavigationContainer>
   </SafeAreaProvider>
   ```

4. **默认值问题**：如果未正确包裹 `SafeAreaProvider`，`insets` 会返回 `{ top: 0, bottom: 0, left: 0, right: 0 }`，可能导致布局错误。  


### 总结
`react-native-safe-area-context` 是处理安全区域适配的最佳实践，核心流程为：  
1. 用 `SafeAreaProvider` 包裹根组件；  
2. 通过 `useSafeAreaInsets`（推荐）或 `SafeAreaView` 灵活适配安全区域。  

通过它可以轻松解决异形屏、刘海屏等设备的布局兼容问题，确保内容展示在可见区域内。