---
title: 從 Azure 媒體服務 v2 遷移至 v3 | Microsoft Docs
description: 本文說明 Azure 媒體服務 v3 中所導入的變更，並說明兩個版本之間的差異。
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: Azure 媒體服務, 串流, 廣播, 即時, 離線
ms.service: media-services
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: a17bad5beb651aaa085c626296c200a00c30647e
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376357"
---
# <a name="migrate-from-media-services-v2-to-v3"></a>從媒體服務 v2 遷移至 v3

本文說明 Azure 媒體服務 (AMS) v3 中所導入的變更，並說明兩個版本之間的差異。

## <a name="why-should-you-move-to-v3"></a>應該改用 v3 的理由？

### <a name="api-is-more-approachable"></a>API 會更平易近人

*  v3 是以統一的 API 表面為基礎的，會公開建置於 Azure Resource Manager 上的管理和操作功能。 Azure Resource Manager 範本可用於建立及部署轉換、串流端點、LiveEvent 等等。
* Open API (也稱為 Swagger) 規格文件。
* SDK 適用於 .Net、.Net Core、Node.js、Python、Java、Ruby。
* Azure CLI 整合。

### <a name="new-features"></a>新功能

* 編碼現在可支援 HTTPS 內嵌 (以 Url 為基礎的輸入)。
* 轉換是 v3 的新功能。 轉換可用來共用設定、建立 Azure Resource Manager 範本，以及找出特定客戶或租用戶的編碼設定。 
* 資產可以有多個 StreamingLocator，各自具有不同的動態封裝和動態加密設定。
* 內容保護可支援多索引鍵功能。 
* LiveEvent 預覽版可支援動態封裝和動態加密。 這可在預覽版上實現內容保護，以及 DASH 和 HLS 封裝。
* LiveOuput 在使用上比舊有的程式實體更簡單。 
* 已新增對實體的 RBAC 支援。

## <a name="changes-from-v2"></a>從 v2 的變更

* 在媒體服務 v3 中，如果您的資產是以媒體服務 v2 建立，則儲存體加密 (AES-256 加密) 只對回溯相容性有所支援。 這表示 v3 可用於現有的儲存體加密資產，但不允許建立新的。

    對於以 v3 建立的資產，媒體服務可支援 [Azure 儲存體](https://docs.microsoft.com/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)伺服器端儲存體加密。
    
* 媒體服務 SDK 已與儲存體 SDK 分離，其掌控力勝過所使用的儲存體 SDK，並可避免版本控制問題。 
* 在 v3 中，所有的編碼位元速率單位都是「位元/秒」。 這不同於 REST v2 媒體編碼器標準預設。 例如，v2 中的位元速率會指定為 128，但在 v3 中，則會是 128000。 
* AssetFiles、AccessPolicies、IngestManifests 不存在於 v3 中。
* ContentKey 不再是實體，StreamingLocator 的屬性。
* 事件方格支援取代了 NotificationEndpoint。
* 已重新命名某些實體

  * JobOutput 取代了工作，現在只是作業的一部分。
  * LiveEvent 取代了通道。
  * LiveOutput 取代了程式。
  * StreamingLocator 取代了定位器。

## <a name="code-changes"></a>程式碼變更

### <a name="create-an-asset-and-upload-a-file"></a>建立資產並上傳檔案 

#### <a name="v2"></a>v2

```csharp
IAsset asset = context.Assets.Create(assetName, storageAccountName, options);

IAssetFile assetFile = asset.AssetFiles.Create(assetFileName);

assetFile.Upload(filePath);
```

#### <a name="v3"></a>v3

```csharp
Asset asset = client.Assets.CreateOrUpdate(resourceGroupName, accountName, assetName, new Asset());

var response = client.Assets.ListContainerSas(resourceGroupName, accountName, assetName, permissions: AssetContainerPermission.ReadWrite, expiryTime: DateTime.Now.AddHours(1));

var sasUri = new Uri(response.AssetContainerSasUrls.First());
CloudBlobContainer container = new CloudBlobContainer(sasUri);

var blob = container.GetBlockBlobReference(Path.GetFileName(fileToUpload));
blob.UploadFromFile(fileToUpload);
```

### <a name="submit-a-job"></a>提交作業

#### <a name="v2"></a>v2

```csharp
IMediaProcessor processor = context.MediaProcessors.GetLatestMediaProcessorByName(mediaProcessorName);

IJob job = jobs.Create($"Job for {inputAsset.Name}");

ITask task = job.Tasks.AddNew($"Task for {inputAsset.Name}", processor, taskConfiguration);

task.InputAssets.Add(inputAsset);

task.OutputAssets.AddNew(outputAssetName, outputAssetStorageAccountName, outputAssetOptions);

job.Submit();
```

#### <a name="v3"></a>v3

```csharp
client.Assets.CreateOrUpdate(resourceGroupName, accountName, outputAssetName, new Asset());

JobOutput[] jobOutputs = { new JobOutputAsset(outputAssetName)};

JobInput jobInput = JobInputAsset(assetName: assetName);

Job job = client.Jobs.Create(resourceGroupName,
accountName, transformName, jobName,
new Job {Input = jobInput, Outputs = jobOutputs});
```

### <a name="publish-an-asset-with-aes-encryption"></a>使用 AES 加密發行資產 

#### <a name="v2"></a>v2

1. 建立 ContentKeyAuthorizationPolicyOption
2. 建立 ContentKeyAuthorizationPolicy
3. 建立 AssetDeliveryPolicy
4. 建立資產並上傳內容，或提交作業並使用輸出資產
5. 讓 AssetDeliveryPolicy 與資產產生關聯
6. 建立 ContentKey
7. 將 ContentKey 連結至資產
8. 建立 AccessPolicy
9. 建立定位器

#### <a name="v3"></a>v3

1. 建立內容金鑰原則
2. 建立資產
3. 上傳內容或將資產作為 JobOutput
4. 建立 StreamingLocator

## <a name="next-steps"></a>後續步驟

若要了解開始撰寫程式碼並串流視訊檔案是多麼容易，請查看[串流檔案](stream-files-dotnet-quickstart.md)。 

