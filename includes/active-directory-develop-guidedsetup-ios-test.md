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
ms.openlocfilehash: eb56712610266d974d2a646412700da5806e9c30
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46466131"
---
## <a name="test-querying-the-microsoft-graph-api-from-your-ios-application"></a>從 iOS 應用程式測試查詢 Microsoft Graph API

若要在模擬器中執行程式碼，請按命令 + **R**。

![在模擬器中測試您的應用程式](media/active-directory-develop-guidedsetup-ios-test/iostestscreenshot.png)

當您準備好進行測試時，請選取**呼叫 Microsoft Graph API**。 出現提示時，請輸入您的使用者名稱和密碼。

### <a name="provide-consent-for-application-access"></a>同意應用程式存取
您第一次登入應用程式時，系統會提示您同意允許應用程式存取您的設定檔，並將您登入：

![同意應用程式存取](media/active-directory-develop-guidedsetup-ios-test/iosconsentscreen.png)

### <a name="view-application-results"></a>檢視應用程式結果
登入之後，您應該會在 [記錄] 區段中看到由 Microsoft Graph API 傳回的使用者設定檔資訊。 

<!--start-collapse-->
### <a name="more-information-about-scopes-and-delegated-permissions"></a>與範圍和委派的權限有關的詳細資訊

Microsoft Graph API 需要 user.read 範圍才能讀取使用者的設定檔。 根據預設，在註冊入口網站上註冊的每個應用程式中，都會自動新增此範圍。 Microsoft Graph 的其他 API 與您後端伺服器的自訂 API 一樣，需要其他範圍。 Microsoft Graph API 需要 Calendars.Read 範圍才能列出使用者的行事曆。

為了在應用程式內容中存取使用者的行事曆，請將 **Calendars.Read** 委派權限新增至應用程式註冊資訊。 接著，將 **Calendars.Read** 範圍新增至 **acquireTokenSilent** 呼叫。 

>[!NOTE]
>系統可能會在您增加範圍數目時，提示使用者同意其他事項。

<!--end-collapse-->

[!INCLUDE [Help and support](./active-directory-develop-help-support-include.md)]
