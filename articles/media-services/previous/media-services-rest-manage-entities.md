---
title: 使用 REST 管理媒體服務實體 | Microsoft Docs
description: 深入了解如何使用 REST API 管理媒體服務實體。
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 95262a32-0f2a-4286-b9e2-1a1ca6399b5b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/19/2017
ms.author: juliako
ms.openlocfilehash: e5c035c4bcf449ecf20a9dfb072ce3ab480110a9
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364324"
---
# <a name="managing-media-services-entities-with-rest"></a>使用 REST 管理媒體服務實體 
> [!div class="op_single_selector"]
> * [REST](media-services-rest-manage-entities.md)
> * [.NET](media-services-dotnet-manage-entities.md)
> 
> 

Microsoft Azure 媒體服務是以 REST 為基礎的服務，內建在 OData v3 上。 您可以使用與任何其他 OData 服務上的相同方式，新增、查詢、更新和刪除實體。 在適用時會呼叫例外狀況。 如需 OData 的詳細資訊，請參閱 [開放式資料通訊協定文件](http://www.odata.org/documentation/)。

本主題說明如何使用 REST 管理 Azure 媒體服務實體。

>[!NOTE]
> 從 2017 年 4 月 1 日起，您的帳戶中任何超過 90 天的作業記錄以及其相關工作記錄都會自動刪除，即使記錄總數低於配額上限亦然。 例如，在 2017 年 4 月 1 日，您帳戶中任何在 2016 年 12 月 31 日以前的作業記錄將會自動刪除。 如果您需要封存作業/工作資訊，您可以使用本主題中所述的程式碼。

## <a name="considerations"></a>考量  

在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

## <a name="connect-to-media-services"></a>連線到媒體服務

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

## <a name="adding-entities"></a>加入實體
媒體服務中的每個實體會透過 POST HTTP 要求加入至實體集 (例如資產)。

下列範例示範如何建立 AccessPolicy。

    POST https://media.windows.net/API/AccessPolicies HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net
    Content-Length: 74
    Expect: 100-continue

    {"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}

## <a name="querying-entities"></a>查詢實體
查詢及列出實體很簡單，只牽涉到 GET HTTP 要求與選用 OData 作業。
下列範例會擷取所有 MediaProcessor 實體的清單。

    GET https://media.windows.net/API/MediaProcessors HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

您也可以擷取特定實體或與特定實體相關聯的所有實體集，如下列範例所示：

    GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c') HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

    GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c')/TaskTemplates HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

下列範例僅會傳回所有工作的狀態屬性。

    GET https://media.windows.net/API/Jobs?$select=State HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

以下範例會傳回名為 "SampleTemplate" 的所有 JobTemplates。

    GET https://media.windows.net/API/JobTemplates?$filter=startswith(Name,%20'SampleTemplate') HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

> [!NOTE]
> 媒體服務中不支援 $Expand 作業，以及 LINQ 考量 (WCF 資料服務) 中所述之不支援的 LINQ 方法。
> 
> 

## <a name="enumerating-through-large-collections-of-entities"></a>透過實體的大型集合列舉
查詢項目時，有一次最多傳回 1000 個實體的限制，因為公用 REST v2 有 1000 個查詢結果數目的限制。 使用 **skip** 和 **top** 來透過實體的大型集合列舉。 

以下範例說明如何使用 **skip** 和 **top** 來略過最前面 2000 項作業並取得接下來的 1000 項作業。  

    GET https://media.windows.net/api/Jobs()?$skip=2000&$top=1000 HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: media.windows.net

## <a name="updating-entities"></a>更新實體
根據實體類型及其所處狀態，您可以透過 PATCH、PUT 或 MERGE HTTP 要求，更新該實體的屬性。 如需這些作業的詳細資訊，請參閱 [PATCH/PUT/MERGE](https://msdn.microsoft.com/library/dd541276.aspx)。

下列程式碼範例示範如何更新資產實體上的 [名稱] 屬性。

    MERGE https://media.windows.net/API/Assets('nb:cid:UUID:80782407-3f87-4e60-a43e-5e4454232f60') HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: media.windows.net
    Content-Length: 21
    Expect: 100-continue

    {"Name" : "NewName" }

## <a name="deleting-entities"></a>刪除實體
可以使用 DELETE HTTP 要求，在媒體服務中刪除實體。 根據實體，您刪除實體的順序可能很重要。 例如，資產等實體需要您撤銷 (或刪除) 參考該特定資產的所有定位器，然後再刪除資產。

下列範例示範如何刪除用於將檔案上傳至 Blob 儲存體的定位器。

    DELETE https://media.windows.net/API/Locators('nb:lid:UUID:76dcc8e8-4230-463d-97b0-ce25c41b5c8d') HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: media.windows.net
    Content-Length: 0

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

