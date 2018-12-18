---
title: Machine Learning 批次執行服務作業的專用功能 | Microsoft Docs
description: 適用於 Machine Learning 作業的 Azure Batch 服務概觀。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.service: machine-learning
ms.component: studio
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.openlocfilehash: 1e4cf34582e28e00108e280d928eea8a8134699a
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834520"
---
# <a name="azure-batch-service-for-machine-learning-jobs"></a>適用於 Machine Learning 作業的 Azure Batch 服務

Machine Learning 批次集區處理提供客戶管理的 Azure Machine Learning 批次執行服務級別。 機器學習服務的傳統批次處理發生於多租用戶環境中，其限制您可以提交的並行作業數目，而且作業會以先進先出為原則排入佇列中。 這種不確定性，表示您無法準確地預測何時會執行您的作業。

批次集區處理可讓您建立集區，以便提交批次作業。 您可控制集區的大小，以及作業要提交至哪個集區。 BES 作業會在自己的處理空間中執行，以提供可預測的處理效能以及建立資源集區 (對應至您提交的處理負載) 的功能。

## <a name="how-to-use-batch-pool-processing"></a>如何使用批次集區處理

您目前無法透過 Azure 入口網站設定批次集區處理。 若要使用批次集區處理，您必須：

-   呼叫 CSS 以建立批次集區帳戶，並取得集區服務 URL 和授權金鑰
-   建立新的 Resource Manager 型 Web 服務和計費方案

若要建立帳戶，請連絡 Microsoft 客戶服務與支援中心 (CSS) 並提供您的訂用帳戶識別碼。 CSS 將與您一起決定您案例的適當容量。 CSS 會接著設定您的帳戶：您可以建立的集區數目上限，以及您可以放在每個集區中的虛擬機器 (VM) 數目上限。 設定您的帳戶後，您便會取得集區服務 URL 和授權金鑰。

建立您的帳戶之後，您可使用集區服務 URL 和授權金鑰，在您的批次集區上執行集區管理作業。

![批次集區服務架構。](./media/dedicated-capacity-for-bes-jobs/pool-architecture.png)

您可以在 CSS 提供給您的集區服務 URL 上呼叫「建立集區」作業，以建立集區。 當您建立集區時，請指定 VM 數目和全新 Resource Manager 型 Machine Learning Web 服務的 swagger.json URL。 此 Web 服務可供建立計費關聯。 批次集區服務會使用 swagger.json 將集區與計費方案建立關聯。 您可以執行您在集區上選擇的任何 BES Web 服務 (全新 Resource Manager 型和傳統)。

您可以使用任何全新 Resource Manager 型 Web 服務，但請注意，作業是根據與該服務相關聯的計費方案收費。 您可以特別建立 Web 服務和新的計費方案，以便執行批次集區作業。

如需關於建立 Web 服務的詳細資訊，請參閱[部署 Azure Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。

建立集區後，您可使用 Web 服務的批次要求 URL 提交 BES 作業。 您可以選擇將它提交至集區或傳統批次處理。 若要將作業提交至批次集區處理，請將下列參數新增至作業提交要求本文︰

"AzureBatchPoolId":"&lt;pool ID&gt;"

如果您未新增此參數，則會在傳統批次處理環境中執行作業。 如果集區有可用的資源，則會立即開始執行作業。 如果集區沒有可用的資源，則作業會排入佇列，直到有可用的資源為止。

如果您發現您定期達到集區的容量而且需要更高的容量，您可以連絡 CSS 並與支援代表一起增加您的配額。

範例要求︰

https://ussouthcentral.services.azureml.net/subscriptions/80c77c7674ba4c8c82294c3b2957990c/services/9fe659022c9747e3b9b7b923c3830623/jobs?api-version=2.0

```json
{

    "Input":{
    
        "ConnectionString":"DefaultEndpointsProtocol=https;BlobEndpoint=https://sampleaccount.blob.core.windows.net/;TableEndpoint
        =https://sampleaccount.table.core.windows.net/;QueueEndpoint=https://sampleaccount.queue.core.windows.net/;FileEndpoint=https://zhguim
        l.file.core.windows.net/;AccountName=sampleaccount;AccountKey=&lt;Key&gt;;",
        
        "BaseLocation":null,
        
        "RelativeLocation":"testint/AdultCensusIncomeBinaryClassificationDataset.csv",
        
        "SasBlobToken":null
    
    },
    
    "GlobalParameters":{ },
    
    "Outputs":{
    
        "output1":{
        
            "ConnectionString":"DefaultEndpointsProtocol=https;BlobEndpoint=https://sampleaccount.blob.core.windows.net/;TableEndpo
            int=https://sampleaccount.table.core.windows.net/;QueueEndpoint=https://sampleaccount.queue.core.windows.net/;FileEndpoint=https://sampleaccount.file.core.windows.net/;AccountName=sampleaccount;AccountKey=&lt;Key&gt;",
            "BaseLocation":null,
            "RelativeLocation":"testintoutput/testint\_results.csv",
            
            "SasBlobToken":null
        
        }
    
    },
    
    "AzureBatchPoolId":"8dfc151b0d3e446497b845f3b29ef53b"

}
```

## <a name="considerations-when-using-batch-pool-processing"></a>使用批次集區處理時的考量

批次集區處理是一項永遠開啟的計費服務，而且您必須將它與 Resource Manager 型計費方案建立關聯。 您只需支付集區執行的計算時數；不管該時間集區內執行的作業數目。 如果您建立集區，即使集區中沒有任何執行中的批次作業，您仍需支付集區中每部虛擬機器的計算時數，直到集區刪除為止。 虛擬機器會在完成佈建時開始計費，而在遭到刪除時停止計費。 您可以使用在 [Machine Learning 價格頁面](https://azure.microsoft.com/pricing/details/machine-learning/)找到的任何方案。

計費範例︰

如果您建立包含 2 部虛擬機器的批次集區，並在 24 小時後將它刪除，您的計費方案會借記 48 個計算小時；不論該期間內執行了多少作業。

如果您建立包含 4 部虛擬機器的批次集區，並在 12 小時後將它刪除，您的計費方案也會借記 48 個計算小時。

我們建議您輪詢作業狀態以判斷作業何時完成。 當所有作業完成執行時，請呼叫「調整集區大小」作業，將集區中的虛擬機器數目設定為零。 如果集區資源不足，而您需要建立新的集區 (例如根據不同計費方案進行計費)，您可以在所有作業都完成執行時刪除集區。


| **使用批次集區處理的時機**    | **使用傳統批次處理的時機**  |
|---|---|
|您需要執行大量的作業<br>或<br/>您需要知道您的作業將立即執行<br/>或<br/>您需要保證的輸送量。 例如，您需要在指定的時間範圍內執行多項作業，而且想要相應放大計算資源以符合您的需求。    | 您正在執行一些作業<br/>和<br/> 您不需要立即執行作業 |
