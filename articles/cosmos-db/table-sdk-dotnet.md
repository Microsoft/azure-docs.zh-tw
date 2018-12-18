---
title: Azure Cosmos DB 資料表 API .NET SDK 和資源 | Microsoft Docs
description: 全面了解 Azure Cosmos DB 資料表 API，包括發行日期、停用日期，以及每個版本之間的變更。
services: cosmos-db
author: rnagpal
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/17/2018
ms.author: rnagpal
ms.openlocfilehash: 1f2fae7bf500a469bc789fc2296fac5b653d1538
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451760"
---
# <a name="azure-cosmos-db-table-net-api-download-and-release-notes"></a>Azure Cosmos DB 資料表 .NET API：下載和版本資訊
> [!div class="op_single_selector"]
> * [.NET](table-sdk-dotnet.md)
> * [Java](table-sdk-java.md)
> * [Node.js](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   |   |
|---|---|
|**SDK 下載**|[NuGet](https://aka.ms/acdbtablenuget)|
|**API 文件**|[.NET API 參考文件](https://aka.ms/acdbtableapiref)|
|**快速入門**|[Azure Cosmos DB：使用 .NET 和資料表 API 建置應用程式 (英文)](create-table-dotnet.md)|
|**教學課程**|[Azure Cosmos DB：使用 .NET 搭配資料表 API 進行開發](tutorial-develop-table-dotnet.md)|
|**目前支援的架構**|[Microsoft .NET Framework 4.5.1](https://www.microsoft.com/en-us/download/details.aspx?id=40779)|

> [!IMPORTANT]
> 如果您在預覽期間建立了資料表 API 帳戶，請建立[新表格 API 帳戶](create-table-dotnet.md#create-a-database-account)以使用正式推出的資料表 API SDK。
>

## <a name="release-notes"></a>版本資訊

### <a name="a-name200200"></a><a name="2.0.0"/>2.0.0
* 已新增多區域寫入支援
* 已修正 Microsoft.Azure.DocumentDB、Microsoft.OData.Core、Microsoft.OData.Edm、Microsoft.Spatial 上的 NuGet 套件相依性

### <a name="a-name113113"></a><a name="1.1.3"/>1.1.3
* 已修正 Microsoft.Azure.Storage.Common 和 Microsoft.Azure.DocumentDB 上的 NuGet 套件相依性。
* 設定 JsonConvert.DefaultSettings 時將資料表序列化的錯誤修正。

### <a name="a-name111111"></a><a name="1.1.1"/>1.1.1
* 新增格式錯誤的 ETAG 在「直接模式」中的驗證。
* 修正閘道模式中 LINQ 的查詢錯誤 (bug)。
* 同步 API 現在使用 SynchronizationContext 在執行緒集區上執行。

### <a name="a-name110110"></a><a name="1.1.0"/>1.1.0
* 將 TableQueryMaxItemCount、TableQueryEnableScan、TableQueryMaxDegreeOfParallelism 及 TableQueryContinuationTokenLimitInKb 新增至 TableRequestOptions
* 錯誤修正

### <a name="a-name100100"></a><a name="1.0.0"/>1.0.0
* 公開上市版本

### <a name="a-name010-preview090-preview"></a><a name="0.1.0-preview"/>0.9.0-preview
* 初始預覽版本

## <a name="release-and-retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

[WindowsAzure.Storage-PremiumTable](https://www.nuget.org/packages/WindowsAzure.Storage-PremiumTable/0.1.0-preview) 預覽套件已由 [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) 套件取代。 WindowsAzure.Storage-PremiumTable SDK 將於 2018 年 11 月 15 日淘汰，屆時不允許使用已停用 SDK 的要求。 `Microsoft.Azure.CosmosDB.Table` 文件庫目前僅適用於 .NET Standard，尚不適用於 .NET Core。

新的功能與最佳化項目只會新增至目前的 SDK，因此建議您一律盡早升級至最新的 SDK 版本。 

服務會拒絕使用已停用 SDK 的任何 Azure Cosmos DB 要求。
<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [1.1.3](#1.1.3) |2018 年 7 月 17 日|--- |
| [1.1.1](#1.1.1) |2018 年 3 月 26 日|--- |
| [1.1.0](#1.1.0) |2018 年 2 月 21 日|--- |
| [1.0.0](#1.0.0) |2017 年 11 月 15 日|--- |
| [0.9.0-preview](#0.9.0-preview) |2017 年 11 月 11 日 |--- |

## <a name="troubleshooting"></a>疑難排解

如果發生錯誤 

```
Unable to resolve dependency 'Microsoft.Azure.Storage.Common'. Source(s) used: 'nuget.org', 
'CliFallbackFolder', 'Microsoft Visual Studio Offline Packages', 'Microsoft Azure Service Fabric SDK'`
```

(在嘗試使用 Microsoft.Azure.CosmosDB.Table NuGet 套件時)，有兩個選項可以修正這個問題：

* 使用套件管理器主控台安裝 Microsoft.Azure.CosmosDB.Table 套件和其相依項目。 若要這樣做，請在方案的套件管理器主控台中輸入下列項目。 
    ```
    Install-Package Microsoft.Azure.CosmosDB.Table -IncludePrerelease
    ```
    
* 使用慣用的 NuGet 套件管理工具，先安裝 Microsoft.Azure.Storage.Common NuGet 套件，再安裝 Microsoft.Azure.CosmosDB.Table。

## <a name="faq"></a>常見問題集

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Azure Cosmos DB 資料表 API，請參閱 [Azure Cosmos DB 資料表 API 的簡介](table-introduction.md)。 
