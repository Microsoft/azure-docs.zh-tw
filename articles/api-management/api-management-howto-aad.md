---
title: 使用 Azure Active Directory 來授權開發人員帳戶 - Azure API 管理 | Microsoft Docs
description: 了解如何在 API 管理中使用 Azure Active Directory 來授權使用者。
services: api-management
documentationcenter: API Management
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/16/2018
ms.author: apimpm
ms.openlocfilehash: 4c1696fc373975eb9857c40366829fbe6a535911
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391873"
---
# <a name="authorize-developer-accounts-by-using-azure-active-directory-in-azure-api-management"></a>在 Azure API 管理中使用 Azure Active Directory 來授權開發人員帳戶

本文說明如何讓使用者能夠從 Azure Active Directory (Azure AD) 來存取開發人員入口網站。 本指南也說明如何新增包含 Azure AD 使用者的外部群組來管理使用者。

> [!NOTE]
> 僅[「開發人員」、「標準」與「進階」](https://azure.microsoft.com/pricing/details/api-management/)層提供 Azure AD 整合。

## <a name="prerequisites"></a>必要條件

- 完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
- 匯入並發佈 Azure API 管理執行個體。 如需詳細資訊，請參閱[匯入和發佈](import-and-publish.md)。

## <a name="authorize-developer-accounts-by-using-azure-ad"></a>使用 Azure AD 來授權開發人員帳戶

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
1. 選取 ![箭號](./media/api-management-howto-aad/arrow.png).
1. 在搜尋方塊中，輸入 **api**。
1. 選取 [API 管理服務]。
1. 選取 API 管理服務執行個體。
1. 在 [安全性] 底下選取 [身分識別]。

1. 從頂端選取 [+新增]。

    [新增識別提供者] 窗格隨即出現在右邊。
1. 在 [提供者類型] 下選取 [Azure Active Directory]。

    窗格中會出現可讓您輸入其他必要資訊的控制項。 控制項包括 [用戶端識別碼] 與 [用戶端密碼]。 (您可在本文稍後取得這些控制項的相關資訊)。
1. 記下 [重新導向 URL] 的內容。
    
   ![在 Azure 入口網站中新增識別提供者的步驟](./media/api-management-howto-aad/api-management-with-aad001.png)  
1. 在瀏覽器中開啟其他索引標籤。 
1. 移至 [Azure 入口網站](https://portal.azure.com)。
1. 選取 ![箭號](./media/api-management-howto-aad/arrow.png).
1. 輸入 **active**。 [Azure Active Directory] 窗格隨即出現。
1. 選取 **Azure Active Directory**。
1. 在 [管理] 底下選取 [應用程式註冊]。
1. 選取 [新增應用程式註冊]。

    ![用於建立新應用程式註冊的選取項目](./media/api-management-howto-aad/api-management-with-aad002.png)

    [建立] 窗格會出現在右邊。 您可以在這裡輸入 Azure AD 應用程式的相關資訊。
1. 輸入應用程式的名稱。
1. 選取 [Web 應用程式/API] 作為應用程式類型。
1. 輸入開發人員入口網站的登入 URL 作為登入 URL。 在此範例中，登入 URL 為 `https://apimwithaad.portal.azure-api.net/signin`。
1. 選取 [建立] 來建立應用程式。
1. 若要尋找您的應用程式，請選取 [應用程式註冊] 並依名稱搜尋。

    ![您搜尋應用程式所在的方塊](./media/api-management-howto-aad/find-your-app.png)
1. 註冊應用程式之後，請移至 [回覆 URL]，並確定 [重新導向 URL] 已設定為您從步驟 9 取得的值。 
1. 如果您想要設定您的應用程式 (例如，變更**應用程式識別碼 URL**)，請選取 [屬性]。

    ![開啟 [屬性] 窗格](./media/api-management-howto-aad/api-management-with-aad004.png)

    如果會有多個 Azure AD 執行個體用於此應用程式，請針對 [多重租用戶] 選取 [是]。 預設值為 [ **否**]。
1. 藉由選取 [所需的權限] 來設定應用程式權限。
1. 選取您的應用程式，然後選取 [讀取目錄資料] 和 [登入及讀取使用者設定檔] 核取方塊。

    ![權限的核取方塊](./media/api-management-howto-aad/api-management-with-aad005.png)

1. 選取 [授與權限] 同意應用程式權限。

    如需應用程式權限和委派權限的詳細資訊，請參閱[存取 Graph API][Accessing the Graph API]。
    
1. 在左側窗格中，複製 [應用程式識別碼] 值。

    ![「應用程式識別碼」的值](./media/api-management-howto-aad/application-id.png)
1. 切換回您的 API 管理應用程式。 

    在 [新增識別提供者] 視窗中，將**應用程式識別碼**的值貼入 [用戶端識別碼] 方塊。
1. 切換回 Azure AD 設定，然後選取 [金鑰]。
1. 指定名稱和持續時間來建立新的金鑰。 
1. 選取 [ **儲存**]。 金鑰隨即產生。

    複製金鑰至剪貼簿。

    ![用於建立金鑰的選取項目](./media/api-management-howto-aad/api-management-with-aad006.png)

    > [!NOTE]
    > 記下此金鑰。 關閉 Azure AD 設定窗格之後，即無法再次顯示金鑰。
    > 
    > 

1. 切換回您的 API 管理應用程式。 

    在 [新增識別提供者] 視窗中，將金鑰貼入 [用戶端密碼] 文字方塊。

    > [!IMPORTANT]
    > 請務必在金鑰到期之前，更新 [用戶端密碼]。 
    >  
    >

1. [新增識別提供者] 視窗也包含 [允許的租用戶] 文字方塊。 請在該處指定 Azure AD 執行個體的網域，您將會對它授與 API 管理服務執行個體的 API 存取權。 您可以使用換行符號、空格或逗號來分隔多個網域。

    您可以在 [允許的租用戶] 區段中指定多個網域。 在使用者可透過與註冊應用程式之原始網域不同的網域登入前，不同網域的全域管理員必須授與權限，應用程式才能存取目錄資料。 若要授與權限，全域管理員應：
    
    a. 移至 `https://<URL of your developer portal>/aadadminconsent` (例如， https://contoso.portal.azure-api.net/aadadminconsent)。
    
    b. 輸入想要授與存取權的 Azure AD 租用戶網域名稱。
    
    c. 選取 [提交]。 
    
    在下列範例中，來自 miaoaad.onmicrosoft.com 的全域管理員嘗試授予權限給此特定開發人員入口網站。 

1. 在您指定所需的設定之後，請選取 [新增]。

    ![[新增識別提供者] 窗格中的 [新增] 按鈕](./media/api-management-howto-aad/api-management-with-aad007.png)

儲存變更之後，所指定 Azure AD 執行個體中的使用者即可依照[使用 Azure AD 帳戶來登入開發人員入口網站](#log_in_to_dev_portal)中的步驟來登入開發人員入口網站。

![輸入 Azure AD 租用戶的名稱](./media/api-management-howto-aad/api-management-aad-consent.png)

在下一個畫面上，系統會提示全域系統管理員確認要給予權限。 

![確認要指派權限](./media/api-management-howto-aad/api-management-permissions-form.png)

如果非全域系統管理員在全域系統管理員授與其權限之前便嘗試登入，登入嘗試就會失敗，並且會顯示錯誤畫面。

## <a name="add-an-external-azure-ad-group"></a>新增外部 Azure AD 群組

在您讓 Azure AD 執行個體中的使用者可以存取之後，即可在 API 管理中新增 Azure AD 群組。 之後，您就可以更輕鬆地管理群組中開發人員與產品之間的關聯。

若要設定外部 Azure AD 群組，您必須依照上一節中的程序進行，先在 [身分識別] 索引標籤上設定 Azure AD 執行個體。 

您可以從 API 管理執行個體的 [群組] 索引標籤來新增外部 Azure AD 群組。

1. 選取 [群組]  索引標籤。
1. 選取 [新增 AAD 群組] 按鈕。
   ![[新增 AAD 群組] 按鈕](./media/api-management-howto-aad/api-management-with-aad008.png)
1. 選取您想要新增的群組。
1. 按 [選取] 按鈕。

新增外部 Azure AD 群組之後，您就可以檢閱並設定其屬性。 從 [群組] 索引標籤中選取群組名稱。您可以在這裡編輯群組的 [名稱] 與 [描述] 資訊。
 
所設定 Azure AD 執行個體的使用者現在可以登入開發人員入口網站。 這些使用者也可以檢視和訂閱所能看見的任何群組。

## <a name="a-idlogintodevportalsign-in-to-the-developer-portal-by-using-an-azure-ad-account"></a><a id="log_in_to_dev_portal"/>使用 Azure AD 帳戶來登入開發人員入口網站

使用您在先前小節中設定的 Azure AD 帳戶來登入開發人員入口網站：

1. 使用 Active Directory 應用程式設定中的登入 URL 來開啟新的瀏覽器視窗，然後選取 [Azure Active Directory]。

   ![登入頁面][api-management-dev-portal-signin]

1. 輸入 Azure AD 中其中一位使用者的認證，然後選取 [登入]。

   ![以使用者名稱和密碼來登入][api-management-aad-signin]

1. 如果需要其他資訊，可能會出現註冊表單的提示。 完成註冊表單，然後選取 [註冊]。

   ![註冊表單上的 [註冊] 按鈕][api-management-complete-registration]

您的使用者現在已登入 API 管理服務執行個體的開發人員入口網站。

![註冊完成之後的開發人員入口網站][api-management-registration-complete]

[api-management-dev-portal-signin]: ./media/api-management-howto-aad/api-management-dev-portal-signin.png
[api-management-aad-signin]: ./media/api-management-howto-aad/api-management-aad-signin.png
[api-management-complete-registration]: ./media/api-management-howto-aad/api-management-complete-registration.png
[api-management-registration-complete]: ./media/api-management-howto-aad/api-management-registration-complete.png

[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: api-management-policy-reference.md
[Caching policies]: api-management-policy-reference.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[http://oauth.net/2/]: http://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[Accessing the Graph API]: http://msdn.microsoft.com/library/azure/dn132599.aspx#BKMK_Graph

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

[Sign in to the developer portal by using an Azure AD account]: #Sign-in-to-the-developer-portal-by-using-an-Azure-AD-account
