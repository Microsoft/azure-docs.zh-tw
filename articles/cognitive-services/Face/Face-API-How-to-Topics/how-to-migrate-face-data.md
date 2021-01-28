---
title: 跨訂用帳戶遷移您的臉部資料-臉部
titleSuffix: Azure Cognitive Services
description: 本指南說明如何將已儲存的臉部資料從某個臉部訂用帳戶遷移至另一個。
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: nitinme
ms.custom: devx-track-csharp
ms.openlocfilehash: b2b3ebdf61349d88d088ebeff5443a9c3e947d73
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98943620"
---
# <a name="migrate-your-face-data-to-a-different-face-subscription"></a>將您的臉部資料移轉至其他臉部訂用帳戶

本指南說明如何將臉部資料（例如具有臉部的已儲存 PersonGroup 物件）移至不同的 Azure 認知服務臉部訂用帳戶。 若要移動資料，您可以使用快照集功能。 如此一來，當您移動或展開作業時，就不必重複建立和定型 PersonGroup 或 FaceList 物件。 例如，您可能建立了具有免費訂用帳戶的 PersonGroup 物件，而現在想要將它遷移至付費訂用帳戶。 或者，您可能需要在不同區域中的訂用帳戶之間同步處理臉部資料，以進行大型企業操作。

相同的移轉策略也適用於 LargePersonGroup 和 LargeFaceList 物件。 如果您不熟悉本指南中的概念，請參閱 [臉部辨識概念](../concepts/face-recognition.md) 指南中的定義。 本指南使用臉部 .NET 用戶端程式庫搭配 c #。

## <a name="prerequisites"></a>先決條件

您需要下列專案：

- 兩個臉部訂用帳戶金鑰，一個具有現有的資料，另一個用於遷移至。 若要訂閱臉部辨識服務並取得您的金鑰，請遵循 [建立認知服務帳戶](../../cognitive-services-apis-create-account.md)中的指示。
- 對應到目標訂用帳戶的臉部訂用帳戶識別碼字串。 若要找到它，請選取 Azure 入口網站中的 **[總覽** ]。 
- 任何 [Visual Studio 2015 或 2017](https://www.visualstudio.com/downloads/) 版本。

## <a name="create-the-visual-studio-project"></a>建立 Visual Studio 專案

本指南使用簡單的主控台應用程式來執行臉部資料移轉。 如需完整的執行，請參閱 GitHub 上的 [臉部快照集範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) 。

1. 在 Visual Studio 中，建立新的主控台應用程式 .NET Framework 專案。 將它命名為 **FaceApiSnapshotSample**。
1. 取得必要的 NuGet 套件。 在方案總管中的專案上按一下滑鼠右鍵，然後選取 [ **管理 NuGet 套件**]。 選取 [ **流覽** ] 索引標籤，然後選取 [ **包含發行** 前版本]。 尋找並安裝下列套件：
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.3.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview) \(英文\)

## <a name="create-face-clients"></a>建立臉部用戶端

在 *Program.cs* 的 **Main** 方法中，為您的來源和目標訂用帳戶建立兩個 [FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) 執行個體。 此範例會使用東亞區域中的臉部訂用帳戶作為來源，並使用美國西部訂用帳戶作為目標。 此範例示範如何將資料從一個 Azure 區域遷移至另一個區域。 

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

```csharp
var FaceClientEastAsia = new FaceClient(new ApiKeyServiceClientCredentials("<East Asia Subscription Key>"))
    {
        Endpoint = "https://southeastasia.api.cognitive.microsoft.com/>"
    };

var FaceClientWestUS = new FaceClient(new ApiKeyServiceClientCredentials("<West US Subscription Key>"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com/"
    };
```

填入您的來源和目標訂閱的訂用帳戶金鑰值和端點 Url。


## <a name="prepare-a-persongroup-for-migration"></a>備妥移轉的 PersonGroup

若要將您來源訂用帳戶中的 PersonGroup 移轉到目標訂用帳戶，您會需要它的識別碼。 使用 [PersonGroupOperationsExtensions. metrics.listasync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperationsextensions.listasync) 方法來取出 PersonGroup 物件的清單。 然後取得 [PersonGroup PersonGroupId](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.persongroup.persongroupid#Microsoft_Azure_CognitiveServices_Vision_Face_Models_PersonGroup_PersonGroupId) 屬性。 根據您的 PersonGroup 物件而定，此程式看起來會不同。 在本指南中，來源 PersonGroup 識別碼會儲存在中 `personGroupId` 。

> [!NOTE]
> [範例程式碼](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample)會建立並定型要遷移的新 PersonGroup。 在大多數情況下，您應該已經有要使用的 PersonGroup。

## <a name="take-a-snapshot-of-a-persongroup"></a>製作 PersonGroup 的快照集

快照集是特定臉部資料類型的暫時性遠端儲存體。 將資料從一個訂用帳戶複製到另一個訂用帳戶時，它的功能是類似剪貼簿。 首先，您會取得來源訂用帳戶中資料的快照集。 然後將它套用至目標訂用帳戶中的新資料物件。

使用來源訂用帳戶的 FaceClient 實例來取得 PersonGroup 的快照。 使用 [TakeAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperationsextensions.takeasync) 搭配 PersonGroup 識別碼和目標訂用帳戶的識別碼。 如果您有多個目標訂用帳戶，請將它們新增為第三個參數中的陣列專案。

```csharp
var takeSnapshotResult = await FaceClientEastAsia.Snapshot.TakeAsync(
    SnapshotObjectType.PersonGroup,
    personGroupId,
    new[] { "<Azure West US Subscription ID>" /* Put other IDs here, if multiple target subscriptions wanted */ });
```

> [!NOTE]
> 採用和套用快照集的程式並不會中斷對來源或目標 Persongroup 或 Facelist 的任何正常呼叫。 請勿進行同時呼叫來變更來源物件，例如 [FaceList 管理呼叫](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations) 或 [PersonGroup 訓練](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations) 通話。 快照集作業可能會在這些作業之前或之後執行，否則可能會發生錯誤。

## <a name="retrieve-the-snapshot-id"></a>取得快照集識別碼

用來取得快照集的方法是非同步，因此您必須等待其完成。 無法取消快照集作業。 在此程式碼中， `WaitForOperation` 方法會監視非同步呼叫。 它會每隔100毫秒檢查一次狀態。 在作業完成之後，藉由剖析欄位來取得作業識別碼 `OperationLocation` 。 

```csharp
var takeOperationId = Guid.Parse(takeSnapshotResult.OperationLocation.Split('/')[2]);
var operationStatus = await WaitForOperation(FaceClientEastAsia, takeOperationId);
```

一般 `OperationLocation` 值看起來像這樣：

```csharp
"/operations/a63a3bdd-a1db-4d05-87b8-dbad6850062a"
```

`WaitForOperation` 協助程式方法如下：

```csharp
/// <summary>
/// Waits for the take/apply operation to complete and returns the final operation status.
/// </summary>
/// <returns>The final operation status.</returns>
private static async Task<OperationStatus> WaitForOperation(IFaceClient client, Guid operationId)
{
    OperationStatus operationStatus = null;
    do
    {
        if (operationStatus != null)
        {
            Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }

        // Get the status of the operation.
        operationStatus = await client.Snapshot.GetOperationStatusAsync(operationId);

        Console.WriteLine($"Operation Status: {operationStatus.Status}");
    }
    while (operationStatus.Status != OperationStatusType.Succeeded
            && operationStatus.Status != OperationStatusType.Failed);

    return operationStatus;
}
```

在作業狀態顯示之後 `Succeeded` ，藉由剖析 `ResourceLocation` 所傳回 >operationstatus 實例的欄位來取得快照集識別碼。

```csharp
var snapshotId = Guid.Parse(operationStatus.ResourceLocation.Split('/')[2]);
```

一般 `resourceLocation` 值看起來像這樣：

```csharp
"/snapshots/e58b3f08-1e8b-4165-81df-aa9858f233dc"
```

## <a name="apply-a-snapshot-to-a-target-subscription"></a>將快照集套用至目標訂用帳戶

接下來，使用隨機產生的識別碼在目標訂用帳戶中建立新的 PersonGroup。 然後使用目標訂用帳戶的 FaceClient 實例，將快照集套用到此 PersonGroup。 傳入快照集識別碼和新的 PersonGroup 識別碼。

```csharp
var newPersonGroupId = Guid.NewGuid().ToString();
var applySnapshotResult = await FaceClientWestUS.Snapshot.ApplyAsync(snapshotId, newPersonGroupId);
```


> [!NOTE]
> 快照集物件只適用于48小時。 如果您想要在不久之後將其用於資料移轉，則只需要使用快照集。

快照套用要求會傳回另一個作業識別碼。 若要取得此識別碼，請剖析 `OperationLocation` 所傳回 applySnapshotResult 實例的欄位。 

```csharp
var applyOperationId = Guid.Parse(applySnapshotResult.OperationLocation.Split('/')[2]);
```

快照集應用程式進程也是非同步，因此，請再次使用 `WaitForOperation` 來等候它完成。

```csharp
operationStatus = await WaitForOperation(FaceClientWestUS, applyOperationId);
```

## <a name="test-the-data-migration"></a>測試資料移轉

套用快照集之後，目標訂用帳戶中的新 PersonGroup 會填入原始的臉部資料。 根據預設，也會複製定型結果。 新的 PersonGroup 已準備好進行臉部辨識通話，而不需要重新定型。

若要測試資料移轉，請執行下列作業，並比較它們列印到主控台的結果：

```csharp
await DisplayPersonGroup(FaceClientEastAsia, personGroupId);
await IdentifyInPersonGroup(FaceClientEastAsia, personGroupId);

await DisplayPersonGroup(FaceClientWestUS, newPersonGroupId);
// No need to retrain the person group before identification,
// training results are copied by snapshot as well.
await IdentifyInPersonGroup(FaceClientWestUS, newPersonGroupId);
```

使用下列協助程式方法：

```csharp
private static async Task DisplayPersonGroup(IFaceClient client, string personGroupId)
{
    var personGroup = await client.PersonGroup.GetAsync(personGroupId);
    Console.WriteLine("Person Group:");
    Console.WriteLine(JsonConvert.SerializeObject(personGroup));

    // List persons.
    var persons = await client.PersonGroupPerson.ListAsync(personGroupId);

    foreach (var person in persons)
    {
        Console.WriteLine(JsonConvert.SerializeObject(person));
    }

    Console.WriteLine();
}
```

```csharp
private static async Task IdentifyInPersonGroup(IFaceClient client, string personGroupId)
{
    using (var fileStream = new FileStream("data\\PersonGroup\\Daughter\\Daughter1.jpg", FileMode.Open, FileAccess.Read))
    {
        var detectedFaces = await client.Face.DetectWithStreamAsync(fileStream);

        var result = await client.Face.IdentifyAsync(detectedFaces.Select(face => face.FaceId.Value).ToList(), personGroupId);
        Console.WriteLine("Test identify against PersonGroup");
        Console.WriteLine(JsonConvert.SerializeObject(result));
        Console.WriteLine();
    }
}
```

現在您可以在目標訂用帳戶中使用新的 PersonGroup。 

若要在未來再次更新目標 PersonGroup，請建立新的 PersonGroup 來接收快照集。 若要這樣做，請遵循本指南中的步驟。 單一 PersonGroup 物件只能套用一次快照集。

## <a name="clean-up-resources"></a>清除資源

當您完成遷移臉部資料之後，請手動刪除快照集物件。

```csharp
await FaceClientEastAsia.Snapshot.DeleteAsync(snapshotId);
```

## <a name="next-steps"></a>後續步驟

接下來，請參閱相關的 API 參考檔、探索使用快照集功能的範例應用程式，或遵循操作指南來開始使用此處所述的其他 API 作業：

- [快照集參考文件 (.NET SDK)](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperations)
- [臉部快照範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample)
- [新增臉部](how-to-add-faces.md)
- [偵測影像中的臉部](HowtoDetectFacesinImage.md)
