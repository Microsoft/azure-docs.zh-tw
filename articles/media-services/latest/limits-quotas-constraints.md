---
title: Azure 媒體服務 v3 配額和限制 | Microsoft Docs
description: 本主題描述 Azure 媒體服務 v3 的配額和限制
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: 5c0fbf396faa0e07ecca4ae16c775a39404c6fc9
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376511"
---
# <a name="quotas-and-limitations-in-azure-media-services-v3"></a>Azure 媒體服務 v3 配額和限制

本文描述 Azure 媒體服務 v3 的配額和限制。

| 資源 | 預設限制 | 
| --- | --- | 
| 每個 Azure 媒體服務帳戶的資產 | 1,000,000|
| 動態資訊清單篩選條件|100|
| 每個工作的工作輸入 | 50 (固定)|
| 每個作業的 JobOutputs/轉換中的 TransformOutputs | 20 (固定) |
| 每個 JobInput 的檔案|10 (固定)|
| 檔案大小| 在某些情況下，對於媒體服務中支援處理的檔案大小上限有所限制。 <sup>(1)</sup> |
| 每個媒體服務帳戶的工作 | 500,000 <sup>(2)</sup> (固定)|
| 列出轉換|將回應分頁，每頁 1000 個轉換|
| 列出作業|將回應分頁，每頁 500 個作業|
| 每個媒體服務帳戶的 LiveEvents |5|
| 單一訂用帳戶的媒體服務帳戶 | 25 (固定) |
| 每個 LiveEvent 其處於執行中狀態的 LiveOutputs |3|
| 儲存體帳戶 | 100<sup>(4)</sup> (固定) |
| 每個媒體服務其處於執行中狀態的串流端點|2|
| StreamingPolicies | 100 <sup>(3)</sup> |
| 每個媒體服務帳戶的轉換 | 100 (固定)|
| 一次與資產相關聯的唯一 StreamingLocator | 100<sup>(5)</sup> (固定) |

<sup>1</sup> 單一 blob 支援的大小上限目前在 Azure Blob 儲存體是最多 5 TB。 不過，其他的限制會以服務所使用的 VM 大小作為基礎套用在 Azure 媒體服務中。 如果原始程式檔超過 260 GB，您的工作可能會失敗。 如果您的 4K 內容超過 260 GB 限制，請來函寄到 amshelp@microsoft.com，我們將針對您的情況，提供解決的方法。

<sup>2</sup> 這個數字包括佇列、已完成、作用中和已取消的工作。 不包含已刪除的工作。 

您的帳戶中任何超過 90 天的工作記錄，都會自動刪除，即使記錄總數低於配額上限亦然。 

<sup>3</sup> 使用自訂的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 時，您應該為媒體服務帳戶設計一組受限的這類原則，並且在需要相同的加密選項和通訊協定時，對 StreamingLocators 重新使用這些原則。 不建議您對每個 StreamingLocator 建立新的 StreamingPolicy。

<sup>4</sup> 儲存體帳戶必須來自相同的 Azure 訂用帳戶。

<sup>5</sup> StreamingLocators 並非設計來管理每個使用者的存取控制。 若要給予個別使用者不同的存取權限，請使用數位版權管理 (DRM) 方案。

## <a name="support-ticket"></a>支援票證

如果是不固定的資源，您可以建立[支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)，要求增加配額。 請在要求中包含所需的配額變更、使用案例情況及所需區域的詳細資訊。 <br/>**請勿**為了嘗試取得更高的限制而建立其他 Azure 媒體服務。

## <a name="next-steps"></a>後續步驟

[概觀](media-services-overview.md)
