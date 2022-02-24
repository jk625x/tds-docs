---
title: 正版验证开发指南
sidebar_label: 开发指南
sidebar_position: 2
---

import {Conditional} from '/src/docComponents/conditional';
import MultiLang from '/src/docComponents/MultiLang';
import CodeBlock from '@theme/CodeBlock';
import sdkVersions from '/src/docComponents/sdkVersions';

正版验证适用于在 TapTap 上架的付费下载游戏。游戏集成 TapSDK 的正版验证之后，当玩家第一次启动游戏时（包含卸载后再次安装），SDK 会前往 TapTap 查询玩家是否购买游戏，如已购买，则可正常进入游戏，如查询到未购买，则会引导购买。

## SDK 获取

可以在 [下载页](/tap-download) 获得 SDK，添加以下依赖：

<MultiLang>

<>

SDK 可以**通过 Unity Package Manger 导入或手动导入**，二者任选其一。

1. 下载 SDK 后即可手动导入。
2. 如果选择 UPM 导入，可以在项目的 `Packages/manifest.json` 文件中添加：

<CodeBlock className="json">
{`"dependencies":{
    // 公共库
    "com.taptap.tds.common":"https://github.com/TapTap/TapCommon-Unity.git#${sdkVersions.taptap.unity}",
    // 付费购买
    "com.taptap.tds.dlc": "https://github.com/TapTap/TapLicense-Unity.git#${sdkVersions.taptap.unity}",
}`}
</CodeBlock>

</>

<>

将 SDK 包导入到项目 `project/app/libs` 目录下。打开项目的 `project/app/build.gradle` 文件，添加：

<CodeBlock className="groovy">
{`repositories{
    flatDir {  
        dirs 'libs'  
    }  
}  
dependencies {
    implementation name:'TapCommon_${sdkVersions.taptap.android}', ext:'aar'
    implementation name:'TapLicense_${sdkVersions.taptap.android}', ext:'aar'
}`}
</CodeBlock>

</>

<>

```objc
// 暂不支持
```

</>

</MultiLang>

## 设置授权回调

<MultiLang>

```cs
// 需要引入 license 库
using TapTap.License;

// 默认情况下 SDK 会弹出不可由玩家手动取消的弹窗来避免未授权玩家进入游戏，如果需要回调来触发流程，请添加如下代码
TapLicense.SetLicencesCallback(ITapLicenseCallback callback);

public interface ITapLicenseCallback
{
    // 授权成功回调
    void OnLicenseSuccess();
}

```

```java
// 默认情况下 SDK 会弹出不可由玩家手动取消的弹窗来避免未授权玩家进入游戏，如果需要回调来触发流程，请添加如下代码
TapLicenseHelper.setLicenseCallback(new TapLicenseCallback() {
    @Override
    public void onLicenseSuccess() {
        // 授权成功回调
    }
});
```

```objc
// 暂不支持
```

</MultiLang>

## 检查是否付费

<MultiLang>

```cs
TapLicense.Check();
```

```java
TapLicenseHelper.check(Activity activity);
```

```objc
// 暂不支持
```

</MultiLang>

## 测试

为了保证上线后，正版验证服务能够正常生效，**请务必按照以下说明完成自测。**

#### 1. 上传 APK

前往开发者中心选择需要测试的游戏，依次转入**游戏服务 > 生态服务 > 正版验证 > 游戏配置 > 包名**。

上传需要测试的 APK 至开发者中心，等待审核通过。

#### 2. 添加测试用户

前往**开发者中心 > 游戏服务 > 生态服务 > 正版验证 > 游戏配置 > 测试用户管理**，填写测试用户的 TapTap ID 。

#### 3. 售卖设置

在**开发者中心 > 商店 > 售卖设置**中为游戏设置测试价格，建议测试价格设置为 0.01 元，提交并等待审核通过。

#### 4. 开始测试

打开 TapTap 客户端，使用上面填写的测试用户账号登录，从游戏商店页面开始正版验证的测试。

测试用户可在 TapTap 购买这个付费游戏，购买后才能顺利下载并进入游戏。

如果测试用户未购买，通过其他途径安装游戏 APK，进入游戏后会弹出不可由玩家手动取消的弹窗，显示「游戏未激活，请前往 TapTap 购买此游戏」。

## 正式售卖

完成上面的测试环节后，游戏可以开始售卖。按照以下步骤操作：

#### 1. 完善应用信息

前往开发者中心，按照[物料要求](/store/store-material/)填写应用信息，等待审核通过。

#### 2. 设置售卖价格

前往**开发者中心 > 商店 > 售卖设置**，开启售卖开关，设置游戏正常售卖金额，提交审核，并同步对接的 TapTap 运营相关信息。

#### 3. 正式上线

所有流程都确保顺利后，游戏可<Conditional region='cn'>[正式上线](/store/store-release/)</Conditional><Conditional region='global'>[正式上线](/store/store-publish-game)</Conditional>.

## 常见问题

### Android 11 或更高版本无法拉起 TapTap 客户端

Android 11（API level 30）之后加强了隐私保护策略，引入了大量变更和限制，其中一个重要变更——[软件包可见性](https://developer.android.com/about/versions/11/privacy/package-visibility)，将会导致第三方应用无法拉起 TapTap 客户端，从而影响 TapTap 相关功能的正常使用，包括但不限于更新唤起 TapTap、购买验证等功能。

**方案一：**

编译时将 `targetSdkVersion` 改为 29（目前设置成 >= 30 会触发该问题）。

**方案二：**

1. 将 gradle build tools 改为 4.1.0+：

    ```java
    classpath 'com.android.tools.build:gradle:4.1.0'
    ```

2. 在 AndroidManifest.xml 里添加如下内容：

    ```xml
    <queries>
        <package android:name="com.taptap" />
        <package android:name="com.taptap.pad" />
        <package android:name="com.taptap.global" />
    </queries>
    ```