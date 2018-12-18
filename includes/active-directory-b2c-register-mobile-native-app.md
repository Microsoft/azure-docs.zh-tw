---
title: 包含檔案
description: 包含檔案
services: active-directory-b2c
author: davidmu1
ms.service: active-directory-b2c
ms.topic: include
ms.date: 04/09/2018
ms.author: davidmu
ms.custom: include file
ms.openlocfilehash: 8363d023e89c77aabc0d123f19264c9a0758a656
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38740456"
---
[!INCLUDE [active-directory-b2c-portal-add-application](active-directory-b2c-portal-add-application.md)]

若要註冊您的行動或原生應用程式，請使用資料表中所指定的設定。

![新行動或原生應用程式的範例註冊設定](./media/active-directory-b2c-register-mobile-native-app/b2c-new-mobile-native-app-settings.png)

| 設定      | 範例值  | 說明                                        |
| ------------ | ------- | -------------------------------------------------- |
| **名稱** | Contoso B2C 應用程式 | 輸入應用程式的 [名稱]，此名稱可為取用者說明您的應用程式。 |
| **原生用戶端** | yes | 針對行動或原生應用程式選取 [是]。 |
| **自訂重新導向 URI** | `com.onmicrosoft.contoso.appname://redirect/path` | 輸入具有自訂配置的重新導向 URI。 務必選擇[良好的重新導向 URI](../articles/active-directory-b2c/active-directory-b2c-app-registration.md)，但不包含特殊字元 (例如底線)。 |

按一下 [建立]  以註冊您的應用程式。

新註冊的應用程式會顯示在 B2C 租用戶的應用程式清單中。 從清單中選取行動或原生應用程式。 應用程式的 [屬性] 窗格隨即顯示。

![應用程式屬性](./media/active-directory-b2c-register-mobile-native-app/b2c-mobile-native-app-properties.png)

請記下全域唯一的 [應用程式用戶端識別碼]。 您可在您的應用程式程式碼中使用此識別碼。

如果您的原生應用程式呼叫 Azure AD B2C 所保護的 Web API，請執行下列步驟：
   1. 前往 [金鑰] 刀鋒視窗，然後按一下 [產生金鑰] 按鈕，以建立應用程式祕密。 請記下 [應用程式金鑰] 值。 您可以使用此值，作為應用程式程式碼中的應用程式祕密。
   2. 按一下 [API 存取][新增] 並選取您的 Web API 和範圍 (權限)。

> [!NOTE]
> **應用程式密鑰** 是重要的安全性認證，應該適當地加以保護。
> 
