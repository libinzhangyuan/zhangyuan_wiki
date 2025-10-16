[return](/react-native/index)

React Native 开发中常用的命令可以分为项目创建、运行调试、打包发布等几类，以下是最常用的核心命令整理：


### 一、环境与工具安装
1. **安装 React Native CLI（命令行工具）**  
   用于创建和管理 React Native 项目（全局安装）：  
   ```bash
   npm install -g react-native-cli   # 使用 npm
   # 或
   yarn global add react-native-cli  # 使用 yarn
   ```


### 二、项目创建与初始化
1. **创建新项目**  
   ```bash
   # 基本用法（默认最新稳定版）
   react-native init MyProjectName

   # 指定版本创建
   react-native init MyProjectName --version X.XX.X

   # 使用 TypeScript 模板创建
   react-native init MyProjectName --template react-native-template-typescript
   ```

   > 注意：现在更推荐使用 `npx` 避免全局安装依赖冲突：  
   > `npx react-native init MyProjectName`


### 三、运行与调试项目
1. **启动 Metro 打包服务器**  
   Metro 是 React Native 的 JavaScript 打包工具，运行项目前需启动（通常会自动启动，也可手动启动）：  
   ```bash
   npm start   # 或 yarn start
   # 等价于
   react-native start
   ```

2. **运行 iOS 应用**  
   ```bash
   # 自动启动默认模拟器
   react-native run-ios

   # 指定模拟器（如 iPhone 14）
   react-native run-ios --simulator "iPhone 14"

   # 运行到连接的真机
   react-native run-ios --device "设备名称"
   ```
   > 注意：首次运行前需进入 `ios` 目录安装 CocoaPods 依赖：  
   > `cd ios && pod install && cd ..`

3. **运行 Android 应用**  
   ```bash
   # 自动检测连接的模拟器/真机并运行
   react-native run-android

   # 指定设备（通过 adb devices 查看设备 ID）
   react-native run-android --deviceId 设备ID
   ```
   > 前提：需配置 `ANDROID_HOME` 环境变量，且有运行中的 Android 模拟器或连接的真机。

4. **查看日志**  
   ```bash
   # iOS 日志
   react-native log-ios

   # Android 日志
   react-native log-android
   ```

5. **开发者菜单**  
   运行时可通过快捷键打开开发者菜单（用于调试、刷新等）：  
   - iOS 模拟器：`Cmd + D`  
   - Android 模拟器：`Cmd + M`（Mac）或 `Ctrl + M`（Windows/Linux）  
   - 真机：摇晃设备


### 四、打包与发布
1. **Android 打包（生成 APK）**  
   ```bash
   # 进入 android 目录
   cd android

   # 生成调试版 APK（用于测试）
   ./gradlew assembleDebug   # Mac/Linux
   # 或
   gradlew.bat assembleDebug  # Windows

   # 生成正式版签名 APK（用于发布）
   ./gradlew assembleRelease  # Mac/Linux
   # 或
   gradlew.bat assembleRelease # Windows
   ```
   生成的 APK 位于 `android/app/build/outputs/apk/` 目录。

2. **iOS 打包**  
   需通过 Xcode 操作：  
   - 打开 `ios/MyProjectName.xcworkspace`  
   - 选择目标设备（如 `Any iOS Device`）  
   - 菜单栏选择 `Product -> Archive` 进行打包，后续可上传至 App Store。


### 五、其他常用命令
1. **升级 React Native 版本**  
   ```bash
   react-native upgrade  # 升级到最新版（需谨慎，可能有兼容问题）
   ```
   建议参考 [官方升级文档](https://reactnative.dev/docs/upgrading) 操作。

2. **清除缓存（解决依赖或打包问题）**  
   ```bash
   # 清除 Metro 缓存
   npm start -- --reset-cache  # 或 yarn start --reset-cache

   # 清除 Android 构建缓存
   cd android && ./gradlew clean && cd ..
   ```


### 补充：Expo 相关命令（若使用 Expo 开发）
如果项目基于 Expo 构建（更简化的开发流程），常用命令：  
```bash
# 安装 Expo CLI
npm install -g expo-cli

# 创建 Expo 项目
expo init MyExpoProject

# 启动项目
expo start  # 或 npm start

# 运行到 iOS/Android
expo run:ios
expo run:android
```


以上命令覆盖了 React Native 开发的主要流程，具体使用时可根据项目需求（原生开发/Expo 开发）选择对应命令。