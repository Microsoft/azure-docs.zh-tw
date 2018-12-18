---
title: 使用 REST 發佈 Azure 媒體服務內容
description: 了解如何建立定位器，用來建置串流 URL。 程式碼使用 REST API。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: ff332c30-30c6-4ed1-99d0-5fffd25d4f23
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2017
ms.author: juliako
ms.openlocfilehash: 8385dedd494c0cef968cb869ded3e92ce213da5e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785857"
---
# <a name="publish-azure-media-services-content-using-rest"></a>使用 REST 發佈 Azure 媒體服務內容
> [!div class="op_single_selector"]
> * [.NET](media-services-deliver-streaming-content.md)
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [入口網站](media-services-portal-publish.md)
> 
> 

您可以建立隨選串流定位器及建置串流 URL，串流處理調適性位元速率 MP4 集。 [為資產編碼](media-services-rest-encode-asset.md)文章說明如何編碼為調適性位元速率 MP4 集。 如果您的內容已加密，請在建立定位器之前設定資產傳遞原則 (如[這個](media-services-rest-configure-asset-delivery-policy.md)文章中所述)。 

您也可以使用隨選串流定位器來建置指向可漸進式下載之 MP4 檔案的 URL。  

本文章說明如何建立隨選串流定位器以發佈資產及建置 Smooth、MPEG DASH 和 HLS 串流 URL。 它也會示範如何建置漸進式下載 URL。

[下列章節](#types) 說明列舉類型，REST 呼叫中會使用這些類型的值。   

> [!NOTE]
> 在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。
> 

## <a name="connect-to-media-services"></a>連線到媒體服務

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

>[!NOTE]
>順利連線到 https://media.windows.net 之後，您將會收到 301 重新導向，指定另一個媒體服務 URI。 後續的呼叫必須送到新的 URI。

## <a name="create-an-ondemand-streaming-locator"></a>建立隨選串流定位器
若要建立隨選串流定位器並取得 URL，您需要執行下列動作：

1. 如果內容已加密，請定義存取原則。
2. 建立隨選串流定位器。
3. 如果您想要串流處理，請取得資產內的串流資訊清單檔案 (.ism)。 
   
   如果您想要漸進式地下載，請取得資產中的 MP4 檔案名稱。 
4. 建置資訊清單檔或 MP4 檔案的 URL。 
5. 您無法使用包含寫入或刪除權限的 AccessPolicy 建立串流訂位器。

### <a name="create-an-access-policy"></a>建立存取原則

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限 (例如要長時間維持就地的定位器原則 (非上載原則))，請使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

要求：

    POST https://media.windows.net/api/AccessPolicies HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
    Host: media.windows.net
    Content-Length: 68

    {"Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}

回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 311
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https:/media.windows.net/api/AccessPolicies('nb%3Apid%3AUUID%3A69c80d98-7830-407f-a9af-e25f4b0d3e5f')
    Server: Microsoft-IIS/8.5
    request-id: a877528a-bdb4-4414-9862-273f8e64f882
    x-ms-request-id: a877528a-bdb4-4414-9862-273f8e64f882
    x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Wed, 18 Feb 2015 06:52:09 GMT

    {"odata.metadata":"https://media.windows.net/api/$metadata#AccessPolicies/@Element","Id":"nb:pid:UUID:69c80d98-7830-407f-a9af-e25f4b0d3e5f","Created":"2015-02-18T06:52:09.8862191Z","LastModified":"2015-02-18T06:52:09.8862191Z","Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}

### <a name="create-an-ondemand-streaming-locator"></a>建立隨選串流定位器
建立指定資產和資產原則的定位器。

要求：

    POST https://media.windows.net/api/Locators HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
    Host: media.windows.net
    Content-Length: 181

    {"AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872Z","Type":2}

回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 637
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://media.windows.net/api/Locators('nb%3Alid%3AUUID%3Abe245661-2bbd-4fc6-b14f-9cf9a1492e5e')
    Server: Microsoft-IIS/8.5
    request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
    x-ms-request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
    x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Wed, 18 Feb 2015 06:58:37 GMT

    {"odata.metadata":"https://media.windows.net/api/$metadata#Locators/@Element","Id":"nb:lid:UUID:be245661-2bbd-4fc6-b14f-9cf9a1492e5e","ExpirationDateTime":"2015-03-20T06:34:47.267872+00:00","Type":2,"Path":"http://amstest1.streaming.mediaservices.windows.net/be245661-2bbd-4fc6-b14f-9cf9a1492e5e/","BaseUri":"http://amstest1.streaming.mediaservices.windows.net","ContentAccessComponent":"be245661-2bbd-4fc6-b14f-9cf9a1492e5e","AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872+00:00","Name":null}

### <a name="build-streaming-urls"></a>建置串流 URL
建立定位器之後，使用傳回的 **Path** 值建置 Smooth、HLS 和 MPEG DASH URL。 

Smooth Streaming： **Path** + 資訊清單檔案名稱 + "/manifest"

例如：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest

HLS： **Path** + 資訊清單檔案名稱 + "/manifest(format=m3u8-aapl)"

例如：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)


DASH： **Path** + 資訊清單檔案名稱 + "/manifest(format=mpd-time-csf)"

例如：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)


### <a name="build-progressive-download-urls"></a>建置漸進式下載 URL
建立定位器之後，使用傳回的 **Path** 值建置漸進式下載 URL。   

URL： **Path** + 資產檔案 MP4 名稱

例如：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

## <a id="types"></a>列舉類型
    [Flags]
    public enum AccessPermissions
    {
        None = 0,
        Read = 1,
        Write = 2,
        Delete = 4,
        List = 8,
    }

    public enum LocatorType
    {
        None = 0,
        Sas = 1,
        OnDemandOrigin = 2,
    }

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[媒體服務作業 REST API 概觀](media-services-rest-how-to-use.md)

[設定資產傳遞原則](media-services-rest-configure-asset-delivery-policy.md)

