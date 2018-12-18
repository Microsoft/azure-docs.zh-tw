---
title: Azure Cosmos DB 中的唯一索引鍵 | Microsoft Docs
description: 了解如何在 Azure Cosmos DB 資料庫中使用唯一索引鍵。
services: cosmos-db
keywords: 唯一索引鍵條件約束, 違反唯一索引鍵條件約束
author: rafats
manager: kfile
editor: monicar
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 08/08/2018
ms.author: rafats
ms.openlocfilehash: 5811cb1e08ed5d02038da2a4460ae4b63580833b
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696134"
---
# <a name="unique-keys-in-azure-cosmos-db"></a>Azure Cosmos DB 中的唯一索引鍵

唯一索引鍵提供開發人員對其資料庫新增一層資料完整性的能力。 您可在建立容器時建立唯一索引鍵原則，以確保每個[資料分割索引鍵](partition-data.md)一或多個值的唯一性。 一旦使用唯一索引鍵原則建立容器，即可防止使用與唯一索引鍵條件約束所指定值重複的值，建立任何新的或已更新的項目。   

> [!NOTE]
> 唯一索引鍵是由最新版的 [.NET](sql-api-sdk-dotnet.md) 和 [.NET Core](sql-api-sdk-dotnet-core.md) SQL SDK 和 [MongoDB API](mongodb-feature-support.md#unique-indexes) 支援。 資料表 API 和 Gremlin API 此時不支援唯一索引鍵。 
> 
>

## <a name="use-case"></a>使用案例

例如，讓我們看看與[社交應用程式](use-cases.md#web-and-mobile-applications)相關聯的使用者資料庫如何受益於電子郵件地址採用唯一索引鍵原則。 讓使用者的電子郵件地址成為唯一索引鍵，可確認每筆記錄有唯一的電子郵件地址，而且無法使用重複的電子郵件地址建立新的記錄。 

如果您希望使用者能夠使用相同的電子郵件地址建立多筆記錄，但名字、姓氏和電子郵件地址不同，您可以將其他路徑新增至唯一索引鍵原則。 因此，您可以建立結合名字、姓氏和電子郵件的唯一索引鍵，而不是建立以一個電子郵件地址為基礎的唯一索引鍵。 在此情況下，允許三個路徑的每個唯一組合，所以資料庫可包含具有下列路徑值的項目。 每筆記錄都會通過唯一索引鍵原則。  

**firstName、lastName 和 email 之唯一索引鍵的允許值**

|名字|姓氏|電子郵件地址|
|---|---|---|
|Gaby|Duperre|gaby@contoso.com |
|Gaby|Duperre|gaby@fabrikam.com|
|Ivan|Duperre|gaby@fabrikam.com|
|    |Duperre|gaby@fabrikam.com|
|    |       |gaby@fabraikam.com|

如果您嘗試使用上表所列的任何組合來插入另一筆記錄，您會收到錯誤，指出不符合唯一索引鍵條件約束。 Azure Cosmos DB 傳回的錯誤為「已存在具有指定識別碼或名稱的資源。」 或「已存在具有指定識別碼、名稱或唯一索引的資源。」 

## <a name="using-unique-keys"></a>使用唯一索引鍵

必須在建立容器時定義唯一索引鍵，且唯一索引鍵的範圍為分割區索引鍵。 若要以先前的範例為基礎，如果您根據郵遞區號進行分割，您可能會使資料表中的記錄複寫在每個分割區中。

您無法將現有的容器更新為使用唯一索引鍵。

一旦使用唯一索引鍵原則建立容器，便無法變更此原則，除非您重新建立容器。 如果您有想要實作唯一索引鍵的現有資料，請建立新的容器，然後使用適當的資料移轉工具將資料移至新的容器。 對於 SQL 容器，使用[資料移轉工具](import-data.md)。 對於 MongoDB 容器，使用 [mongoimport.exe 或 mongorestore.exe](mongodb-migrate.md)。

每個唯一索引鍵最多可以包含 16 個路徑值 (例如 /firstName、/lastName、/address/zipCode 等)。 

每個唯一索引鍵原則最多可以有 10 個唯一索引鍵條件約束或組合，而且所有唯一索引屬性的結合路徑不應該超過 60 個字元。 因此先前使用名字、姓氏和電子郵件地址的範例只是一個條件約束，而且使用 16 個可能路徑中的三個。 

當容器上有唯一索引鍵原則時，建立、更新及刪除項目的要求單位費用稍微高一些。 

不支援疏鬆的唯一索引鍵。 如果有些唯一路徑的值遺漏，這些值會被視為特殊 null 值並納入唯一性條件約束。

## <a name="sql-api-sample"></a>SQL API 範例

下列程式碼範例示範如何使用兩個唯一索引鍵條件約束來建立新的 SQL 容器。 第一個條件約束是先前範例中所述的 firstName、lastName、email 條件約束。 第二個條件約束是使用者地址/郵遞區號。 使用此唯一索引鍵原則中路徑的範例 JSON 檔案會遵循此程式碼範例。 

```csharp
// Create a collection with two separate UniqueKeys, one compound key for /firstName, /lastName,
// and /email, and another for /address/zipCode.
private static async Task CreateCollectionIfNotExistsAsync(string dataBase, string collection)
{
    try
    {
        await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(dataBase, collection));
    }
    catch (DocumentClientException e)
    {
        if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
        {
            DocumentCollection myCollection = new DocumentCollection();
            myCollection.Id = collection;
            myCollection.PartitionKey.Paths.Add("/pk");
            myCollection.UniqueKeyPolicy = new UniqueKeyPolicy
            {
                UniqueKeys =
                new Collection<UniqueKey>
                {
                    new UniqueKey { Paths = new Collection<string> { "/firstName" , "/lastName" , "/email" }}
                    new UniqueKey { Paths = new Collection<string> { "/address/zipcode" } },
          }
            };
            await client.CreateDocumentCollectionAsync(
                UriFactory.CreateDatabaseUri(dataBase),
                myCollection,
                new RequestOptions { OfferThroughput = 2500 });
        }
        else
        {
            throw;
        }
    }
```

範例 JSON 文件。

```json
{
    "id": "1",
    "pk": "1234",
    "firstName": "Gaby",
    "lastName": "Duperre",
    "email": "gaby@contoso.com",
    "address": 
        {            
            "line1": "100 Some Street",
            "line2": "Unit 1",
            "city": "Seattle",
            "state": "WA",
            "zipcode": 98012
        }
    
}
```
> [!NOTE]
> 請注意，唯一索引鍵名稱有區分大小寫。 如上述範例所示，已設定了 /address/zipcode 的唯一名稱。 如果您的資料有 ZipCode，那麼它會在唯一索引鍵中插入「null」，因為 zipcode 不等於 ZipCode。 因為區分大小寫的緣故，將無法插入所有其他包含 ZipCode 的記錄，因為重複的「null」會違反唯一索引鍵限制式。

## <a name="mongodb-api-sample"></a>MongoDB API 範例

下列命令範例示範如何在 MongoDB API 之 users 集合的 firstName、lastName 和 email 欄位上建立唯一索引。 這可確保集合中全部文件的這三個欄位組合的唯一性。 對於 MongoDB 集合，唯一索引會在集合建立之後，但在填入集合之前建立。

> [!NOTE]
> MongoDB API 帳戶與 SQL API 帳戶的唯一索引鍵格式不同，您不必在欄位名稱前面指定反斜線 (/) 字元。 

```
db.users.createIndex( { firstName: 1, lastName: 1, email: 1 }, { unique: true } )
```
## <a name="configure-unique-keys-by-using-azure-portal"></a>使用 Azure 入口網站設定唯一索引鍵

在以上章節中，您可以找到程式碼範例，示範如何在使用 SQL API 或 MongoDB API 建立集合時，定義唯一索引鍵限制式。 不過在透過 Azure 入口網站的 web UI 建立集合時，也可以定義唯一索引鍵。 

- 在 Cosmos DB 帳戶中瀏覽至 [資料總管]
- 按一下 [新增集合] 
- 在 [唯一索引鍵] 區段中，**您可以按一下 [新增唯一索引鍵]，新增所需的唯一索引鍵限制式

![在資料總管中定義唯一索引鍵](./media/unique-keys/unique-keys-azure-portal.png)

- 如果您想要在 lastName 路徑中建立唯一索引鍵限制式，請新增 `/lastName`。
- 如果您想要為 lastName firstName 組合建立唯一索引鍵限制式，請新增 `/lastName,/firstName`

完成時，按一下 [確定] 以建立集合。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何為資料庫中的項目建立唯一索引鍵。 如果您第一次建立容器，請檢閱[在 Azure Cosmos DB 中分割資料](partition-data.md)，因為唯一索引鍵和分割區索引鍵會互相依賴。 


