---
title: 使用 REST 傳遞點播內容入門 | Microsoft Docs
description: 本教學課程會帶您逐步完成使用 REST API 實作含 Azure 媒體服務的點播內容傳遞應用程式。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 88194b59-e479-43ac-b179-af4f295e3780
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/13/2018
ms.author: juliako
ms.openlocfilehash: 757ba9e999bfbb46be96e653e7939d91bdf67679
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287120"
---
# <a name="get-started-with-delivering-content-on-demand-using-rest"></a>使用 REST 傳遞點播內容入門
[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

本快速入門將逐步引導您使用 REST API 完成利用 Azure 媒體服務 (AMS) 來實作點播視訊 (VoD) 內容傳遞應用程式。

教學課程中介紹基本的媒體服務工作流程，以及媒體服務開發最常用的程式設計物件和必要工作。 完成本教學課程時，您將能夠串流或以漸進方式下載您已上傳、編碼和下載的範例媒體檔案。

下列影像顯示針對媒體服務 OData 模型開發 VoD 應用程式時一些最常用的物件。

按一下影像可以完整大小檢視。  

<a href="./media/media-services-rest-get-started/media-services-overview-object-model.png" target="_blank"><img src="./media/media-services-rest-get-started/media-services-overview-object-model-small.png"></a> 

## <a name="prerequisites"></a>必要條件
需要下列必要條件，才能開始使用 REST API 用媒體服務進行開發。

* 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* 了解如何使用媒體服務 REST API 進行開發。 如需詳細資訊，請參閱[媒體服務 REST API 概觀](media-services-rest-how-to-use.md)。
* 由您選擇可以傳送 HTTP 要求和回應的應用程式。 本教學課程使用 [Fiddler](http://www.telerik.com/download/fiddler)。

本快速入門會顯示下列工作。

1. 啟動串流端點 (使用 Azure 入口網站)。
2. 使用 REST API 連接到媒體服務帳戶。
3. 使用 REST API 建立新資產並上傳視訊檔案。
4. 使用 REST API 將來源檔案編碼為一組調適性位元速率 MP4 檔案。
5. 使用 REST API 發行資產及取得串流和漸進式下載 URL。
6. 播放您的內容。

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限 (例如要長時間維持就地的定位器原則 (非上載原則))，請使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

如需本文章中所用之 AMS REST 實體的詳細資訊，請參閱 [Azure 媒體服務 REST API 參考](https://docs.microsoft.com/en-us/rest/api/media/services/azure-media-services-rest-api-reference)。 此外，請參閱 [Azure 媒體服務概念](media-services-concepts.md)。

>[!NOTE]
>在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>使用 Azure 入口網站開始串流端點

使用 Azure 媒體服務時，其中一個最常見的案例是透過自適性串流提供影片。 媒體服務提供動態封裝，這讓您以媒體服務即時支援的串流格式 (MPEG DASH、HLS、Smooth Streaming) 提供自適性 MP4 編碼內容，而不必儲存這些串流格式個別的預先封裝版本。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。

若要啟動串流端點，請執行下列作業︰

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 [設定] 視窗中，按一下 [串流端點]。
3. 按一下預設串流端點。

    [預設串流端點詳細資料] 視窗隨即出現。

4. 按一下 [啟動] 圖示。
5. 按一下 [儲存] 按鈕以儲存您的變更。

## <a id="connect"></a>使用 REST API 連接到媒體服務帳戶

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

## <a id="upload"></a>使用 REST API 建立新資產並上傳視訊檔案

在媒體服務中，您會將數位檔案上傳到到資產。 **資產**實體可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式輔助字幕檔案 (以及這些檔案的相關中繼資料)。一旦檔案會上傳到資產，您的內容會安全地儲存在雲端，以便進行進一步的處理和串流。

您必須建立資產時提供的值是資產建立選項。 **Options** 屬性是描述可以使用建立資產之加密選項的列舉值。 有效的值是以下清單的其中一個值，而不是值的組合。

* **None** = **0** - 不使用加密。 使用此選項時，您的內容在傳輸或儲存體中靜止時不會受到保護。
    如果您計劃使用漸進式下載傳遞 MP4，請使用此選項。
* **StorageEncrypted** = **1** - 使用 AES-256 位元加密對您的內容進行本機加密，接著上傳到已靜止加密儲存的 Azure 儲存體。 以儲存體加密保護的資產會自動解除加密並在編碼前放置在加密的檔案系統中，並且會在上傳為新輸出資產之前選擇性地重新編碼。 儲存體加密的主要使用案例是讓您可以使用強式加密來保護磁碟中靜止的高品質輸入媒體檔。
* **CommonEncryptionProtected** = **2** - 如果您要上傳已經使用一般加密或 PlayReady DRM (例如使用 PlayReady DRM 保護的 Smooth Streaming) 加密及保護的內容，請使用這個選項。
* **EnvelopeEncryptionProtected** = **4** – 如果您要上傳使用 AES 加密的 HLS，請使用這個選項。 檔案必須已由 Transform Manager 編碼和加密。

### <a name="create-an-asset"></a>建立資產
資產是媒體服務中多種類型或物件集的容器，包括視訊、音訊、影像、縮圖集合、文字播放軌和隱藏式字幕檔案。 在 REST API 中，建立資產必須傳送 POST 要求給媒體服務，並將關於您資產的任何屬性資訊放在要求主體中。

下列範例示範如何建立資產。

**HTTP 要求**

    POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
    Host: wamsbayclus001rest-hs.cloudapp.net
    Content-Length: 45

    {"Name":"BigBuckBunny.mp4", "Options":"0"}


**HTTP 回應**

如果成功，則會傳回下列內容：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 452
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
    request-id: e98be122-ae09-473a-8072-0ccd234a0657
    x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Sun, 18 Jan 2015 22:06:40 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
       "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "State":0,
       "Created":"2015-01-18T22:06:40.6010903Z",
       "LastModified":"2015-01-18T22:06:40.6010903Z",
       "AlternateId":null,
       "Name":"BigBuckBunny2.mp4",
       "Options":0,
       "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "StorageAccountName":"storagetestaccount001"
    }

### <a name="create-an-assetfile"></a>建立 AssetFile
[AssetFile](https://docs.microsoft.com/rest/api/media/operations/assetfile) 實體代表儲存在 blob 容器中的視訊或音訊檔案。 資產檔案一律會與資產相關聯，資產可包含一或多個 Assetfile。 如果資產檔案物件並未與 blob 容器中的數位檔案相關聯，媒體服務編碼器工作將會失敗。

您將數位媒體檔案上傳至 blob 容器之後，您將使用 **MERGE** HTTP 要求，以媒體檔案的相關資訊來更新 AssetFile (如本主題稍後所示)。

**HTTP 要求**

    POST https://wamsbayclus001rest-hs.cloudapp.net/api/Files HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net
    Content-Length: 164

    {  
       "IsEncrypted":"false",
       "IsPrimary":"false",
       "MimeType":"video/mp4",
       "Name":"BigBuckBunny.mp4",
       "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
    }


**HTTP 回應**

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 535
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
    Server: Microsoft-IIS/8.5
    request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
    x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 00:34:07 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
       "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
       "Name":"BigBuckBunny.mp4",
       "ContentFileSize":"0",
       "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "EncryptionVersion":null,
       "EncryptionScheme":null,
       "IsEncrypted":false,
       "EncryptionKeyId":null,
       "InitializationVector":null,
       "IsPrimary":false,
       "LastModified":"2015-01-19T00:34:08.1934137Z",
       "Created":"2015-01-19T00:34:08.1934137Z",
       "MimeType":"video/mp4",
       "ContentChecksum":null
    }


### <a name="creating-the-accesspolicy-with-write-permission"></a>建立具有寫入權限的 AccessPolicy
將任何檔案上傳到 blob 儲存體之前，請設定寫入資產的存取原則權限。 若要這樣做，請 POST HTTP 要求到 AccessPolicies 實體集。 請在建立時定義 DurationInMinutes 值，否則您會在回應中收到 500 內部伺服器錯誤訊息。 如需 AccessPolicies 的詳細資訊，請參閱 [AccessPolicy](https://docs.microsoft.com/rest/api/media/operations/accesspolicy)。

下列範例示範如何建立 AccessPolicy：

**HTTP 要求**

    POST https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net
    Content-Length: 74

    {"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"}

**HTTP 回應**

如果成功，則會傳回下列回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 312
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
    Server: Microsoft-IIS/8.5
    request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
    x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Sun, 18 Jan 2015 22:18:06 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
       "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
       "Created":"2015-01-18T22:18:06.6370575Z",
       "LastModified":"2015-01-18T22:18:06.6370575Z",
       "Name":"NewUploadPolicy",
       "DurationInMinutes":440.0,
       "Permissions":2
    }

### <a name="get-the-upload-url"></a>取得上傳 URL

若要接收實際的上傳 URL，請建立 SAS 定位器。 定位器為想要存取資產中之檔案的用戶端定義連線端點的開始時間和類型。 您可以為指定的 AccessPolicy 與 Asset 配對建立多個 Locator 實體，以處理不同的用戶端要求與需求。 這些 Locator 每個都會使用 StartTime 值加上 AccessPolicy 的 DurationInMinutes 值，以判斷可以使用 URL 的時間長度。 如需詳細資訊，請參閱 [定位器](https://docs.microsoft.com/rest/api/media/operations/locator)。

SAS URL 具有下列格式：

    {https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}

適用一些考量事項：

* 您一次不能有超過五個唯一定位器與指定的資產相關聯。 
* 如果您需要立即上傳檔案，您應該將 StartTime 值設為目前時間的五分鐘前。 這是因為用戶端電腦與媒體服務之間可能有時間差。 此外，您的 StartTime 值必須是以下日期時間格式：YYYY-MM-DDTHH:mm:ssZ (例如，"2014-05-23T17:53:50Z")。    
* 建立 Locator 之後到它可供使用時，中間可能會有 30 到 40 秒的延遲。 此問題同時適用於 [SAS URL](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1) 與原始定位器。

下列範例會示範如何建立 SAS URL 定位器，如要求主體中的 Type 屬性所定義 ("1" 代表 SAS 定位器，"2" 代表隨選原始定位器)。 傳回的 **Path** 屬性包含上傳檔案必須使用的 URL。

**HTTP 要求**

    POST https://wamsbayclus001rest-hs.cloudapp.net/api/Locators HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net
    Content-Length: 178

    {  
       "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
       "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "StartTime":"2015-02-18T16:45:53",
       "Type":1
    }


**HTTP 回應**

如果成功，則會傳回下列回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 949
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
    Server: Microsoft-IIS/8.5
    request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
    x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 03:01:29 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
       "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
       "ExpirationDateTime":"2015-02-19T00:05:53",
       "Type":1,
       "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
       "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
       "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
       "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
       "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "StartTime":"2015-02-18T16:45:53",
       "Name":null
    }

### <a name="upload-a-file-into-a-blob-storage-container"></a>將檔案上傳至 blob 儲存體容器
一旦設定 AccessPolicy 與 Locator，就會使用「Azure 儲存體 REST API」將實際檔案上傳到 Azure Blob 儲存體容器。 您必須將檔案以區塊 Blob 形式上傳。 「Azure 媒體服務」不支援分頁 Blob。  

> [!NOTE]
> 您必須將要上傳的檔案名稱新增到上一節中所收到的 Locator **Path** 值。 例如， https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . .
>
>

如需使用 Azure 儲存體 blob 的詳細資訊，請參閱 [Blob 服務 REST API](https://docs.microsoft.com/rest/api/storageservices/Blob-Service-REST-API)。

### <a name="update-the-assetfile"></a>更新 AssetFile
現在，您已上傳您的檔案，請更新 FileAsset 大小 (及其他) 資訊。 例如︰

    MERGE https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net

    {  
       "ContentFileSize":"1186540",
       "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
       "MimeType":"video/mp4",
       "Name":"BigBuckBunny.mp4",
       "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
    }


**HTTP 回應**

如果成功，則會傳回下列內容：

    HTTP/1.1 204 No Content
    ...

## <a name="delete-the-locator-and-accesspolicy"></a>刪除 Locator 和 AccessPolicy
**HTTP 要求**

    DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net


**HTTP 回應**

如果成功，則會傳回下列內容：

    HTTP/1.1 204 No Content
    ...

**HTTP 要求**

    DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net

**HTTP 回應**

如果成功，則會傳回下列內容：

    HTTP/1.1 204 No Content
    ...

## <a id="encode"></a>將來源檔案編碼為一組調適性位元速率 MP4 檔案

將資產內嵌到媒體服務之後，可以先將媒體編碼、轉碼多工處理、加上浮水印等，再傳遞給用戶端。 這些活動會針對多個背景角色執行個體排定和執行，以確保高效能與可用性。 這些活動稱為作業，每個作業包含對資產檔案執行實際工作的不可部分完成的工作 (如需詳細資訊，請參閱[作業](https://docs.microsoft.com/en-us/rest/api/media/operations/job)、[工作](https://docs.microsoft.com/en-us/rest/api/media/operations/task)說明)。

如稍早所提及，使用 Azure 媒體服務時，其中一個最常見的案例是將調適性位元速率串流傳遞給用戶端。 媒體服務可以以下列其中一種格式動態封裝一組自適性 MP4 檔案：HTTP 即時串流 (HLS)、Smooth Streaming 和 MPEG DASH。

下一節示範如何建立包含一個編碼工作的工作。 此工作指定使用 **媒體編碼器標準**，將夾層檔案轉換為一組調適性位元速率 MP4。 此節也會示範如何監視工作處理進度。 工作完成時，您將能夠建立存取資產所需的定位器。

### <a name="get-a-media-processor"></a>取得媒體處理器
在媒體服務中，媒體處理器是可處理特定處理工作的元件，例如編碼、格式轉換、加密或解密媒體內容。 此教學課程中所示的編碼工作，我們將使用媒體編碼器標準。

下列程式碼要求編碼器的識別碼。

**HTTP 要求**

    GET https://wamsbayclus001rest-hs.cloudapp.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net


**HTTP 回應**

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 273
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Server: Microsoft-IIS/8.5
    request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
    x-ms-request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 07:54:09 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#MediaProcessors",
       "value":[  
          {  
             "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
             "Description":"Media Encoder Standard",
             "Name":"Media Encoder Standard",
             "Sku":"",
             "Vendor":"Microsoft",
             "Version":"1.1"
          }
       ]
    }

### <a name="create-a-job"></a>建立工作
每個工作可以有一或多個工作，視您想要完成的處理類型而定。 您可以透過 REST API 以兩種方式的其中之一建立工作和其相關工作：工作可以透過 Job 實體上的 Tasks 導覽屬性，或透過 OData 批次處理進行內嵌定義。 媒體服務 SDK 使用批次處理。 不過，為了本文章中程式碼範例的可讀性，工作都是以內嵌定義。 如需批次處理的資訊，請參閱 [開放式資料通訊協定 (OData) 批次處理](http://www.odata.org/documentation/odata-version-3-0/batch-processing/)。

下列範例會示範如何建立和張貼工作，並將一個工作設為在特定的解析度與品質將視訊編碼。 下列文件區段包含媒體編碼器標準處理器支援的所有 [工作預設](http://msdn.microsoft.com/library/mt269960) 清單。  

**HTTP 要求**

    POST https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net
    Content-Length: 482

    {  
       "Name":"NewTestJob",
       "InputMediaAssets":[  
          {  
             "__metadata":{  
                "uri":"https://wamsbayclus001rest-hs.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')"
             }
          }
       ],
       "Tasks":[  
          {  
             "Configuration":"Adaptive Streaming",
             "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
             "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset>
                <outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
          }
       ]
    }

**HTTP 回應**

如果成功，則會傳回下列回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 1215
    Content-Type: application/json;odata=verbose;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')
    Server: Microsoft-IIS/8.5
    request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
    x-ms-request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 08:04:35 GMT

    {  
       "d":{  
          "__metadata":{  
             "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
             "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
             "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Job"
          },
          "Tasks":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Tasks"
             }
          },
          "OutputMediaAssets":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets"
             }
          },
          "InputMediaAssets":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')/InputMediaAssets"
             }
          },
          "Id":"nb:jid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
          "Name":"NewTestJob",
          "Created":"2015-01-19T08:04:34.3287057Z",
          "LastModified":"2015-01-19T08:04:34.3287057Z",
          "EndTime":null,
          "Priority":0,
          "RunningDuration":0,
          "StartTime":null,
          "State":0,
          "TemplateId":null,
          "JobNotificationSubscriptions":{  
             "__metadata":{  
                "type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.JobNotificationSubscription)"
             },
             "results":[  

             ]
          }
       }
    }


有任何工作要求中有一些重要事項要注意：

* TaskBody 屬性必須使用 XML 常值來定義工作所使用的輸入或輸出資產數目。 工作文章包含 XML 的 XML 結構描述定義。
* 在 TaskBody 定義中，<inputAsset> 和 <outputAsset> 的每一個內部值必須設定為 JobInputAsset(value) 或 JobOutputAsset(value)。
* 每個工作可以有多個輸出資產。 一個 JobOutputAsset(x) 只能使用一次做為工作中的工作輸出。
* 您可以指定 JobInputAsset 或 JobOutputAsset 做為工作的輸入資產。
* 工作不能形成循環。
* 您傳遞至 JobInputAsset 或 JobOutputAsset 的 value 參數代表資產的索引值。 實際資產定義在作業實體定義上的 InputMediaAsset 與 OutputMediaAsset 導覽屬性。

> [!NOTE]
> 由於媒體服務建置在 OData v3 之上，因此 InputMediaAsset 與 OutputMediaAsset 導覽屬性集合中的個別資產會透過 "__metadata : uri" 名稱 / 值組參考。
>
>

* InputMediaAsset 對應至您在媒體服務中建立的一個或多個資產。 OutputMediaAsset 由系統建立。 它們不會參考現有的資產。
* OutputMediaAsset 可以使用 assetName 屬性命名。 如果這個屬性不存在，則 OutputMediaAsset 的名稱是 <outputAsset> 元素的任何內部文字值，並且尾碼為工作名稱值或工作識別碼值 (在未定義 Name 屬性的情況下)。 例如，如果您將 assetName 的值設為 "Sample"，則 OutputMediaAsset Name 屬性會設為 "Sample"。 不過，如果您未設定 assetName 的值，但已將工作名稱設為 "NewJob"，則 OutputMediaAsset Name 會是 "JobOutputAsset(value)_NewJob"。

    下列範例示範如何設定 assetName 屬性：

        "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"
* 啟用工作鏈結：

  * 工作必須有至少 2 個工作
  * 必須有至少一個工作的輸入是工作中另一項工作的輸出。

如需詳細資訊，請參閱 [使用媒體服務 REST API 建立編碼工作](media-services-rest-encode-asset.md)。

### <a name="monitor-processing-progress"></a>監看處理進度
您可以使用 State 屬性擷取工作狀態，如下列範例所示：

**HTTP 要求**

    GET https://wamsbayclus001rest-hs.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/State HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: wamsbayclus001rest-hs.net
    Content-Length: 0


**HTTP 回應**

如果成功，則會傳回下列回應：

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 17
    Content-Type: application/json;odata=verbose;charset=utf-8
    Server: Microsoft-IIS/7.5
    x-ms-request-id: 01324d81-18e2-4493-a3e5-c6186209f0c8
    X-Content-Type-Options: nosniff
    DataServiceVersion: 1.0;
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET
    Date: Sun, 13 May 2012 05:16:53 GMT

    {"d":{"State":2}}


### <a name="cancel-a-job"></a>取消工作
媒體服務可讓您透過 CancelJob 函式取消正在執行的工作。 如果您嘗試取消的工作狀態為已取消、取消中、錯誤或已完成，這項呼叫將傳回 400 錯誤碼。

下列範例示範如何呼叫 CancelJob。

**HTTP 要求**

    GET https://wamsbayclus001rest-hs.net/API/CancelJob?jobid='nb%3ajid%3aUUID%3a71d2dd33-efdf-ec43-8ea1-136a110bd42c' HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.2
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: wamsbayclus001rest-hs.net


如果成功，會傳回 204 回應碼且沒有訊息主體。

> [!NOTE]
> 將工作識別碼做為參數傳遞到 CancelJob 時，您必須將工作識別碼進行 URL 編碼 (通常是 nb:jid:UUID: somevalue)。
>
>

### <a name="get-the-output-asset"></a>取得輸出資產
下列程式碼示範如何要求輸出資產識別碼。

**HTTP 要求**

    GET https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets() HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.17
    Host: wamsbayclus001rest-hs.cloudapp.net


**HTTP 回應**

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 445
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Server: Microsoft-IIS/8.5
    request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
    x-ms-request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 08:28:13 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets",
       "value":[  
          {  
             "Id":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
             "State":0,
             "Created":"2015-01-19T07:52:15.603",
             "LastModified":"2015-01-19T07:52:15.603",
             "AlternateId":null,
             "Name":"Multibitrate MP4s",
             "Options":0,
             "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c",
             "StorageAccountName":"storagetestaccount001"
          }
       ]
    }

## <a id="publish_get_urls"></a>使用 REST API 發行資產及取得串流和漸進式下載 URL

若要串流處理或下載資產，您必須先建立定位器來「發佈」它。 定位器提供對於資產中包含之檔案的存取。 媒體服務支援兩種類型的定位器：OnDemandOrigin 定位器，用於串流媒體 (例如，MPEG DASH、HLS 或 Smooth Streaming) 和存取簽章 (SAS) 定位器，用來下載媒體檔案。 

建立定位器之後，您便可以建立用來串流或下載檔案的 URL。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。

Smooth Streaming 的串流 URL 具有下列格式：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest

HLS 的串流 URL 具有下列格式：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

MPEG DASH 的串流 URL 具有下列格式：

    {streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)

用來下載檔案的 SAS URL 具有下列格式：

    {blob container name}/{asset name}/{file name}/{SAS signature}

本節示範如何執行「發佈」您的資產所需的下列工作。  

* 建立具有讀取權限的 AccessPolicy
* 建立下載內容用的 SAS URL
* 建立串流內容用的原始 URL

### <a name="creating-the-accesspolicy-with-read-permission"></a>建立具有讀取權限的 AccessPolicy
下載或串流任何媒體內容之前，請先定義具有讀取權限的 AccessPolicy，並建立適當的 Locator 實體，指定您想要為用戶端啟用的傳遞機制類型。 如需可用屬性的詳細資訊，請參閱 [AccessPolicy 實體屬性](https://docs.microsoft.com/rest/api/media/operations/accesspolicy#accesspolicy_properties)。

下列範例示範如何針對指定資產指定讀取權限的 AccessPolicy。

    POST https://wamsbayclus001rest-hs.net/API/AccessPolicies HTTP/1.1
    Content-Type: application/json
    Accept: application/json
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: wamsbayclus001rest-hs.net
    Content-Length: 74
    Expect: 100-continue

    {"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}

如果成功，會傳回 201 成功碼，描述您所建立的 AccessPolicy 實體。 然後，您將使用 AccessPolicy 識別碼以及資產的資產識別碼，其中此資產包含您要傳遞 (例如做為輸出資產) 以建立 Locator 實體的檔案。

> [!NOTE]
> 這個基本工作流程在擷取資產 時 (如本主題前面所討論) 與上傳檔案相同。 此外，就像上傳檔案，如果您 (或您的用戶端) 需要立即存取檔案，請將 StartTime 值設定為目前時間之前五分鐘。 需要進行此動作是因為用戶端電腦與媒體服務之間可能有時間差。 StartTime 值必須是以下日期時間格式：YYYY-MM-DDTHH:mm:ssZ (例如，"2014-05-23T17:53:50Z")。
>
>

### <a name="creating-a-sas-url-for-downloading-content"></a>建立下載內容用的 SAS URL
下列程式碼示範如何取得可以用來下載先前建立及上傳之媒體檔案的 URL。 AccessPolicy 具有讀取權限集，而定位器路徑指向 SAS 下載 URL。

    POST https://wamsbayclus001rest-hs.net/API/Locators HTTP/1.1
    Content-Type: application/json
    Accept: application/json
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: wamsbayclus001rest-hs.net
    Content-Length: 182
    Expect: 100-continue

    {"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c", "StartTime" : "2014-05-17T16:45:53", "Type":1}

如果成功，則會傳回下列回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 1150
    Content-Type: application/json;odata=verbose;charset=utf-8
    Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')
    Server: Microsoft-IIS/7.5
    x-ms-request-id: 8cfd8556-3064-416a-97f2-67d9e35f0911
    X-Content-Type-Options: nosniff
    DataServiceVersion: 1.0;
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET
    Date: Mon, 14 May 2012 21:41:32 GMT

    {  
       "d":{  
          "__metadata":{  
             "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
             "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
             "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
          },
          "AccessPolicy":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')/AccessPolicy"
             }
          },
          "Asset":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Asset"
             }
          },
          "Id":"nb:lid:UUID:8e5a821d-2194-4d00-8884-adf979856874",
          "ExpirationDateTime":"\/Date(1337049393000)\/",
          "Type":1,
          "Path":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c?st=2012-05-14T21%3A36%3A33Z&se=2012-05-15T02%3A36%3A33Z&sr=c&si=8e5a821d-2194-4d00-8884-adf979856874&sig=y75dViDpC5V8WutrXM%2B%2FGpR3uOtqmlISiNlHU1YUBOg%3D",
          "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
          "AssetId":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
          "StartTime":"\/Date(1337031393000)\/"
       }
    }

傳回的 **Path** 屬性包含 SAS URL。

> [!NOTE]
> 如果您下載儲存體加密內容，則必須手動將其解密才能呈現，或是在處理工作中使用儲存體解密 MediaProcessor，以純文字將處理的檔案輸出到 OutputAsset，然後從該資產下載。 如需有關處理的詳細資訊，請參閱＜使用媒體服務 REST API 建立編碼工作＞。 此外，已建立 SAS URL 定位器之後，無法更新它們。 例如，您無法以更新的 StartTime 值重複使用相同的定位器。 這是因為建立 SAS URL 的方式。 如果您想要在定位器過期之後存取資產以便下載，您必須用新的 StartTime 建立一個新的定位器。
>
>

### <a name="download-files"></a>下載檔案
設定 AccessPolicy 與 Locator 之後，您可以使用 Azure 儲存體 REST API 下載檔案。  

> [!NOTE]
> 您必須將要下載的檔案名稱新增到前一節中所收到的 Locator **Path** 值。 例如， https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . .
>
>

如需使用 Azure 儲存體 blob 的詳細資訊，請參閱 [Blob 服務 REST API](https://docs.microsoft.com/rest/api/storageservices/Blob-Service-REST-API)。

由於您先前執行的編碼工作 (編碼成自調適性 MP4 集)，您有多個可以漸進式下載的 MP4 檔案。 例如︰    

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

    https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z

### <a name="creating-a-streaming-url-for-streaming-content"></a>建立串流內容用的串流 URL
下列程式碼示範如何建立串流 URL 定位器：

    POST https://wamsbayclus001rest-hs/API/Locators HTTP/1.1
    Content-Type: application/json
    Accept: application/json
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.17
    Authorization: Bearer <ENCODED JWT TOKEN>
    Host: wamsbayclus001rest-hs
    Content-Length: 182
    Expect: 100-continue

    {"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3", "StartTime" : "2014-05-17T16:45:53",, "Type":2}

如果成功，則會傳回下列回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 981
    Content-Type: application/json;odata=verbose;charset=utf-8
    Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')
    Server: Microsoft-IIS/7.5
    x-ms-request-id: 2eac4158-fc78-4fa2-81ee-c9f582dc385f
    X-Content-Type-Options: nosniff
    DataServiceVersion: 1.0;
    X-AspNet-Version: 4.0.30319
    X-Powered-By: ASP.NET
    Date: Mon, 14 May 2012 21:41:39 GMT

    {  
       "d":{  
          "__metadata":{  
             "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
             "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
             "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
          },
          "AccessPolicy":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/AccessPolicy"
             }
          },
          "Asset":{  
             "__deferred":{  
                "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/Asset"
             }
          },
          "Id":"nb:lid:UUID:52034bf6-dfae-4d83-aad3-3bd87dcb1a5d",
          "ExpirationDateTime":"\/Date(1337049395000)\/",
          "Type":2,
          "Path":"http://wamsbayclus001rest-hs.net/52034bf6-dfae-4d83-aad3-3bd87dcb1a5d/",
          "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
          "AssetId":"nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3",
          "StartTime":"\/Date(1337031395000)\/"
       }
    }

若要在串流媒體播放器中串流 Smooth Streaming 原始 URL，您必須在 Path 屬性後附加 Smooth Streaming 資訊清單檔案的名稱，後面再接 "/manifest"。

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest

若要串流 HLS，請在 "/manifest" 之後附加 (format=m3u8-aapl) 。

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)

若要串流 MPEG DASH，請在 "/manifest" 之後附加 (format=mpd-time-csf) 。

    http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)


## <a id="play"></a>播放您的內容
若要串流您的視訊，請使用 [Azure 媒體服務播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)。

若要測試漸進式下載，請將 URL 貼入瀏覽器 (例如，IE、Chrome、Safari)。

## <a name="next-steps-media-services-learning-paths"></a>後續步驟：媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
