[return](../index)

你想了解在 React Native 开发中，`AsyncStorage` 和 `MMKV` 这两种本地键值存储方案的核心区别，以便根据项目需求选择合适的存储方式。

---

## 一、核心概念与基础对比

先从本质上理解两者：
- **AsyncStorage**：React Native 官方（现已移至独立包）提供的异步持久化存储方案，底层封装了 iOS 的 `NSUserDefaults` 和 Android 的 `SharedPreferences`，仅支持字符串存储，需手动序列化/反序列化。
- **MMKV**：腾讯开源的高性能键值存储库，基于 `mmap` 内存映射技术实现，跨平台且性能远超传统存储方案，React Native 可通过第三方库 `react-native-mmkv` 集成。

### 关键维度对比表
```
| 特性                | AsyncStorage                          | MMKV                                  |
|---------------------|---------------------------------------|---------------------------------------|
| **性能**            | 低（异步IO+序列化开销，数据量越大越慢） | 极高（内存级读写，大数据仍保持高性能） |
| **存储原理**        | iOS→plist文件/Android→XML文件，异步IO  | 内存映射文件（mmap），同步写入        |
| **数据类型**        | 仅支持字符串（需JSON序列化/反序列化）  | 原生支持字符串、数字、布尔、对象、二进制 |
| **异步/同步**       | 仅异步（所有方法返回Promise）          | 默认同步，也支持异步API               |
| **安全性**          | 明文存储，无加密（需自行实现）         | 原生支持AES加密，可配置密钥           |
| **持久化可靠性**    | 有数据丢失风险（如应用崩溃）           | 写入即时持久化，几乎无丢失风险        |
| **大小限制**        | 无硬限制，但性能随数据量骤降          | 无实际限制（仅受设备存储影响）        |
| **平台支持**        | iOS/Android                           | iOS/Android/Windows/macOS             |
| **易用性**          | 需手动处理序列化/解析错误             | 自动处理类型转换，API更简洁           |
```


## 二、使用示例（完整可运行）

### 1. AsyncStorage 使用示例

#### 前置条件

先安装依赖：
```bash
npm install @react-native-async-storage/async-storage --save
# iOS 需额外执行
cd ios && pod install && cd ..
```

#### 核心代码

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

// 存储数据（必须转字符串）
const storeData = async (key, value) => {
  try {
    const jsonValue = JSON.stringify(value); // 手动序列化
    await AsyncStorage.setItem(key, jsonValue);
    console.log('存储成功');
  } catch (e) {
    console.error('AsyncStorage存储失败：', e);
  }
};

// 读取数据（需解析，处理null）
const getData = async (key) => {
  try {
    const jsonValue = await AsyncStorage.getItem(key);
    return jsonValue != null ? JSON.parse(jsonValue) : null;
  } catch (e) {
    console.error('AsyncStorage读取失败：', e);
    return null;
  }
};

// 调用示例
storeData('user', { name: '张三', age: 20 });
getData('user').then(res => console.log('读取结果：', res)); // { name: '张三', age: 20 }
```

### 2. MMKV 使用示例

#### 前置条件

安装依赖：
```bash
npm install react-native-mmkv --save
# iOS 需额外执行
cd ios && pod install && cd ..
```

#### 核心代码
```javascript
import { MMKV } from 'react-native-mmkv';

// 初始化实例（可选加密）
const storage = new MMKV({
  id: 'my-app-storage', // 多实例唯一标识
  encryptionKey: 'my-secret-key-123', // 加密密钥（可选）
});

// 存储数据（无需手动序列化）
storage.set('token', 'abc123456'); // 字符串
storage.set('isLogin', true); // 布尔
storage.set('age', 25); // 数字
storage.set('user', { name: '李四', age: 22 }); // 对象

// 读取数据（自动类型转换）
const token = storage.getString('token'); // 'abc123456'
const isLogin = storage.getBoolean('isLogin'); // true
const user = storage.getObject('user'); // { name: '李四', age: 22 }

console.log('MMKV读取结果：', token, isLogin, user);
```

---

## 三、适用场景建议

- **选 AsyncStorage**：
  - 项目仅需存储少量简单数据（如用户token、基础配置）；
  - 对性能无高要求，且不想引入第三方依赖；
  - 团队对官方方案的兼容性要求更高。

- **选 MMKV**：
  - 需要存储大量数据（如离线缓存、复杂配置）；
  - 对读写性能有高要求（如高频存储/读取场景）；
  - 需存储敏感数据（如用户密码、支付信息）；
  - 跨平台（如同时支持iOS/Android/Windows）需求。

---

### 总结

1. **性能核心**：MMKV 基于内存映射技术，读写速度远超依赖文件IO的 AsyncStorage；
2. **易用性**：MMKV 无需手动处理 JSON 序列化，支持多数据类型，开发效率更高；
3. **选型原则**：简单小数据场景可复用 AsyncStorage，高性能/加密/大数据场景优先 MMKV。