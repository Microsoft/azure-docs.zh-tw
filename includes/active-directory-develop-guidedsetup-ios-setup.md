---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: ios
ms.workload: identity
ms.date: 09/19/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: 4ed4f7e15a21e1565031994bd377c15aebd535bc
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46466132"
---
## <a name="setting-up-your-ios-application"></a>設定您的 iOS 應用程式

本節提供逐步指示，說明如何建立新的專案來示範整合 iOS 應用程式 (Swift) 與使用 Microsoft 登入，以便它可以查詢需要權杖的 Web API。

> 想要改為下載此範例的 XCode 專案嗎？ [下載專案](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/archive/master.zip)並跳至[設定步驟](#register-your-application)，以在執行之前先設定程式碼範例。


## <a name="install-carthage-to-download-and-build-msal"></a>安裝 Carthage 以下載並建置 MSAL
MSAL 預覽期間會使用 Carthage 套件管理員 – 與 XCode 整合，同時維持 Microsoft 對程式庫進行變更的能力。

- 在[這裡](https://github.com/Carthage/Carthage/releases "Carthage 下載 URL") 下載並安裝最新版的 Carthage

## <a name="creating-your-application"></a>建立應用程式

1.  開啟 Xcode 並選取 `Create a new Xcode project`
2.  選取 `iOS`  >  `Single view Application`，然後按 [下一步]
3.  提供產品名稱，然後按 [下一步]
4.  選取用來建立應用程式的資料夾，然後按一下 [建立]

## <a name="build-the-msal-framework"></a>建置 MSAL 架構

請遵循以下指示以使用 Carthage，提取然後建置最新版本的 MSAL 程式庫：

1.  開啟 bash 終端機並移至應用程式的根資料夾
2.  複製下列項目並在 bash 終端機貼上，以建立 'Cartfile' 檔案：

```bash
echo "github \"AzureAD/microsoft-authentication-library-for-objc\" \"master\"" > Cartfile
```
<!-- Workaround for Docs conversion bug -->
<ol start="3">
<li>
複製並貼上下列項目。 此命令會將相依性擷取至 Carthage/Checkouts 資料夾，然後建置 MSAL 程式庫：
</li>
</ol>

```bash
carthage update
```

> 上述程序是用來下載並建置 Microsoft Authentication Library (MSAL)。 MSAL 會處理使用者權杖的取得、快取及重新整理作業，這些權杖是用來存取受 Azure Active Directory v2 保護的 API。

## <a name="add-the-msal-framework-to-your-application"></a>將 MSAL 架構新增至您的應用程式
1.  在 Xcode 中，開啟 [`General`] 索引標籤
2.  移至 [`Linked Frameworks and Libraries`] 區段，然後按一下 [`+`]
3.  選取 `Add other…`
4.  選取：`Carthage` > `Build` > `iOS` > `MSAL.framework`，然後按一下 [開啟]。 您應該會看到 `MSAL.framework` 新增至清單。
5.  移至 [`Build Phases`] 索引標籤，然後按一下 [`+`] 圖示，選擇 [`New Run Script Phase`]
6.  將下列內容新增至 [指令碼區域]：

```text
/usr/local/bin/carthage copy-frameworks
```

<!-- Workaround for Docs conversion bug -->
<ol start="7">
<li>
藉由按一下 [<code>+</code>]，將下列項目新增至 [<code>Input Files</code>]：
</li>
</ol>

```text
$(SRCROOT)/Carthage/Build/iOS/MSAL.framework
```

## <a name="creating-your-applications-ui"></a>建立應用程式的 UI
系統應會自動建立 Main.storyboard 檔案，作為專案範本的一部分。 請遵循下列指示以建立應用程式 UI：

1.  按住 Ctrl 鍵並按一下 `Main.storyboard` 以顯示內容功能表，然後按一下：`Open As` > `Source Code`
2.  使用下列程式碼來取代 `<scenes>` 節點：

```xml
 <scenes>
    <scene sceneID="tne-QT-ifu">
        <objects>
            <viewController id="BYZ-38-t0r" customClass="ViewController" customModule="MSALiOS" customModuleProvider="target" sceneMemberID="viewController">
                <layoutGuides>
                    <viewControllerLayoutGuide type="top" id="y3c-jy-aDJ"/>
                    <viewControllerLayoutGuide type="bottom" id="wfy-db-euE"/>
                </layoutGuides>
                <view key="view" contentMode="scaleToFill" id="8bC-Xf-vdC">
                    <rect key="frame" x="0.0" y="0.0" width="375" height="667"/>
                    <autoresizingMask key="autoresizingMask" widthSizable="YES" heightSizable="YES"/>
                    <subviews>
                        <label opaque="NO" userInteractionEnabled="NO" contentMode="left" horizontalHuggingPriority="251" verticalHuggingPriority="251" fixedFrame="YES" text="Microsoft Authentication Library" textAlignment="natural" lineBreakMode="tailTruncation" baselineAdjustment="alignBaselines" adjustsFontSizeToFit="NO" translatesAutoresizingMaskIntoConstraints="NO" id="ifd-fu-zjm">
                            <rect key="frame" x="64" y="28" width="246" height="21"/>
                            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
                            <fontDescription key="fontDescription" type="system" pointSize="17"/>
                            <nil key="textColor"/>
                            <nil key="highlightedColor"/>
                        </label>
                        <label opaque="NO" userInteractionEnabled="NO" contentMode="left" horizontalHuggingPriority="251" verticalHuggingPriority="251" fixedFrame="YES" text="Logging" textAlignment="natural" lineBreakMode="tailTruncation" baselineAdjustment="alignBaselines" adjustsFontSizeToFit="NO" translatesAutoresizingMaskIntoConstraints="NO" id="98g-dc-BPL">
                            <rect key="frame" x="16" y="277" width="62" height="21"/>
                            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
                            <fontDescription key="fontDescription" type="system" pointSize="17"/>
                            <nil key="textColor"/>
                            <nil key="highlightedColor"/>
                        </label>
                        <button opaque="NO" contentMode="scaleToFill" fixedFrame="YES" contentHorizontalAlignment="center" contentVerticalAlignment="center" buttonType="roundedRect" lineBreakMode="middleTruncation" translatesAutoresizingMaskIntoConstraints="NO" id="2rX-Vv-T1i">
                            <rect key="frame" x="87" y="100" width="200" height="30"/>
                            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
                            <state key="normal" title="Call Microsoft Graph API"/>
                            <connections>
                                <action selector="callGraphButton:" destination="BYZ-38-t0r" eventType="touchUpInside" id="Kx0-JL-Bv9"/>
                            </connections>
                        </button>
                        <textView clipsSubviews="YES" multipleTouchEnabled="YES" contentMode="scaleToFill" fixedFrame="YES" editable="NO" textAlignment="natural" selectable="NO" translatesAutoresizingMaskIntoConstraints="NO" id="qXW-2z-J7K">
                            <rect key="frame" x="16" y="324" width="343" height="291"/>
                            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
                            <color key="backgroundColor" white="1" alpha="1" colorSpace="calibratedWhite"/>
                            <accessibility key="accessibilityConfiguration">
                                <accessibilityTraits key="traits" updatesFrequently="YES"/>
                            </accessibility>
                            <fontDescription key="fontDescription" type="system" pointSize="14"/>
                            <textInputTraits key="textInputTraits" autocapitalizationType="sentences"/>
                        </textView>
                        <button opaque="NO" contentMode="scaleToFill" fixedFrame="YES" contentHorizontalAlignment="center" contentVerticalAlignment="center" buttonType="roundedRect" lineBreakMode="middleTruncation" translatesAutoresizingMaskIntoConstraints="NO" id="9u4-b8-vmX">
                            <rect key="frame" x="137" y="138" width="100" height="30"/>
                            <autoresizingMask key="autoresizingMask" flexibleMaxX="YES" flexibleMaxY="YES"/>
                            <state key="normal" title="Sign Out"/>
                            <connections>
                                <action selector="signoutButton:" destination="BYZ-38-t0r" eventType="touchUpInside" id="kZT-P8-0Zy"/>
                            </connections>
                        </button>
                    </subviews>
                    <color key="backgroundColor" red="1" green="1" blue="1" alpha="1" colorSpace="custom" customColorSpace="sRGB"/>
                </view>
                <connections>
                    <outlet property="loggingText" destination="qXW-2z-J7K" id="uqO-Yw-AsK"/>
                    <outlet property="signoutButton" destination="9u4-b8-vmX" id="OCh-qk-ldv"/>
                </connections>
            </viewController>
            <placeholder placeholderIdentifier="IBFirstResponder" id="dkx-z0-nzr" sceneMemberID="firstResponder"/>
        </objects>
        <point key="canvasLocation" x="140" y="137.18140929535232"/>
    </scene>
</scenes>
```
