# 智齿 Unity SDK 集成演示 Demo

## 项目说明

本工程是一个干净的 Unity 集成演示项目，已集成智齿 Unity SDK 插件，并复用了开发工程中的启动页样例。

需要注意：

- `SobotUnityIntegrationDemo` 展示的是“空 Unity 项目导入智齿 Unity SDK 后”的最终状态。
- Unity 的 `.unitypackage` 导入后会展开到项目目录中，不会在项目里保持为一个单独的包文件。
- 因此导入后看到 `Assets/Plugins/Sobot/Android`、`Assets/Plugins/Sobot/iOS`、iOS framework、Android bridge 等原生对接文件是正常的。
- 用户真正集成时，只需要在自己的空项目里导入 `SobotUnitySDK_latest.unitypackage`；导入完成后的目录结构就会和本 Demo 中的插件目录类似。

- Unity 版本：2021.3.45f2c1
- Demo 场景：`Assets/Scenes/SampleScene.unity`
- Demo 页面：`Assets/StartPage.cs`
- 插件目录：`Assets/Plugins/Sobot`
- 构建辅助脚本：`Assets/Editor`

## 打开方式

1. 打开 Unity Hub。
2. 选择 `Add project from disk`。
3. 选择本目录：`/Users/lizh/Documents/智齿项目文件/U3D/Zhichi/SobotUnityIntegrationDemo`。
4. 使用 Unity `2021.3.45f2c1` 打开。
5. 打开 `Assets/Scenes/SampleScene.unity`，运行后即可看到 Demo 按钮页面。

## Demo 参数位置

Demo 参数在 `Assets/StartPage.cs` 中：

- `APP_KEY`：替换为智齿管理平台分配的 `app_key`
- `PARTNER_ID`：替换为业务侧用户唯一标识
- `CreateDemoParams()`：统一维护初始化、聊天、帮助中心、留言、主动发送消息等参数

## Android 真机运行

1. `File > Build Settings` 切换到 Android。
2. 点击 Unity 菜单 `Sobot > Apply Android Build Settings`，自动设置 `IL2CPP + ARM64`。
3. 连接 Android 真机并开启 USB 调试。
4. `Build And Run`。

Android 依赖会在 Gradle 工程生成后由 `Assets/Editor/SobotPostProcessBuild.cs` 自动注入。

## iOS 真机运行

1. `File > Build Settings` 切换到 iOS。
2. 点击 `Build` 导出 Xcode 工程。
3. 使用 Xcode 打开导出的工程。
4. 配置 Apple Team、Bundle Identifier、证书和描述文件。
5. 连接 iPhone 真机运行。

iOS 构建后处理会自动加入 `UserNotifications.framework`，复制 `SobotKit.bundle`，并补充常用权限文案。
