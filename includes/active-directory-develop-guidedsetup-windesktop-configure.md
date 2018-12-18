---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: fb521fc9d4927a953cdd66948101969dfc102de4
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843542"
---
## <a name="register-your-application"></a>註冊您的應用程式
您可以使用兩種方式之一來註冊您的應用程式。

### <a name="option-1-express-mode"></a>選項 1：快速模式
執行下列動作，即可快速註冊您的應用程式：
1. 移至 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/portal/register-app?appType=mobileAndDesktopApp&appTech=windowsDesktop&step=configure)。

2. 選取 [新增應用程式]。

3. 在 [應用程式名稱] 方塊中，輸入應用程式的名稱。

4. 確認已選取 [引導式設定] 核取方塊，然後選取 [建立]。

5. 依照指示取得應用程式識別碼，然後將它貼到您的程式碼中。

### <a name="option-2-advanced-mode"></a>選項 2：進階模式
若要註冊您的應用程式並將應用程式註冊資訊新增到您的解決方案，請執行下列作業：
1. 如果您尚未註冊您的應用程式，請移至 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/portal/register-app)。

2. 選取 [新增應用程式]。

3. 在 [應用程式名稱] 方塊中，輸入應用程式的名稱。 

4. 確認已清除 [引導式設定] 核取方塊，然後選取 [建立]。

5. 選取 [新增平台]，選取 [原生應用程式]，然後選取 [儲存]。

6. 在 [應用程式 ID] 方塊中，複製 GUID。

7. 移至 Visual Studio，開啟 App.xaml.cs 檔案，然後將 `your_client_id_here` 取代為您剛剛註冊及複製的應用程式 ID。

    ```csharp
    private static string ClientId = "your_application_id_here";
    ```
