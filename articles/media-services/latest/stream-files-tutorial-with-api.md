---
標題：使用媒體服務 v3 上傳、編碼和串流： Azure 媒體服務描述：示範如何使用 Azure 媒體服務 v3 上傳檔案、編碼影片及串流內容的教學課程。
服務： media services documentationcenter： ' ' author： IngridAtMicrosoft manager： femila editor： ' '

ms. 服務：媒體-服務 ms. 工作負載： ms. 主題：教學課程 ms. 自訂： mvc ms. 日期： 08/31/2020 ms. 作者： inhenkel
---

# <a name="tutorial-upload-encode-and-stream-videos-with-media-services-v3"></a>教學課程：使用媒體服務 v3 上傳、編碼和串流處理影片

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> 雖然本教學課程使用 [.NET SDK](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet) 範例，但是 [REST API](/rest/api/media/liveevents)、[CLI](/cli/azure/ams/live-event?view=azure-cli-latest) 或其他受支援 [SDK](media-services-apis-overview.md#sdks) 的一般步驟都相同。

Azure 媒體服務可讓您將媒體檔案編碼成可在各種不同的瀏覽器和裝置上播放的格式。 例如，您可能會想要串流處理 Apple 的 HLS 或 MPEG DASH 格式的內容。 在進行串流處理之前，您應先編碼高品質數位媒體檔案。 如需編碼的說明，請參閱[編碼概念](encoding-concept.md)。 本教學課程會上傳本機視訊檔案，並編碼上傳的檔案。 您也可以編碼可透過 HTTPS URL 存取的內容。 如需詳細資訊，請參閱[從 HTTP URL 建立作業輸入](job-input-from-http-how-to.md)。

![使用 Azure 媒體播放器播放影片](./media/stream-files-tutorial-with-api/final-video.png)

本教學課程說明如何：

> [!div class="checklist"]
> * 下載本主題中說明的範例應用程式。
> * 檢查上傳、編碼和串流處理的程式碼。
> * 執行應用程式。
> * 測試串流 URL。
> * 清除資源。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

- 如果您未安裝 Visual Studio，您可以取得 [Visual Studio Community 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15)。
- [建立媒體服務帳戶](./create-account-howto.md)。<br/>請務必記住您用於資源群組名稱和「媒體服務」帳戶名稱的值。
- 請依照[使用 Azure CLI 存取 Azure 媒體服務 API](./access-api-howto.md) 中的步驟，並儲存認證。 您必須使用這些認證來存取 API。

## <a name="download-and-set-up-the-sample"></a>下載並設定範例

使用下列命令，將具有串流 .NET 範例的 GitHub 存放庫複製到您的機器：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

此範例位於 [UploadEncodeAndStreamFiles](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/UploadEncodeAndStreamFiles) 資料夾中。

在您下載的專案中開啟 [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/appsettings.json)。 將值取代為從[存取 API](./access-api-howto.md) 中取得的認證。

## <a name="examine-the-code-that-uploads-encodes-and-streams"></a>檢查上傳、編碼和串流處理的程式碼

本節將針對 UploadEncodeAndStreamFiles 專案的 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs) 檔案，檢查其中定義的函式。

此範例會執行下列動作：

1. 建立新的 **轉換** (首先，檢查指定的轉換是否存在)。
2. 建立輸出 **資產**，作為編碼 **作業** 的輸出。
3. 建立輸入 **資產**，並將指定的本機視訊檔案上傳到其中。 此資產會作為作業的輸入。
4. 使用已建立的輸入和輸出提交編碼作業。
5. 檢查作業的狀態。
6. 建立 **串流定位器**。
7. 建置串流 URL。

### <a name="start-using-media-services-apis-with-net-sdk"></a>開始搭配使用媒體服務 API 與 .NET SDK

若要開始搭配使用媒體服務 API 與 .NET，您需要建立 **AzureMediaServicesClient** 物件。 若要建立物件，您需要提供必要的認證，讓用戶端使用 Azure AD 連線至 Azure。 在您於本文一開始複製的程式碼中，**GetCredentialsAsync** 函式會根據本機組態檔中提供的認證建立 ServiceClientCredentials 物件。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>建立輸入資產並將本機檔案上傳到其中

**CreateInputAsset** 函式會建立新的輸入 [資產](/rest/api/media/assets)，並將指定的本機影片檔案上傳到其中。 這項 **資產** 會用來作為編碼作業的輸入。 在媒體服務 v3 中，**作業** 的輸入可以是 **資產**，或是您透過 HTTPS URL 讓媒體服務帳戶可存取的內容。 若要了解如何從 HTTPS URL 進行編碼，請參閱[這篇](job-input-from-http-how-to.md)文章。

在媒體服務 v3 中，您會使用 Azure 儲存體 API 來上傳檔案。 下列 .NET 程式碼片段將說明執行方式。

下列函式會執行下列動作：

* 建立 **資產**。
* 取得可寫入的 [SAS URL](../../storage/common/storage-sas-overview.md)，以存取[儲存體中的資產容器](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container)。

    如果使用資產的 [ListContainerSas](/rest/api/media/assets/listcontainersas) 函式來取得 SAS URL，請注意到此函式會傳回多個 SAS URL，因為每個儲存體帳戶都有兩個儲存體帳戶金鑰。 儲存體帳戶之所以有兩個金鑰，是因為其允許無縫輪替儲存體帳戶金鑰 (例如，在使用一個金鑰時變更另一個金鑰，然後開始使用新金鑰並輪替另一個金鑰)。 第一個 SAS URL 代表儲存體金鑰 1，而第二個代表儲存體金鑰 2。
* 使用 SAS URL，將檔案上傳至儲存體中的容器。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-a-job"></a>建立要儲存作業結果的輸出資產

輸出[資產](/rest/api/media/assets)會儲存您的編碼作業結果。 專案會定義 **DownloadResults** 函式，以將結果從此輸出資產下載至「輸出」資料夾，因此您可以看到結果。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>建立轉換和編碼上傳檔案的作業

在媒體服務中編碼或處理內容時，將編碼設定設為配方 (recipe) 是很常見的模式。 然後您可以透過提交 **作業**，將該配方套用到影片。 藉由為每部新影片提交新的作業，您可以將該配方套用到媒體櫃中的所有影片。 配方在媒體服務中稱為 **轉換**。 如需詳細資訊，請參閱[轉換和作業](./transforms-jobs-concept.md)。 本教學課程中所述的範例會定義編碼影片的配方，以便將影片串流到各種 iOS 和 Android 裝置。

#### <a name="transform"></a>轉換

建立新的[轉換](/rest/api/media/transforms)執行個體時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件，如下列程式碼所示。 每個 **TransformOutput** 都會包含 **Preset (預設)** 。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 本文中所述的範例會使用稱為 **AdaptiveStreaming** 的內建 Preset。 Preset 會根據輸入解析度和位元速率，將輸入影片編碼為自動產生的位元速率階梯 (位元速率-解析度配對)，並產生 H.264 影片與 AAC 音訊標準 (對應到每個 位元速率-解析度配對) 的 ISO MP4 檔案。 如需此 Preset 的相關資訊，請參閱[自動產生位元速率階梯](autogen-bitrate-ladder.md)。

您可以使用內建的 EncoderNamedPreset 或使用自訂預設值。 如需詳細資訊，請參閱[如何自訂編碼器預設值](customize-encoder-presets-how-to.md)。

建立 [轉換](/rest/api/media/transforms)時，您應該先使用 **Get** 方法檢查是否已有轉換存在，如下列程式碼所示。 在媒體服務 v3 中，如果實體不存在，對實體執行的 **Get** 方法會傳回 **null** (檢查名稱時不區分大小寫)。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>工作 (Job)

如同前面所述，[轉換](/rest/api/media/transforms)物件是配方，而 [作業](/rest/api/media/jobs)則是實際要求媒體服務，將 **轉換** 套用至指定的輸入影片或音訊內容。 **作業** 會指定輸入影片的位置、輸出的位置等資訊。

在此範例中，輸入影片已從本機電腦上傳。 如果您想要了解如何從 HTTPS URL 進行編碼，請參閱[這篇](job-input-from-http-how-to.md)文章。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>請等待作業完成

此作業需要一些時間來完成，而您可以選擇在完成時收到通知。 下列程式碼範例說明如何輪詢服務以取得[作業](/rest/api/media/jobs)狀態。 對生產應用程式而言，輪詢不是建議的最佳做法，因為可能會發生延遲。 如果過度使用帳戶，輪詢可能會進行節流處理。 開發人員應改為使用事件方格。

事件方格專為高可用性、一致效能及動態調整而設計。 透過事件方格，您的應用程式幾乎可以從所有 Azure 服務和自訂來源接聽及回應事件。 以 HTTP 為基礎的簡單回應式事件處理，可協助您透過智慧型事件篩選和路由來建置有效率的解決方案。  請參閱[將事件路由至自訂 Web 端點](job-state-events-cli-how-to.md)。

**作業** 通常會經歷下列狀態：**已排程**、**已排入佇列**、**正在處理**、**已完成** (最後一個狀態)。 如果作業發生錯誤，您會收到 **錯誤** 狀態。 如果正在取消作業，您會收到 **正在取消** 的狀態，以及完成時的 **已取消** 狀態。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#WaitForJobToFinish)]

### <a name="job-error-codes"></a>作業錯誤碼

請參閱[錯誤碼](/rest/api/media/jobs/get#joberrorcode) \(英文\)。

### <a name="get-a-streaming-locator"></a>取得串流定位器

編碼完成後，下一個步驟是要讓用戶端可播放輸出資產中的視訊。 您可以透過兩個步驟來執行此動作：第一步，建立[串流定位器](/rest/api/media/streaminglocators)，第二步，建置用戶端可使用的串流 URL。

建立 [串流定位器] 的程序稱為發佈。 根據預設，**串流定位器** 會在進行 API 呼叫後立即生效，而且會持續運作到遭到刪除為止 (除非您有設定選擇性的開始和結束時間)。

建立 [StreamingLocator](/rest/api/media/streaminglocators) 時，您必須指定所需的 **StreamingPolicyName**。 在此範例中，您將串流處理乾淨 (或未加密的) 內容，而使用預先定義的乾淨串流原則 (**PredefinedStreamingPolicy.ClearStreamingOnly**)。

> [!IMPORTANT]
> 使用自訂的[串流原則](/rest/api/media/streamingpolicies)時，您應該為媒體服務帳戶設計一組受限的這類原則，並且在需要相同的加密選項和通訊協定時，對 StreamingLocators 重新使用這些原則。 媒體服務帳戶有串流原則項目的數量配額。 您不應為每個串流定位器建立新的串流原則。

下列程式碼假設您要呼叫具有唯一 locatorName 的函式。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateStreamingLocator)]

雖然本主題中的範例是在討論串流處理，但您可以使用相同的呼叫來建立串流定位器，以透過漸進式下載來傳遞影片。

### <a name="get-streaming-urls"></a>取得串流 URL

建立了 [串流定位器](/rest/api/media/streaminglocators)之後，現在您可以取得串流 URL，如 **GetStreamingURLs** 中所示。 若要建置 URL，您需要串連 [串流端點](/rest/api/media/streamingendpoints)主機名稱和 **串流定位器** 路徑。 此範例會使用預設的 **串流端點**。 初次建立媒體服務帳戶時，此預設的 **串流端點** 會處於停止狀態，因此您需要呼叫 **Start**。

> [!NOTE]
> 在此方法中，您需要 locatorName，也就是為輸出資產建立 **串流定位器** 時所使用的項目。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#GetStreamingURLs)]

### <a name="clean-up-resources-in-your-media-services-account"></a>清除媒體服務帳戶中的資源

一般而言，您應清除所有項目，只保留想要重複使用的物件 (您通常會重複使用轉換並且保存 StreamingLocators 等)。 如果您想要在實驗之後有乾淨的帳戶，請刪除您不打算重複使用的資源。 例如，下列程式碼會刪除作業、已建立的資產和內容金鑰原則：

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>執行範例應用程式

1. 按 Ctrl+F5 以執行 EncodeAndStreamFiles 應用程式。
2. 從主控台中複製其中一個串流 URL。

此範例顯示可以使用其他不同通訊協定來播放影片的 URL：

![此輸出範例顯示媒體服務串流處理影片的 URL](./media/stream-files-tutorial-with-api/output.png)

## <a name="test-the-streaming-url"></a>測試串流 URL

本文使用 Azure 媒體播放器測試串流。

> [!NOTE]
> 如果播放程式裝載在 HTTPS 網站上，請務必將 URL 更新為 "https"。

1. 開啟瀏覽器並巡覽至 [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/)。
2. 在 [URL:] 方塊中，貼上您在執行應用程式時取得的其中一個串流 URL 值。
3. 選取 [更新播放程式]  。

Azure 媒體播放器可以用於測試，但不應用於生產環境。

## <a name="clean-up-resources"></a>清除資源

如果您不再需要資源群組中的任何資源 (包含您為教學課程建立的媒體服務和儲存體帳戶)，請將稍早建立的資源群組刪除。

執行下列 CLI 命令：

```azurecli
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>多執行緒

Azure 媒體服務 v3 SDK 不是安全執行緒。 開發多執行緒應用程式時，您應該為每個執行緒產生及使用新的 AzureMediaServicesClient 物件。

## <a name="ask-questions-give-feedback-get-updates"></a>提出問題、提供意見反應、取得更新

請參閱 [Azure 媒體服務社群](media-services-community.md)文章，以了解詢問問題、提供意見反應及取得媒體服務相關更新的不同方式。

## <a name="next-steps"></a>後續步驟

在您已知道如何上傳、編碼和串流影片後，請參閱下列文章： 

> [!div class="nextstepaction"]
> [分析影片](analyze-videos-tutorial-with-api.md)
