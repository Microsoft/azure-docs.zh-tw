---
title: Azure Cosmos DB 教學課程︰在Apache TinkerPops Gremlin 主控台中建立、查詢和周遊 | Microsoft Docs
description: Azure Cosmos DB 快速入門，說明如何使用 Azure Cosmos DB Gremlin API 建立頂點、邊緣和查詢。
services: cosmos-db
author: luisbosquez
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-graph
ms.devlang: na
ms.topic: quickstart
ms.date: 01/08/2018
ms.author: lbosq
ms.openlocfilehash: e73b0e88a98c1b06216378078626b4338c598816
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47422961"
---
# <a name="azure-cosmos-db-create-query-and-traverse-a-graph-in-the-gremlin-console"></a>Azure Cosmos DB︰在 Gremlin 主控台中建立、查詢和周遊圖形

> [!div class="op_single_selector"]
> * [Gremlin 主控台](create-graph-gremlin-console.md)
> * [.NET](create-graph-dotnet.md)
> * [Java](create-graph-java.md)
> * [Node.js](create-graph-nodejs.md)
> * [Python](create-graph-python.md)
> * [PHP](create-graph-php.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。 

本快速入門示範如何使用 Azure 入口網站建立 Azure Cosmos DB [Gremlin API](graph-introduction.md) 帳戶、資料庫和圖表 (容器)，然後從 [Apache TinkerPop](http://tinkerpop.apache.org) (英文) 使用 [Gremlin 主控台](https://tinkerpop.apache.org/docs/current/reference/#gremlin-console) (英文) 來處理Gremlin API 資料。 在本教學課程中，您將建立和查詢頂點和邊緣、更新頂點屬性、查詢頂點、周遊該圖形，以及刪除頂點。

![Apache Gremlin 主控台中的 Azure Cosmos DB](./media/create-graph-gremlin-console/gremlin-console.png)

Gremlin 主控台是以 Groovy/Java 為基礎並且在 Linux、Mac 和 Windows 上執行。 您可以從 [Apache TinkerPop 網站](http://tinkerpop.apache.org/downloads.html)進行下載。

## <a name="prerequisites"></a>必要條件

您必須擁有 Azure 訂用帳戶，才能針對本快速入門建立 Azure Cosmos DB 帳戶。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

您也需要安裝 [Gremlin 主控台](http://tinkerpop.apache.org/)。 使用 3.2.5 版或更高版本。

## <a name="create-a-database-account"></a>建立資料庫帳戶

[!INCLUDE [cosmos-db-create-dbaccount-graph](../../includes/cosmos-db-create-dbaccount-graph.md)]

## <a name="add-a-graph"></a>新增圖形

[!INCLUDE [cosmos-db-create-graph](../../includes/cosmos-db-create-graph.md)]

## <a id="ConnectAppService"></a>連線到您的應用程式服務
1. 啟動 Gremlin 主控台之前，請建立或修改 `apache-tinkerpop-gremlin-console-3.2.5/conf` 目錄中的 remote-secure.yaml 組態檔。
2. 填入如下表中定義的 host、port、username、password、connectionPool 和 serializer 組態︰

    設定|建議的值|說明
    ---|---|---
    主機|針對 2017 年 12 月 20 日之前建立的帳戶，[account-name.gremlin.cosmosdb.azure.com] 或 [account-name.graphs.azure.com]|請參閱下列螢幕擷取畫面。 這是 Azure 入口網站的 [概觀] 頁面上的 Gremlin URI 值，其以方括號括住並已移除尾端的 :443/。
    連接埠|443|設為 443。
    username|您的使用者名稱|`/dbs/<db>/colls/<coll>` 表單的資源，其中 `<db>` 是您的資料庫名稱，而 `<coll>` 是您的集合名稱。
    password|您的主要金鑰| 請看下方的第二個螢幕擷取畫面。 這是您的主要金鑰，可以從 Azure 入口網站 [金鑰] 頁面的 [主鑰金鑰] 方塊中擷取。 使用方塊左側的 [複製] 按鈕來複製此值。
    connectionPool|{enableSsl: true}|SSL 的連線集區設定。
    序列化程式|{ className: org.apache.tinkerpop.gremlin.<br>driver.ser.GraphSONMessageSerializerV1d0,<br> config: { serializeResultToString: true }}|設定此值，並在貼入此值時刪除任何 `\n` 分行符號。

    對於主機值，從 [概觀] 頁面複製 [Gremlin URI] 值：![在 Azure 入口網站的 [概觀] 頁面上檢視和複製 Gremlin URI 值](./media/create-graph-gremlin-console/gremlin-uri.png)

    對於密碼值，從 [金鑰] 頁面複製 [主要金鑰]：![在 Azure 入口網站的 [金鑰] 頁面上檢視和複製主要金鑰](./media/create-graph-gremlin-console/keys.png)

remote-secure.yaml 檔案看起來應該像這樣：

```
hosts: [your_database_server.gremlin.cosmosdb.azure.com]
port: 443
username: /dbs/your_database_account/colls/your_collection
password: your_primary_key
connectionPool: {
  enableSsl: true
}
serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0, config: { serializeResultToString: true }}
```

3. 在您的終端機執行 `bin/gremlin.bat` 或 `bin/gremlin.sh`，以啟動 [Gremlin 主控台](http://tinkerpop.apache.org/docs/3.2.5/tutorials/getting-started/)。
4. 在您的終端機執行 `:remote connect tinkerpop.server conf/remote-secure.yaml`，以連線到您的應用程式服務。

    > [!TIP]
    > 如果您收到 `No appenders could be found for logger` 錯誤，確定您如步驟 2 所述更新了 remote-secure.yaml 檔案中的序列化程式值。 

5. 接下來，執行 `:remote console` 以將所有主控台命令重新導向至遠端伺服器。

   > [!NOTE]
   > 如果您不執行 `:remote console` 命令，但想要將所有主控台命令重新導向至遠端伺服器，則應在命令前面加上 `:>` 前置詞，例如，您應以 `:> g.V().count()` 的形式執行此命令。 此前置詞是命令的一部分，在搭配使用 Gremlin 主控台與 Azure Cosmos DB 時務必要使用。 省略此前置詞會指示主控台在本機執行命令，通常是針對記憶體中的圖形。 使用此前置詞 `:>` 會指示主控台要執行遠端命令，在此案例中是針對 Azure Cosmos DB (localhost 模擬器或 Azure 執行個體)。

太棒了！ 現在已完成安裝程式，讓我們開始執行一些主控台命令。

我們來試試簡單的 count() 命令。 在提示字元中，將下列內容輸入到主控台：

```
g.V().count()
```

## <a name="create-vertices-and-edges"></a>建立頂點和邊緣

首先我們會新增五個人員頂點 Thomas、Mary Kay、Robin、Ben 和 Jack。

輸入 (Thomas)：

```
g.addV('person').property('firstName', 'Thomas').property('lastName', 'Andersen').property('age', 44).property('userid', 1)
```

輸出：

```
==>[id:796cdccc-2acd-4e58-a324-91d6f6f5ed6d,label:person,type:vertex,properties:[firstName:[[id:f02a749f-b67c-4016-850e-910242d68953,value:Thomas]],lastName:[[id:f5fa3126-8818-4fda-88b0-9bb55145ce5c,value:Andersen]],age:[[id:f6390f9c-e563-433e-acbf-25627628016e,value:44]],userid:[[id:796cdccc-2acd-4e58-a324-91d6f6f5ed6d|userid,value:1]]]]
```
輸入 (Mary Kay)：

```
g.addV('person').property('firstName', 'Mary Kay').property('lastName', 'Andersen').property('age', 39).property('userid', 2)

```

輸出：

```
==>[id:0ac9be25-a476-4a30-8da8-e79f0119ea5e,label:person,type:vertex,properties:[firstName:[[id:ea0604f8-14ee-4513-a48a-1734a1f28dc0,value:Mary Kay]],lastName:[[id:86d3bba5-fd60-4856-9396-c195ef7d7f4b,value:Andersen]],age:[[id:bc81b78d-30c4-4e03-8f40-50f72eb5f6da,value:39]],userid:[[id:0ac9be25-a476-4a30-8da8-e79f0119ea5e|userid,value:2]]]]

```

輸入 (Robin)：

```
g.addV('person').property('firstName', 'Robin').property('lastName', 'Wakefield').property('userid', 3)
```

輸出：

```
==>[id:8dc14d6a-8683-4a54-8d74-7eef1fb43a3e,label:person,type:vertex,properties:[firstName:[[id:ec65f078-7a43-4cbe-bc06-e50f2640dc4e,value:Robin]],lastName:[[id:a3937d07-0e88-45d3-a442-26fcdfb042ce,value:Wakefield]],userid:[[id:8dc14d6a-8683-4a54-8d74-7eef1fb43a3e|userid,value:3]]]]
```

輸入 (Ben)：

```
g.addV('person').property('firstName', 'Ben').property('lastName', 'Miller').property('userid', 4)

```

輸出：

```
==>[id:ee86b670-4d24-4966-9a39-30529284b66f,label:person,type:vertex,properties:[firstName:[[id:a632469b-30fc-4157-840c-b80260871e9a,value:Ben]],lastName:[[id:4a08d307-0719-47c6-84ae-1b0b06630928,value:Miller]],userid:[[id:ee86b670-4d24-4966-9a39-30529284b66f|userid,value:4]]]]
```

輸入 (Jack)：

```
g.addV('person').property('firstName', 'Jack').property('lastName', 'Connor').property('userid', 5)
```

輸出：

```
==>[id:4c835f2a-ea5b-43bb-9b6b-215488ad8469,label:person,type:vertex,properties:[firstName:[[id:4250824e-4b72-417f-af98-8034aa15559f,value:Jack]],lastName:[[id:44c1d5e1-a831-480a-bf94-5167d133549e,value:Connor]],userid:[[id:4c835f2a-ea5b-43bb-9b6b-215488ad8469|userid,value:5]]]]
```


接下來，我們會新增人員之間關聯性的邊緣。

輸入 (Thomas -> Mary Kay)：

```
g.V().hasLabel('person').has('firstName', 'Thomas').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Mary Kay'))
```

輸出：

```
==>[id:c12bf9fb-96a1-4cb7-a3f8-431e196e702f,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:0d1fa428-780c-49a5-bd3a-a68d96391d5c,outV:1ce821c6-aa3d-4170-a0b7-d14d2a4d18c3]
```

輸入 (Thomas -> Robin)：

```
g.V().hasLabel('person').has('firstName', 'Thomas').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Robin'))
```

輸出：

```
==>[id:58319bdd-1d3e-4f17-a106-0ddf18719d15,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:3e324073-ccfc-4ae1-8675-d450858ca116,outV:1ce821c6-aa3d-4170-a0b7-d14d2a4d18c3]
```

輸入 (Robin -> Ben)：

```
g.V().hasLabel('person').has('firstName', 'Robin').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Ben'))
```

輸出：

```
==>[id:889c4d3c-549e-4d35-bc21-a3d1bfa11e00,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:40fd641d-546e-412a-abcc-58fe53891aab,outV:3e324073-ccfc-4ae1-8675-d450858ca116]
```

## <a name="update-a-vertex"></a>更新頂點

我們會以新的年齡 45 更新 Thomas 頂點。

輸入：
```
g.V().hasLabel('person').has('firstName', 'Thomas').property('age', 45)
```
輸出：

```
==>[id:ae36f938-210e-445a-92df-519f2b64c8ec,label:person,type:vertex,properties:[firstName:[[id:872090b6-6a77-456a-9a55-a59141d4ebc2,value:Thomas]],lastName:[[id:7ee7a39a-a414-4127-89b4-870bc4ef99f3,value:Andersen]],age:[[id:a2a75d5a-ae70-4095-806d-a35abcbfe71d,value:45]]]]
```

## <a name="query-your-graph"></a>查詢圖形

現在，我們會對您的圖形執行各種查詢。

首先，我們會使用篩選條件嘗試查詢，只傳回超過 40 歲的人員。

輸入 (篩選查詢)︰

```
g.V().hasLabel('person').has('age', gt(40))
```

輸出：

```
==>[id:ae36f938-210e-445a-92df-519f2b64c8ec,label:person,type:vertex,properties:[firstName:[[id:872090b6-6a77-456a-9a55-a59141d4ebc2,value:Thomas]],lastName:[[id:7ee7a39a-a414-4127-89b4-870bc4ef99f3,value:Andersen]],age:[[id:a2a75d5a-ae70-4095-806d-a35abcbfe71d,value:45]]]]
```

接下來，我們會預測超過 40 歲的第一個人員名稱。

輸入 (篩選 + 預測查詢)：

```
g.V().hasLabel('person').has('age', gt(40)).values('firstName')
```

輸出：

```
==>Thomas
```

## <a name="traverse-your-graph"></a>查詢圖形

我們會周遊圖形，以傳回 Thomas 的所有朋友。

輸入 (Thomas 的朋友)：

```
g.V().hasLabel('person').has('firstName', 'Thomas').outE('knows').inV().hasLabel('person')
```

輸出： 

```
==>[id:f04bc00b-cb56-46c4-a3bb-a5870c42f7ff,label:person,type:vertex,properties:[firstName:[[id:14feedec-b070-444e-b544-62be15c7167c,value:Mary Kay]],lastName:[[id:107ab421-7208-45d4-b969-bbc54481992a,value:Andersen]],age:[[id:4b08d6e4-58f5-45df-8e69-6b790b692e0a,value:39]]]]
==>[id:91605c63-4988-4b60-9a30-5144719ae326,label:person,type:vertex,properties:[firstName:[[id:f760e0e6-652a-481a-92b0-1767d9bf372e,value:Robin]],lastName:[[id:352a4caa-bad6-47e3-a7dc-90ff342cf870,value:Wakefield]]]]
```

接下來，我們會取得下一層的頂點。 周遊圖形，以傳回 Thomas 朋友的所有朋友。

輸入 (Thomas 朋友的朋友)：

```
g.V().hasLabel('person').has('firstName', 'Thomas').outE('knows').inV().hasLabel('person').outE('knows').inV().hasLabel('person')
```
輸出：

```
==>[id:a801a0cb-ee85-44ee-a502-271685ef212e,label:person,type:vertex,properties:[firstName:[[id:b9489902-d29a-4673-8c09-c2b3fe7f8b94,value:Ben]],lastName:[[id:e084f933-9a4b-4dbc-8273-f0171265cf1d,value:Miller]]]]
```

## <a name="drop-a-vertex"></a>刪除頂點

我們現在會從圖形資料庫中刪除頂點。

輸入 (置放 Jack 頂點)：

```
g.V().hasLabel('person').has('firstName', 'Jack').drop()
```

## <a name="clear-your-graph"></a>清除圖形

最後，我們會清除所有頂點和邊緣的資料庫。

輸入：

```
g.E().drop()
g.V().drop()
```

恭喜！ 您已經完成此 Azure Cosmos DB：Gremlin API 教學課程！

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶、如何使用 [資料總管] 來建立圖形、如何建立頂點和邊緣，以及如何使用 Gremlin 主控台來周遊圖形。 您現在可以使用 Gremlin 來建置更複雜的查詢和實作強大的圖形周遊邏輯。 

> [!div class="nextstepaction"]
> [使用 Gremlin 進行查詢](tutorial-query-graph.md)
