---
title: 設定單一頁面應用程式 |蔚藍
titleSuffix: Microsoft identity platform
description: '瞭解如何建立單一頁面應用程式 (應用程式的程式碼設定) '
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 5cbb576a7fcfb2daf492a149130aa7c99fe10ac5
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98753616"
---
# <a name="single-page-application-code-configuration"></a>單一頁面應用程式：程式碼設定

瞭解如何設定單一頁面應用程式的程式碼， (SPA) 。

## <a name="msal-libraries-for-spas-and-supported-authentication-flows"></a>適用于 Spa 的 MSAL 程式庫和支援的驗證流程

Microsoft 身分識別平臺提供下列適用于 JavaScript 的 Microsoft 驗證程式庫 ( # A0) ，藉由使用業界建議的安全性作法，以支援 PKCE 的隱含流程和授權碼流程：

| MSAL 程式庫 | 流程 | 描述 |
|--------------|------|-------------|
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (2.x) ](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) | 授權碼流程 (PKCE)  | 單純的 JavaScript 程式庫，可用於透過 JavaScript 或 SPA 架構（例如角、Vue.js 和 React.js）所建立的任何用戶端 web 應用程式。 |
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (1.x) ](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core) | 隱含流程 | 單純的 JavaScript 程式庫，可用於透過 JavaScript 或 SPA 架構（例如角、Vue.js 和 React.js）所建立的任何用戶端 web 應用程式。 |
| ![MSAL 角度](media/sample-v2-code/logo_angular.png) <br/> [MSAL 角度](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | 隱含流程 | 核心 MSAL.js 程式庫的包裝函式，可簡化透過角度架構建立的單一頁面應用程式中的使用。 |

## <a name="application-code-configuration"></a>應用程式程式碼設定

在 MSAL 程式庫中，應用程式註冊資訊會在程式庫初始化期間以設定的形式傳遞。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// Configuration object constructed.
const config = {
    auth: {
        clientId: 'your_client_id'
    }
};

// create UserAgentApplication instance
const userAgentApplication = new UserAgentApplication(config);
```

如需可設定選項的詳細資訊，請參閱 [使用 MSAL.js初始化應用程式 ](msal-js-initializing-client-applications.md)。

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
// App.module.ts
import { MsalModule } from '@azure/msal-angular';

@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id'
            }
        })
    ]
})

export class AppModule { }
```

---

## <a name="next-steps"></a>後續步驟

請移至本案例的下一篇文章，登 [入並登出](scenario-spa-sign-in.md)。
