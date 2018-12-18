---
title: 使用用戶端憑證驗證保護後端服務 - Azure API 管理 | Microsoft Docs
description: 了解如何在 Azure API 管理中使用用戶端憑證驗證來保護後端服務。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: apimpm
ms.openlocfilehash: 5983e6236d1aee80b2cc5b83b65dd2f7945039a1
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579956"
---
# <a name="how-to-secure-back-end-services-using-client-certificate-authentication-in-azure-api-management"></a>如何在 Azure API 管理中使用用戶端憑證驗證來保護後端服務

API 管理可讓您使用用戶端憑證來保護對 API 後端服務的存取。 本指南說明如何在 Azure 入口網站中管理 Azure API 管理服務執行個體中的憑證。 此外也會說明如何設定 API 以使用憑證來存取後端服務。

如需使用 API 管理 REST API 來管理憑證的詳細資訊，請參閱 <a href="https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity">Azure API 管理 REST API 憑證實體</a>。

## <a name="prerequisites"> </a>先決條件

本指南將示範如何設定 API 管理服務執行個體，以使用用戶端憑證驗證來存取 API 的後端服務。 在依照本文中的步驟操作之前，您應先設定後端服務以進行用戶端憑證驗證 ([若要在 Azure WebSites 中設定憑證驗證，請參閱此文章][to configure certificate authentication in Azure WebSites refer to this article])。 您必須存取憑證和密碼以將它上傳至 API 管理服務。

## <a name="step1"> </a>上傳用戶端憑證

![新增用戶端憑證](media/api-management-howto-mutual-certificates/apim-client-cert.png)

請依照下列步驟來上傳新的用戶端憑證。 如果您尚未建立 API 管理服務執行個體，請參閱[建立 API 管理服務執行個體][Create an API Management service instance]教學課程。

1. 在 Azure 入口網站中瀏覽至您的 Azure API 管理服務執行個體。
2. 從功能表中選取 [用戶端憑證]。
3. 按一下 [+新增] 按鈕。  
    ![新增用戶端憑證](media/api-management-howto-mutual-certificates/apim-client-cert-add.png)  
4. 瀏覽憑證，並提供其識別碼和密碼。  
5. 按一下頁面底部的 [新增] 。

> [!NOTE]
> 憑證必須是 **.pfx** 格式。 可接受自我簽署憑證。

憑證上傳完畢後，會顯示在 [用戶端憑證] 中。  如果您有許多憑證，請記下所需憑證的指紋，以[設定 API 以使用用戶端憑證來驗證閘道][Configure an API to use a client certificate for gateway authentication]。

> [!NOTE]
> 若要在使用自我簽署的憑證時關閉憑證鏈結驗證，請遵循此常見問題集[項目](api-management-faq.md#can-i-use-a-self-signed-ssl-certificate-for-a-back-end)中所述的步驟。

## <a name="step1a"> </a>刪除用戶端憑證

若要刪除憑證，請按一下內容功能表的 **...**，然後選取憑證旁的 [刪除]。

![刪除用戶端憑證](media/api-management-howto-mutual-certificates/apim-client-cert-delete.png)

如果有 API 佔用憑證，系統會顯示警告畫面。 若要刪除憑證，您必須先從已設定為使用該憑證的 API 中將其移除。

![刪除用戶端憑證失敗](media/api-management-howto-mutual-certificates/apim-client-cert-delete-failure.png)

## <a name="step2"> </a>設定 API 以使用用戶端憑證來驗證閘道

1. 從左側的 [API 管理] 功能表按一下 [API]，然後瀏覽至 API。  
    ![啟用用戶端憑證](media/api-management-howto-mutual-certificates/apim-client-cert-enable.png)

2. 在 [設計] 索引標籤中，按一下 [後端] 區段的鉛筆圖示。 
3. 將 [閘道認證] 變更為 [用戶端憑證]，並從下拉式清單中選取您的憑證。  
    ![啟用用戶端憑證](media/api-management-howto-mutual-certificates/apim-client-cert-enable-select.png)

4. 按一下 [檔案] 。 

> [!WARNING]
> 此變更將立即生效，且該 API 之作業的呼叫將使用憑證以在後端伺服器上進行驗證。


> [!TIP]
> 當您將憑證指定用於 API 後端服務的閘道驗證時，憑證遂成為該 API 之原則的一部分，因此可以在原則編輯器中檢視。

## <a name="self-signed-certificates"></a>自我簽署憑證

如果您使用自我簽署憑證，則需要停用信任鏈結驗證，API 管理才能與後端系統通訊。 否則將會傳回 500 錯誤碼。 若要這樣設定，請使用 [`New-AzureRmApiManagementBackend`](https://docs.microsoft.com/powershell/module/azurerm.apimanagement/new-azurermapimanagementbackend) (適用於新的後端) 或 [`Set-AzureRmApiManagementBackend`](https://docs.microsoft.com/powershell/module/azurerm.apimanagement/set-azurermapimanagementbackend) (適用於現有的後端) PowerShell Cmdlet，並將 `-SkipCertificateChainValidation` 參數設定為 `True`。

```
$context = New-AzureRmApiManagementContext -resourcegroup 'ContosoResourceGroup' -servicename 'ContosoAPIMService'
New-AzureRmApiManagementBackend -Context  $context -Url 'https://contoso.com/myapi' -Protocol http -SkipCertificateChainValidation $true
```

[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: ../api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: api-management-policy-reference.md
[Caching policies]: api-management-policy-reference.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[Azure API Management REST API Certificate entity]: http://msdn.microsoft.com/library/azure/dn783483.aspx
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[to configure certificate authentication in Azure WebSites refer to this article]: ../app-service/app-service-web-configure-tls-mutual-auth.md

[Prerequisites]: #prerequisites
[Upload a client certificate]: #step1
[Delete a client certificate]: #step1a
[Configure an API to use a client certificate for gateway authentication]: #step2
[Test the configuration by calling an operation in the Developer Portal]: #step3
[Next steps]: #next-steps
