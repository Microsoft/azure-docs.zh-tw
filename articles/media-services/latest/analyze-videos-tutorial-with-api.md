---
title: 使用 Azure 媒體服務分析影片 | Microsoft Docs
description: 請遵循本教學課程的步驟，使用 Azure 媒體服務分析影片。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: 96ed5c17501aeb195ee584ce6d41a84fc2118bac
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376647"
---
# <a name="tutorial-analyze-videos-with-azure-media-services"></a>教學課程：使用 Azure 媒體服務分析影片 

本教學課程會示範如何使用 Azure 媒體服務分析影片。 在許多情況下，您會想從記錄影片或音訊內容中取得深入的見解。 例如，為達到更高的客戶滿意度，組織可以執行語音轉文字處理，將客戶支援記錄轉換為可搜尋的目錄，且其中可包含索引及儀表板。 然後，他們可以取得有關其業務的見解，例如常見的抱怨、這類抱怨的來源等清單。

本教學課程說明如何：    

> [!div class="checklist"]
> * 建立媒體服務帳戶
> * 存取媒體服務 API
> * 設定範例應用程式
> * 檢查用來分析指定視訊的程式碼
> * 執行應用程式
> * 檢查輸出
> * 清除資源

> [!Note]
> 使用 Azure 入口網站 (如[調整媒體處理](../previous/media-services-scale-media-processing-overview.md)所述) 將您的媒體服務帳戶設為 10 個 S3 媒體保留單元。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

如果沒有安裝 Visual Studio，您可以取得 [Visual Studio Community 2017](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15)。

## <a name="download-the-sample"></a>下載範例

使用以下命令將包含 .NET 範例的 GitHub 存放庫複製到您的機器：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

此範例位於 [AnalyzeVideos](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/AnalyzeVideos) 資料夾中。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

[!INCLUDE [media-services-cli-create-v3-account-include](../../../includes/media-services-cli-create-v3-account-include.md)]

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

## <a name="examine-the-code-that-analyzes-the-specified-video"></a>檢查用來分析指定視訊的程式碼

本節將針對 AnalyzeVideos 專案的 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/Program.cs) 檔案，檢查其中定義的函式。

此範例會執行下列動作：

1. 建立用來分析視訊的轉換和作業。
2. 建立輸入資產，並將視訊上傳到其中。 此資產會作為作業的輸入。
3. 建立輸出資產，用以儲存作業的輸出。 
4. 提交作業。
5. 檢查作業的狀態。
6. 下載執行作業所產生的檔案。 

### <a name="start-using-media-services-apis-with-net-sdk"></a>開始搭配使用媒體服務 API 與 .NET SDK

若要開始搭配使用媒體服務 API 與 .NET，您需要建立 **AzureMediaServicesClient** 物件。 若要建立物件，您需要提供必要的認證，讓用戶端使用 Azure AD 連線至 Azure。 在您於本文一開始複製的程式碼中，**GetCredentialsAsync** 函式會根據本機組態檔中提供的認證建立 ServiceClientCredentials 物件。 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>建立輸入資產並將本機檔案上傳到其中 

**CreateInputAsset** 函式會建立新的輸入[資產](https://docs.microsoft.com/rest/api/media/assets)，並將指定的本機影片檔案上傳到其中。 這項資產會用來作為編碼作業的輸入。 在媒體服務 v3 中，作業的輸入可以是資產，或是您透過 HTTPS URL 讓媒體服務帳戶可存取的內容。 如果您想要了解如何從 HTTPS URL 進行編碼，請參閱[這篇](job-input-from-http-how-to.md)文章。  

在媒體服務 v3 中，您會使用 Azure 儲存體 API 來上傳檔案。 下列 .NET 程式碼片段將說明執行方式。

下列函式會執行下列動作：

* 建立資產 
* 取得可寫入的 [SAS URL](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)，以存取[儲存體中的資產容器](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-dotnet?tabs=windows#upload-blobs-to-the-container)
* 使用 SAS URL，將檔案上傳至儲存體中的容器

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-the-job"></a>建立要儲存作業結果的輸出資產 

輸出[資產](https://docs.microsoft.com/rest/api/media/assets)會儲存您的作業結果。 專案會定義 **DownloadResults** 函式，以將結果從此輸出資產下載至「輸出」資料夾，因此您可以看到結果。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-analyzes-videos"></a>建立分析影片的轉換和作業

在媒體服務中編碼或處理內容時，將編碼設定設為配方 (recipe) 是很常見的模式。 然後您可以透過提交**作業**，將該配方套用到影片。 藉由為每部新影片提交新的作業，您可以將該配方套用到媒體櫃中的所有影片。 配方在媒體服務中稱為「**轉換 (Transform)**」。 如需詳細資訊，請參閱[轉換和作業](transform-concept.md)。 本教學課程中所述的範例會定義用來分析指定影片的配方。 

#### <a name="transform"></a>轉換

建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)執行個體時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件，如上方程式碼所示。 每個 **TransformOutput** 都會包含 **Preset (預設)**。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 此範例中會使用 **VideoAnalyzerPreset** 預設，以及將語言 ("en-US") 傳遞至其建構函式。 此預設可讓您從影片擷取多個音訊和影片見解。 如果您需要從影片擷取多個音訊見解，可以使用 **AudioAnalyzerPreset** 預設。 

建立**轉換**時，您應該先使用 **Get** 方法檢查是否已有轉換存在，如下列程式碼所示。  在媒體服務 v3 中，如果實體不存在，對實體執行的 **Get** 方法會傳回 **null** (檢查名稱時不區分大小寫)。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>工作 (Job)

如同前面所述，[轉換](https://docs.microsoft.com/rest/api/media/transforms)物件是配方，而[作業](https://docs.microsoft.com/rest/api/media/jobs)則是實際要求媒體服務，將**轉換**套用至指定的輸入影片或音訊內容。 **作業**會指定輸入影片的位置、輸出的位置等資訊。 您可以使用下列項目指定影片的位置：HTTPS URL、SAS URL 或媒體服務帳戶的資產。 

在此範例中，工作輸入是本機影片。  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>請等待作業完成

此作業需要一些時間來完成，而您可以選擇在完成時收到通知。 取得[作業](https://docs.microsoft.com/rest/api/media/jobs)完成通知的選項有很多種。 最簡單的選項 (如此處所示) 是使用輪詢。 

對生產應用程式而言，輪詢不是建議的最佳做法，因為可能會發生延遲。 如果過度使用帳戶，輪詢可能會進行節流處理。 開發人員應改為使用事件方格。

事件方格專為高可用性、一致效能及動態調整而設計。 透過事件方格，您的應用程式幾乎可以從所有 Azure 服務和自訂來源接聽及回應事件。 以 HTTP 為基礎的簡單回應式事件處理，可協助您透過智慧型事件篩選和路由來建置有效率的解決方案。 請參閱[將事件路由至自訂 Web 端點](job-state-events-cli-how-to.md)。

**作業**通常會經歷下列狀態：**已排程**、**已排入佇列**、**正在處理**、**已完成** (最後一個狀態)。 如果作業發生錯誤，您會收到**錯誤**狀態。 如果正在取消作業，您會收到**正在取消**的狀態，以及完成時的**已取消**狀態。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#WaitForJobToFinish)]

### <a name="download-the-result-of-the-job"></a>下載作業結果

下列函式會將結果從輸出[資產](https://docs.microsoft.com/rest/api/media/assets)下載至「輸出」資料夾，因此您可以檢查作業的結果。 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#DownloadResults)]

### <a name="clean-up-resource-in-your-media-services-account"></a>清除媒體服務帳戶中的資源

一般而言，您應清除所有項目，而只保留您預計要重複使用的物件 (您通常會重複使用轉換並且保存 StreamingLocators)。 如果您想要在實驗之後有乾淨的帳戶，您應刪除不打算重複使用的資源。 例如，下列程式碼會刪除作業。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>執行範例應用程式

按 Ctrl+F5 執行 *AnalyzeVideos* 應用程式。

當我們執行程式時，此作業會為影片中找到的每個臉部製作縮圖。 並且也會產生 insights.json 檔案。

## <a name="examine-the-output"></a>檢查輸出

分析影片的輸出檔稱為 insights.json。 此檔案包含影片的見解。 您可以在[媒體智慧](intelligence-concept.md)一文中，為 json 檔案中發現的元素找到說明。

## <a name="clean-up-resources"></a>清除資源

如果您不再需要資源群組中的任何資源 (包含您為教學課程建立的媒體服務和儲存體帳戶)，請將稍早建立的資源群組刪除。 您可以使用 [CloudShell] 工具。

在 **CloudShell** 中，執行以下命令：

```azurecli-interactive
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>多執行緒

Azure 媒體服務 v3 SDK 不是安全執行緒。 當使用多執行緒應用程式時，您應該為每個執行緒產生新的 AzureMediaServicesClient 物件。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [教學課程：上傳、編碼和串流檔案](stream-files-tutorial-with-api.md)
