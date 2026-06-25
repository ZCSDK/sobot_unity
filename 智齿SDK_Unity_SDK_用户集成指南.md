# 智齿 SDK Unity 插件用户集成指南

本文档面向需要在 Unity 项目中接入智齿客服能力的开发者，说明 Unity 插件包的导入方式、平台配置、主要功能和常用 API。

适用平台：Unity Android / iOS  
Unity 版本：`2021.3.45f2c1`

示例工程：`SobotUnityIntegrationDemo`

## 1. 接入前准备

### 1.1 获取 app_key

使用管理员账号登录智齿管理后台，在「桌面网站客服 - APP - 添加 APP」创建应用，获取 `app_key`。后续初始化 SDK、打开客服、获取未读消息等能力都需要使用该值。

### 1.2 插件包

导入文件：

```text
SobotClientSDK.unitypackage
```

导入后 Unity 项目中会出现以下主要目录：

```text
Assets/Plugins/Sobot
Assets/Plugins/Sobot/Android
Assets/Plugins/Sobot/iOS
Assets/Editor
```

核心 C# 入口类：

```csharp
using SobotChatClient;

ZCSobotApi
SobotParams
```

## 2. 导入 Unity 插件

1. 打开 Unity 项目。
2. 选择 `Assets > Import Package > Custom Package...`。
3. 选择 `SobotClientSDK.unitypackage` 并全部导入。
4. 确认存在：

```text
Assets/Plugins/Sobot/ZCSobotApi.cs
Assets/Plugins/Sobot/Android/SobotBridge.java
Assets/Plugins/Sobot/iOS/SobotUnityBridge.mm
Assets/Editor/SobotPostProcessBuild.cs
```

业务脚本中引用：

```csharp
using SobotChatClient;
```

## 3. 平台配置

### 3.1 Android

建议配置：

```text
Scripting Backend: IL2CPP
Target Architectures: ARM64
Min SDK Version: 22 或以上
```

插件提供菜单：

```text
Sobot > Apply Android Build Settings
```

执行后会自动设置 Android 构建所需的 `IL2CPP + ARM64`。

Android Gradle 工程生成后，插件会自动注入智齿 SDK 依赖和必要 Maven 仓库。客户项目如使用自定义 Gradle 模板，需要确认仓库至少包含：

```gradle
google()
mavenCentral()
```

### 3.2 iOS

建议配置：

```text
iOS Target: iOS 12.0 或以上
Scripting Backend: IL2CPP
```

iOS 构建后，插件会自动处理：

- 添加 `UserNotifications.framework`
- 拷贝 `SobotKit.bundle`
- 将 `SobotKit.bundle` 加入 Xcode 工程 Copy Bundle Resources
- 写入相机、麦克风、相册权限文案

导出 Xcode 工程后，需要在 Xcode 中配置：

- Bundle Identifier
- Team
- Signing Certificate
- Provisioning Profile

## 4. 快速接入

### 4.1 创建参数

`app_key` 必填，`partnerid` 建议必传并保持稳定，用于标识当前业务用户。

```csharp
private SobotParams CreateSobotParams()
{
    return new SobotParams
    {
        api_host = "https://www.sobot.com",
        app_key = "替换为你的 app_key",
        partnerid = "user_unique_id",
        user_nick = "Unity 用户",
        user_name = "Unity 用户",
        user_tels = "18510002000",
        user_emails = "test@example.com"
    };
}
```

### 4.2 初始化 SDK

建议在打开客服页面前先初始化，并在初始化成功后再调用其它功能。

```csharp
ZCSobotApi.initSobotSDK(CreateSobotParams(), (success, message) =>
{
    Debug.Log($"Sobot init: {success}, {message}");
});
```

### 4.3 打开客服聊天

```csharp
ZCSobotApi.openSobotChat(CreateSobotParams());
```

### 4.4 打开帮助中心

```csharp
ZCSobotApi.openSobotHelpCenter(CreateSobotParams());
```

### 4.5 打开留言页面

```csharp
ZCSobotApi.openSobotLeave(CreateSobotParams());
```

### 4.6 获取未读消息数

```csharp
ZCSobotApi.getUnReadMessage(CreateSobotParams(), (unRead, offline, unAck) =>
{
    Debug.Log($"unRead={unRead}, offline={offline}, unAck={unAck}");
});
```

### 4.7 关闭会话

```csharp
ZCSobotApi.closeSobotChat(CreateSobotParams());
```

iOS 如需关闭推送，可设置：

```csharp
new SobotParams
{
    closePush = true
};
```

## 5. 事件监听

### 5.1 链接点击监听

```csharp
ZCSobotApi.setMessageLinkClick(url =>
{
    Debug.Log("Sobot link: " + url);

    // true 表示业务侧已处理，SDK 不再默认处理
    // false 表示交给 SDK 默认处理
    return false;
});
```

### 5.2 功能点击监听

```csharp
ZCSobotApi.setFunctionClickListener(type =>
{
    Debug.Log("Sobot function type: " + type);
});
```

常用事件类型：

| type | 含义 |
| --- | --- |
| 1 | 留言返回 |
| 2 | 聊天返回 |
| 3 | 帮助中心返回 |
| 4 | 电商消息中心返回 |
| 5 | 点击电话按钮 |

## 6. 主动发送消息

主动发送消息需要先初始化 SDK。Android 端建议在客服聊天页已经打开并加载完成后再调用，否则可能没有聊天页接收消息。

### 6.1 发送商品卡片

必填字段：`goodsTitle`、`goodsLink`

```csharp
var param = CreateSobotParams();
param.goodsTitle = "商品标题";
param.goodsDesc = "商品描述";
param.goodsLabel = "¥99";
param.goodsLink = "https://www.example.com/goods/10001";
param.goodsImage = "https://www.example.com/goods.png";

ZCSobotApi.sendProductInfo(param, (success, code, message) =>
{
    Debug.Log($"send product: {success}, {code}, {message}");
});
```

### 6.2 发送订单卡片

必填字段：`orderCode`

```csharp
var param = CreateSobotParams();
param.orderCode = "ORDER_10001";
param.orderStatus = "0";
param.statusCustom = "已发货";
param.goodsCount = "2";
param.totalFee = "199";
param.orderUrl = "https://www.example.com/order/10001";

ZCSobotApi.sendOrderGoodsInfo(param, (success, code, message) =>
{
    Debug.Log($"send order: {success}, {code}, {message}");
});
```

### 6.3 发送定位消息

必填字段：`lat`、`lng`

```csharp
var param = CreateSobotParams();
param.lat = "39.908823";
param.lng = "116.397470";
param.localLabel = "北京市东城区东长安街";
param.localName = "天安门";

ZCSobotApi.sendLocation(param, (success, code, message) =>
{
    Debug.Log($"send location: {success}, {code}, {message}");
});
```

### 6.4 发送自定义卡片

必填字段：`custom_card`

`custom_card` 为 JSON 字符串，具体字段请按业务卡片结构生成，可参考 Demo 中 `Assets/StartPage.cs` 的 `CreateDemoCustomCardJson()`。

```csharp
var param = CreateSobotParams();
param.custom_card = "{\"cardId\":\"10001\",\"cardDesc\":\"自定义卡片内容\"}";

ZCSobotApi.sendCustomCardToChat(param, (success, code, message) =>
{
    Debug.Log($"send custom card: {success}, {code}, {message}");
});
```

## 7. 常用参数说明

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `app_key` | string | 是 | 智齿后台分配的 App Key |
| `partnerid` | string | 建议 | 业务侧用户唯一 ID |
| `api_host` | string | 否 | API 域名，默认可使用 `https://www.sobot.com` |
| `user_nick` | string | 否 | 用户昵称 |
| `user_name` | string | 否 | 用户姓名 |
| `user_tels` | string | 否 | 用户电话 |
| `user_emails` | string | 否 | 用户邮箱 |
| `groupid` | string | 否 | 技能组 ID |
| `group_name` | string | 否 | 技能组名称 |
| `choose_adminid` | string | 否 | 指定客服 ID |
| `tran_flag` | string | 否 | 指定客服转接策略 |
| `content` | string | 否 | 自动发送文本 |
| `good_msg_type` | int | 否 | 自动发送模式：1 机器人，2 人工，3 全部 |
| `params` | string | 否 | 自定义扩展字段 JSON 字符串 |
| `customer_fields` | string | 否 | 客户字段 JSON 字符串 |
| `multi_params` | string | 否 | 多轮会话参数 JSON 字符串 |
| `absolute_language` | string | 否 | 指定语言，例如 `en_lproj` |
| `showNotification` | bool | 否 | 通知开关 |
| `deviceToken` | string | iOS 可选 | iOS 远程推送 token |

注意：C# 中 `params` 是关键字，代码中字段名写作 `@params`：

```csharp
new SobotParams
{
    @params = "{\"level\":\"vip\"}"
};
```

SDK 内部会转换为 JSON key：`params`。

## 8. Demo 参考

示例工程：

```text
SobotUnityIntegrationDemo
```

重点文件：

```text
Assets/StartPage.cs
Assets/Plugins/Sobot/ZCSobotApi.cs
```

Demo 中包含：

- SDK 初始化
- 打开客服聊天
- 打开帮助中心
- 打开留言页面
- 获取未读消息数
- 注册链接点击监听
- 注册功能点击监听
- 主动发送商品卡片、订单卡片、定位消息、自定义卡片

## 9. 常见问题

### 9.1 导入后找不到 ZCSobotApi

确认已导入：

```text
Assets/Plugins/Sobot/ZCSobotApi.cs
```

业务脚本中需要添加：

```csharp
using SobotChatClient;
```

### 9.2 Android 真机安装失败或架构不匹配

确认 Android 构建设置为：

```text
Scripting Backend: IL2CPP
Target Architectures: ARM64
```

可使用菜单：

```text
Sobot > Apply Android Build Settings
```

### 9.3 iOS 构建后资源缺失

确认 Xcode 工程中 `SobotKit.bundle` 已加入 Copy Bundle Resources。插件的 `SobotPostProcessBuild.cs` 会自动处理该步骤。

如项目迁移路径后出现 IL2CPP/Tundra 旧路径错误，可在 Unity 菜单中清理：

```text
Sobot > Clean Local Build Cache > iOS IL2CPP Cache
```

### 9.4 Android 主动发送返回失败

请确认：

- 已初始化成功
- 已打开客服聊天页
- 聊天页已经加载完成

Android 端主动发送依赖聊天页接收消息，聊天页未打开时可能发送失败。

## 10. 集成检查清单

- 已导入 `SobotClientSDK.unitypackage`
- 已引用 `using SobotChatClient;`
- 已替换真实 `app_key`
- `partnerid` 使用真实且稳定的用户唯一 ID
- Android 已设置 `IL2CPP + ARM64`
- iOS 已配置 Xcode Signing
- 初始化成功后再打开客服页面
- 主动发送消息前确认聊天页已打开
- 真机测试通过初始化、打开客服、帮助中心、留言、未读数等基础功能
