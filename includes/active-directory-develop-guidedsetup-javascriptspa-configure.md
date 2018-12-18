---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: nacanuma
ms.custom: include file
ms.openlocfilehash: 038ea48bedeb31416627f99b38ebb083846747e4
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843454"
---
## <a name="register-your-application"></a>註冊您的應用程式

有多種方式可建立應用程式，請選取其中一個：

### <a name="option-1-register-your-application-express-mode"></a>選項 1：註冊您的應用程式 (快速模式)
現在您需要在「Microsoft 應用程式註冊入口網站」註冊應用程式：

1.  透過 [Microsoft 應用程式註冊入口網站 (英文)](https://apps.dev.microsoft.com/portal/register-app?appType=singlePageApp&appTech=javascriptSpa&step=configure) 註冊您的應用程式。
2.  輸入應用程式的名稱和您的電子郵件。
3.  確定已選取 [引導式設定] 選項。
4.  依照指示取得應用程式識別碼並貼到您的程式碼中。

### <a name="option-2-register-your-application-advanced-mode"></a>選項 2：註冊您的應用程式 (進階模式)

1. 前往 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/portal/register-app)註冊應用程式。
2. 輸入應用程式的名稱和您的電子郵件。
3. 確定已取消選取 [引導式設定] 選項。
4.  按一下 [新增平台]，然後選取 [Web]。
5. 以您的網頁伺服器作為基礎，新增對應到應用程式 URL 的 [重新導向 URL]。 如需有關如何設定及取得 Visual Studio 和 Node 中重新導向 URL 的相關指示，請參閱下列各節。
6. 選取 [ **儲存**]。

> #### <a name="setting-redirect-url-for-node"></a>設定 Node 的重新導向 URL
> 對於 Node.js，您可以在 server.js 檔案中設定網頁伺服器連接埠。 此教學課程會使用連接埠 30662 作為參考，但是可以隨意使用任何其他可用的連接埠。 在任何情況下，請遵循下列指示以在應用程式註冊資訊中設定重新導向 URL：<br/>
> - 切換回應用程式註冊入口網站，並將 `http://localhost:30662/` 設定為 `Redirect URL`，或如果您是使用自訂 TCP 通訊埠 (其中[通訊埠] 是自訂 TCP 通訊埠號碼)，請使用 `http://localhost:[port]/`，然後按一下 [儲存]

<p/>

> #### <a name="visual-studio-instructions-for-obtaining-the-redirect-url"></a>取得重新導向 URL 的 Visual Studio 指示
> 請遵循下列步驟以取得重新導向 URL：
> 1.    在 [方案總管] 中，選取專案並查看 [屬性] 視窗。 如果您沒有看到 [屬性] 視窗，請按 **F4**。
> 2.    將此值從 **URL** 複製到剪貼簿：<br/> ![專案屬性](media/active-directory-develop-guidedsetup-javascriptspa-configure/vs-project-properties-screenshot.png)<br />
> 3.    切換回「應用程式註冊入口網站」，並貼上此值作為 [重新導向 URL]，然後按一下 [儲存]


#### <a name="configure-your-javascript-spa"></a>設定您的 JavaScript SPA

1.  在專案設定期間建立的 `index.html` 檔案中，新增應用程式註冊資訊。 在 `index.html` 檔案主體頂端的 `<script></script>` 標記內，新增下列程式碼：

```javascript
var applicationConfig = {
    clientID: "[Enter the application Id here]",
    graphScopes: ["user.read"],
    graphEndpoint: "https://graph.microsoft.com/v1.0/me"
};
```
<ol start="3">
<li>
用您剛剛註冊的應用程式識別碼取代 <code>Enter the application Id here</code>。
</li>
</ol>
