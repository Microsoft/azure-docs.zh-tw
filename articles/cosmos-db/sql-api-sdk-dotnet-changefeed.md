---
title: Azure Cosmos DB：.NET 變更摘要處理器 API、SDK 和資源 | Microsoft Docs
description: 了解「變更摘要處理器 API 和 SDK」的所有相關資訊，包括發行日期、停用日期，以及「.NET 變更摘要處理器 SDK」每個版本之間的變更。
services: cosmos-db
author: ealsur
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 05/21/2018
ms.author: maquaran
ms.openlocfilehash: 443cb2786c12bddd87535a7267d66c0332473b1b
ms.sourcegitcommit: 17fe5fe119bdd82e011f8235283e599931fa671a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2018
ms.locfileid: "42141059"
---
# <a name="net-change-feed-processor-sdk-download-and-release-notes"></a>.NET 變更摘要處理器 SDK：下載和版本資訊
> [!div class="op_single_selector"]
> * [.NET](sql-api-sdk-dotnet.md)
> * [.NET 變更摘要](sql-api-sdk-dotnet-changefeed.md)
> * [.NET Core](sql-api-sdk-dotnet-core.md)
> * [Node.js](sql-api-sdk-node.md)
> * [非同步 Java](sql-api-sdk-async-java.md)
> * [Java](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](https://docs.microsoft.com/rest/api/cosmos-db/)
> * [REST 資源提供者](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/)
> * [SQL](https://msdn.microsoft.com/library/azure/dn782250.aspx)
> * [BulkExecutor - .NET](sql-api-sdk-bulk-executor-dot-net.md)
> * [BulkExecutor - Java](sql-api-sdk-bulk-executor-java.md)

|   |   |
|---|---|
|**SDK 下載**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.ChangeFeedProcessor/)|
|**API 文件**|[變更摘要處理器程式庫 API 參考文件](/dotnet/api/microsoft.azure.documents.changefeedprocessor?view=azure-dotnet)|
|**開始使用**|[開始使用變更摘要處理器 .NET SDK](change-feed.md)|
|**目前支援的架構**| [Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)</br> [Microsoft .NET Core](https://www.microsoft.com/net/download/core) |

## <a name="release-notes"></a>版本資訊

### <a name="v2-builds"></a>v2 組建

### <a name="a-name206206"></a><a name="2.0.6"/>2.0.6
* 已新增 ChangeFeedEventHost.HostName 公用屬性以與 v1 相容。

### <a name="a-name205205"></a><a name="2.0.5"/>2.0.5
* 已修正會在分割區進行分割期間發生的競爭條件。 競爭條件可能會導致在取得租用後立即於分割區進行分割期間失去它，並導致競爭情形。 競爭條件問題已在此版本中修正。

### <a name="a-name204204"></a><a name="2.0.4"/>2.0.4
* GA SDK

### <a name="a-name203-prerelease203-prerelease"></a><a name="2.0.3-prerelease"/>2.0.3-prerelease
* 修正下列問題：
  * 當分割區進行分割時，可能會重複處理分割前修改的文件。
  * 租用集合中沒有租用時，GetEstimatedRemainingWork API 傳回 0。

* 下列例外狀況設為公開。 實作 IPartitionProcessor 的擴充可能擲回以下例外狀況。
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.LeaseLostException. 
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionException. 
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionNotFoundException.
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionSplitException. 

### <a name="a-name202-prerelease202-prerelease"></a><a name="2.0.2-prerelease"/>2.0.2 發行前版本
* 次要 API 變更：
  * 移除已標記為過時的 ChangeFeedProcessorOptions.IsAutoCheckpointEnabled。

### <a name="a-name201-prerelease201-prerelease"></a><a name="2.0.1-prerelease"/>2.0.1 發行前版本
* 穩定性改進：
  * 改善租用存放區初始化的處理。 如果租用存放區是空的，則只有一個處理器的執行個體可以將它初始化，其他執行個體則須等候。
  * 更穩定/有效的租用更新/版本。 更新和釋放每一個分割區的租用會與其他租用的更新各自獨立。 在 V1 中，所有分割區會循序完成此動作。
* 新的 v2 API：
  * 處理器彈性建構的產生器模式：ChangeFeedProcessorBuilder 類別。
    * 可取用參數的任意組合。
    * 可將 DocumentClient 執行個體用於監視和 (或) 租用集合 (不適用於 v1)。
  * IChangeFeedObserver.ProcessChangesAsync 現在會取用 CancellationToken。
  * IRemainingWorkEstimator - 剩餘工作估算器可在處理器以外單獨使用。
  * 新的擴充點：
    * IParitionLoadBalancingStrategy - 用於處理器執行個體之間的分割區自訂負載平衡。
    * ILease、ILeaseManager - 用於自訂租用管理。
    * IPartitionProcessor - 用於分割區上的自訂處理變更。
* 記錄 - 使用 [LibLog](https://github.com/damianh/LibLog) 程式庫。
* 可 100% 回溯相容於 v1 API。
* 新的程式碼基底。
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.21.1 版和更新版本相容。

### <a name="v1-builds"></a>v1 組建

### <a name="a-name133133"></a><a name="1.3.3"/>1.3.3
* 已新增更多記錄。
* 已修正在多次呼叫擱置中工作估計時的 DocumentClient 流失。

### <a name="a-name132132"></a><a name="1.3.2"/>1.3.2
* 修正擱置的工作評估。

### <a name="a-name131131"></a><a name="1.3.1"/>1.3.1
* 穩定性改進。
  * 修正處理取消工作的問題，此問題可能會導致某些分割區上的觀察者停止運作。
* 支援手動檢查點。
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.21 版和更新版本相容。

### <a name="a-name120120"></a><a name="1.2.0"/>1.2.0
* 新增 .NET Standard 2.0 的支援。 套件現在支援 `netstandard2.0` 和 `net451` Framework Moniker。
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.17.0 或以上版本相容。
* 與 [SQL .NET Core SDK](sql-api-sdk-dotnet-core.md) 1.5.1 或以上版本相容。

### <a name="a-name111111"></a><a name="1.1.1"/>1.1.1
* 變更摘要為空白或沒有已暫止的工作時，會修正計算剩餘工作估計的問題。
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.13.2 或以上版本相容。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0
* 新增方法以取得要在變更摘要中處理之剩餘工作的估算。
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.13.2 或以上版本相容。

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* GA SDK
* 與 [SQL .NET SDK](sql-api-sdk-dotnet.md) 1.14.1 或以上版本相容。


## <a name="release--retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

新的功能與最佳化項目只會新增至目前的 SDK，因此建議您一律盡早升級至最新的 SDK 版本。 

服務將會拒絕使用已停用 SDK 的任何 Cosmos DB 要求。

<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [1.3.3](#1.3.3) |2018 年 5 月 8 日 |--- |
| [1.3.2](#1.3.2) |2018 年 4 月 18 日 |--- |
| [1.3.1](#1.3.1) |2018 年 3 月 13 日 |--- |
| [1.2.0](#1.2.0) |2017 年 10 月 31 日 |--- |
| [1.1.1](#1.1.1) |2017 年 8 月 29 日 |--- |
| [1.1.0](#1.1.0) |2017 年 8 月 13 日 |--- |
| [1.0.0](#1.0.0) |2017 年 7 月 7 日 |--- |


## <a name="faq"></a>常見問題集
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。 

