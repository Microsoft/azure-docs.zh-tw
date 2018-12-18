---
title: 使用 REST 將檔案上傳至 Azure 媒體服務帳戶 | Microsoft Docs
description: 了解如何建立並上傳資產，以將媒體內容移至媒體服務中。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: juliako
ms.openlocfilehash: 1e51439ec0a6c6658b28ae0f02ff3eaeb4c551e4
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34070414"
---
# <a name="upload-files-into-a-media-services-account-using-rest"></a>使用 REST 將檔案上傳至媒體服務帳戶
> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-upload-files.md)
> * [REST](media-services-rest-upload-files.md)
> * [入口網站](media-services-portal-upload-files.md)
> 

在媒體服務中，您會將數位檔案上傳到到資產。 [資產](https://docs.microsoft.com/rest/api/media/operations/asset)實體可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式輔助字幕檔案 (以及這些檔案的相關中繼資料)。一旦檔案會上傳到資產，您的內容會安全地儲存在雲端，以便進行進一步的處理和串流。 

在本教學課程中，您可以了解如何上傳檔案，以及其他相關聯的作業：

> [!div class="checklist"]
> * 為所有上傳作業設定 Postman
> * 連線到媒體服務 
> * 建立具有寫入權限的存取原則
> * 建立資產
> * 建立 SAS 定位器並建立上傳 URL
> * 使用上傳 URL 將檔案上傳至 blob 儲存體
> * 針對您上傳的媒體檔案，在資產中建立中繼資料

## <a name="prerequisites"></a>先決條件

- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) 。
- [使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。
- 請先複習＜[使用 AAD 驗證存取 Azure 媒體服務 API 概觀](media-services-use-aad-auth-to-access-ams-api.md)＞一文。
- 如＜[設定 Postman 以進行媒體服務 REST API 呼叫](media-rest-apis-with-postman.md)＞中所述，設定 **Postman**。

## <a name="considerations"></a>考量

使用媒體服務 REST API 時，適用下列考量事項：
 
* 使用媒體服務 REST API 存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。 <br/>本教學課程中使用的 Postman 集合會負責設定所有必要的標頭。
* 建置串流內容的 URL (例如，http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters) 時，媒體服務會使用 IAssetFile.Name 屬性的值。基於這個理由，不允許 percent-encoding。 **Name** 屬性的值不能有下列任何[百分比編碼保留字元](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#[]"。 而且，副檔名只能有一個 '.'。
* 名稱長度不應超過 260 個字元。
* 對於在媒體服務處理檔案，支援的檔案大小有上限。 請參閱[這篇](media-services-quotas-and-limitations.md)文章，以取得有關檔案大小限制的詳細資料。

## <a name="set-up-postman"></a>設定 Postman

如需如何設定 Postman 以用於本教學課程的步驟，請參閱[設定 Postman](media-rest-apis-with-postman.md)。

## <a name="connect-to-media-services"></a>連線到媒體服務

1. 將連線值新增至您的環境。 

    您需要手動設定一些屬於 **MediaServices** [環境](postman-environment.md)的變數，才能開始執行[集合](postman-collection.md)中定義的作業。

    若要取得前五個變數的值，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

    ![上傳檔案](./media/media-services-rest-upload-files/postman-import-env.png)
2. 指定 **MediaFileName** 環境變數的值。

    指定您計劃上傳的媒體檔案名稱。 在此範例中，我們將上傳 BigBuckBunny.mp4。 
3. 檢查 **AzureMediaServices.postman_environment.json** 檔案。 您將會看到集合中幾乎所有作業都執行著 "test" 指令碼。 指令碼會採用一些回應所傳回的值，並設定適當的環境變數。

    例如，第一項作業會取得存取權杖，並將其設定在其他所有作業使用的 **AccessToken** 環境變數上。

    ```    
    "listen": "test",
    "script": {
        "type": "text/javascript",
        "exec": [
            "var json = JSON.parse(responseBody);",
            "postman.setEnvironmentVariable(\"AccessToken\", json.access_token);"
        ]
    }
    ```
4. 在 **Postman** 視窗的左側，按一下 [1.取得 AAD 驗證權杖] -> [取得服務主體的 Azure AD 權杖]。

    URL 部分會填入 **AzureADSTSEndpoint** 環境變數 (在稍早的教學課程中，您已設定了支援[集合](#configure-the-collection)[環境變數](#configure-the-environment)的值)。

    ![上傳檔案](./media/media-services-rest-upload-files/postment-get-token.png)

5. 按 [傳送]。

    您可以看到包含 "access_token" 的回應。 "test" 指令碼會採用此值，並設定 **AccessToken** 環境變數 (如上方所述)。 如果檢查您的環境變數，您會看到此變數現在包含其餘作業中使用的存取權杖 (持有人權杖) 值。 

    如果權杖過期，請再次執行「取得服務主體的 Azure AD 權杖」的步驟。 

## <a name="create-an-access-policy-with-write-permission"></a>建立具有寫入權限的存取原則

### <a name="overview"></a>概觀 

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

將任何檔案上傳到 blob 儲存體之前，請設定寫入資產的存取原則權限。 若要這樣做，請 POST HTTP 要求到 AccessPolicies 實體集。 請在建立時定義 DurationInMinutes 值，否則您會在回應中收到 500 內部伺服器錯誤訊息。 如需 AccessPolicies 的詳細資訊，請參閱 [AccessPolicy](https://docs.microsoft.com/rest/api/media/operations/accesspolicy)。

### <a name="create-an-access-policy"></a>建立存取原則

1. 選取 [AccessPolicy] -> 為上傳檔案建立 AccessPolicy。
2. 按 [傳送]。

    ![上傳檔案](./media/media-services-rest-upload-files/postman-access-policy.png)

    "test" 指令碼會取得 AccessPolicy 識別碼，並設定適當的環境變數。

## <a name="create-an-asset"></a>建立資產

### <a name="overview"></a>概觀

[資產](https://docs.microsoft.com/rest/api/media/operations/asset)是媒體服務中多種類型或物件集的容器，包括視訊、音訊、影像、縮圖集合、文字播放軌和隱藏式字幕檔案。 在 REST API 中，建立資產必須傳送 POST 要求給媒體服務，並將關於您資產的任何屬性資訊放在要求主體中。

您可以在建立資產時新增的其中一個屬性是 **Options**。 您可以指定下列其中一個加密選項：**None** (預設值，不使用加密)、**StorageEncrypted** (適用於已透過用戶端儲存體加密進行預先加密的內容)、**CommonEncryptionProtected** 或 **EnvelopeEncryptionProtected**。 如果您有加密的資產，則需要設定傳遞原則。 如需詳細資訊，請參閱[設定資產傳遞原則](media-services-rest-configure-asset-delivery-policy.md)。

如果您的資產已加密，您必須建立 **ContentKey** 並將它連結到您的資產，如下列文章中所述：[如何建立 ContentKey](media-services-rest-create-contentkey.md)。 當您將檔案上傳到資產之後，您必須將 **AssetFile** 實體上的加密屬性更新為在**資產**加密期間得到的值。 請使用 **MERGE** HTTP 要求執行此作業。 

在此範例中，我們會建立未加密的資產。 

### <a name="create-an-asset"></a>建立資產

1. 選取 [資產] -> [建立資產]。
2. 按 [傳送]。

    ![上傳檔案](./media/media-services-rest-upload-files/postman-create-asset.png)

    "test" 指令碼會取得 Asset 識別碼，並設定適當的環境變數。

## <a name="create-a-sas-locator-and-create-the-upload-url"></a>建立 SAS 定位器並建立上傳 URL

### <a name="overview"></a>概觀

一旦設定 AccessPolicy 與 Locator，實際檔案會使用 Azure 儲存體 REST API 上傳至 Azure Blob 儲存容器。 您必須將檔案以區塊 Blob 形式上傳。 「Azure 媒體服務」不支援分頁 Blob。  

如需使用 Azure 儲存體 blob 的詳細資訊，請參閱 [Blob 服務 REST API](https://docs.microsoft.com/rest/api/storageservices/Blob-Service-REST-API)。

若要接收實際的上傳 URL，請建立 SAS 定位器 (如下所示)。 定位器為想要存取資產中之檔案的用戶端定義連線端點的開始時間和類型。 您可以為指定的 AccessPolicy 與 Asset 配對建立多個 Locator 實體，以處理不同的用戶端要求與需求。 這些 Locator 每個都會使用 StartTime 值加上 AccessPolicy 的 DurationInMinutes 值，以判斷可以使用 URL 的時間長度。 如需詳細資訊，請參閱 [定位器](https://docs.microsoft.com/rest/api/media/operations/locator)。

SAS URL 具有下列格式：

    {https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}

### <a name="considerations"></a>考量

適用一些考量事項：

* 您一次不能有超過五個唯一定位器與指定的資產相關聯。 如需詳細資訊，請參閱＜定位器＞。
* 如果您需要立即上傳檔案，您應該將 StartTime 值設為目前時間的五分鐘前。 這是因為用戶端電腦與媒體服務之間可能有時間差。 此外，您的 StartTime 值必須是以下日期時間格式：YYYY-MM-DDTHH:mm:ssZ (例如，"2014-05-23T17:53:50Z")。    
* 建立 Locator 之後到它可供使用時，中間可能會有 30 到 40 秒的延遲。

### <a name="create-a-sas-locator"></a>建立 SAS 定位器

1. 選取 [定位器] -> [建立 SAS 定位器]。
2. 按 [傳送]。

    "test" 指令碼會根據您指定的媒體檔案名稱和 SAS 定位器資訊建立「上傳 URL」，並設定適當的環境變數。

    ![上傳檔案](./media/media-services-rest-upload-files/postman-create-sas-locator.png)

## <a name="upload-a-file-to-blob-storage-using-the-upload-url"></a>使用上傳 URL 將檔案上傳至 blob 儲存體

### <a name="overview"></a>概觀

現在您已經有了上傳 URL，您需要直接使用 Azure Blob API 來撰寫一些程式碼，將您的檔案上傳至 SAS 容器。 如需詳細資訊，請參閱下列文章：

- [使用 Azure 儲存體 REST API](https://docs.microsoft.com/azure/storage/common/storage-rest-api-auth?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [PUT Blob](https://docs.microsoft.com/rest/api/storageservices/put-blob)
- [將 blob 上傳至 blob 儲存體](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy#upload-blobs-to-blob-storage)

### <a name="upload-a-file-with-postman"></a>使用 Postman 上傳檔案

我們會使用 Postman 上傳小型的 .mp4 檔案作為範例。 透過 Postman 上傳二進位檔可能會有檔案大小限制。

上傳要求不是 **AzureMedia** 集合的一部份。 

建立並設定新的要求：
1. 按 **+**，以建立新的要求索引標籤。
2. 選取 **PUT** 作業並在 URL 中貼上 **{{UploadURL}}**。
2. [授權] 索引標籤保持不變 (請勿將其設為**持有人權杖**)。
3. 在 [標頭] 索引標籤中，指定：**金鑰**："x-ms-blob-type" 和**值**："BlockBlob"。
2. 在 [本文] 索引標籤上，按一下 [二進位]。
4. 根據您在 **MediaFileName** 環境變數中指定的名稱選擇檔案。
5. 按 [傳送]。

    ![上傳檔案](./media/media-services-rest-upload-files/postman-upload-file.png)

##  <a name="create-a-metadata-in-the-asset"></a>在資產中建立中繼資料

一旦檔案已上傳，您需要針對上傳到 blob 儲存體 (與資產相關聯) 的媒體檔案，在資產中建立中繼資料。

1. 選取 [Assetfile] -> [CreateFileInfos]。
2. 按 [傳送]。

    ![上傳檔案](./media/media-services-rest-upload-files/postman-create-file-info.png)

應上傳此檔案，並設定其中繼資料。

## <a name="validate"></a>驗證

若要驗證檔案是否已上傳成功，您可以查詢 [AssetFile](https://docs.microsoft.com/rest/api/media/operations/assetfile)，並比對 **ContentFileSize** (或其他詳細資料) 和您預期在新資產中看到的內容。 

例如，下列 **GET** 作業會帶出資產檔案的檔案資料 (也就是我們案例中的 BigBuckBunny.mp4 檔案)。 查詢會使用您稍早設定的[環境變數](postman-environment.md)。

    {{RESTAPIEndpoint}}/Assets('{{LastAssetId}}')/Files

回應會包含大小、名稱和其他資訊。

    "Id": "nb:cid:UUID:69e72ede-2886-4f2a-8d36-80a59da09913",
    "Name": "BigBuckBunny.mp4",
    "ContentFileSize": "3186542",
    "ParentAssetId": "nb:cid:UUID:0b8f3b04-72fb-4f38-8e7b-d7dd78888938",
            
## <a name="next-steps"></a>後續步驟

您現在可以將上傳的資產編碼。 如需詳細資訊，請參閱 [為資產編碼](media-services-portal-encode.md)。

您也可以使用 Azure Functions，以根據在所設定容器到達的檔案來觸發編碼作業。 如需詳細資訊，請參閱[此範例](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ )。

