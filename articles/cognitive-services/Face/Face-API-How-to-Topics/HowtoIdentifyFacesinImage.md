---
title: 範例：識別影像中的人臉 - 臉部 API
titleSuffix: Azure Cognitive Services
description: 使用臉部 API 識別影像中的人臉。
services: cognitive-services
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: sample
ms.date: 03/01/2018
ms.author: sbowles
ms.openlocfilehash: a26f7d6057f92fd3ab92405ecca6965dbd6e37ad
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129066"
---
# <a name="example-how-to-identify-faces-in-images"></a>範例：如何識別影像中的人臉

本指南示範如何使用 PersonGroup 識別不明的臉部，而這些臉部由事先已知的人員所建立。 這些範例是使用「臉部 API」用戶端程式庫以 C# 撰寫的。

## <a name="concepts"></a>概念

若您不熟悉本指南中的下列概念，請隨時搜尋[詞彙](../Glossary.md)中的定義：

- 臉部 - 偵測
- 臉部 - 識別
- PersonGroup

## <a name="preparation"></a>準備工作

此範例示範下列功能：

- 如何建立 PersonGroup - 此 PersonGroup 包含已知的人員清單。
- 如何指派每個人的臉部 - 這些臉部用做識別人員的基準。 建議使用清楚的正臉，就像您的相片識別碼一樣。 一組不錯的相片應該包含具有不同姿勢、衣服顏色或髮型之同一個人的臉部。

若要執行此範例的示範，您需要準備一堆圖片：

- 一些具有人臉的相片。 [按一下這裡下載範例相片](https://github.com/Microsoft/Cognitive-Face-Windows/tree/master/Data)，即 Anna、Bill 和 Clare 的相片。
- 一系列測試相片，這或許未包含用來測試識別的 Anna、Bill 或 Clare 的臉部。 您也可以從上述連結選取一些範例影像。

## <a name="step-1-authorize-the-api-call"></a>步驟 1：授權 API 呼叫

每次呼叫臉部 API 時，都需要訂用帳戶金鑰。 此金鑰可以透過查詢字串參數傳遞，或者在要求標頭中指定。 若要透過查詢字串傳遞訂用帳戶金鑰，請參閱所舉出的[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)要求 URL 範例：
```
https://westus.api.cognitive.microsoft.com/face/v1.0/detect[?returnFaceId][&returnFaceLandmarks][&returnFaceAttributes]
&subscription-key=<Subscription key>
```

或者，您也可以在 HTTP 要求標頭中指定訂用帳戶金鑰：**ocp-apim-subscription-key: &lt;訂用帳戶金鑰&gt;**。若您使用用戶端程式庫，則會透過 FaceServiceClient 類別的建構函式傳入訂用帳戶金鑰。 例如︰
 
```CSharp 
faceServiceClient = new FaceServiceClient("<Subscription Key>");
```
 
您可以從 Azure 入口網站的 Marketplace 頁面取得訂用帳戶金鑰。 請參閱[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)。

## <a name="step-2-create-the-persongroup"></a>步驟 2：建立 PersonGroup

在此步驟中，我們已建立名為 "MyFriends" 的 PersonGroup，其中包含三位人員：Anna、Bill 和 Clare。 每個人都會註冊數個臉部。 您需要從影像中偵測到這些臉部。 在所有這些步驟之後，您會有 PersonGroup，如下圖所示：

![HowToIdentify1](../Images/group.image.1.jpg)

### <a name="21-define-people-for-the-persongroup"></a>2.1 定義 PersonGroup 的人員
人員是基本識別單位。 一個人可以註冊一或多個已知臉部。 不過，PersonGroup 是人員的集合，而且每位人員都是定義在特定 PersonGroup 中。 識別是針對 PersonGroup 執行。 因此，此工作是建立 PersonGroup，然後在其中建立人員 (例如 Anna、Bill 和 Clare)。

首先，您必須建立新的 PersonGroup。 這是使用 [PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API 所執行。 對應的用戶端程式庫 API 是 FaceServiceClient 類別的 CreatePersonGroupAsync 方法。 指定用來建立群組的群組識別碼對於每個訂用帳戶而言都是唯一的；您也可以使用其他 PersonGroup API 來取得、更新或刪除 PersonGroup。 定義群組之後，接著可以使用 [PersonGroup Person - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API 在其內定義人員。 用戶端程式庫方法是 CreatePersonAsync。 您可以在建立人員之後新增其臉部。

```CSharp 
// Create an empty PersonGroup
string personGroupId = "myfriends";
await faceServiceClient.CreatePersonGroupAsync(personGroupId, "My Friends");
 
// Define Anna
CreatePersonResult friend1 = await faceServiceClient.CreatePersonAsync(
    // Id of the PersonGroup that the person belonged to
    personGroupId,    
    // Name of the person
    "Anna"            
);
 
// Define Bill and Clare in the same way
```
### <a name="step2-2"></a> 2.2 偵測臉部並將它們註冊到正確人員
在 HTTP 要求本文中，使用影像檔將 "POST" Web 要求傳送給[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) API，即可完成偵測。 使用用戶端程式庫時，會透過 FaceServiceClient 類別的 DetectAsync 方法執行臉部偵測。

針對每個偵測到的臉部，您可以呼叫 [PersonGroup Person - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) 將它新增至正確人員。

下列程式碼示範如何從影像中偵測到臉部並將它新增給人員的程序：
```CSharp 
// Directory contains image files of Anna
const string friend1ImageDir = @"D:\Pictures\MyFriends\Anna\";
 
foreach (string imagePath in Directory.GetFiles(friend1ImageDir, "*.jpg"))
{
    using (Stream s = File.OpenRead(imagePath))
    {
        // Detect faces in the image and add to Anna
        await faceServiceClient.AddPersonFaceAsync(
            personGroupId, friend1.PersonId, s);
    }
}
// Do the same for Bill and Clare
``` 
請注意，若影像包含多個臉部，則只會新增最大的一張臉。 新增人員之其他臉部的方式是將 "targetFace = left, top, width, height" 格式的字串傳遞給 [PersonGroup Person - 新增臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) API 的 targetFace 查詢參數，或使用 AddPersonFaceAsync 方法的 targetFace 選擇性參數來新增其他臉部。 每個新增至人員的臉部都會獲指定唯一持續性臉部識別碼，以用於 [PersonGroup Person - 刪除臉部](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523e)和[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239)。

## <a name="step-3-train-the-persongroup"></a>步驟 3：為 PersonGroup 定型

必須先訓練 PersonGroup，才能使用它來執行識別。 甚至，在新增或移除任何人員之後，或者，若有任何人員編輯過其註冊的臉部，則必須重新予以訓練。 訓練是透過 [PersonGroup - 訓練](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) API 完成。 使用用戶端程式庫時，它就只是 TrainPersonGroupAsync 方法呼叫：
 
```CSharp 
await faceServiceClient.TrainPersonGroupAsync(personGroupId);
```
 
訓練是非同步程序。 這可能未完成，即使傳回 TrainPersonGroupAsync 方法之後也是一樣。 您可能需要透過 [PersonGroup - 取得訓練狀態](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395247) API 或是用戶端程式庫的 GetPersonGroupTrainingStatusAsync 方法，來查詢訓練狀態。 下列程式碼示範等待 PersonGroup 訓練完成的簡單邏輯：
 
```CSharp 
TrainingStatus trainingStatus = null;
while(true)
{
    trainingStatus = await faceServiceClient.GetPersonGroupTrainingStatusAsync(personGroupId);
 
    if (trainingStatus.Status != Status.Running)
    {
        break;
    }
 
    await Task.Delay(1000);
} 
``` 

## <a name="step-4-identify-a-face-against-a-defined-persongroup"></a>步驟 4：根據定義的 PersonGroup 來識別臉部

執行識別時，臉部 API 可以計算測試臉部與群組內所有臉部的相似度，並傳回與該測試臉部最像的人員。 這是透過[臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) API 或是用戶端程式庫的 IdentifyAsync 方法完成。

需要使用先前的步驟偵測測試臉部，接著將臉部識別碼傳遞給識別 API 作為第二個引數。 一次可以識別多個臉部識別碼，而結果會包含所有識別結果。 識別預設只會傳回一個最符合測試臉部的人。 若您想要的話，可以指定選擇性參數 maxNumOfCandidatesReturned，讓識別傳回多個候選項目。

下列程式碼示範識別程序︰

```CSharp 
string testImageFile = @"D:\Pictures\test_img1.jpg";

using (Stream s = File.OpenRead(testImageFile))
{
    var faces = await faceServiceClient.DetectAsync(s);
    var faceIds = faces.Select(face => face.FaceId).ToArray();
 
    var results = await faceServiceClient.IdentifyAsync(personGroupId, faceIds);
    foreach (var identifyResult in results)
    {
        Console.WriteLine("Result of face: {0}", identifyResult.FaceId);
        if (identifyResult.Candidates.Length == 0)
        {
            Console.WriteLine("No one identified");
        }
        else
        {
            // Get top 1 among all candidates returned
            var candidateId = identifyResult.Candidates[0].PersonId;
            var person = await faceServiceClient.GetPersonAsync(personGroupId, candidateId);
            Console.WriteLine("Identified as {0}", person.Name);
        }
    }
}
``` 

當您完成這些步驟時，可以嘗試識別不同的臉部，並根據上傳來偵測臉部的影像，確認是否可以正確地識別 Anna、Bill 或 Clare 的臉部。 請參閱下列範例：

![HowToIdentify2](../Images/identificationResult.1.jpg )

## <a name="step-5-request-for-large-scale"></a>步驟 5：大規模的要求

基於先前設計的限制，已知 PersonGroup 最多可以保留 10,000 個人。
如需高達百萬規模之情節的詳細資訊，請參閱[如何使用大規模功能](how-to-use-large-scale.md)。

## <a name="summary"></a>總結

在本指南中，您已了解建立 PersonGroup 以及識別人員的程序。 以下是您先前所說明和示範之功能的快速提醒：

- 使用[臉部 - 偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d) API 偵測臉部
- 使用 [PersonGroup - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API 建立 PersonGroup
- 使用 [PersonGroup Person - 建立](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API 建立人員
- 使用 [PersonGroup - 訓練](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) API 訓練 PersonGroup
- 使用 [臉部 - 識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) API，根據 PersonGroup 識別不明的臉部

## <a name="related-topics"></a>相關主題

- [如何偵測影像中的人臉](HowtoDetectFacesinImage.md)
- [如何新增臉部](how-to-add-faces.md)
- [如何使用大規模功能](how-to-use-large-scale.md)
