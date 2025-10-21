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