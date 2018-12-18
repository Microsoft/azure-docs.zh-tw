---
title: 使用 Azure Media Indexer 編輯媒體檔案索引
description: Azure Media Indexer 讓您能將媒體檔案的內容變成可搜尋，並產生隱藏式字幕和關鍵字的全文檢索記錄。 本主題說明如何使用 Media Indexer。
services: media-services
documentationcenter: ''
author: Asolanki
manager: cfowler
editor: ''
ms.assetid: 827a56b2-58a5-4044-8d5c-3e5356488271
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 07/20/2017
ms.author: adsolank;juliako;johndeu
ms.openlocfilehash: 320d8cfa21c36e0f0d751d29d461a0023ab563e7
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783767"
---
# <a name="indexing-media-files-with-azure-media-indexer"></a>使用 Azure Media Indexer 編輯媒體檔案索引
Azure Media Indexer 讓您能將媒體檔案的內容變成可搜尋，並產生隱藏式字幕和關鍵字的全文檢索記錄。 您可以處理一份媒體檔或是批次處理多個媒體檔案。  

> [!IMPORTANT]
> 在編製內容索引時，請務必使用語音清楚的媒體檔案 (不含背景音樂、噪音、效果或麥克風雜音)。 適當內容的一些範例有：錄製的會議、演講或簡報。 下列內容可能不適合用來編製索引：電影、電視節目、任何具有混合音訊與音效的內容、錄製效果不良有背景噪音 (雜音) 的內容。
> 
> 

索引工作可產生下列輸出檔案：

* 下列格式的隱藏式輔助字幕檔案：**SAMI**、**TTML** 和 **WebVTT**。
  
    隱藏式輔助字幕檔案包含稱為 Recognizability 的標記，它會根據來源視訊中的語音可辨識度來為索引工作評分。  您可以使用 Recognizability 值篩選輸出檔的實用性。 較低的分數表示由於音訊品質所致的不良索引結果。
* 關鍵字檔案 (XML)。
* 與 SQL Server 搭配使用的音訊編製索引 blob 檔案 (AIB)。
  
    如需詳細資訊，請參閱 [搭配 Azure Media Indexer 和 SQL Server 使用 AIB 檔案](https://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)。

本文示範如何建立索引作業來**建立資產的索引**和**建立多個檔案的索引**。

如需最新的 Azure Media Indexer 更新，請參閱 [媒體服務部落格](#preset)。

## <a name="using-configuration-and-manifest-files-for-indexing-tasks"></a>針對索引工作使用組態和資訊清單檔
您可以使用工作組態來為索引工作指定更多詳細資料。 例如，您可以指定要用於媒體檔案的中繼資料。 語言引擎會使用此中繼資料來擴充其詞彙，並大幅提升語音辨識準確度。  您也可以指定想要的輸出檔案。

您也可以使用資訊清單檔，一次處理多個媒體檔案。

如需詳細資訊，請參閱 [Azure Media Indexer 的工作預設](https://msdn.microsoft.com/library/dn783454.aspx)。

## <a name="index-an-asset"></a>編製資產索引
下列方法會將媒體檔案上傳為資產，並建立工作來編製資產索引。

如果未指定組態檔案，則會使用所有預設設定編製媒體檔案的索引。

```csharp
    static bool RunIndexingJob(string inputMediaFilePath, string outputFolder, string configurationFile = "")
    {
        // Create an asset and upload the input media file to storage.
        IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Indexing Input Asset",
            AssetCreationOptions.None);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration from file if specified.
        string configuration = string.IsNullOrEmpty(configurationFile) ? "" : File.ReadAllText(configurationFile);

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset", AssetCreationOptions.None);

        // Use the following event handler to check job progress.  
        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

        // Launch the job.
        job.Submit();

        // Check job execution and wait for job to finish.
        Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
        progressJobTask.Wait();

        // If job state is Error, the event handling
        // method for job progress should log errors.  Here we check
        // for error state and exit if needed.
        if (job.State == JobState.Error)
        {
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
        assetFile.Upload(filePath);

        return asset;
    }

    static void DownloadAsset(IAsset asset, string outputDirectory)
    {
        foreach (IAssetFile file in asset.AssetFiles)
        {
            file.Download(Path.Combine(outputDirectory, file.Name));
        }
    }

    static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
    {
        var processor = _context.MediaProcessors
        .Where(p => p.Name == mediaProcessorName)
        .ToList()
        .OrderBy(p => new Version(p.Version))
        .LastOrDefault();

        if (processor == null)
            throw new ArgumentException(string.Format("Unknown media processor",
                                                       mediaProcessorName));

        return processor;
    }  
```

<!-- __ -->
### <a id="output_files"></a>輸出檔案
索引工作預設會產生下列輸出檔案。 檔案會儲存在第一個輸出資產中。

當有多個輸入媒體檔案時，索引子會產生作業輸出的資訊清單檔，名為 'JobResult.txt'。 針對每個輸入媒體檔案，系統會把所產生的 AIB、SAMI、TTML、WebVTT 及關鍵字檔案循序編號，並使用「別名」來命名。

| 檔案名稱 | 說明 |
| --- | --- |
| **InputFileName.aib** |音訊索引 blob 檔案。 <br/><br/> 音訊索引 Blob (AIB) 檔案是二進位檔案，可以使用全文檢索搜尋在 Microsoft SQL Server 中搜尋。  AIB 檔比簡單的字幕檔案強大，因為它包含每個字的替代字，允許更豐富的搜尋經驗。 <br/> <br/>它需要在執行 Microsoft SQL 2008 或更新版本的電腦上安裝 Indexer SQL 附加元件。 使用 Microsoft SQL Server 全文檢索搜尋來搜尋 AIB 可以比搜尋 WAMI 產生之隱藏式字幕檔案提供更正確的搜尋結果。 這是因為 AIB 包含發音類似的替代字，而隱藏式字幕檔案則包含音訊每一節的最高信賴字。 如果搜尋說的話重要性最高，則建議一起使用 AIB 和 Microsoft SQL Server。<br/><br/> 若要下載附加元件，請按一下 [<a href="http://aka.ms/indexersql">Azure 媒體索引器 SQL 附加元件</a>]。 <br/><br/>也可以利用其他搜尋引擎，例如 Apache Lucene/Solr，只根據隱藏式字幕和關鍵字 XML 檔案編製視訊的索引，但這會導致搜尋結果較不正確。 |
| **InputFileName.smi**<br/>**InputFileName.ttml**<br/>**InputFileName.vtt** |SAMI、TTML 和 WebVTT 格式的隱藏式輔助字幕 (CC) 檔案。<br/><br/>它們可以用來讓具有聽力障礙的人存取音訊和視訊檔案。<br/><br/>隱藏式輔助字幕檔案包含稱為 <b>Recognizability</b> 的標記，它會根據來源視訊中的語音可辨識度來為索引工作評分。  您可以使用 <b>Recognizability</b> 的值，針對實用性來篩選輸出檔。 較低的分數表示由於音訊品質所致的不良索引結果。 |
| **InputFileName.kw.xml<br/>InputFileName.info** |關鍵字與資訊檔案。 <br/><br/>關鍵字檔案是 XML 檔案，其中包含從語音內容擷取的關鍵字，以及關鍵字的頻率和位移資訊。 <br/><br/>資訊檔案是純文字檔案，包含每個已辨識字詞的細微資訊。 第一行是特殊行並包含可辨識分數。 後續每一行皆是下列資料的清單 (以 tab 鍵分隔)：開始時間、結束時間、文字/片語、信賴值。 時間是以秒為單位，信賴值則是以 0-1 的數字標示。 <br/><br/>範例行："1.20    1.45    word    0.67" <br/><br/>這些檔案的用途眾多，例如執行語音分析，或是公開到搜尋引擎 (例如 Bing、Google 或 Microsoft SharePoint) 來讓媒體檔案更容易被找到，或甚至用來放送更多相關的廣告。 |
| **JobResult.txt** |包含下列資訊的輸出資訊清單 (只會在編製多個檔案的索引時顯示)：<br/><br/><table border="1"><tr><th>InputFile</th><th>Alias</th><th>MediaLength</th><th>Error</th></tr><tr><td>a.mp4</td><td>Media_1</td><td>300</td><td>0</td></tr><tr><td>b.mp4</td><td>Media_2</td><td>0</td><td>3000</td></tr><tr><td>c.mp4</td><td>Media_3</td><td>600</td><td>0</td></tr></table><br/> |

如果不是所有輸入媒體檔案都成功編製索引，則索引作業會失敗，錯誤碼為 4000。 如需詳細資訊，請參閱 [錯誤碼](#error_codes)。

## <a name="index-multiple-files"></a>編製多個檔案的索引
下列方法會將多個媒體檔案上傳為資產，並建立工作來批次編製這些檔案的索引。

會建立 ".lst" 副檔名的資訊清單檔，並上傳到資產。 資訊清單檔案包含所有資產檔案的清單。 如需詳細資訊，請參閱 [Azure Media Indexer 的工作預設](https://msdn.microsoft.com/library/dn783454.aspx)。

```csharp
    static bool RunBatchIndexingJob(string[] inputMediaFiles, string outputFolder)
    {
        // Create an asset and upload to storage.
        IAsset asset = CreateAssetAndUploadMultipleFiles(inputMediaFiles,
            "My Indexing Input Asset - Batch Mode",
            AssetCreationOptions.None);

        // Create a manifest file that contains all the asset file names and upload to storage.
        string manifestFile = "input.lst";            
        File.WriteAllLines(manifestFile, asset.AssetFiles.Select(f => f.Name).ToArray());
        var assetFile = asset.AssetFiles.Create(Path.GetFileName(manifestFile));
        assetFile.Upload(manifestFile);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job - Batch Mode");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration.
        string configuration = File.ReadAllText("batch.config");

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task - Batch Mode",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset - Batch Mode", AssetCreationOptions.None);

        // Use the following event handler to check job progress.  
        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

        // Launch the job.
        job.Submit();

        // Check job execution and wait for job to finish.
        Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
        progressJobTask.Wait();

        // If job state is Error, the event handling
        // method for job progress should log errors.  Here we check
        // for error state and exit if needed.
        if (job.State == JobState.Error)
        {
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    private static IAsset CreateAssetAndUploadMultipleFiles(string[] filePaths, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        foreach (string filePath in filePaths)
        {
            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);
        }

        return asset;
    }
```

### <a name="partially-succeeded-job"></a>部分成功的工作
如果不是所有輸入媒體檔案都成功編製索引，則索引工作將會失敗，錯誤碼為 4000。 如需詳細資訊，請參閱 [錯誤碼](#error_codes)。

會產生 (與成功工作) 相同的輸出。 您可以參閱輸出資訊清單檔，根據 Error 欄位值找出哪些輸入檔案失敗。 針對失敗的輸入檔案，將不會產生結果的 AIB、SAMI、TTML、WebVTT 和關鍵字檔案。

### <a id="preset"></a> Azure 媒體索引器的工作預設
在工作旁邊提供選擇性工作預設，即可自訂從 Azure 媒體索引器處理。  下表說明此組態 xml 的格式。

| Name | 必要 | 說明 |
| --- | --- | --- |
| **input** |false |您想要編製索引的資產檔案。</p><p>Azure 媒體索引器支援下列媒體檔案格式︰MP4、WMV、MP3、M4A、WMA、AAC、WAV。</p><p>您可以在 **input** 元素的 **name** 或 **list** 屬性中指定檔案名稱 (如下所示)。如果您未指定要編制索引的資產檔案，則會選擇主要檔案。 如果未設定主要資產檔案，則會對輸入資產中的第一個檔案編製索引。</p><p>若要明確指定資產檔案名稱，請執行︰<br/>`<input name="TestFile.wmv">`<br/><br/>您也可以一次對多個資產檔案編制索引 (最多 10 個檔案)。 作法：<br/><br/><ol class="ordered"><li><p>建立文字檔 (資訊清單檔) 並指定 .lst 副檔名。 </p></li><li><p>將輸入資產中所有資產檔案名稱的清單加入至此資訊清單檔案。 </p></li><li><p>將此資訊檔案新增 (上傳) 到資產。  </p></li><li><p>在輸入的 list 屬性中指定資訊清單檔的名稱。<br/>`<input list="input.lst">`</li></ol><br/><br/>附註︰如果您在資訊清單檔中新增超過 10 個檔案，編製索引作業將失敗，並出現 2006 錯誤碼。 |
| **metadata** |false |用於詞彙調節之指定資產檔案的中繼資料。  適合用來準備 Indexer 以辨識非標準詞彙文字，例如專有名詞。<br/>`<metadata key="..." value="..."/>` <br/><br/>您可以提供預先定義的**索引鍵**的**值**。 目前支援下列索引鍵：<br/><br/>"title" 和 "description" - 用於詞彙調整，以微調您的工作的語言模型及改進語音辨識準確度。  值會植入網際網路搜尋來尋找內容相關的文字文件，並使用內容來加強索引工作期間的內部字典。<br/>`<metadata key="title" value="[Title of the media file]" />`<br/>`<metadata key="description" value="[Description of the media file] />"` |
| **features** <br/><br/> 在 1.2 版中新增。 目前唯一支援的功能是語音辨識 ("ASR")。 |false |語音辨識功能具有下列設定索引鍵︰<table><tr><th><p>Key</p></th>        <th><p>說明</p></th><th><p>範例值</p></th></tr><tr><td><p>語言</p></td><td><p>要在多媒體檔案中辨識的自然語言。</p></td><td><p>英文、西班牙文</p></td></tr><tr><td><p>CaptionFormats</p></td><td><p>所需輸出字幕格式的分號分隔清單 (如果有的話)</p></td><td><p>ttml;sami;webvtt</p></td></tr><tr><td><p>GenerateAIB</p></td><td><p>布林值旗標，用來指定是否需要 AIB 檔案 (適用於 SQL Server 和客戶索引器 IFilter)。  如需詳細資訊，請參閱 <a href="http://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/">搭配 Azure Media Indexer 和 SQL Server 使用 AIB 檔案</a>。</p></td><td><p>True; False</p></td></tr><tr><td><p>GenerateKeywords</p></td><td><p>布林值旗標，用於指定是否需要關鍵字 XML 檔案。</p></td><td><p>True; False。 </p></td></tr><tr><td><p>ForceFullCaption</p></td><td><p>布林值旗標，用於指定是否強制完整字幕 (不管信賴等級為何)。  </p><p>預設值為 false，在此情況下，會省略最終字幕輸出中信賴等級小於 50% 的單字和片語並以省略符號 ("...") 取代。  省略符號適合用於字幕品質控制和稽核。</p></td><td><p>True; False。 </p></td></tr></table> |

### <a id="error_codes"></a>錯誤碼
如果發生錯誤，Azure 媒體索引器應回報下列其中一個錯誤碼：

| 代碼 | Name | 可能的原因 |
| --- | --- | --- |
| 2000 |無效的組態 |無效的組態 |
| 2001 |無效的輸入資產 |遺漏輸入資產或資產是空的。 |
| 2002 |無效的資訊清單 |資訊清單是空的或資訊清單包含無效的項目。 |
| 2003 |無法下載媒體檔案 |資訊清單檔中的 URL 無效。 |
| 2004 |不支援的通訊協定 |不支援媒體 URL 的通訊協定。 |
| 2005 |不支援的檔案類型 |不支援輸入媒體檔案類型。 |
| 2006 |太多輸入檔案 |輸入資訊清單中有超過 10 個檔案。 |
| 3000 |無法解碼媒體檔案 |不支援的媒體轉碼器  <br/>或<br/> 損毀的媒體檔案 <br/>或<br/> 輸入媒體沒有音訊資料流。 |
| 4000 |批次編製索引已部分成功 |某些輸入媒體檔案無法編製索引。 如需詳細資訊，請參閱<a href="#output_files">輸出檔案</a>。 |
| 其他 |內部錯誤 |請連絡技術支援小組。 indexer@microsoft.com |

## <a id="supported_languages"></a>支援的語言
目前支援英文和西班牙文。 如需詳細資訊，請參閱 [v1.2 版部落格文章](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>相關連結
[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[搭配 Azure Media Indexer 和 SQL Server 使用 AIB 檔案](https://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)

[使用 Azure Media Indexer 2 Preview 編製媒體檔案索引](media-services-process-content-with-indexer2.md)

