---
title: 從 Azure AI 資源庫檢視和刪除資料 | Microsoft Docs
description: 您可以使用介面或 AI 資源庫目錄 API，從 Azure AI 資源庫匯出和刪除產品中的使用者資料。 本文將說明如何做到。
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
manager: cgronlun
ms.reviewer: jmartens, mldocs
ms.service: machine-learning
ms.component: studio
ms.topic: conceptual
ms.date: 05/25/2018
ms.openlocfilehash: bc6ffa912d7914c8662dbde623e04947540ac149
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34659114"
---
# <a name="view-and-delete-in-product-user-data-from-azure-ai-gallery"></a>從 Azure AI 資源庫檢視和刪除產品中的使用者資料

您可以使用介面或 AI 資源庫目錄 API，從 Azure AI 資源庫檢視和刪除產品中的使用者資料。 本文會說明如何做到。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="view-your-data-in-ai-gallery-with-the-ui"></a>使用 UI 檢視 AI 資源庫中的資料

您可以透過 Azure AI 資源庫網站 UI 檢視所發行的項目。 使用者可以檢視公開的和未列出的解決方案、專案、實驗和其他已發行的項目：

1.  登入 [Azure AI 資源庫](https://gallery.azure.ai/)。
2.  按一下右上角的個人資料圖片，然後按一下帳戶名稱以載入個人資料頁面。
3.  載入個人資料頁面會顯示所有已發行至資源庫的項目，包括未列出的項目。

## <a name="use-the-ai-gallery-catalog-api-to-view-your-data"></a>使用 AI 資源庫目錄 API 來檢視資料

您可以利用程式設計，透過 AI 資源庫目錄 API (可於 https://catalog.cortanaanalytics.com/entities 存取) 檢視所收集的資料。 若要檢視資料，您需要建立者識別碼。 若要透過目錄 API 檢視未列出的實體，您需要有存取權杖。

目錄回應會以 JSON 格式傳回。

### <a name="get-an-author-id"></a>取得建立者識別碼
建立者識別碼是以發行至 Azure AI 資源庫時所使用的電子郵件地址為基礎的。 它不會變更：

1.  登入 [Azure AI 資源庫](https://gallery.azure.ai/)。
2.  按一下右上角的個人資料圖片，然後按一下帳戶名稱以載入個人資料頁面。
3.  網址列中的 URL 會在 `authorId=` 後面顯示英數字元識別碼。 例如，若 URL 為：`https://gallery.cortanaintelligence.com/Home/Author?authorId=99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`
        
    則建立者識別碼為：`99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

### <a name="get-your-access-token"></a>取得存取權杖

您需要有存取權杖才能透過目錄 API 檢視未列出的實體。 如果沒有存取權杖，使用者仍可檢視公開的實體和其他使用者資訊。

若要取得存取權杖，您必須檢查瀏覽器在登入時對目錄 API 所提出的 HTTP 要求 `DataLabAccessToken` 標頭：

1.  登入 [Azure AI 資源庫](https://gallery.azure.ai/)。
2.  按一下右上角的個人資料圖片，然後按一下帳戶名稱以載入個人資料頁面。
3.  按 F12 來開啟瀏覽器的 [開發人員工具] 窗格、選取 [網路] 索引標籤，然後重新整理頁面。 
4. 在 [篩選條件] 文字方塊中輸入「目錄」字串來對該字串篩選要求。
5.  在 URL `https://catalog.cortanaanalytics.com/entities` 的要求中，尋找 GET 要求，然後選取 [標頭] 索引標籤。向下捲動至＜要求標頭＞一節。
6.  `DataLabAccessToken`　標頭下方是英數字元權杖。 為了讓資料保持安全，請勿共用此權杖。

### <a name="view-user-information"></a>檢視使用者資訊
使用您在先前步驟中取得的建立者識別碼取代下列 URL 中的 `[AuthorId]`，來檢視使用者個人資料中的資訊：

    https://catalog.cortanaanalytics.com/users/[AuthorID]

例如，此 URL 要求：
    
    https://catalog.cortanaanalytics.com/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA

傳回如下回應：

    {"entities_count":9,"contribution_score":86.351575190956922,"scored_at":"2018-05-07T14:30:25.9305671+00:00","contributed_at":"2018-05-07T14:26:55.0381756+00:00","created_at":"2017-12-15T00:49:15.6733094+00:00","updated_at":"2017-12-15T00:49:15.6733094+00:00","name":"First Last","slugs":["First-Last"],"tenant_id":"14b2744cf8d6418c87ffddc3f3127242","community_id":"9502630827244d60a1214f250e3bbca7","id":"99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA","_links":{"self":"https://catalog.azureml.net/tenants/14b2744cf8d6418c87ffddc3f3127242/communities/9502630827244d60a1214f250e3bbca7/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA"},"etag":"\"2100d185-0000-0000-0000-5af063010000\""}


### <a name="view-public-entities"></a>檢視公開實體

目錄 API 會儲存已發行至 Azure AI 資源庫的實體相關資訊，這些資訊也可直接在 [AI 資源庫網站](https://gallery.azure.ai/)上檢視。 

若要檢視已發行的實體，請瀏覽下列 URL，並將 `[AuthorId]` 取代為上面的[取得建立者識別碼](#get-an-author-ID)中所取得的建立者識別碼。

    https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '[AuthorId]'

例如︰

    https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA'

### <a name="view-unlisted-and-public-entities"></a>檢視未列出和公開的實體

此查詢只會顯示公開實體。 若要檢視所有實體 (包括未列出的實體)，請提供上一節所取得的存取權杖。

1.  使用 [Postman](https://www.getpostman.com) 等工具，建立目錄 URL 的 HTTP GET 要求，如[取得存取權杖](#get-your-access-token)所述。
2.  建立名為 `DataLabAccessToken` 的 HTTP 要求標頭，並將值設為存取權杖。
3.  提交 HTTP 要求。

> [!TIP]
> 如果來自目錄 API 的回應未能顯示未列出的實體，表示使用者的存取權杖可能無效或已過期。 登出 Azure AI 資源庫，然後重複[取得存取權杖](#get-your-access-token)中的步驟來更新權杖。 
