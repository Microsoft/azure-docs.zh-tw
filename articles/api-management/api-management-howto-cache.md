---
title: 新增快取以改善 Azure API 管理的效能 | Microsoft Docs
description: 了解如何改善 API 管理服務呼叫的延遲、頻寬耗用量和 Web 服務負載。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 740f6a27-8323-474d-ade2-828ae0c75e7a
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: 7458ad6e0a864d742f74ce743ce3179594113c00
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2017
ms.locfileid: "26127772"
---
# <a name="add-caching-to-improve-performance-in-azure-api-management"></a>新增快取以改善 Azure API 管理的效能
可以設定 API 管理中的作業進行回應快取。 對於不常變更的資料，回應快取可大幅降低 API 延遲、頻寬耗用量和 Web 服務負載。
 
如需快取的詳細資訊，請參閱 [API 管理快取原則](api-management-caching-policies.md)和[在 Azure API 管理中自訂快取](api-management-sample-cache-by-key.md)。

![快取原則](media/api-management-howto-cache/cache-policies.png)

您將學到什麼：

> [!div class="checklist"]
> * 為您的 API 新增回應快取
> * 驗證快取的運作方式

## <a name="prerequisites"></a>先決條件

若要完成本教學課程：

+ [建立 Azure API 管理執行個體](get-started-create-service-instance.md)
+ [匯入及發佈 API 產品](import-and-publish.md)

## <a name="caching-policies"> </a>新增快取原則

根據此範例中顯示的快取原則，對 **GetSpeakers** 作業的第一個要求會從後端服務傳回回應。 此回應會被快取，並依指定的標頭和查詢字串參數來識別。 後續使用相符的參數呼叫此操作時，將傳回快取的回應，直到快取期間間隔到期為止。

1. 登入 Azure 入口網站，位址是 [https://portal.azure.com](https://portal.azure.com)。
2. 瀏覽至您的 APIM 執行個體。
3. 選取 [API] 索引標籤。
4. 按一下 API 清單中的 [示範會議 API]。
5. 選取 **GetSpeakers**。
6. 選取畫面頂端的 [設計] 索引標籤。
7. 在 [輸入處理] 視窗中，按一下三角形 (鉛筆旁邊)。

    ![程式碼編輯器](media/api-management-howto-cache/code-editor.png)
8. 選取 [程式碼編輯器]。
9. 在 **inbound** 元素中，新增下列原則：

        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false">
            <vary-by-header>Accept</vary-by-header>
            <vary-by-header>Accept-Charset</vary-by-header>
            <vary-by-header>Authorization</vary-by-header>
        </cache-lookup>

10. 在 **outbound** 元素中，新增下列原則：

        <cache-store caching-mode="cache-on" duration="20" />

    [持續期間] 指定快取回應的到期間隔。 在本範例中，間隔為 **20** 秒。

## <a name="test-operation"> </a>呼叫作業和測試快取
若要瞭解快取的運作方式，請從開發人員入口網站呼叫此作業。

1. 在 Azure 入口網站中，瀏覽至您的 APIM 執行個體。
2. 選取 [API] 索引標籤。
3. 選取您已新增快取原則的 API。
4. 選取 **GetSpeakers** 作業。
5. 按一下右上方功能表中的 [測試] 索引標籤。
6. 按 [傳送]。

## <a name="next-steps"> </a>後續步驟
* 如需快取原則的詳細資訊，請參閱 [API 管理原則參考文件][API Management policy reference]中的[快取原則][Caching policies]。
* 如需使用原則運算式依索引鍵快取項目的詳細資訊，請參閱 [在 Azure API 管理中自訂快取](api-management-sample-cache-by-key.md)。

[api-management-management-console]: ./media/api-management-howto-cache/api-management-management-console.png
[api-management-echo-api]: ./media/api-management-howto-cache/api-management-echo-api.png
[api-management-echo-api-operations]: ./media/api-management-howto-cache/api-management-echo-api-operations.png
[api-management-caching-tab]: ./media/api-management-howto-cache/api-management-caching-tab.png
[api-management-operation-dropdown]: ./media/api-management-howto-cache/api-management-operation-dropdown.png
[api-management-policy-editor]: ./media/api-management-howto-cache/api-management-policy-editor.png
[api-management-developer-portal-menu]: ./media/api-management-howto-cache/api-management-developer-portal-menu.png
[api-management-apis-echo-api]: ./media/api-management-howto-cache/api-management-apis-echo-api.png
[api-management-open-console]: ./media/api-management-howto-cache/api-management-open-console.png
[api-management-console]: ./media/api-management-howto-cache/api-management-console.png


[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md

[API Management policy reference]: https://msdn.microsoft.com/library/azure/dn894081.aspx
[Caching policies]: https://msdn.microsoft.com/library/azure/dn894086.aspx

[Create an API Management service instance]: get-started-create-service-instance.md

[Configure an operation for caching]: #configure-caching
[Review the caching policies]: #caching-policies
[Call an operation and test the caching]: #test-operation
[Next steps]: #next-steps
