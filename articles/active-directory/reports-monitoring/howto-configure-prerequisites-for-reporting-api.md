---
title: 存取 Azure Active Directory 報告 API 的必要條件 | Microsoft Docs
description: 了解存取 Azure AD 報告 API 的必要條件
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 05/07/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: 8d610dc74b7e2ef10295bc0a3407cf7c3d781b51
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42144194"
---
# <a name="prerequisites-to-access-the-azure-active-directory-reporting-api"></a>存取 Azure Active Directory 報告 API 的必要條件

[Azure Active Directory (Azure AD) 報告 API](https://msdn.microsoft.com/library/azure/ad/graph/howto/azure-ad-reports-and-events-preview) 透過一組以 REST 為基礎的 API 為您提供資料的程式設計方式存取。 您可以從各種程式設計語言和工具呼叫這些 API。

報告 API 會使用 [OAuth](https://msdn.microsoft.com/library/azure/dn645545.aspx) 授權存取 Web API。

若要準備存取報告 API，您必須︰

1. 指派角色
2. 註冊應用程式
3. 授與權限
4. 收集組態設定



## <a name="assign-roles"></a>指派角色

若要透過 API 來存取報告資料，您必須已具備下列其中一個指派的角色︰

- 安全性讀取者

- 安全性系統管理員

- 全域管理員




## <a name="register-an-application"></a>註冊應用程式

即使您使用指令碼來存取報告 API，也必須註冊應用程式。 註冊可讓您取得**應用程式識別碼**，您必須有此識別碼才能進行授權呼叫，以便讓程式碼獲得權杖。

若要設定您的目錄以存取 Azure AD 報告 API，您必須以 Azure 管理員帳戶登入 Azure 入口網站，而且該帳戶同時也是 Azure AD 租用戶**全域管理員**目錄角色的成員。

> [!IMPORTANT]
> 像這樣使用具有「管理員」權限的認證所執行的應用程式會非常強大，請務必確保應用程式的識別碼/密碼認證的安全性。
> 


**若要註冊 Azure Active Directory 應用程式：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，按一下 [Azure Active Directory]。
   
    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. 在 [Azure Active Directory] 頁面中，按一下 [應用程式註冊]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/02.png) 

3. 在 [應用程式註冊] 頁面中，按一下頂端工具列中的 [新增應用程式註冊]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/03.png)

4. 在 [建立] 頁面上，執行下列步驟：

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/04.png)

    a. 在 [名稱] 文字方塊中，輸入 `Reporting API application`。

    b. 對於 [應用程式類型]，選取 [Web 應用程式/API]。

    c. 在 [登入 URL] 文字方塊中，輸入 `https://localhost`。

    d. 按一下頁面底部的 [新增] 。 


## <a name="grant-permissions"></a>授與權限 

根據您想要存取的 API，您需要授與應用程式下列權限：  

| API | 權限 |
| --- | --- |
| Windows Azure Active Directory | 讀取目錄資料 |
| Microsoft Graph | 讀取所有稽核記錄資料 |


![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/36.png)


下節列出適用於這兩個 API 的步驟。 如果您不想存取其中一個 API，可以跳過相關步驟。
 

**若要對應用程式授與 API 使用權限：**

1. 在 [應用程式註冊] 頁面中，按一下應用程式清單中的 [報告 API 應用程式]。

2. 在 [報告 API 應用程式] 頁面中，按一下頂端工具列中的 [設定]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/05.png)

3. 在 [設定] 頁面中，按一下 [必要權限]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/06.png)

4. 在 [必要權限] 頁面中，按一下 [API] 清單中的 [Windows Azure Active Directory]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/07.png)

5. 在 [啟用存取] 頁面中，選取 [讀取目錄資料]，並取消選取 [登入及讀取使用者設定檔]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/08.png)

6. 在頂端工具列中，按一下 [儲存]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/15.png)

7. 在 [必要權限] 頁面頂端的工具列中，按一下 [新增]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/32.png)

8. 在 [新增 API 存取權] 頁面中，按一下 [選取 API]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/31.png)

9. 在 [選取 API] 頁面中，按一下 [Microsoft Graph]，然後按一下 [選取]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/33.png)

10. 在 [啟用存取] 頁面中，選取 [讀取所有稽核記錄資料]，然後按一下 [選取]。  

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/34.png)


11. 在 [新增 API 存取權] 頁面中，按一下 [完成]。  

12. 在 [必要權限] 頁面頂端的工具列中， 按一下 [授與權限]，然後按一下 [是]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/17.png)


## <a name="gather-configuration-settings"></a>收集組態設定 

本節說明如何從您的目錄取得下列設定︰

- 網域名稱
- 用戶端識別碼
- 用戶端密碼

您在設定報告 API 的呼叫時需要這些值。 

### <a name="get-your-domain-name"></a>取得您的網域名稱

**若要取得網域名稱：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，按一下 [Azure Active Directory]。
   
    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. 在 [Azure Active Directory] 頁面中，按一下 [自訂網域名稱]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/09.png) 

3. 從網域清單中複製網域名稱。


### <a name="get-your-applications-client-id"></a>取得應用程式的用戶端識別碼

**若要取得應用程式的用戶端識別碼：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，按一下 [Azure Active Directory]。
   
    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. 在 [應用程式註冊] 頁面中，按一下應用程式清單中的 [報告 API 應用程式]。

3. 在 [報告 API 應用程式] 頁面中，按一下 [應用程式識別碼] 中的 [按一下以複製]。

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/11.png) 



### <a name="get-your-applications-client-secret"></a>取得應用程式的用戶端祕密
若要取得應用程式的用戶端密碼，您需要建立新的金鑰並在儲存新金鑰時儲存其值，因為稍後不可能再擷取此值。

**若要取得應用程式的用戶端祕密：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，按一下 [Azure Active Directory]。
   
    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/01.png) 

2. 在 [應用程式註冊] 頁面中，按一下應用程式清單中的 [報告 API 應用程式]。


3. 在 [報告 API 應用程式] 頁面中，按一下頂端工具列中的 [設定]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/05.png)

4. 在 [設定] 頁面中，按一下 [APIR 存取] 區段中的 [金鑰]。 

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/12.png)


5. 在 [金鑰] 頁面中，執行下列步驟︰

    ![註冊應用程式](./media/howto-configure-prerequisites-for-reporting-api/14.png)

    a. 在 [描述] 文字方塊中，輸入 `Reporting API`。

    b. 選取 [2 年後] 來作為 [到期]。

    c. 按一下 [檔案] 。

    d. 複製金鑰值。


## <a name="next-steps"></a>後續步驟

* [使用 Azure Active Directory 報告 API 搭配憑證來取得資料](tutorial-access-api-with-certificates.md)
* [取得報告 API 的第一印象](concept-reporting-api.md)
* [稽核 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/directoryaudit) 
* [登入活動報告 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/signin)
