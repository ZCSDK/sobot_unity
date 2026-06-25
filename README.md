
# Sobot SDK Unity Plugin Integration Guide

This guide is intended for developers who need to integrate Sobot customer service features into a Unity project. It explains how to import the Unity plugin package, configure Android and iOS builds, and use the main APIs.

Demo project: `SobotUnityIntegrationDemo`

## 1. Before You Start

### 1.1 Get the app_key

Log in to the Sobot admin console with an administrator account. Create an app under `Desktop Website Customer Service > APP > Add APP`, then obtain the `app_key`.

The `app_key` is required for SDK initialization, opening the customer service page, querying unread messages, and other SDK features.

### 1.2 Plugin Package

Import this package:

```text
SobotClientSDK.unitypackage
```

After importing, the following main directories will appear in your Unity project:

```text
Assets/Plugins/Sobot
Assets/Plugins/Sobot/Android
Assets/Plugins/Sobot/iOS
Assets/Editor
```

Core C# entry points:

```csharp
using SobotChatClient;

ZCSobotApi
SobotParams
```

## 2. Import the Unity Plugin

1. Open your Unity project.
2. Select `Assets > Import Package > Custom Package...`.
3. Select `SobotClientSDK.unitypackage` and import all files.
4. Make sure the following files exist:

```text
Assets/Plugins/Sobot/ZCSobotApi.cs
Assets/Plugins/Sobot/Android/SobotBridge.java
Assets/Plugins/Sobot/iOS/SobotUnityBridge.mm
Assets/Editor/SobotPostProcessBuild.cs
```

Add the namespace in your business script:

```csharp
using SobotChatClient;
```

## 3. Platform Configuration

### 3.1 Android

Recommended settings:

```text
Scripting Backend: IL2CPP
Target Architectures: ARM64
Min SDK Version: 22 or later
```

The plugin provides this Unity menu:

```text
Sobot > Apply Android Build Settings
```

Running this menu item automatically applies the required Android build settings: `IL2CPP + ARM64`.

After Unity generates the Android Gradle project, the plugin automatically injects the ZhiChi SDK dependencies and required Maven repositories. If your project uses custom Gradle templates, make sure the repositories include at least:

```gradle
google()
mavenCentral()
```

### 3.2 iOS

Recommended settings:

```text
iOS Target: iOS 12.0 or later
Scripting Backend: IL2CPP
```

After the iOS build is generated, the plugin automatically:

- Adds `UserNotifications.framework`
- Copies `SobotKit.bundle`
- Adds `SobotKit.bundle` to Copy Bundle Resources in the Xcode project
- Adds permission descriptions for camera, microphone, and photo library

After exporting the Xcode project, configure the following in Xcode:

- Bundle Identifier
- Team
- Signing Certificate
- Provisioning Profile

## 4. Quick Start

### 4.1 Create Parameters

`app_key` is required. `partnerid` is strongly recommended and should be stable, because it identifies the current business user.

```csharp
private SobotParams CreateSobotParams()
{
    return new SobotParams
    {
        api_host = "https://www.sobot.com",
        app_key = "replace_with_your_app_key",
        partnerid = "user_unique_id",
        user_nick = "Unity User",
        user_name = "Unity User",
        user_tels = "18510002000",
        user_emails = "test@example.com"
    };
}
```

### 4.2 Initialize the SDK

Initialize the SDK before opening any customer service page. Call other SDK features after initialization succeeds.

```csharp
ZCSobotApi.initSobotSDK(CreateSobotParams(), (success, message) =>
{
    Debug.Log($"Sobot init: {success}, {message}");
});
```

### 4.3 Open Customer Service Chat

```csharp
ZCSobotApi.openSobotChat(CreateSobotParams());
```

### 4.4 Open Help Center

```csharp
ZCSobotApi.openSobotHelpCenter(CreateSobotParams());
```

### 4.5 Open Leave Message Page

```csharp
ZCSobotApi.openSobotLeave(CreateSobotParams());
```

### 4.6 Query Unread Message Count

```csharp
ZCSobotApi.getUnReadMessage(CreateSobotParams(), (unRead, offline, unAck) =>
{
    Debug.Log($"unRead={unRead}, offline={offline}, unAck={unAck}");
});
```

### 4.7 Close Session

```csharp
ZCSobotApi.closeSobotChat(CreateSobotParams());
```

On iOS, set `closePush` if you also need to close push notification handling:

```csharp
new SobotParams
{
    closePush = true
};
```

## 5. Event Listeners

### 5.1 Link Click Listener

```csharp
ZCSobotApi.setMessageLinkClick(url =>
{
    Debug.Log("Sobot link: " + url);

    // true: handled by your app, SDK will not process it by default
    // false: let the SDK process it by default
    return false;
});
```

### 5.2 Function Click Listener

```csharp
ZCSobotApi.setFunctionClickListener(type =>
{
    Debug.Log("Sobot function type: " + type);
});
```

Common event types:

| type | Description |
| --- | --- |
| 1 | Leave message page closed |
| 2 | Chat page closed |
| 3 | Help center closed |
| 4 | E-commerce message center closed |
| 5 | Phone button clicked |

## 6. Proactively Send Messages

The SDK must be initialized before proactively sending messages. On Android, it is recommended to call these APIs only after the customer service chat page has been opened and fully loaded; otherwise, there may be no chat page available to receive the message.

### 6.1 Send Product Card

Required fields: `goodsTitle`, `goodsLink`

```csharp
var param = CreateSobotParams();
param.goodsTitle = "Product Title";
param.goodsDesc = "Product Description";
param.goodsLabel = "$99";
param.goodsLink = "https://www.example.com/goods/10001";
param.goodsImage = "https://www.example.com/goods.png";

ZCSobotApi.sendProductInfo(param, (success, code, message) =>
{
    Debug.Log($"send product: {success}, {code}, {message}");
});
```

### 6.2 Send Order Card

Required field: `orderCode`

```csharp
var param = CreateSobotParams();
param.orderCode = "ORDER_10001";
param.orderStatus = "0";
param.statusCustom = "Shipped";
param.goodsCount = "2";
param.totalFee = "199";
param.orderUrl = "https://www.example.com/order/10001";

ZCSobotApi.sendOrderGoodsInfo(param, (success, code, message) =>
{
    Debug.Log($"send order: {success}, {code}, {message}");
});
```

### 6.3 Send Location Message

Required fields: `lat`, `lng`

```csharp
var param = CreateSobotParams();
param.lat = "39.908823";
param.lng = "116.397470";
param.localLabel = "East Chang'an Avenue, Dongcheng District, Beijing";
param.localName = "Tiananmen";

ZCSobotApi.sendLocation(param, (success, code, message) =>
{
    Debug.Log($"send location: {success}, {code}, {message}");
});
```

### 6.4 Send Custom Card

Required field: `custom_card`

`custom_card` is a JSON string. Generate it according to your business card structure. You can also refer to `CreateDemoCustomCardJson()` in `Assets/StartPage.cs` of the demo project.

```csharp
var param = CreateSobotParams();
param.custom_card = "{\"cardId\":\"10001\",\"cardDesc\":\"Custom card content\"}";

ZCSobotApi.sendCustomCardToChat(param, (success, code, message) =>
{
    Debug.Log($"send custom card: {success}, {code}, {message}");
});
```

## 7. Common Parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `app_key` | string | Yes | App Key assigned by the ZhiChi admin console |
| `partnerid` | string | Recommended | Stable unique user ID from your business system |
| `api_host` | string | No | API host, usually `https://www.sobot.com` |
| `user_nick` | string | No | User nickname |
| `user_name` | string | No | User name |
| `user_tels` | string | No | User phone number |
| `user_emails` | string | No | User email |
| `groupid` | string | No | Skill group ID |
| `group_name` | string | No | Skill group name |
| `choose_adminid` | string | No | Assigned agent ID |
| `tran_flag` | string | No | Agent transfer policy |
| `content` | string | No | Text to send automatically |
| `good_msg_type` | int | No | Auto-send target: 1 robot, 2 human agent, 3 both |
| `params` | string | No | Custom extension fields as a JSON string |
| `customer_fields` | string | No | Customer fields as a JSON string |
| `multi_params` | string | No | Multi-round conversation parameters as a JSON string |
| `absolute_language` | string | No | Language code, for example `en_lproj` |
| `showNotification` | bool | No | Notification switch |
| `deviceToken` | string | iOS optional | iOS remote push device token |

Note: `params` is a C# keyword, so the C# field name is `@params`:

```csharp
new SobotParams
{
    @params = "{\"level\":\"vip\"}"
};
```

The SDK converts it to the JSON key `params` internally.

## 8. Demo Reference

Demo project:

```text
SobotUnityIntegrationDemo
```

Important files:

```text
Assets/StartPage.cs
Assets/Plugins/Sobot/ZCSobotApi.cs
```

The demo includes:

- SDK initialization
- Opening customer service chat
- Opening help center
- Opening leave message page
- Querying unread message count
- Registering link click listener
- Registering function click listener
- Proactively sending product cards, order cards, location messages, and custom cards

## 9. FAQ

### 9.1 ZCSobotApi cannot be found after import

Make sure this file exists:

```text
Assets/Plugins/Sobot/ZCSobotApi.cs
```

Add the namespace in your business script:

```csharp
using SobotChatClient;
```

### 9.2 Android device installation fails or architecture does not match

Make sure Android build settings are:

```text
Scripting Backend: IL2CPP
Target Architectures: ARM64
```

You can also use:

```text
Sobot > Apply Android Build Settings
```

### 9.3 Resources are missing after iOS build

Make sure `SobotKit.bundle` is included in Copy Bundle Resources in the Xcode project. The plugin handles this automatically through `SobotPostProcessBuild.cs`.

If an IL2CPP/Tundra old-path error appears after moving or copying the Unity project, clean the iOS IL2CPP cache from the Unity menu:

```text
Sobot > Clean Local Build Cache > iOS IL2CPP Cache
```

### 9.4 Proactive sending fails on Android

Check the following:

- SDK initialization succeeded
- The customer service chat page has been opened
- The chat page has finished loading

On Android, proactive sending relies on the chat page receiving the message. If the chat page is not open, sending may fail.

## 10. Integration Checklist

- `SobotClientSDK.unitypackage` has been imported
- `using SobotChatClient;` has been added
- The real `app_key` has been configured
- `partnerid` is a real and stable unique user ID
- Android is configured as `IL2CPP + ARM64`
- iOS Xcode Signing has been configured
- Customer service pages are opened only after SDK initialization succeeds
- The chat page is open before proactively sending messages
- Initialization, customer service chat, help center, leave message, and unread message query have been tested on a real device
