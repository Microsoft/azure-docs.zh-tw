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
ms.openlocfilehash: ff465891075bbb16774b7ecad306de222c2f2a7c
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46473861"
---
## <a name="add-the-applications-registration-information-to-your-app"></a>將應用程式的註冊資訊新增到您的應用程式

在這個步驟中，您需要將應用程式識別碼新增到您的專案：

1.  在 `ViewController.swift` 中，將開頭為 '`let kClientID`' 的那一行取代為：
```swift
let kClientID = "[Enter the application Id here]"
```
<!-- Workaround for Docs conversion bug -->
<ol start="2">
<li>
按住 Ctrl 鍵並按一下 <code>Info.plist</code> 以顯示內容功能表，然後按一下：<code>Open As</code> > <code>Source Code</code>
</li>
<li>
在 <code>dict</code> 根節點下，新增下列項目：
</li>
</ol>

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>msal[Enter the application Id here]</string>
        </array>
    </dict>
</array>
```
