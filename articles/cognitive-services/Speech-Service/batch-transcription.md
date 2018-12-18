---
title: Azure Batch 轉譯 API
description: 範例
services: cognitive-services
author: PanosPeriorellis
ms.service: cognitive-services
ms.technology: Speech to Text
ms.topic: article
ms.date: 04/26/2018
ms.author: panosper
ms.openlocfilehash: 860b58a18fbc14532a8591fc753453d60492d3c0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981367"
---
# <a name="batch-transcription"></a>批次轉譯

如果您有大量音訊，則適合使用 Batch 轉譯。 您可以在非同步模式中指向音訊檔案並取回轉譯。

## <a name="batch-transcription-api"></a>批次轉譯 API

Batch 轉譯 API 會提供非同步語音轉換文字轉譯及其他功能。

> [!NOTE]
> Batch 轉譯 API 適用於通常會累積數千個小時音訊的話務中心。 此 API 遵循「射後不理」哲學，可讓您輕鬆轉譯大量錄製的音訊。

### <a name="supported-formats"></a>支援的格式

Batch 轉譯 API 支援下列格式：

名稱| 通道  |
----|----------|
mp3 |   Mono   |   
mp3 |  立體聲  | 
wav |   Mono   |
wav |  立體聲  |

針對立體聲音訊資料流，Batch 轉譯會在轉譯期間分割左右聲道。 這會建立兩個 JSON 結果檔案，每個聲道各建立一個。 每個語句的時間戳記可讓開發人員建立排序的最終文字記錄。 下列 JSON 範例顯示聲道的輸出。

```json
       {
        "recordingsUrl": "https://mystorage.blob.core.windows.net/cris-e2e-datasets/TranscriptionsDataset/small_sentence.wav?st=2018-04-19T15:56:00Z&se=2040-04-21T15:56:00Z&sp=rl&sv=2017-04-17&sr=b&sig=DtvXbMYquDWQ2OkhAenGuyZI%2BYgaa3cyvdQoHKIBGdQ%3D",
        "resultsUrls": {
            "channel_0": "https://mystorage.blob.core.windows.net/bestor-87a0286f-304c-4636-b6bd-b3a96166df28/TranscriptionData/24265e4c-e459-4384-b572-5e3e7795221f?sv=2017-04-17&sr=b&sig=IY2qd%2Fkgtz2PwRe2C88BphH4Hv%2F1VCb1UVJ33xsw%2BEY%3D&se=2018-04-23T14:48:24Z&sp=r"
        },
        "statusMessage": "None.",
        "id": "0bb95356-ff06-469d-acc7-81f9144a269a",
        "createdDateTime": "2018-04-20T14:11:57.167",
        "lastActionDateTime": "2018-04-20T14:12:54.643",
        "status": "Succeeded",
        "locale": "en-US"
    },
```

> [!NOTE]
> 批次轉譯 API 使用 REST 服務來要求轉譯、其狀態及相關結果。 您可以使用任何語言的 API。 下一節描述其使用方式。

## <a name="authorization-token"></a>授權權杖

如同統一語音服務的所有功能，您可以從 [Azure 入口網站](https://portal.azure.com)依照我們的[快速入門指南](get-started.md)建立訂用帳戶金鑰。 如果您打算從我們的基準模型取得轉譯，就需要這樣做。 

如果打算自訂並使用自訂模型，則需要將此訂用帳戶金鑰新增到自訂語音入口網站，如下所示：

1. 登入[自訂語音](https://customspeech.ai)。

2. 選取 **訂用帳戶** 。

3. 選取 [連線現有的訂用帳戶]。

4. 在快顯檢視中輸入訂用帳戶金鑰和別名

    ![自訂語音訂用帳戶頁面的螢幕擷取畫面](media/stt/Subscriptions.jpg)

5. 複製該金鑰並貼到下列範例的用戶端程式碼中。

> [!NOTE]
> 如果您打算使用自訂模型，您也需要該模型的識別碼。 請注意，這不是您在 [端點詳細資料] 檢視中所找到的端點識別碼。 這是您選取該模型的詳細資料時可擷取的模型識別碼。

## <a name="sample-code"></a>範例程式碼

使用訂用帳戶金鑰和 API 金鑰來自訂下列範例程式碼。 這可讓您取得持有人權杖。

```cs
     public static CrisClient CreateApiV2Client(string key, string hostName, int port)

        {
            var client = new HttpClient();
            client.Timeout = TimeSpan.FromMinutes(25);
            client.BaseAddress = new UriBuilder(Uri.UriSchemeHttps, hostName, port).Uri;
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);
         
            return new CrisClient(client);
        }
```

在您取得權杖之後，您必須指定 SAS URI，該 URI 會指向需要轉譯的音訊檔案。 其餘的程式碼會逐一查看狀態，並顯示結果。

```cs
   static async Task TranscribeAsync()
        { 
            private const string SubscriptionKey = "<your Speech subscription key>";
            private const string HostName = "westus.cris.ai";
            private const int Port = 443;
    
            // Creating a Batch transcription API Client
            var client = CrisClient.CreateApiV2Client(SubscriptionKey, HostName, Port);
            
            var transcriptions = await client.GetTranscriptionAsync().ConfigureAwait(false);

            var transcriptionLocation = await client.PostTranscriptionAsync(Name, Description, Locale, new Uri(RecordingsBlobUri), new[] { AdaptedAcousticId, AdaptedLanguageId }).ConfigureAwait(false);

            // get the transcription Id from the location URI
            var createdTranscriptions = new List<Guid>();
            createdTranscriptions.Add(new Guid(transcriptionLocation.ToString().Split('/').LastOrDefault()))

            while (true)
            {
                // get all transcriptions for the user
                transcriptions = await client.GetTranscriptionAsync().ConfigureAwait(false);
                completed = 0; running = 0; notStarted = 0;

                // for each transcription in the list we check the status
                foreach (var transcription in transcriptions)
                {
                    switch(transcription.Status)
                    {
                        case "Failed":
                        case "Succeeded":

                            // we check to see if it was one of the transcriptions we created from this client.
                            if (!createdTranscriptions.Contains(transcription.Id))
                            {
                                // not creted form here, continue
                                continue;
                            }
                            
                            completed++;
                            
                            // if the transcription was successfull, check the results
                            if (transcription.Status == "Succeeded")
                            {
                                var resultsUri = transcription.ResultsUrls["channel_0"];
                                WebClient webClient = new WebClient();
                                var filename = Path.GetTempFileName();
                                webClient.DownloadFile(resultsUri, filename);
                                var results = File.ReadAllText(filename);
                                Console.WriteLine("Transcription succedded. Results: ");
                                Console.WriteLine(results);
                            }
                            break;
                        case "Running":
                            running++;
                            break;
                        case "NotStarted":
                            notStarted++;
                            break;
                    }
                }

                Console.WriteLine(string.Format("Transcriptions status: {0} completed, {1} running, {2} not started yet", completed, running, notStarted));

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }

            Console.WriteLine("Press any key...");
        }
```

> [!NOTE]
> 在上述程式碼中，訂用帳戶金鑰來自您在 Azure 入口網站上建立的語音資源。 從自訂語音服務資源取得的金鑰無法運作。

請注意，張貼音訊和接收轉譯狀態是非同步設定。 建立的用戶端是 .NET HTTP 用戶端。 您可以使用 `PostTranscriptions` 方法來傳送音訊檔案詳細資料，並使用 `GetTranscriptions` 方法來接收結果。 `PostTranscriptions` 會傳回控制代碼，而 `GetTranscriptions` 會使用此控制代碼來建立控制代碼，以取得轉譯狀態。

目前的範例程式碼未指定任何自訂模型。 該服務會使用基準模型來轉譯一或多個檔案。 若要指定模型，您可以將原音和語言模型的模型識別碼傳遞給相同的方法。 

如果您不想要使用基準，則必須傳遞原音和語言模型的模型識別碼。

> [!NOTE]
> 針對基準轉譯，您不需要宣告基準模型的端點。 如果您想要使用自訂模型，就必須提供其端點識別碼作為[範例](https://github.com/PanosPeriorellis/Speech_Service-BatchTranscriptionAPI)。 如果您想要搭配使用原音基準和基準語言模型，您只需宣告自訂模型的端點識別碼。 Microsoft 會偵測夥伴基準模型 (原音或語言)，然後使用該模型來完成轉譯要求。

### <a name="supported-storage"></a>支援的儲存體

Azure Blob 儲存體是目前唯一支援的儲存體。

## <a name="downloading-the-sample"></a>下載範例

此處顯示的範例是在 [GitHub](https://github.com/PanosPeriorellis/Speech_Service-BatchTranscriptionAPI) 上。

> [!NOTE]
> 音訊轉譯所需的時間長度，通常相當於音訊檔案持續時間加上 2-3 分鐘的額外時間。

## <a name="next-steps"></a>後續步驟

* [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
