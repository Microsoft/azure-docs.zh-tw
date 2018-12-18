---
title: 使用 Azure Active Directory B2C 授權開發人員帳戶 - Azure API 管理 | Microsoft Docs
description: 了解如何在 API 管理中使用 Azure Active Directory B2C 授權使用者。
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
ms.date: 10/30/2017
ms.author: apimpm
ms.openlocfilehash: a6e7aad6c3d20a67ecba66c49be4efcdebdf718a
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32153416"
---
> [!WARNING]
> 僅[「開發人員」、「標準」與「進階」](https://azure.microsoft.com/pricing/details/api-management/)層提供 Azure Active Directory B2C 整合。

# <a name="how-to-authorize-developer-accounts-by-using-azure-active-directory-b2c-in-azure-api-management"></a>如何在 Azure API 管理中使用 Azure Active Directory B2C 授權開發人員帳戶
## <a name="overview"></a>概觀
Azure Active Directory B2C 是適用於取用者導向 Web 與行動應用程式的雲端身分識別管理解決方案。 您可以使用它來管理開發人員入口網站的存取。 本指南會說明要與 Azure Active Directory B2C 整合所必須在 API 管理服務中進行的設定。 如需實現使用傳統 Azure Active Directory 來存取開發人員入口網站的相關資訊，請參閱[如何使用 Azure Active Directory 授權開發人員帳戶]。

> [!NOTE]
> 若要完成本指南中的步驟，您必須先具備要在其中建立應用程式的 Azure Active Directory B2C 租用戶。 您還必須備妥註冊和登入原則。 如需詳細資訊，請參閱 [Azure Active Directory B2C 概觀]。

## <a name="authorize-developer-accounts-by-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 授權開發人員帳戶

1. 若要開始，請在 API 管理服務的 Azure 入口網站中按一下 [發佈者入口網站]。 這會帶您前往 API 管理發行者入口網站。

   ![發行者入口網站][api-management-management-console]

   > [!NOTE]
   > 如果您尚未建立 API 管理服務執行個體，請參閱[開始使用 Azure API 管理教學課程][Get started with Azure API Management]中的[建立 API 管理服務執行個體][Create an API Management service instance]。

2. 在 [API 管理] 功能表上，按一下 [安全性]。 在 [身分識別] 索引標籤上，選擇 [Azure Active Directory B2C]。

  ![外部身分識別 1][api-management-howto-aad-b2c-security-tab]

3. 記下 [重新導向 URL]，然後切換到 Azure 入口網站中的 Azure Active Directory B2C。

  ![外部身分識別 2][api-management-howto-aad-b2c-security-tab-reply-url]

4. 按一下 [應用程式] 按鈕。

  ![註冊新的應用程式 1][api-management-howto-aad-b2c-portal-menu]

5. 按一下 [新增] 按鈕以建立新的 Azure Active Directory B2C 應用程式。

  ![註冊新的應用程式 2][api-management-howto-aad-b2c-add-button]

6. 在 [新增應用程式] 刀鋒視窗中，輸入應用程式的名稱。 在 [Web 應用程式/Web API] 下選擇 [是]，然後在 [允許隱含流程] 下選擇 [是]。 然後，從發佈者入口網站中 [身分識別] 索引標籤的 [Azure Active Directory B2C] 區段，複製 [重新導向 URL] 並貼到 [回覆 URL] 文字方塊。

  ![註冊新的應用程式 3][api-management-howto-aad-b2c-app-details]

7. 按一下 [ **建立** ] 按鈕。 應用程式在建立後會出現在 [應用程式] 刀鋒視窗。 按一下應用程式名稱可查看其詳細資料。

  ![註冊新的應用程式 4][api-management-howto-aad-b2c-app-created]

8. 從 [屬性] 刀鋒視窗複製 [應用程式識別碼] 到剪貼簿。

  ![應用程式識別碼 1][api-management-howto-aad-b2c-app-id]

9. 切換回發佈者入口網站，並將識別碼貼到 [用戶端識別碼] 文字方塊。

  ![應用程式識別碼 2][api-management-howto-aad-b2c-client-id]

10. 切換回 Azure 入口網站，按一下 [金鑰] 按鈕，然後按一下 [產生金鑰]。 按一下 [儲存] 以儲存組態並顯示**應用程式金鑰**。 複製金鑰至剪貼簿。

  ![應用程式金鑰 1][api-management-howto-aad-b2c-app-key]

11. 切換回發行者入口網站，並將金鑰貼上至 [ **用戶端密碼** ] 文字方塊。

  ![應用程式金鑰 2][api-management-howto-aad-b2c-client-secret]

12. 在 [允許的租用戶] 中，指定 Azure Active Directory B2C 租用戶的網域名稱。

  ![允許的租用戶][api-management-howto-aad-b2c-allowed-tenant]

13. 指定 [註冊原則] 和 [登入原則]。 (選擇性) 您也可以提供 [設定檔編輯原則] 和 [密碼重設原則]。

  ![原則][api-management-howto-aad-b2c-policies]

  > [!NOTE]
  > 如需原則的詳細資訊，請參閱 [Azure Active Directory B2C：可延伸的原則架構]。

14. 指定需要的組態之後，按一下 [儲存]。

  儲存變更後，開發人員就可以使用 Azure Active Directory B2C 建立新帳戶和登入開發人員入口網站。

## <a name="sign-up-for-a-developer-account-by-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 註冊開發人員帳戶

1. 若要使用 Azure Active Directory B2C 註冊開發人員帳戶，請開啟新的瀏覽器視窗並移至開發人員入口網站。 按一下 [註冊] 按鈕。

   ![開發人員入口網站 1][api-management-howto-aad-b2c-dev-portal]

2. 選擇使用 [Azure Active Directory B2C] 進行註冊。

   ![開發人員入口網站 2][api-management-howto-aad-b2c-dev-portal-b2c-button]

3. 系統會將您重新導向至您在上一節中設定的註冊原則。 選擇使用電子郵件地址或其中一個現有的社交帳戶登入。

   > [!NOTE]
   > 如果 Azure Active Directory B2C 是發佈者入口網站的 [身分識別] 索引標籤上唯一啟用的選項，系統會直接將您重新導向至註冊原則。

   ![開發人員入口網站][api-management-howto-aad-b2c-dev-portal-b2c-options]

   註冊完成後，系統會將您重新導向回到開發人員入口網站。 您現在已登入 API 管理服務執行個體的開發人員入口網站。

    ![註冊完成][api-management-registration-complete]

## <a name="next-steps"></a>後續步驟

*  [Azure Active Directory B2C 概觀]
*  [Azure Active Directory B2C：可延伸的原則架構]
*  [使用 Microsoft 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]
*  [使用 Google 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]
*  [使用 Linkedin 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]
*  [使用 Facebook 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]




[api-management-howto-aad-b2c-security-tab]: ./media/api-management-howto-aad-b2c/api-management-b2c-security-tab.PNG
[api-management-howto-aad-b2c-security-tab-reply-url]: ./media/api-management-howto-aad-b2c/api-management-b2c-security-tab-reply-url.PNG
[api-management-howto-aad-b2c-portal-menu]: ./media/api-management-howto-aad-b2c/api-management-b2c-portal-menu.PNG
[api-management-howto-aad-b2c-add-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-add-button.PNG
[api-management-howto-aad-b2c-app-details]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-details.PNG
[api-management-howto-aad-b2c-app-created]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-created.PNG
[api-management-howto-aad-b2c-app-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-id.PNG
[api-management-howto-aad-b2c-client-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-id.PNG
[api-management-howto-aad-b2c-app-key]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key.PNG
[api-management-howto-aad-b2c-app-key-saved]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key-saved.PNG
[api-management-howto-aad-b2c-client-secret]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-secret.PNG
[api-management-howto-aad-b2c-allowed-tenant]: ./media/api-management-howto-aad-b2c/api-management-b2c-allowed-tenant.PNG
[api-management-howto-aad-b2c-policies]: ./media/api-management-howto-aad-b2c/api-management-b2c-policies.PNG
[api-management-howto-aad-b2c-dev-portal]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-button.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-options]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-options.PNG
[api-management-complete-registration]: ./media/api-management-howto-aad/api-management-complete-registration.PNG
[api-management-registration-complete]: ./media/api-management-howto-aad/api-management-registration-complete.png

[api-management-management-console]: ./media/api-management-howto-aad/api-management-management-console.png
[api-management-security-external-identities]: ./media/api-management-howto-aad/api-management-b2c-security-tab.png
[api-management-security-aad-new]: ./media/api-management-howto-aad/api-management-security-aad-new.png
[api-management-new-aad-application-menu]: ./media/api-management-howto-aad/api-management-new-aad-application-menu.png
[api-management-new-aad-application-1]: ./media/api-management-howto-aad/api-management-new-aad-application-1.png
[api-management-new-aad-application-2]: ./media/api-management-howto-aad/api-management-new-aad-application-2.png
[api-management-new-aad-app-created]: ./media/api-management-howto-aad/api-management-new-aad-app-created.png
[api-management-aad-app-permissions]: ./media/api-management-howto-aad/api-management-aad-app-permissions.png
[api-management-aad-app-client-id]: ./media/api-management-howto-aad/api-management-aad-app-client-id.png
[api-management-client-id]: ./media/api-management-howto-aad/api-management-client-id.png
[api-management-aad-key-before-save]: ./media/api-management-howto-aad/api-management-aad-key-before-save.png
[api-management-aad-key-after-save]: ./media/api-management-howto-aad/api-management-aad-key-after-save.png
[api-management-client-secret]: ./media/api-management-howto-aad/api-management-client-secret.png
[api-management-client-allowed-tenants]: ./media/api-management-howto-aad/api-management-client-allowed-tenants.png
[api-management-client-allowed-tenants-save]: ./media/api-management-howto-aad/api-management-client-allowed-tenants-save.png
[api-management-aad-delegated-permissions]: ./media/api-management-howto-aad/api-management-aad-delegated-permissions.png
[api-management-dev-portal-signin]: ./media/api-management-howto-aad/api-management-dev-portal-signin.png
[api-management-aad-signin]: ./media/api-management-howto-aad/api-management-aad-signin.png
[api-management-aad-app-multi-tenant]: ./media/api-management-howto-aad/api-management-aad-app-multi-tenant.png
[api-management-aad-reply-url]: ./media/api-management-howto-aad/api-management-aad-reply-url.png
[api-management-permissions-form]: ./media/api-management-howto-aad/api-management-permissions-form.png
[api-management-configure-product]: ./media/api-management-howto-aad/api-management-configure-product.png
[api-management-add-groups]: ./media/api-management-howto-aad/api-management-add-groups.png
[api-management-select-group]: ./media/api-management-howto-aad/api-management-select-group.png
[api-management-aad-groups-list]: ./media/api-management-howto-aad/api-management-aad-groups-list.png
[api-management-aad-group-added]: ./media/api-management-howto-aad/api-management-aad-group-added.png
[api-management-groups]: ./media/api-management-howto-aad/api-management-groups.png
[api-management-edit-group]: ./media/api-management-howto-aad/api-management-edit-group.png

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
[Azure Active Directory B2C 概觀]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview
[如何使用 Azure Active Directory 授權開發人員帳戶]: https://docs.microsoft.com/azure/api-management/api-management-howto-aad
[Azure Active Directory B2C：可延伸的原則架構]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-policies
[使用 Microsoft 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-msa-app
[使用 Google 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-goog-app
[使用 Facebook 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-fb-app
[使用 Linkedin 帳戶做為 Azure Active Directory B2C 中的身分識別提供者]: https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-li-app

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

[Log in to the Developer portal using an Azure Active Directory account]: #Log-in-to-the-Developer-portal-using-an-Azure-Active-Directory-account
