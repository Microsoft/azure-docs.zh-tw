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
ms.openlocfilehash: 3cbcae9c02abaf11ded31304f97abdd74f905c6e
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843631"
---
## <a name="test-your-code"></a>測試您的程式碼

### <a name="test-with-node"></a>使用 Node 測試
如果您不是使用 Visual Studio，請確定您的 web 伺服器已啟動。
1. 將伺服器設定為接聽以您 **index.html** 檔案位置為基礎的 TCP 通訊埠。 對於 Node，請從應用程式資料夾在命令提示字元中執行命令，以啟動網頁伺服器來接聽此連接埠：

    ```bash
    npm install
    node server.js
    ```
1. 開啟瀏覽器並輸入 http://<span></span>localhost:30662 or http://<span></span>localhost:{port}，其中的 port 是網頁伺服器正在接聽的連接埠。 您應會看到 index.html 檔案的內容和 [登入] 按鈕。

<p/><!-- -->

### <a name="test-with-visual-studio"></a>使用 Visual Studio 進行測試
如果您使用 Visual Studio，請務必選取專案解決方案並且按 **F5** 來執行專案。 瀏覽器會開啟至 http://<span></span>localhost:{port} 位置，而且您會看到 [登入] 按鈕。


## <a name="test-your-application"></a>測試您的應用程式

在瀏覽器載入您的 index.html 檔案之後，請按一下 [登入]。 系統會提示您使用 Microsoft Azure Active Directory (Azure AD) v2.0 端點登入：

![登入您的 JavaScript SPA 帳戶](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspascreenshot1.png)


### <a name="provide-consent-for-application-access"></a>同意應用程式存取

您第一次登入應用程式時，系統會提示您同意允許應用程式存取您的設定檔，並將您登入：

![同意應用程式存取](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

### <a name="view-application-results"></a>檢視應用程式結果
登入之後，您應該會在頁面上看到以 Microsoft Graph API 回應傳回的使用者設定檔資訊。

![Microsoft Graph API 呼叫的預期結果](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptsparesults.png)

<!--start-collapse-->
### <a name="more-information-about-scopes-and-delegated-permissions"></a>與範圍和委派的權限有關的詳細資訊

Microsoft Graph API 需要 `user.read` 範圍以讀取使用者的設定檔。 根據預設，在註冊入口網站上註冊的每個應用程式中，都會自動新增此範圍。 Microsoft Graph 的其他 API 與您後端伺服器的自訂 API 一樣，需要其他範圍。 例如，Microsoft Graph API 需要 `Calendars.Read` 範圍才能列出使用者的行事曆。

為了在應用程式內容中存取使用者的行事曆，請將 `Calendars.Read` 委派權限新增至應用程式註冊資訊。 接著，將 `Calendars.Read` 範圍新增至 `acquireTokenSilent` 呼叫。

>[!NOTE]
>系統可能會在您增加範圍數目時，提示使用者同意其他事項。

如果後端 API 不需要範圍 (不建議)，您可以在呼叫中使用 `clientId` 作為範圍以取得權杖。

<!--end-collapse-->

[!INCLUDE [Help and support](./active-directory-develop-help-support-include.md)]
