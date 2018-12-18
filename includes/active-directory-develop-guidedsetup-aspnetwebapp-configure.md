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
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/04/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: f1dc23729f32a7a9535b887acf638cf5464c24bd
ms.sourcegitcommit: c851842d113a7078c378d78d94fea8ff5948c337
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/18/2018
ms.locfileid: "36206110"
---
## <a name="register-your-application"></a>註冊您的應用程式

若要註冊應用程式並將應用程式註冊資訊新增到解決方案，您有兩個選項可以選擇：

### <a name="option-1-express-mode"></a>選項 1：快速模式

執行下列動作，即可快速註冊您的應用程式：

1. 透過 [Microsoft 應用程式註冊入口網站 (英文)](https://apps.dev.microsoft.com/portal/register-app?appType=serverSideWebApp&appTech=aspNetWebAppOwin&step=configure) 註冊您的應用程式
2.  輸入應用程式的名稱和您的電子郵件
3.  確定已選取 [Guided Setup (引導式設定)] 選項
4.  依照指示將重新導向 URL 新增到您的應用程式

### <a name="option-2-advanced-mode"></a>選項 2：進階模式

若要註冊您的應用程式並將應用程式註冊資訊新增到您的解決方案，請執行下列作業：

1. 前往 [Microsoft 應用程式註冊入口網站 (英文)](https://apps.dev.microsoft.com/portal/register-app) 註冊應用程式
2. 輸入應用程式的名稱和您的電子郵件 
3. 確定已取消選取 [Guided Setup (引導式設定)] 選項
4. 按一下 `Add Platform`，然後選取 `Web`
5. 回到 Visual Studio，在「方案總管」中，選取專案並查看 [屬性] 視窗 (如果您沒有看到 [屬性] 視窗，請按 F4)
6. 將 [SSL 已啟用] 變更為 `True`
7. 在 Visual Studio 專案上以滑鼠右鍵按一下，然後選擇 [屬性] 和 [Web] 索引標籤。在 [伺服器] 區段中，將 [專案 URL] 變更為 SSL URL
8. 複製 SSL URL 並將這個 URL 新增到註冊入口網站的重新導向 URL 清單：<br/><br/>![專案屬性](media/active-directory-develop-guidedsetup-aspnetwebapp-configure/vsprojectproperties.png)<br />
9. 在位於根資料夾之 `web.config` 的 `configuration\appSettings` 區段下新增下列內容：

    ```xml
    <add key="ClientId" value="Enter_the_Application_Id_here" />
    <add key="redirectUri" value="Enter_the_Redirect_URL_here" />
    <add key="Tenant" value="common" />
    <add key="Authority" value="https://login.microsoftonline.com/{0}/v2.0" />
    ```

10. 用您剛剛註冊的應用程式識別碼取代 `ClientId`
11. 用您專案的 SSL URL 取代 `redirectUri`

