# App内嵌H5屈光档案 JSBridge接口文档

## 文档版本历史

| 版本 | 日期       | 修改内容     | 修改人 |
| :--- | :--------- | :----------- | :----- |
| v1.0 | 2024-01-10 | 初始版本创建 | -      |

## 一、文档介绍

### 1.1 目的

本文档旨在规范App与内嵌H5页面（屈光档案）之间的JSBridge交互流程，指导原生App如何正确调用H5暴露的JavaScript方法，实现用户登录、主题色设置等功能。

### 1.2 适用范围

- 原生Android/iOS开发人员
- H5前端开发人员
- 测试人员

## 二、环境说明

### 2.1 H5页面信息

- **页面名称**：屈光档案
- **H5链接**：`http://localhost:8080/wechat/?route=jdf`
- **运行环境**：App内嵌WebView

### 2.2 通信方式

采用JSBridge模式，H5页面加载完成后向`window`对象注册全局函数，App通过`webview.evaluateJavascript`或类似机制调用这些函数。

## 三、JSBridge初始化流程

### 3.1 H5端初始化代码

javascript

```
// H5页面中的初始化逻辑
setupJSBridge() {
  // 1. 注册全局登录函数
  window.h5Login = this.pwdLogin
  
  // 2. 注册全局主题设置函数
  window.setThemeVariables = this.setThemeVariables
  
  // 3...
  
  console.log('H5方法已暴露，等待原生调用...')
}
```



### 3.2 初始化时序图

text

```
App WebView                    H5页面
    |                            |
    |---- 加载H5页面 ------------->|
    |                            |
    |                            |---- 执行setupJSBridge()
    |                            |---- 注册window.h5Login
    |                            |---- 注册window.setThemeVariables
    |                            |
    |<--- 页面加载完成 --------------|
    |                            |
    |---- 调用h5Login() ---------->|
    |---- 调用setThemeVariables() ->|
    |                            |
```



## 四、接口详情

### 4.1 登录接口 (h5Login)

#### 接口说明

App调用此方法实现H5页面的自动登录，传递手机号信息。

#### 方法名称

```
h5Login
```

#### 参数说明

| 参数名      | 类型   | 必填 | 说明                                 |
| :---------- | :----- | :--- | :----------------------------------- |
| phonenumber | string | 是   | 手机号码，需转换为JSON字符串格式传入 |

#### 调用示例

**正确示例（推荐）**：

javascript

```
// 将手机号作为字符串处理
const params = JSON.stringify({
  phonenumber: "19919910664"
});
h5Login(params);
```



**错误示例**：

javascript

```
// ❌ 直接传入对象（部分WebView可能不支持）
h5Login({phonenumber: "19919910664"});

// ❌ 手机号作为数字传入（可能丢失精度）
h5Login(JSON.stringify({phonenumber: 19919910664}));
```



#### Android调用示例

java

```
// Android端调用代码
String jsonParams = "{\"phonenumber\":\"19919910664\"}";
webView.evaluateJavascript("javascript:h5Login('" + jsonParams + "')", null);
```



#### iOS调用示例

objc

```
// iOS端调用代码
NSString *jsonParams = @"{\"phonenumber\":\"19919910664\"}";
[webView evaluateJavaScript:[NSString stringWithFormat:@"h5Login('%@')", jsonParams] completionHandler:nil];
```



### 4.2 主题色设置接口 (setThemeVariables)

#### 接口说明

App调用此方法动态设置H5页面的主题背景色。

#### 方法名称

```
setThemeVariables
```

#### 参数说明

| 参数名  | 类型   | 必填 | 说明                                         |
| :------ | :----- | :--- | :------------------------------------------- |
| bgColor | string | 是   | 背景色十六进制值，需转换为JSON字符串格式传入 |

#### 调用示例

**正确示例**：

javascript

```
// 将主题色参数转换为JSON字符串
const params = JSON.stringify({
  bgColor: "#4a76de"
});
setThemeVariables(params);
```



**错误示例**：

javascript

```
// ❌ 直接传入对象
setThemeVariables({bgColor: "#4a76de"});

// ❌ 参数格式错误
setThemeVariables("bgColor=#4a76de");
```



#### Android调用示例

java

```
// Android端调用代码
String jsonParams = "{\"bgColor\":\"#4a76de\"}";
webView.evaluateJavascript("javascript:setThemeVariables('" + jsonParams + "')", null);
```



#### iOS调用示例

objc

```
// iOS端调用代码
NSString *jsonParams = @"{\"bgColor\":\"#4a76de\"}";
[webView evaluateJavaScript:[NSString stringWithFormat:@"setThemeVariables('%@')", jsonParams] completionHandler:nil];
```



## 五、注意事项

### 5.1 参数格式要求

- **必须**将参数转换为JSON字符串格式传入
- 不能直接传入JavaScript对象
- 手机号必须作为字符串处理，避免数字精度问题

### 5.2 调用时机

- 建议在WebView页面加载完成（`onPageFinished`）后调用H5方法
- H5端会在初始化时注册全局函数，确保调用时函数已存在

### 5.3 错误处理

javascript

```
// H5端建议添加函数存在性检查
if (typeof window.h5Login === 'function') {
  // 安全调用
}
```



### 5.4 注意事项清单

- 所有参数必须经过`JSON.stringify`处理
- 手机号统一使用字符串类型
- 颜色值需包含#前缀（如`#4a76de`）
- 避免在页面未加载完成时调用
- 生产环境建议添加调用失败重试机制

## 六、常见问题

### Q1: 为什么参数必须为JSON字符串格式？

A: 某些低版本Android WebView在直接传递JavaScript对象时可能解析失败，使用JSON字符串格式可以保证最大兼容性。

### Q2: 手机号为什么要用字符串？

A: JavaScript中数字类型有精度限制（最大安全整数为2^53-1），11位手机号虽然未超出范围，但可能丢失前导零（如手机号以0开头的情况），因此统一使用字符串最安全。

### Q3: 如何确认H5方法调用成功？

A: H5端会在控制台输出日志，可以通过WebView的调试工具查看；同时建议H5端在方法执行后通过JSBridge回调App。

## 七、附录

### 7.1 JSON格式参考

json

```
// h5Login参数格式
{"phonenumber":"19919910664"}

// setThemeVariables参数格式
{"bgColor":"#4a76de"}
```