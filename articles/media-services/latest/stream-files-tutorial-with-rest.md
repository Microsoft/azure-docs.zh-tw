---
title: 使用 Azure 媒體服務執行上傳、編碼和串流 | Microsoft Docs
description: 遵循此教學課程的步驟，使用 Azure 媒體服務和 REST 來上傳檔案、編碼視訊及串流處理內容。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: e49b450ef2c731e9ddbafa0c8366d9eae29dc5ef
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377422"
---
# <a name="tutorial-upload-encode-and-stream-videos-with-rest"></a>教學課程：使用 REST 上傳、編碼和串流處理視訊

本教學課程說明如何使用 Azure 媒體服務來上傳、編碼和串流處理視訊檔案。

媒體服務可讓您將媒體檔案編碼成可在各種不同的瀏覽器和裝置上播放的格式。 例如，您可能會想要串流處理 Apple 的 HLS 或 MPEG DASH 格式的內容。 在進行串流處理之前，您應先編碼高品質數位媒體檔案。 如需編碼指引，請參閱[編碼概念](encoding-concept.md)。

![播放影片](./media/stream-files-tutorial-with-api/final-video.png)

本教學課程說明如何：    

> [!div class="checklist"]
> * 建立媒體服務帳戶
> * 存取媒體服務 API
> * 下載 Postman 檔案
> * 設定 Postman
> * 使用 Postman 傳送要求
> * 測試串流 URL
> * 清除資源

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

- 安裝 [Postman](https://www.getpostman.com/) \(英文\) REST 用戶端，來執行在某些 AMS REST 教學課程中所示範的 REST API。 

    我們使用的是 **Postman**，但任何 REST 工具都適用。 其他替代方案為：搭配 REST 外掛程式的 **Visual Studio Code**，或 **Telerik Fiddler**。 

## <a name="download-postman-files"></a>下載 Postman 檔案

複製包含 Postman 集合和環境檔案的 GitHub 存放庫。

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

[!INCLUDE [media-services-cli-create-v3-account-include](../../../includes/media-services-cli-create-v3-account-include.md)]

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

## <a name="configure-postman"></a>設定 Postman

此區段會設定 Postman。

### <a name="configure-the-environment"></a>建立環境 

1. 開啟 **Postman**。
2. 選取畫面右側的 [管理環境] 選項。

    ![管理 env](./media/develop-with-postman/postman-import-env.png)
4. 按一下 [管理環境] 對話方塊中的 [匯入]。
2. 瀏覽至您在複製 `https://github.com/Azure-Samples/media-services-v3-rest-postman.git` 時下載的 `Azure Media Service v3 Environment.postman_environment.json` 檔案。
6. 新增 **Azure Media Service v3 Environment** 環境。

    > [!Note]
    > 使用您從前述的**存取媒體服務 API** 一節中取得的值來更新存取變數。

7. 按兩下選取的檔案，並輸入您依照[存取 API](#access-the-media-services-api) 步驟所取得的值。
8. 關閉對話方塊。
9. 從下拉式清單中選取 **Azure Media Service v3 Environment** 環境。

    ![選擇 env](./media/develop-with-postman/choose-env.png)
   
### <a name="configure-the-collection"></a>設定集合

1. 按一下 [匯入] 來匯入集合檔案。
1. 瀏覽至您在複製 `https://github.com/Azure-Samples/media-services-v3-rest-postman.git` 時下載的 `Media Services v3.postman_collection.json` 檔案
3. 選擇 **Media Services v3.postman_collection.json** 檔案。

    ![匯入檔案](./media/develop-with-postman/postman-import-collection.png)

## <a name="send-requests-using-postman"></a>使用 Postman 傳送要求

在本節中，我們會傳送與編碼和建立 URL 有關的要求，讓您能夠串流處理檔案。 具體來說，我們會傳送下列要求：

1. 取得服務主體驗證的 Azure AD 權杖
2. 建立輸出資產
3. 建立轉換
4. 建立工作 
5. 建立串流定位器
6. 列出串流定位器的路徑

> [!Note]
>  本教學課程假設您會使用唯一名稱建立所有資源。  

### <a name="get-azure-ad-token"></a>取得 Azure AD 權杖 

1. 在 Postman 的左側視窗中，選取 [步驟 1：取得 AAD 驗證權杖]。
2. 然後，選取 [取得服務主體驗證的 Azure AD 權杖]。
3. 按 [傳送]。

    此時會傳送下列 **POST** 作業。

    ```
    https://login.microsoftonline.com/:tenantId/oauth2/token
    ```

4. 傳回的回應會包含權杖，並將 "AccessToken" 環境變數設定為權杖值。 若要查看設定 "AccessToken" 的程式碼，請按一下 [測試] 索引標籤。 

    ![取得 AAD 權杖](./media/develop-with-postman/postman-get-aad-auth-token.png)

### <a name="create-an-output-asset"></a>建立輸出資產

輸出[資產](https://docs.microsoft.com/rest/api/media/assets)會儲存您的編碼作業結果。 

1. 在 Postman 的左側視窗中，選取 [資產]。
2. 然後，選取 [建立或更新資產]。
3. 按 [傳送]。

    * 此時會傳送下列 **PUT** 作業：

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/assets/:assetName?api-version={{api-version}}
        ```
    * 作業會具有下列主體：

        ```json
        {
        "properties": {
            "description": "My Asset",
            "alternateId" : "some GUID"
         }
        }
        ```

### <a name="create-a-transform"></a>建立轉換

在媒體服務中編碼或處理內容時，將編碼設定設為配方 (recipe) 是很常見的模式。 然後您可以透過提交**作業**，將該配方套用到影片。 藉由為每部新影片提交新的作業，您可以將該配方套用到媒體櫃中的所有影片。 配方在媒體服務中稱為「**轉換 (Transform)**」。 如需詳細資訊，請參閱[轉換和作業](transform-concept.md)。 本教學課程中所述的範例會定義編碼影片的配方，以便將影片串流到各種 iOS 和 Android 裝置。 

建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)執行個體時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件。 每個 **TransformOutput** 都會包含 **Preset (預設)**。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 本文中所述的範例會使用稱為 **AdaptiveStreaming** 的內建 Preset。 Preset 會根據輸入解析度和位元速率，將輸入影片編碼為自動產生的位元速率階梯 (位元速率-解析度配對)，並產生 H.264 影片與 AAC 音訊標準 (對應到每個 位元速率-解析度配對) 的 ISO MP4 檔案。 如需此 Preset 的相關資訊，請參閱[自動產生位元速率階梯](autogen-bitrate-ladder.md)。

您可以使用內建的 EncoderNamedPreset 或使用自訂預設值。 

> [!Note]
> 建立[轉換](https://docs.microsoft.com/rest/api/media/transforms)時，您應該先使用 **Get** 方法檢查是否已有轉換存在。 本教學課程假設您會使用唯一名稱建立轉換。

1. 在 Postman 的左側視窗中，選取 [編碼和分析]。
2. 然後，選取 [建立轉換]。
3. 按 [傳送]。

    * 此時會傳送下列 **PUT** 作業。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
        ```
    * 作業會具有下列主體：

        ```json
        {
            "properties": {
                "description": "Basic Transform using an Adaptive Streaming encoding preset from the libray of built-in Standard Encoder presets",
                "outputs": [
                    {
                    "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                    "preset": {
                        "@odata.type": "#Microsoft.Media.BuiltInStandardEncoderPreset",
                        "presetName": "AdaptiveStreaming"
                    }
                    }
                ]
            }
        }
        ```

### <a name="create-a-job"></a>建立工作

[作業](https://docs.microsoft.com/rest/api/media/jobs)是要媒體服務將已建立的轉換套用至指定輸入視訊或音訊內容的實際要求。 **作業**會指定輸入影片的位置、輸出的位置等資訊。

在此範例中，作業的輸入會以 HTTPS URL ("https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/") 為基礎。

1. 在 Postman 的左側視窗中，選取 [編碼和分析]。
2. 然後，選取 [建立或更新作業]。
3. 按 [傳送]。

    * 此時會傳送下列 **PUT** 作業。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName/jobs/:jobName?api-version={{api-version}}
        ```
    * 作業會具有下列主體：

        ```json
        {
        "properties": {
            "input": {
            "@odata.type": "#Microsoft.Media.JobInputHttp",
            "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
            "files": [
                    "Ignite-short.mp4"
                ]
            },
            "outputs": [
            {
                "@odata.type": "#Microsoft.Media.JobOutputAsset",
                "assetName": "testAsset1"
            }
            ]
        }
        }
        ```

此作業需要一些時間來完成，而您可以選擇在完成時收到通知。 若要查看作業的進度，建議您使用事件方格。 事件方格是為高可用性、一致效能及動態調整而設計的。 透過事件方格，您的應用程式幾乎可以從所有 Azure 服務和自訂來源接聽及回應事件。 以 HTTP 為基礎的簡單回應式事件處理，可協助您透過智慧型事件篩選和路由來建置有效率的解決方案。  請參閱[將事件路由至自訂 Web 端點](job-state-events-cli-how-to.md)。

**作業**通常會經歷下列狀態：**已排程**、**已排入佇列**、**正在處理**、**已完成** (最後一個狀態)。 如果作業發生錯誤，您會收到**錯誤**狀態。 如果正在取消作業，您會收到**正在取消**的狀態，以及完成時的**已取消**狀態。

### <a name="create-a-streaming-locator"></a>建立串流定位器

編碼作業完成後，下一個步驟是要讓用戶端可播放輸出資產中的視訊。 您可以透過兩個步驟來執行此動作：第一步，建立[StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators)，第二步，建置用戶端可以使用的串流 URL。 

建立 **StreamingLocator** 的程序稱為發佈。 根據預設，**StreamingLocator** 會在進行 API 呼叫後立即生效，而且會持續運作到遭到刪除為止 (除非您有設定選擇性的開始和結束時間)。 

建立 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 時，您必須指定需要的 **StreamingPolicyName**。 在此範例中，您將串流處理乾淨 (或未加密的) 內容，而使用預先定義的乾淨串流原則 (**PredefinedStreamingPolicy.ClearStreamingOnly**)。

> [!IMPORTANT]
> 使用自訂的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 時，您應該為媒體服務帳戶設計一組受限的這類原則，並且在需要相同的加密選項和通訊協定時，對 StreamingLocators 重新使用這些原則。 

媒體服務帳戶有 StreamingPolicy 項目的數量配額。 不建議您對每個 StreamingLocator 建立新的 StreamingPolicy。

1. 在 Postman 的左側視窗中，選取 [串流原則]。
2. 然後，選取 [建立串流定位器]。
3. 按 [傳送]。

    * 此時會傳送下列 **PUT** 作業。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingPolicies/:streamingPolicyName?api-version={{api-version}}
        ```
    * 作業會具有下列主體：

        ```json
        {
            "properties":{
            "assetName": "{{assetName}}",
            "streamingPolicyName": "{{streamingPolicyName}}"
            }
        }
        ```

### <a name="list-paths-and-build-streaming-urls"></a>列出路徑並建置串流 URL

#### <a name="list-paths"></a>列出路徑

建立了 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 之後，現在您可以取得串流 URL

1. 在 Postman 的左側視窗中，選取 [串流原則]。
2. 然後，選取 [列出路徑]。
3. 按 [傳送]。

    * 此時會傳送下列 **POST** 作業。

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName/listPaths?api-version={{api-version}}
        ```
        
    * 此作業沒有主體：
        
4. 記下其中一個要用於串流處理的路徑，您將在下一節中使用該路徑。 在此案例中會傳回下列路徑：
    
    ```
    "streamingPaths": [
        {
            "streamingProtocol": "Hls",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)"
            ]
        },
        {
            "streamingProtocol": "Dash",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=mpd-time-csf)"
            ]
        },
        {
            "streamingProtocol": "SmoothStreaming",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest"
            ]
        }
    ]
    ```

#### <a name="build-the-streaming-urls"></a>建置串流 URL

在本節中，我們將建置 HLS 串流 URL。 URL 由下列值組成：

1. 用來傳送資料的通訊協定。 在此案例中為 "HTTPS"。

    > [!NOTE]
    > 如果播放程式裝載在 HTTPS 網站上，請務必將 URL 更新為 "https"。

2. StreamingEndpoint 的主機名稱。 在此案例中，名稱為 "amsaccount-usw22.streaming.media.azure.net"。

    若要取得主機名稱，您可以使用下列 GET 作業：
    
    ```
    https://management.azure.com/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingEndpoints/default?api-version={{api-version}}
    ```
    
3. 您在上一節 (列出路徑) 中取得的路徑。  

因此，會建置下列 HLS URL

```
https://amsaccount-usw22.streaming.media.azure.net/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)
```

## <a name="test-the-streaming-url"></a>測試串流 URL


> [!NOTE]
> 確定要作為串流來源的串流端點正在執行中。

本文使用 Azure 媒體播放器測試串流。 

1. 開啟瀏覽器並巡覽至 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在 [URL:] 方塊中，貼上您所建置的 URL。 
3. 按一下 [更新播放程式]。

Azure 媒體播放器可以用於測試，但不應用於生產環境。 

## <a name="clean-up-resources-in-your-media-services-account"></a>清除媒體服務帳戶中的資源

一般而言，您應清除所有項目，只保留想要重複使用的物件 (您通常會重複使用轉換並且保存 StreamingLocators 等)。 如果您想要在實驗之後有乾淨的帳戶，您應刪除不打算重複使用的資源。  

若要刪除資源，請在您要刪除的任何資源下方選取 [刪除...] 作業。

## <a name="clean-up-resources"></a>清除資源

如果您不再需要資源群組中的任何資源 (包含您為教學課程建立的媒體服務和儲存體帳戶)，請將稍早建立的資源群組刪除。 您可以使用 [CloudShell] 工具。

在 **CloudShell** 中，執行以下命令：

```azurecli-interactive
az group delete --name amsResourceGroup
```

## <a name="next-steps"></a>後續步驟

在您已知道如何上傳、編碼和串流影片後，請參閱下列文章： 

> [!div class="nextstepaction"]
> [分析影片](analyze-videos-tutorial-with-api.md)
