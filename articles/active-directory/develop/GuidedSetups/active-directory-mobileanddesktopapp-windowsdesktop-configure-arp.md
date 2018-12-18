---
title: Azure AD v2 Windows Desktop 快速入門 - 設定 | Microsoft Docs
description: Windows Desktop .NET (XAML) 應用程式可取得存取權杖，以及呼叫受 Azure Active Directory v2 端點保護之 API 的方法。
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/09/2017
ms.author: andret
ms.custom: aaddev
ms.openlocfilehash: c42f40252733f0c4fd7fdbabb49714ea537acc70
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38473956"
---
## <a name="add-the-applications-registration-information-to-your-app"></a>將應用程式的註冊資訊新增到您的應用程式
在這個步驟中，您需要將應用程式識別碼新增到您的專案。

1.  開啟 `App.xaml.cs` 並將包含 `ClientId` 的那一行取代為：

```csharp
private static string ClientId = "[Enter the application Id here]";
```

### <a name="what-is-next"></a>下一步

[!INCLUDE [Test and Validate](..\..\..\..\includes\active-directory-develop-guidedsetup-windesktop-test.md)]
