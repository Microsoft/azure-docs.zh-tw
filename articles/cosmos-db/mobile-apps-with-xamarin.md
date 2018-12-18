---
title: 使用 Xamarin 和 Azure Cosmos DB 建置行動應用程式 | Microsoft Docs
description: 使用 Azure Cosmos DB 建立 Xamarin iOS、Android 或 Forms 應用程式的教學課程。 Azure Cosmos DB 是快速的全球級行動應用程式雲端資料庫。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/15/2017
ms.author: sngun
ms.openlocfilehash: 0187bf51d36f16001f0f7f67f4f72f5f3df2e05d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974131"
---
# <a name="build-mobile-applications-with-xamarin-and-azure-cosmos-db"></a>使用 Xamarin 和 Azure Cosmos DB 建置行動應用程式

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

大部分的行動應用程式都需要將資料儲存在雲端，而 Azure Cosmos DB 正是適用於行動應用程式的雲端資料庫。 行動應用程式開發人員所需的一切盡在其中。 它是完全受控的資料庫即服務，可依需求進行調整。 無論您的使用者位於全球哪個地方，它都可以透明方式將資料送至您的應用程式。 使用 [Azure Cosmos DB .NET Core SDK](sql-api-sdk-dotnet-core.md)，您可以讓 Xamarin 行動應用程式直接與 Azure Cosmos DB 互動，而不需要經過中介層。

本文提供的教學課程可讓您了解如何使用 Xamarin 和 Azure Cosmos DB 建置行動應用程式。 您可以在 [GitHub 上的 Xamarin 和 Azure Cosmos DB](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin) 中找到本教學課程的完整原始程式碼，包括如何管理使用者和權限。

## <a name="azure-cosmos-db-capabilities-for-mobile-apps"></a>適用於行動應用程式的 Azure Cosmos DB 功能
Azure Cosmos DB 為行動應用程式開發人員提供了下列重要功能：

![適用於行動應用程式的 Azure Cosmos DB 功能](media/mobile-apps-with-xamarin/documentdb-for-mobile.png)

* 豐富的無結構描述資料查詢。 Azure Cosmos DB 會將資料以無結構描述 JSON 文件的形式儲存在異質集合中。 它提供[豐富且快速的查詢](sql-api-sql-query.md)，不需要擔心結構描述或索引。
* 快速的輸送量。 只需要幾毫秒就能使用 Azure Cosmos DB 讀取和寫入文件。 開發人員可以指定他們需要的輸送量，Azure Cosmos DB 則以 99.99% 可用性的 SLA 加以實現，適用於一致性很寬鬆的所有單一區域帳戶和所有多重區域帳戶，而所有多重區域資料庫帳戶則有 99.999% 的讀取可用性。
* 規模無限制。 您的 Azure Cosmos DB 集合可[隨著應用程式成長](partition-data.md)。 您可以從小型資料大小和每秒數百個要求的輸送量著手。 您的集合或資料庫可以成長至數 PB 的資料以及每秒數億個要求的極大輸送量。
* 散布世界各地。 行動應用程式使用者總是忙個不停，經常在世界各地奔波。 Azure Cosmos DB 是[分散在世界各地的資料庫](distribute-data-globally.md)。 按一下地圖，讓使用者能夠存取您的資料。
* 內建豐富授權。 使用 Azure Cosmos DB 就能輕鬆實作熱門模式，例如[每位使用者的資料](https://aka.ms/documentdb-xamarin-todouser)或多位使用者共用的資料，而不需要複雜的自訂授權程式碼。
* 地理空間查詢。 許多行動應用程式目前都有提供地理情境體驗。 透過一流的[地理空間類型](geospatial.md)支援，Azure Cosmos DB 可讓您輕鬆實現這些體驗。
* 二進位附件。 您的應用程式資料通常包含二進位 Blob。 原生的附件支援可讓您更輕鬆地使用 Azure Cosmos DB 做為應用程式資料的集散中心。

## <a name="azure-cosmos-db-and-xamarin-tutorial"></a>Azure Cosmos DB 和 Xamarin 教學課程
下列教學課程示範如何使用 Xamarin 和 Azure Cosmos DB 建置行動應用程式。 您可以在 [GitHub 上的 Xamarin 和 Azure Cosmos DB](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin) 中找到本教學課程的完整原始程式碼。

### <a name="get-started"></a>開始使用
您可以輕鬆地開始使用 Azure Cosmos DB。 移至 Azure 入口網站，然後建立新的 Azure Cosmos DB 帳戶。 按一下 [快速入門] 索引標籤。下載已連接至您 Azure Cosmos DB 帳戶的 Xamarin 表單待辦事項清單範例。 

![適用於行動應用程式的 Azure Cosmos DB 快速入門](media/mobile-apps-with-xamarin/cosmos-db-quickstart.png)

或者，如果您有現有的 Xamarin 應用程式，您可以新增 [Azure Cosmos DB NuGet 套件](sql-api-sdk-dotnet-core.md)。 Azure Cosmos DB 支援 Xamarin.IOS、Xamarin.Android 和 Xamarin Forms 共用程式庫。

### <a name="work-with-data"></a>使用資料
您的資料記錄會以無結構描述 JSON 文件的形式儲存在 Azure Cosmos DB 的異質集合中。 您可以在相同的集合中使用不同結構儲存文件：

```cs
    var result = await client.CreateDocumentAsync(collectionLink, todoItem);
```

在您的 Xamarin 專案中，您可以使用語言整合式的無結構描述資料查詢︰

```cs
    var query = await client.CreateDocumentQuery<ToDoItem>(collectionLink)
                    .Where(todoItem => todoItem.Complete == false)
                    .AsDocumentQuery();

    Items = new List<TodoItem>();
    while (query.HasMoreResults) {
        Items.AddRange(await query.ExecuteNextAsync<TodoItem>());
    }
```
### <a name="add-users"></a>新增使用者
如同許多開始使用範例，您下載的 Azure Cosmos DB 範例會使用應用程式程式碼中硬式編碼的主要金鑰來向服務進行驗證。 除了本機模擬器外，此預設值不適合用於您想要執行的應用程式。 如果未經授權的使用者取得主要金鑰，則整個 Azure Cosmos DB 帳戶的所有資料可能會外洩。 您反而會希望您的應用程式只存取已登入使用者的記錄。 Azure Cosmos DB 可讓開發人員對應用程式授與集合、依分割區索引鍵群組的一組文件或特定文件的讀取或讀取/寫入權限。 

請依照下列步驟，將待辦事項清單應用程式修改為多使用者的待辦事項清單應用程式︰ 

  1. 使用 Facebook、Active Directory 或其他任何提供者，對應用程式新增登入。

  2. 使用 **/userId** 做為分割區索引鍵來建立 Azure Cosmos DB UserItems 集合。 為集合指定分割區索引鍵，可讓 Azure Cosmos DB 隨著應用程式使用者人數的成長無限制地擴充，同時繼續提供快速的查詢。

  3. 新增 Azure Cosmos DB 資源權杖訊息代理程式。 這個簡單的 Web API 可驗證使用者，並對已登入的使用者發出只能存取其資料分割內文件的短期權杖。 在此範例中，資源權杖訊息代理程式裝載於 App Service 中。

  4. 修改應用程式以使用 Facebook 向資源權杖訊息代理程式驗證，並且向已登入的 Facebook 使用者要求資源權杖。 然後您可以存取其在 UserItems 集合中的資料。  

您可以在 [GitHub 上的資源權杖訊息代理程式](http://aka.ms/documentdb-xamarin-todouser)中找到此模式的完整程式碼範例。 下圖可說明解決方案︰

![Azure Cosmos DB 使用者和權限訊息代理程式](media/mobile-apps-with-xamarin/documentdb-resource-token-broker.png)

如果您想讓兩位使用者存取同一份待辦事項清單，您可以在資源權杖訊息代理程式的存取權杖中新增額外的權限。

### <a name="scale-on-demand"></a>依需求進行調整
Azure Cosmos DB 是受控資料庫即服務。 隨著使用者人數成長，您不必為佈建 VM 或增加核心而煩惱。 您只需告訴 Azure Cosmos DB 您的應用程式每秒需要多少作業數 (輸送量)。 您可以透過入口網站的 [調整] 索引標籤，使用稱為每秒要求單位數 (RU) 的量值來指定輸送量。 例如，對 1-KB 文件進行的讀取作業需要 1 RU。 您也可以為 [輸送量] 計量新增警示來監視流量成長，並以設計程式方式在引發警示時變更輸送量。

![Azure Cosmos DB 依需求調整輸送量](media/mobile-apps-with-xamarin/cosmos-db-xamarin-scale.png)

### <a name="go-planet-scale"></a>進入全球級別
隨著您的應用程式日益受到歡迎，您可能會吸引全球各地的使用者。 或者，您或許想要針對無法預見的事件做準備。 移至 Azure 入口網站，然後開啟您的 Azure Cosmos DB 帳戶。 按一下地圖，讓您的資料持續複寫到世界各地任意數目的區域。 這項功能可讓您的使用者隨時隨地使用您的資料。 您也可以新增容錯移轉原則來因應偶發狀況。

![跨地理區域的 Azure Cosmos DB 調整](media/mobile-apps-with-xamarin/cosmos-db-xamarin-replicate.png)

恭喜！ 您已完成解決方案，並使用 Xamarin 和 Azure Cosmos DB 建置了行動應用程式。 請遵循類似的步驟，使用 Azure Cosmos DB JavaScript SDK 來建置 Cordova 應用程式，以及使用 Azure Cosmos DB REST API 來建置原生 iOS/Android 應用程式。

## <a name="next-steps"></a>後續步驟
* 檢視 [GitHub 上的 Xamarin 和 Azure Cosmos DB](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin) 原始程式碼。
* 下載 [Azure Cosmos DB .NET Core SDK](sql-api-sdk-dotnet-core.md)。
* 尋找更多 [.NET 應用程式](sql-api-dotnet-samples.md)的程式碼範例。
* 了解 [Azure Cosmos DB 豐富查詢功能](sql-api-sql-query.md)。
* 了解 [Azure Cosmos DB 中的地理空間支援](geospatial.md)。



