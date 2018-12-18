---
title: Azure Cosmos DB 的非同步 Java 效能祕訣 | Microsoft Docs
description: 了解用以改善 Azure Cosmos DB 資料庫效能的用戶端設定選項
keywords: 如何改善資料庫效能
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: java
ms.topic: conceptual
ms.date: 03/27/2018
ms.author: sngun
ms.openlocfilehash: 48555dc8d1cc027cb771e0ba0678c6cb12d6785f
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697970"
---
# <a name="performance-tips-for-azure-cosmos-db-and-async-java"></a>Azure Cosmos DB 和非同步 Java 的效能祕訣

> [!div class="op_single_selector"]
> * [非同步 Java](performance-tips-async-java.md)
> * [Java](performance-tips-java.md)
> * [.NET](performance-tips.md)
> 

Azure Cosmos DB 是一個既快速又彈性的分散式資料庫，可在獲得延遲與輸送量保證的情況下順暢地調整。 使用 Azure Cosmos DB 時，您不須進行主要的架構變更，或是撰寫複雜的程式碼來調整您的資料庫。 相應增加和減少就像進行單一 API 呼叫或 SDK 方法呼叫一樣簡單。 不過，由於 Azure Cosmos DB 是透過網路呼叫存取，所以您可以在使用 [SQL 非同步 Java SDK](sql-api-sdk-async-java.md) 時，進行用戶端最佳化以達到最高效能。

如果您詢問「如何改善我的資料庫效能？ 」，請考慮下列選項：

## <a name="networking"></a>網路功能
   <a id="same-region"></a>
1. **為了效能在相同 Azure 區域中共置用戶端**

    可能的話，請將任何呼叫 Azure Cosmos DB 的應用程式放在與 Azure Cosmos DB 資料庫相同的區域中。 以約略的比較來說，在相同區域內對 Azure Cosmos DB 進行的呼叫會在 1-2 毫秒內完成，但美國西岸和美國東岸之間的延遲則會大於 50 毫秒。 視要求所採用的路由而定，各項要求從用戶端傳遞至 Azure 資料中心界限時的這類延遲可能有所不同。 確保呼叫端應用程式與佈建的 Azure Cosmos DB 端點位於相同的 Azure 區域中，將可能達到最低的延遲。 如需可用區域的清單，請參閱 [Azure 區域](https://azure.microsoft.com/regions/#services)。

    ![Azure Cosmos DB 連接原則的圖例](./media/performance-tips/same-region.png)
   
## <a name="sdk-usage"></a>SDK 的使用方式
1. **安裝最新的 SDK**

    Azure Cosmos DB SDK 會持續改善以提供最佳效能。 請參閱 [Azure Cosmos DB SDK](sql-api-sdk-async-java.md) 頁面來判斷最新的 SDK 並檢閱改善項目。
2. **在應用程式存留期內使用單一 Azure Cosmos DB 用戶端**

    每個 AsyncDocumentClient 執行個體都是安全執行緒，並且會執行有效率的連線管理和位址快取。 若要藉由 AsyncDocumentClient 獲得有效率的連線管理和更佳的效能，建議在應用程式存留期內，對每個 AppDomain 使用單一 AsyncDocumentClient 執行個體。

   <a id="max-connection"></a>

3. **調整 ConnectionPolicy**

    使用非同步 Java SDK 時，Azure Cosmos DB 要求是透過 HTTPS/REST 發出，並受制於預設最大連線集區大小 (1000)。 這個預設值應該適合大部分的使用案例。 不過，如果您有包含多個分割區的大型集合，可以使用 setMaxPoolSize 將最大連線集區大小設為較大的數字 (例如 1500)。

4. **微調分割之集合的平行查詢**

    Azure Cosmos DB SQL 非同步 Java SDK 支援平行查詢，可讓您平行查詢分割的集合 (詳細資訊請參閱[使用 SDK](sql-api-partition-data.md#working-with-the-azure-cosmos-db-sdks) 和相關的[程式碼範例](https://github.com/Azure/azure-cosmosdb-java/tree/master/examples/src/test/java/com/microsoft/azure/cosmosdb/rx/examples))。 平行查詢的設計目的是要改善其連續對應項目的查詢延遲和輸送量。

    (a) ***微調 setMaxDegreeOfParallelism\:*** 平行查詢的運作方式是以平行方式查詢多個分割。 不過，對於查詢會以循序方式擷取來自個別分割集合的資料。 因此，使用 setMaxDegreeOfParallelism 設定分割數目會最有機會達到最高效能的查詢，但前提是其他所有系統條件皆維持不變。 如果您不知道分割數目，您可以使用 setMaxDegreeOfParallelism 設定為較高的數字，然後系統會選擇最小值 (分割數目、使用者提供的輸入) 作為平行處理原則的最大刻度。 

    請務必注意，若對於查詢是以平均方式將資料分佈於所有分割，平行查詢便會產生最佳效益。 如果分割之集合的分割方式是查詢所傳回的所有或大多數資料集中在少數幾個分割中 (最差的情況是集中在一個分割)，則這些分割會成為查詢效能的瓶頸。

    (b) ***微調 setMaxBufferedItemCount\:*** 平行查詢是設計成可以在用戶端處理目前的結果批次時預先擷取結果。 預先擷取有助於改善查詢的整體延遲。 setMaxBufferedItemCount 可限制預先擷取的結果數目。 將 setMaxBufferedItemCount 設定為預期傳回的結果數目 (或更高的數目)，可讓查詢透過預先擷取獲得最大效益。

    預先擷取會以相同方式運作，不受 MaxDegreeOfParallelism 的影響，而且來自所有分割的資料會有單一緩衝區。  

5. **依 getRetryAfterInMilliseconds 間隔實作輪詢**

    在進行效能測試期間，您應該增加負載，直到系統對小部分要求進行節流處理為止。 如果進行節流處理，用戶端應用程式應該降速，且持續時間達伺服器指定的重試間隔。 採用降速可確保您在重試之間花費最少的等待時間。 如需詳細資訊，請參閱[超過保留的輸送量限制](request-units.md#RequestRateTooLarge)和 DocumentClientException.getRetryAfterInMilliseconds。
6. **相應放大用戶端工作負載**

    如果您是以高輸送量層級 (> 50,000 RU/秒) 進行測試，用戶端應用程式可能會成為瓶頸，因為電腦對 CPU 或網路的使用率將達到上限。 如果到了這一刻，您可以將用戶端應用程式向外延展至多部伺服器，以繼續將 Azure Cosmos DB 帳戶再往前推進一步。

7. **使用名稱定址**

    使用連結格式為 `dbs/MyDatabaseId/colls/MyCollectionId/docs/MyDocumentId` 的名稱定址來取代格式為 `dbs/<database_rid>/colls/<collection_rid>/docs/<document_rid>` 的 SelfLinks (\_self)，以避免擷取用來建構連結之所有資源的 ResourceId。 此外，由於會重新建立這些資源 (可能使用相同名稱)，因此快取這些資源並沒有幫助。

   <a id="tune-page-size"></a>
8. **調整查詢/讀取摘要的頁面大小以獲得更好的效能**

    使用讀取摘要功能 (例如 readDocuments) 執行大量文件讀取時，或發出 SQL 查詢時，如果結果集太大，則會以分段方式傳回結果。 根據預設，會以 100 個項目或 1 MB 的區塊傳回結果 (以先達到的限制為準)。

    若要減少擷取所有適用結果所需的網路來回行程次數，您可以使用 [x-ms-max-item-count](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 要求標頭將頁面大小最高增加至 1000。 在您只需要顯示幾個結果的情況下 (例如，您的使用者介面或應用程式 API 一次只傳回 10 筆結果)，您也可以將頁面大小縮小為 10，以降低讀取和查詢所耗用的輸送量。

    您也可以使用 setMaxItemCount 方法來設定頁面大小。
    
9. **使用適當排程器 (避免竊取事件迴圈 IO Netty 執行緒)**

    非同步 Java SDK 會將 [netty](https://netty.io/) 用於非封鎖 IO。 SDK 會使用固定數目的 IO netty 事件迴圈執行緒 (和您電腦所擁有的 CPU 核心數一樣多) 來執行 IO 作業。 API 所傳回的 Observable 會在其中一個共用的 IO 事件迴圈 netty 執行緒上發出結果。 因此，請切勿封鎖共用的 IO 事件迴圈 netty 執行緒。 若執行 CPU 密集工作或封鎖 IO 事件迴圈 netty 執行緒上的作業，可能會導致鎖死或大幅降低 SDK 輸送量。

    例如，下列程式碼會在事件迴圈 IO netty 執行緒上執行 CPU 密集工作：

    ```java
    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribe(
      resourceResponse -> {
        //this is executed on eventloop IO netty thread.
        //the eventloop thread is shared and is meant to return back quickly.
        //
        // DON'T do this on eventloop IO netty thread.
        veryCpuIntensiveWork();
      });
    ```

    在收到結果之後，如果您需要對結果進行 CPU 密集工作，請避免在事件迴圈 IO netty 執行緒上進行。 您可以改為提供您自己的排程器，以提供自己的執行緒來執行工作。

    ```java
    import rx.schedulers;

    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribeOn(Schedulers.computation())
    subscribe(
      resourceResponse -> {
        // this is executed on threads provided by Scheduler.computation()
        // Schedulers.computation() should be used only when:
        //   1. The work is cpu intensive 
        //   2. You are not doing blocking IO, thread sleep, etc. in this thread against other resources.
        veryCpuIntensiveWork();
      });
    ```

    根據您的工作類型，您應該將適當的現有 RxJava 排程器用於您的工作。 閱讀這裡 [``Schedulers``](http://reactivex.io/RxJava/1.x/javadoc/rx/schedulers/Schedulers.html)。

    如需詳細資訊，請查看非同步 Java SDK 的 [Github 頁面](https://github.com/Azure/azure-cosmosdb-java)。

10. **停用 netty 的記錄** Netty 程式庫記錄通訊頻繁，而且必須加以關閉 (隱藏組態中的登入可能不夠) 以避免額外的 CPU 成本。 如果您不是在偵錯模式中，請停用 netty 全部的記錄。 因此，如果您要使用 log4j 來移除 netty 中的 ``org.apache.log4j.Category.callAppenders()`` 所產生的額外 CPU 成本，請將下列行新增至程式碼基底：

    ```java
    org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
    ```

11. **OS 開啟檔案資源限制** 某些 Linux 系統 (例如 Red Hat) 有開啟檔案數目的上限，以及連線總數的上限。 執行下列命令來檢視目前的限制：

    ```bash
    ulimit -a
    ```

    開啟檔案 (nofile) 的數目必須夠大，才能有足夠空間供您設定的連線集區大小和 OS 的其他開啟檔案使用。 您可以進行修改，以允許較大的連線集區大小。

    開啟 limits.conf 檔案：

    ```bash
    vim /etc/security/limits.conf
    ```
    
    新增/修改下列幾行：

    ```
    * - nofile 100000
    ```

12. **將原生 SSL 實作用於 netty** Netty 可以直接將 OpenSSL 用於 SSL 實作堆疊，以達到更佳的效能。 如果沒有這個設定，netty 將會改為使用 Java 的預設 SSL 實作。

    在 Ubuntu 上：
    ```bash
    sudo apt-get install openssl
    sudo apt-get install libapr1
    ```

    並將下列相依性新增至專案 maven 相依性：
    ```xml
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-tcnative</artifactId>
      <version>2.0.7.Final</version>
      <classifier>linux-x86_64</classifier>
    </dependency>
    ```

針對其他平台 (Red Hat、Windows、Mac 等等)，請參閱這些指示 https://netty.io/wiki/forked-tomcat-native.html

## <a name="indexing-policy"></a>索引原則
 
1. **從索引編製中排除未使用的路徑以加快寫入速度**

    Azure Cosmos DB 的索引編製原則可讓您利用檢索路徑 (setIncludedPaths 和 setExcludedPaths)，指定要在索引編製中包含或排除的文件路徑。 在事先知道查詢模式的案例中，使用檢索路徑可改善寫入效能並降低索引儲存空間，因為檢索成本與檢索的唯一路徑數目直接相互關聯。  例如，以下程式碼示範如何將文件的整個區段 (也稱為 樹狀子目錄) 自索引編製作業中排除 (透過使用 "*" 萬用字元)。

    ```Java
    Index numberIndex = Index.Range(DataType.Number);
    numberIndex.set("precision", -1);
    indexes.add(numberIndex);
    includedPath.setIndexes(indexes);
    includedPaths.add(includedPath);
    indexingPolicy.setIncludedPaths(includedPaths);
    collectionDefinition.setIndexingPolicy(indexingPolicy);
    ```

    如需詳細資訊，請參閱 [Azure Cosmos DB 索引編製原則](indexing-policies.md)。

## <a name="throughput"></a>Throughput
<a id="measure-rus"></a>

1. **測量和調整較低的要求單位/秒使用量**

    Azure Cosmos DB 提供許多的資料庫作業，包括使用 UDF、預存程序和觸發程序進行關聯式和階層式查詢，而這些作業全都是對資料庫集合內的文件來進行。 與上述各項作業相關聯的成本，會因為完成作業所需的 CPU、IO 和記憶體而不同。 您不需要考慮和管理硬體資源，您可以將要求單位 (RU) 想成是執行各種資料庫作業以及服務應用程式要求時所需的資源數量。

    輸送量是根據為每個容器所設定的[要求單位](request-units.md)數量來佈建。 要求單位消耗量是以每秒的速率來計算。 如果應用程式的速率超過為其容器佈建的要求單位速率，便會受到限制，直到該速率降到容器的佈建層級以下。 如果您的應用程式需要較高的輸送量，您可以藉由佈建其他的要求單位來增加輸送量。 

    查詢的複雜性會影響針對作業所耗用的要求單位數量。 述詞數目、述詞性質、UDF 數目，以及來源資料集的大小，全都會影響查詢作業的成本。

    若要測量任何作業 (建立、更新或刪除) 的額外負荷，請檢查 [x-ms-request-charge](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 標頭，來測量這些作業所耗用的要求單位數量。 您也可以查看 ResourceResponse<T> 或 FeedResponse<T> 中的對等 RequestCharge 屬性。

    ```Java
    ResourceResponse<Document> response = asyncClient.createDocument(collectionLink, documentDefinition, null,
                                                     false).toBlocking.single();
    response.getRequestCharge();
    ```             

    在此標頭中傳回的要求費用是佈建輸送量的一小部分。 例如，如果您佈建了 2000 RU/秒，且前述查詢傳回 1000 份 1 KB 文件，則作業成本會是 1000。 因此在一秒內，伺服器在對後續要求進行速率限制前，只會接受兩個這類要求。 如需詳細資訊，請參閱[要求單位](request-units.md)和[要求單位計算機](https://www.documentdb.com/capacityplanner)。
<a id="429"></a>
2. **處理速率限制/要求速率太大**

    當用戶端嘗試超過帳戶保留的輸送量時，伺服器的效能不會降低，而且不會使用超過保留層級的輸送量容量。 伺服器將預先使用 RequestRateTooLarge (HTTP 狀態碼 429) 來結束要求，並傳回 [x-ms-retry-after-ms](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 標頭，以指出使用者重試要求之前必須等候的時間量 (毫秒)。

        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK 全都隱含地攔截這個回應，採用伺服器指定的 retry-after 標頭，並重試此要求。 除非有多個用戶端同時存取您的帳戶，否則下次重試將會成功。

    如果您有多個用戶端都以高於要求速率的方式累積運作，則用戶端在內部設定為 9 的預設重試計數可能會不敷使用；在此情況下，用戶端會擲回 DocumentClientException (狀態碼 429) 到應用程式。 在 ConnectionPolicy 執行個體上使用 setRetryOptions，即可變更預設重試計數。 根據預設，如果要求繼續以高於要求速率的方式運作，則會在 30 秒的累計等候時間後傳回 DocumentClientException (狀態碼 429)。 即使目前的重試計數小於最大重試計數 (預設值 9 或使用者定義的值)，也會發生這種情況。

    雖然自動重試行為有助於改善大部分應用程式的恢復功能和可用性，但是在進行效能基準測試時可能會有所歧異 (尤其是在測量延遲時)。  如果實驗達到伺服器節流並導致用戶端 SDK 以無訊息模式重試，則用戶端觀察到的延遲將會突然增加。 若要避免效能實驗期間的延遲尖峰，測量每個作業所傳回的費用，並確保要求是以低於保留要求速率的方式運作。 如需詳細資訊，請參閱 [要求單位](request-units.md)。
3. **輸送量較高之少量文件的設計**

    指定之作業的要求費用 (要求處理成本) 與文件大小直接相互關聯。 大型文件的作業成本高於小型文件的作業成本。

## <a name="next-steps"></a>後續步驟
若要深入了解如何針對規模和高效能設計您的應用程式，請參閱 [Azure Cosmos DB 的資料分割與調整規模](partition-data.md)。
