[return](/react-native/index)

在React Native中，当自定义组件需要包含交互能力时（如点击、长按等），可以结合 `Pressable` 组件和 `PressableProps` 来实现更规范的属性传递。`PressableProps` 是 `Pressable` 组件的属性类型定义，包含了 `onPress`、`onLongPress`、`style` 等交互相关的属性，让自定义组件能够像原生组件一样接收标准的触摸事件属性。


### 示例：基于 PressableProps 的自定义组件
下面通过一个带交互的自定义按钮组件，演示如何利用 `PressableProps` 传递参数：


#### 1. 定义带 PressableProps 的自定义组件（使用 TypeScript 更规范）
```tsx
// CustomButton.tsx
import React from 'react';
import { Pressable, Text, StyleProp, ViewStyle, PressableProps } from 'react-native';

// 自定义组件的 props：结合自定义属性和 PressableProps
type CustomButtonProps = {
  // 自定义属性：按钮文本
  text: string;
  // 自定义属性：文本样式（可选）
  textStyle?: StyleProp<ViewStyle>;
  // 继承 Pressable 的所有属性（如 onPress、onLongPress、style 等）
} & PressableProps;

const CustomButton: React.FC<CustomButtonProps> = (props) => {
  // 从 props 中解构出自定义属性，剩余属性通过 ...rest 传递给 Pressable
  const { text, textStyle, ...rest } = props;

  return (
    // 将 PressableProps 传递给 Pressable 组件
    <Pressable
      {...rest} // 传递所有 Pressable 相关属性（如 onPress、style 等）
      style={({ pressed }) => [
        // 默认样式
        {
          padding: 12,
          borderRadius: 8,
          backgroundColor: pressed ? '#666' : '#333', // 按压状态变化
          alignItems: 'center',
        },
        // 支持父组件传递的 style（覆盖默认样式）
        props.style,
      ]}
    >
      <Text
        style={[
          { color: 'white', fontSize: 16 }, // 默认文本样式
          textStyle, // 父组件传递的文本样式
        ]}
      >
        {text}
      </Text>
    </Pressable>
  );
};

export default CustomButton;
```


#### 2. 使用自定义组件（传递 PressableProps 和自定义属性）
父组件使用时，可以像使用原生 `Pressable` 一样传递 `onPress`、`onLongPress` 等属性，同时传递自定义的 `text` 等属性：

```tsx
// ParentComponent.tsx
import React from 'react';
import { View, Alert } from 'react-native';
import CustomButton from './CustomButton';

const ParentComponent = () => {
  return (
    <View style={{ padding: 20, gap: 10 }}>
      {/* 基础用法：传递自定义 text 和 onPress */}
      <CustomButton
        text="点击我"
        onPress={() => Alert.alert('提示', '按钮被点击了')}
      />

      {/* 传递更多 PressableProps 和自定义样式 */}
      <CustomButton
        text="长按试试"
        textStyle={{ fontWeight: 'bold' }} // 自定义文本样式
        style={{ backgroundColor: 'blue' }} // 覆盖按钮默认背景色
        onLongPress={() => Alert.alert('提示', '按钮被长按了')} // 长按事件
        delayLongPress={500} // 长按延迟（Pressable 原生属性）
      />
    </View>
  );
};

export default ParentComponent;
```


### 核心说明：
1. **类型组合**：通过 `& PressableProps` 让自定义组件的 props 继承 `Pressable` 的所有属性，实现类型安全。
   
2. **属性透传**：使用 `...rest` 将 `PressableProps` 透传给内部的 `Pressable` 组件，确保所有原生交互属性（如 `onPress`、`disabled`、`hitSlop` 等）都能生效。

3. **样式覆盖**：在 `style` 中通过数组形式，先定义默认样式，再合并父组件传递的 `style`，实现样式的可定制性。

4. **状态获取**：`Pressable` 的 `style` 支持函数形式（`({ pressed }) => Style`），可以根据按压状态动态调整样式，这也是 `Pressable` 相比 `TouchableOpacity` 更灵活的地方。


### 若使用 JavaScript（非 TypeScript）：
可以省略类型定义，但仍可通过 `prop-types` 进行类型检查：

```jsx
// CustomButton.js
import React from 'react';
import { Pressable, Text } from 'react-native';
import PropTypes from 'prop-types';

const CustomButton = (props) => {
  const { text, textStyle, ...rest } = props;
  return (
    <Pressable {...rest} style={({ pressed }) => [/* 样式 */, props.style]}>
      <Text style={[{ color: 'white' }, textStyle]}>{text}</Text>
    </Pressable>
  );
};

// 类型检查（结合 Pressable 的属性）
CustomButton.propTypes = {
  text: PropTypes.string.isRequired,
  textStyle: PropTypes.object,
  // 透传 Pressable 的属性（可根据需要指定）
  onPress: PropTypes.func,
  onLongPress: PropTypes.func,
  style: PropTypes.oneOfType([PropTypes.object, PropTypes.array]),
};

export default CustomButton;
```


通过这种方式，自定义组件既能保持自身的业务逻辑（如文本展示），又能完整支持 `Pressable` 的所有交互能力，使组件更通用、更符合 React Native 的设计规范。

<br>
<br>
<br>
<br>
<br>
<br>
<br>

在 React/React Native 中，`ComponentProps` 是一个类型工具（来自 `react` 或 `react-native`），用于**获取任意组件的 props 类型**，让自定义组件能够轻松继承其他组件的所有属性（包括原生组件或第三方组件）。这在封装组件时非常有用，既能保留原组件的所有功能，又能添加自定义逻辑。


### 核心用法
`ComponentProps` 的语法为：  
`ComponentProps<typeof 组件名>`  
它会返回该组件的完整 props 类型，便于自定义组件继承。


### 示例 1：基于原生组件封装（如 Text）
假设我们要封装一个 `CustomText` 组件，它需要支持原生 `Text` 组件的所有属性（如 `style`、`numberOfLines` 等），同时添加自定义属性（如 `isHighlighted` 高亮状态）。

```tsx
// CustomText.tsx
import React from 'react';
import { Text, StyleSheet, ComponentProps } from 'react-native';

// 1. 获取 Text 组件的 props 类型
type TextProps = ComponentProps<typeof Text>;

// 2. 自定义组件的 props：继承 Text 的所有属性 + 自定义属性
type CustomTextProps = TextProps & {
  isHighlighted?: boolean; // 自定义：是否高亮
};

// 3. 实现组件，透传所有 TextProps
const CustomText: React.FC<CustomTextProps> = (props) => {
  // 从 props 中解构出自定义属性，剩余属性透传给原生 Text
  const { isHighlighted, ...textProps } = props;

  return (
    <Text
      {...textProps} // 透传所有原生 Text 属性
      style={[
        styles.default,
        isHighlighted && styles.highlighted, // 自定义高亮样式
        props.style, // 允许父组件覆盖样式
      ]}
    />
  );
};

const styles = StyleSheet.create({
  default: {
    fontSize: 16,
    color: '#333',
  },
  highlighted: {
    color: 'red',
    fontWeight: 'bold',
  },
});

export default CustomText;
```


### 示例 2：使用自定义组件
父组件使用 `CustomText` 时，既能传递原生 `Text` 的所有属性，也能传递自定义的 `isHighlighted`：

```tsx
// Parent.tsx
import React from 'react';
import { View } from 'react-native';
import CustomText from './CustomText';

const Parent = () => {
  return (
    <View style={{ padding: 20 }}>
      {/* 仅使用原生 Text 属性 */}
      <CustomText style={{ fontSize: 18 }}>普通文本</CustomText>

      {/* 结合自定义属性和原生属性 */}
      <CustomText
        isHighlighted={true}
        numberOfLines={1} // 原生 Text 属性：最多显示1行
        ellipsizeMode="tail" // 原生 Text 属性：超出部分省略
      >
        这是一段需要高亮且自动省略的长文本...
      </CustomText>
    </View>
  );
};
```


### 示例 3：继承第三方组件的 Props
如果封装第三方组件（如 `react-native-paper` 的 `Button`），同样可以用 `ComponentProps` 继承其属性：

```tsx
// CustomPaperButton.tsx
import React from 'react';
import { Button } from 'react-native-paper';
import { ComponentProps } from 'react-native';

// 获取第三方 Button 的 props 类型
type PaperButtonProps = ComponentProps<typeof Button>;

// 自定义 props：继承第三方 Button 属性 + 自定义属性
type CustomPaperButtonProps = PaperButtonProps & {
  isCritical?: boolean; // 自定义：是否为危险操作按钮
};

const CustomPaperButton: React.FC<CustomPaperButtonProps> = (props) => {
  const { isCritical, ...buttonProps } = props;

  return (
    <Button
      {...buttonProps}
      mode={isCritical ? 'destructive' : buttonProps.mode} // 自定义危险模式
    />
  );
};
```


### 关键说明
1. **类型安全**：  
   在 TypeScript 中，`ComponentProps` 会自动推导组件的 props 类型，确保传递的属性符合原组件要求（如类型错误时编译报错）。

2. **属性透传**：  
   通过 `...rest` 语法将继承的 props 透传给内部组件，确保原组件的所有功能（如事件、样式、属性）都能正常使用。

3. **优先级处理**：  
   自定义逻辑可以覆盖原组件的默认行为（如示例1中 `isHighlighted` 优先于默认样式，父组件传递的 `style` 又覆盖自定义样式）。

4. **JavaScript 中使用**：  
   即使不使用 TypeScript，也可以通过 `...props` 透传属性，只是缺少类型提示：
   ```jsx
   const CustomText = (props) => {
     const { isHighlighted, ...textProps } = props;
     return <Text {...textProps} style={[styles.default, isHighlighted && styles.highlighted]} />;
   };
   ```


### 与 `PressableProps` 的区别
- `PressableProps` 是特定于 `Pressable` 组件的 props 类型（固定集合）。  
- `ComponentProps<typeof Pressable>` 与 `PressableProps` 效果一致，但 `ComponentProps` 更通用，适用于**任何组件**（包括自定义组件）。

例如：  
```tsx
// 两种方式等价（针对 Pressable）
type Props1 = PressableProps;
type Props2 = ComponentProps<typeof Pressable>;
```


通过 `ComponentProps`，可以让自定义组件在保留原组件功能的基础上扩展新能力，同时保证类型一致性，是封装复用组件的最佳实践之一。