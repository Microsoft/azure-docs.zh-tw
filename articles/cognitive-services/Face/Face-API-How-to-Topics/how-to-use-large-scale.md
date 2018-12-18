---
title: 範例：使用大規模功能 - 臉部 API
titleSuffix: Azure Cognitive Services
description: 使用臉部 API 中的大規模功能。
services: cognitive-services
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: sample
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: e8bbf78da84ddb77ce956e37f91be46e96144991
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46123074"
---
# <a name="example-how-to-use-the-large-scale-feature"></a>範例：如何使用大規模功能

本指南是一篇有關移轉程式碼以從現有 PersonGroup 和 FaceList 將規模分別放大至 LargePersonGroup 和 LargeFaceList 的進階文章。
本指南將示範移轉程序，其中假設您已具備 PersonGroup 和 FaceList 的基本使用知識。
如需熟悉基本作業，請參閱其他教學課程，例如[如何識別影像中的臉部](HowtoIdentifyFacesinImage.md)。

「臉部 API」最近發行 LargePersonGroup 與 LargeFaceList 這兩個功能來啟用大規模案例，兩者統稱為大規模作業。
LargePersonGroup 可包含多達 1,000,000 個人員，每個人員最多可有 248 張臉，而 LargeFaceList 則可包含多達 1,000,000 張臉。

大規模作業與傳統的 PersonGroup 和 FaceList 類似，但由於採用新架構，因此有一些明顯的差異。
本指南將示範移轉程序，其中假設您已具備 PersonGroup 和 FaceList 的基本使用知識。
這些範例是使用「臉部 API」用戶端程式庫以 C# 撰寫的。

若要在大型規模中啟用「識別」和 FindSimilar 的臉部搜尋效能，您必須導入「定型」作業來對 LargeFaceList 和 LargePersonGroup 進行前處理。
定型時間從幾秒到大約半小時不等，取決於實際的容量。
如果先前已進行過成功的定型，則在定型期間，仍有可能執行「識別」和 FindSimilar。
不過，缺點是，在完成新的大規模定型後移轉之前，新增的人員/臉部不會出現在結果中。

## <a name="concepts"></a>概念

如果您不熟悉本指南中的下列概念，可以在[詞彙](../Glossary.md)中找到相關定義：

- LargePersonGroup：「人員」的集合，容量上限為 1,000,000。
- LargeFaceList：「臉部」的集合，容量上限為 1,000,000。
- 定型：一種用來確保「識別/FindSimilar」效能的前處理。
- 識別：從 PersonGroup 或 LargePersonGroup 中識別一或多張臉。
- FindSimilar：從 FaceList 或 LargeFaceList 中搜尋相似的臉部。

## <a name="step-1-authorize-the-api-call"></a>步驟 1：授權 API 呼叫

使用「臉部 API」用戶端程式庫時，會透過 FaceServiceClient 類別的建構函式，傳入訂用帳戶金鑰和訂用帳戶端點。 例如︰

```CSharp
string SubscriptionKey = "<Subscription Key>";
// Use your own subscription endpoint corresponding to the subscription key.
string SubscriptionRegion = "https://westcentralus.api.cognitive.microsoft.com/face/v1.0/";
FaceServiceClient FaceServiceClient = new FaceServiceClient(SubscriptionKey, SubscriptionRegion);
```

您可以從 Azure 入口網站的 Marketplace 頁面取得訂用帳戶金鑰及對應的端點。
請參閱[訂用帳戶](https://azure.microsoft.com/services/cognitive-services/directory/vision/)。

## <a name="step-2-code-migration-in-action"></a>步驟 2：實際操作程式碼移轉

本節只著重在將 PersonGroup/FaceList 移轉至 LargePersonGroup/LargeFaceList。
雖然 LargePersonGroup/LargeFaceList 與 PersonGroup/FaceList 在設計上及內部實作上不同，但 API 介面類似以支援回溯相容性。

不支援資料移轉，您必須改為重新建立 LargePersonGroup/LargeFaceList。

## <a name="step-21-migrate-persongroup-to-largepersongroup"></a>步驟 2.1：將 PersonGroup 移轉至 LargePersonGroup

從 PersonGroup 到 LargePersonGroup 的移轉相當順暢，因為它們共用完全相同的群組層級作業。

針對 PersonGroup/Person 相關的實作，只需將 API 路徑或 SDK 類別/模組變更為 LargePersonGroup 和「LargePersonGroup 人員」即可.

在資料移轉方面，請參閱[如何新增臉部](how-to-add-faces.md)。

## <a name="step-22-migrate-facelist-to-largefacelist"></a>步驟 2.2：將 FaceList 移轉到 LargeFaceList

| FaceList API | LargeFaceList API |
|:---:|:---:|
| 建立 | 建立 |
| 刪除 | 刪除 |
| 取得 | 取得 |
| 列出 | 列出 |
| 更新 | 更新 |
| - | 定型 |
| - | 取得定型訓練 |

上表是 FaceList 與 LargeFaceList 之間的清單層級作業比較。
如所示，與 FaceList 相比，LargeFaceList 附帶「定型」和「取得定型狀態」這兩項新作業。
將 LargeFaceList 定型是 [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) 作業的先決條件，而 FaceList 則不需要「定型」。
以下程式碼片段是一個等候 LargeFaceList 定型作業的協助程式函式。

```CSharp
/// <summary>
/// Helper function to train LargeFaceList and wait for finish.
/// </summary>
/// <remarks>
/// The time interval can be adjusted considering the following factors:
/// - The training time which depends on the capacity of the LargeFaceList.
/// - The acceptable latency for getting the training status.
/// - The call frequency and cost.
///
/// Estimated training time for LargeFaceList in different scale:
/// -     1,000 faces cost about  1 to  2 seconds.
/// -    10,000 faces cost about  5 to 10 seconds.
/// -   100,000 faces cost about  1 to  2 minutes.
/// - 1,000,000 faces cost about 10 to 30 minutes.
/// </remarks>
/// <param name="largeFaceListId">The Id of the LargeFaceList for training.</param>
/// <param name="timeIntervalInMilliseconds">The time interval for getting training status in milliseconds.</param>
/// <returns>A task of waiting for LargeFaceList training finish.</returns>
private static async Task TrainLargeFaceList(
    string largeFaceListId,
    int timeIntervalInMilliseconds = 1000)
{
    // Trigger a train call.
    await FaceServiceClient.TrainLargeFaceListAsync(largeFaceListId);

    // Wait for training finish.
    while (true)
    {
        Task.Delay(timeIntervalInMilliseconds).Wait();
        var status = await FaceServiceClient.GetLargeFaceListTrainingStatusAsync(largeFaceListId);

        if (status.Status == Status.Running)
        {
            continue;
        }
        else if (status.Status == Status.Succeeded)
        {
            break;
        }
        else
        {
            throw new Exception("The train operation is failed!");
        }
    }
}
```

先前，FaceList 搭配新增臉部和 FindSimilar 的一般用法會是

```CSharp
// Create a FaceList.
const string FaceListId = "myfacelistid_001";
const string FaceListName = "MyFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
FaceServiceClient.CreateFaceListAsync(FaceListId, FaceListName).Wait();

// Add Faces to the FaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await FaceServiceClient.AddFaceToFaceListAsync(FaceListId, stream);
            }
        });

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = FaceServiceClient.DetectAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await FaceServiceClient.FindSimilarAsync(face.FaceId, FaceListId, 20));
    }
}
```

將其移轉成 LargeFaceList 時，則應該變成

```CSharp
// Create a LargeFaceList.
const string LargeFaceListId = "mylargefacelistid_001";
const string LargeFaceListName = "MyLargeFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
FaceServiceClient.CreateLargeFaceListAsync(LargeFaceListId, LargeFaceListName).Wait();

// Add Faces to the LargeFaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await FaceServiceClient.AddFaceToLargeFaceListAsync(LargeFaceListId, stream);
            }
        });

// Train() is newly added operation for LargeFaceList.
// Must call it before FindSimilarAsync() to ensure the newly added faces searchable.
await TrainLargeFaceList(LargeFaceListId);

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = FaceServiceClient.DetectAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await FaceServiceClient.FindSimilarAsync(face.FaceId, largeFaceListId: LargeFaceListId));
    }
}
```

如以上所示，資料管理和 FindSimilar 部分幾乎相同。
唯一的例外就是必須先在 LargeFaceList 中完成全新的前處理「定型」作業，FindSimilar 才能運作。

## <a name="step-3-train-suggestions"></a>步驟 3：定型建議

雖然「定型」作業可加快 [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) 和[識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239)的速度，但定型需要花費時間，尤其在面對大型規模時更為耗時。
下表列出針對各種不同規模預估的定型時間：

| 規模 (臉部或人員) | 預估的定型時間 |
|:---:|:---:|
| 1,000 | 1-2 秒 |
| 10,000 | 5-10 秒 |
| 100,000 | 1 - 2 分鐘 |
| 1,000,000 | 10 - 30 分鐘 |

若要更充分利用大規模功能，建議您考量採用一些策略。

## <a name="step-31-customize-time-interval"></a>步驟 3.1：自訂時間間隔

如 `TrainLargeFaceList()` 中所示，有一個延遲無限定型狀態檢查程序的 `timeIntervalInMilliseconds`。
針對臉部數量較多的 LargeFaceList，使用較長的間隔將可降低呼叫計數和成本。
您應該根據預期的 LargeFaceList 容量自訂時間間隔。

相同的策略也適用於 LargePersonGroup。
例如，為含有 1,000,000 個人員的 LargePersonGroup 進行定型時，`timeIntervalInMilliseconds` 可為 60,000 (也稱為 1 分鐘間隔)。

## <a name="step-32-small-scale-buffer"></a>步驟 3.2：小規模緩衝區

LargePersonGroup/LargeFaceList 中的「人員/臉部」只有在定型後才可供搜尋。
在動態案例中，會不斷新增人員/臉部且必須立即可供搜尋，而定型所花費的時間可能會超出預期。
為了解決這個問題，您可以僅針對剛新增的項目使用額外的小規模 LargePersonGroup/LargeFaceList 作為緩衝區。
由於規模小很多，因此這個緩衝區所需的定型時間較短，而應該能夠讓這個暫存緩衝區立即可供搜尋。
請將此緩衝區與主要 LargePersonGroup/LargeFaceList 的定型搭配使用，方法是以更疏鬆的間隔 (例如在午夜和每天) 執行主要定型。

範例工作流程：
1. 建立一個主要 LargePersonGroup/LargeFaceList (主要集合) 和一個緩衝區 LargePersonGroup/LargeFaceList (緩衝區集合)。 緩衝區集合僅適用於剛新增的「人員/臉部」。
1. 將新的「人員/臉部」同時新增至主要集合和緩衝區集合。
1. 只以短時間間隔進行緩衝區集合定型，以確保剛新增的項目生效。
1. 同時針對主要集合和緩衝區集合呼叫「識別/FindSimilar」，然後合併結果。
1. 在緩衝區集合大小增加達到閾值，或處於系統閒置時間時，建立新的緩衝區集合並觸發主要集合定型。
1. 完成主要集合定型之後，刪除舊的緩衝區集合。

## <a name="step-33-standalone-training"></a>步驟 3.3：獨立定型

如果可以接受相當長的延遲，就不需要在新增資料後立即觸發「定型」作業。
可以改為將「定型」作業從主要邏輯中分割出來，而以定期方式觸發。
這個策略適用於具有可接受延遲的動態案例，也可套用至靜態案例來進一步降低「定型」頻率。

假設有一個類似於 `TrainLargeFaceList` 的 `TrainLargePersonGroup` 函式。
透過叫用 `System.Timers` 中的 [`Timer`](https://msdn.microsoft.com/library/system.timers.timer(v=vs.110).aspx)
類別以在 LargePersonGroup 上進行獨立「定型」的一般實作會是：

```CSharp
private static void Main()
{
    // Create a LargePersonGroup.
    const string LargePersonGroupId = "mylargepersongroupid_001";
    const string LargePersonGroupName = "MyLargePersonGroupDisplayName";
    FaceServiceClient.CreateLargePersonGroupAsync(LargePersonGroupId, LargePersonGroupName).Wait();

    // Setup a standalone training at regular intervals.
    const int TimeIntervalForStatus = 1000 * 60; // 1 minute interval for getting training status.
    const double TimeIntervalForTrain = 1000 * 60 * 60; // 1 hour interval for training.
    var trainTimer = new Timer(TimeIntervalForTrain);
    trainTimer.Elapsed += (sender, args) => TrainTimerOnElapsed(LargePersonGroupId, TimeIntervalForStatus);
    trainTimer.AutoReset = true;
    trainTimer.Enabled = true;

    // Other operations like creating persons, adding faces and Identification except for Train.
    // ...
}

private static void TrainTimerOnElapsed(string largePersonGroupId, int timeIntervalInMilliseconds)
{
    TrainLargePersonGroup(largePersonGroupId, timeIntervalInMilliseconds).Wait();
}
```

如需有關資料管理和識別相關實作的詳細資訊，請參閱[如何新增臉部](how-to-add-faces.md)和[如何識別影像中的臉部](HowtoIdentifyFacesinImage.md)。

## <a name="summary"></a>總結

在本指南中，您以了解如何將現有的 PersonGroup/FaceList 程式碼 (非資料) 移轉成 LargePersonGroup/LargeFaceList：

- LargePersonGroup 和 LargeFaceList 的運作方式與 PersonGroup/FaceList 類似，差別在於 LargeFaceList 需要「定型」作業。
- 採用適當的定型策略，以針對大規模資料集進行動態資料更新。

## <a name="related-topics"></a>相關主題

- [如何識別影像中的人臉](HowtoIdentifyFacesinImage.md)
- [如何新增臉部](how-to-add-faces.md)
