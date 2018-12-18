---
title: 範例：新增人臉 - 臉部 API
titleSuffix: Azure Cognitive Services
description: 使用臉部 API 新增影像中的人臉。
services: cognitive-services
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: sample
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: fb5d03e2cb3c11daf7a94966fda46345ee910ded
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46125097"
---
# <a name="example-how-to-add-faces"></a>範例：如何新增人臉

本指南示範將大量人員和人臉新增至 PersonGroup 的最佳做法。
相同的策略也適用於 FaceList 和 LargePersonGroup。
這些範例是使用「臉部 API」用戶端程式庫以 C# 撰寫的。

## <a name="step-1-initialization"></a>步驟 1：初始化

已宣告數個變數並實作協助程式函式來排程要求。

- `PersonCount` 是人員的總數。
- `CallLimitPerSecond` 是根據訂用帳戶層而定的每秒呼叫數上限。
- `_timeStampQueue` 是要記錄要求時間戳記的佇列。
- `await WaitCallLimitPerSecondAsync()` 會等到有效才會傳送下一個要求。

```CSharp
const int PersonCount = 10000;
const int CallLimitPerSecond = 10;
static Queue<DateTime> _timeStampQueue = new Queue<DateTime>(CallLimitPerSecond);

static async Task WaitCallLimitPerSecondAsync()
{
    Monitor.Enter(_timeStampQueue);
    try
    {
        if (_timeStampQueue.Count >= CallLimitPerSecond)
        {
            TimeSpan timeInterval = DateTime.UtcNow - _timeStampQueue.Peek();
            if (timeInterval < TimeSpan.FromSeconds(1))
            {
                await Task.Delay(TimeSpan.FromSeconds(1) - timeInterval);
            }
            _timeStampQueue.Dequeue();
        }
        _timeStampQueue.Enqueue(DateTime.UtcNow);
    }
    finally
    {
        Monitor.Exit(_timeStampQueue);
    }
}
```

## <a name="step-2-authorize-the-api-call"></a>步驟 2：授權 API 呼叫

使用用戶端程式庫時，訂用帳戶金鑰會透過 FaceServiceClient 類別的建構函式傳入。 例如︰

```CSharp
FaceServiceClient faceServiceClient = new FaceServiceClient("<Subscription Key>");
```

您可以從 Azure 入口網站的 Marketplace 頁面取得訂用帳戶金鑰。 請參閱[訂用帳戶](https://www.microsoft.com/cognitive-services/en-us/sign-up)。

## <a name="step-3-create-the-persongroup"></a>步驟 3：建立 PersonGroup

名為 "MyPersonGroup" 的 PersonGroup 用來儲存人員。
要求時間已排入 `_timeStampQueue` 佇列中，以確保整體驗證。

```CSharp
const string personGroupId = "mypersongroupid";
const string personGroupName = "MyPersonGroup";
_timeStampQueue.Enqueue(DateTime.UtcNow);
await faceServiceClient.CreatePersonGroupAsync(personGroupId, personGroupName);
```

## <a name="step-4-create-the-persons-to-the-persongroup"></a>步驟 4：建立 PersonGroup 的人員

同時建立人員，而且會套用 `await WaitCallLimitPerSecondAsync()` 以避免超出呼叫限制。

```CSharp
CreatePersonResult[] persons = new CreatePersonResult[PersonCount];
Parallel.For(0, PersonCount, async i =>
{
    await WaitCallLimitPerSecondAsync();

    string personName = $"PersonName#{i}";
    persons[i] = await faceServiceClient.CreatePersonAsync(personGroupId, personName);
});
```

## <a name="step-5-add-faces-to-the-persons"></a>步驟 5：將人臉新增至人員

同時處理將人臉新增至不同的人員，而一個特定人員的人臉則會循序新增。
同樣會叫用 `await WaitCallLimitPerSecondAsync()` 以確保要求頻率在限制的範圍內。

```CSharp
Parallel.For(0, PersonCount, async i =>
{
    Guid personId = persons[i].PersonId;
    string personImageDir = @"/path/to/person/i/images";

    foreach (string imagePath in Directory.GetFiles(personImageDir, "*.jpg"))
    {
        await WaitCallLimitPerSecondAsync();

        using (Stream stream = File.OpenRead(imagePath))
        {
            await faceServiceClient.AddPersonFaceAsync(personGroupId, personId, stream);
        }
    }
});
```

## <a name="summary"></a>總結

在本指南中，您已了解使用大量人員和人臉建立 PersonGroup 的程序。 幾項提醒：

- 此策略也適用於 FaceList 和 LargePersonGroup。
- 可以同時處理對 LargePersonGroup 中的不同 FaceList 或 Person 新增/刪除人臉。
- 對於 LargePersonGroup 中的一個特定 FaceList 或 Person，應該循序執行相同的作業。
- 為了簡化內容，本指南略過潛在例外狀況的處理。 如果您想要變得更強固，就應該套用適當的重試原則。

以下是先前所說明和示範功能的快速提醒：

- 使用 [PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API 建立 PersonGroup
- 使用 [PersonGroup Person - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API 建立人員
- 使用 [PersonGroup Person - 新增人臉](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) API 將人臉新增至人員

## <a name="related-topics"></a>相關主題

- [如何識別影像中的人臉](HowtoIdentifyFacesinImage.md)
- [如何偵測影像中的人臉](HowtoDetectFacesinImage.md)
- [如何使用大規模功能](how-to-use-large-scale.md)
