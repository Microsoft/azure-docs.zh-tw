---
title: Azure Cosmos DB Cassandra API 支援的 Apache Cassandra 功能與命令
description: 了解 Azure Cosmos DB Cassandra API 中的 Apache Cassandra 功能
services: cosmos-db
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.topic: overview
ms.date: 09/24/2018
ms.openlocfilehash: e3de78bdf38a326498b984dc2a9f8eaa42233d22
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091278"
---
# <a name="apache-cassandra-features-supported-by-azure-cosmos-db-cassandra-api"></a>Azure Cosmos DB Cassandra API 支援的 Apache Cassandra 功能 

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以透過 Cassandra 查詢語言 (CQL) v4 [有線通訊協定](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec)合規性開放原始碼 Cassandra 用戶端[驅動程式](http://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver)與 Azure Cosmos DB Cassandra API 通訊。 

透過使用 Azure Cosmos DB Cassandra API，您可以享受 Apache Cassandra API 的好處，以及 Azure Cosmos DB 所提供的企業功能。 企業功能包括[全域散發](distribute-data-globally.md)、[自動相應放大分割處理](partition-data.md)、可用性與延遲保證、靜態資料加密、備份等。

## <a name="cassandra-protocol"></a>Cassandra 通訊協定 

Azure Cosmos DB Cassandra API 與 CQL 版本 **v4** 相容。 支援的 CQL 命令、工具、限制與例外，如下所示。 能解析這些通訊協定的任何用戶端驅動程式，都應該能夠使用連線到 Azure Cosmos DB Cassandra API。

## <a name="cassandra-driver"></a>Cassandra 驅動程式

Azure Cosmos DB Cassandra API 支援下列 Cassandra 驅動程式：

* [Java 3.5+](https://github.com/datastax/java-driver)  
* [C# 3.5+](https://github.com/datastax/csharp-driver)  
* [Nodejs 3.5+](https://github.com/datastax/nodejs-driver)  
* [Python 3.15+](https://github.com/datastax/python-driver)  
* [C++ 2.9](https://github.com/datastax/cpp-driver)   
* [PHP 1.3](https://github.com/datastax/php-driver)  
* [Gocql](https://github.com/gocql/gocql)  
 
## <a name="cql-data-types"></a>CQL 資料類型 

Azure Cosmos DB Cassandra API 支援下列 CQL 資料類型：

* ascii  
* bigint  
* blob  
* 布林值  
* counter  
* 日期  
* decimal  
* double  
* float  
* 冰雪奇緣  
* inet  
* int  
* list  
* set  
* smallint  
* text  
* 分析  
* timestamp  
* timeuuid  
* tinyint  
* Tuple  
* uuid  
* varchar  
* varint  
* Tuple  
* udts  
* map  

## <a name="cql-functions"></a>CQL 功能

Azure Cosmos DB Cassandra API 支援下列 CQL 函式：

* token  
* Blob 轉換函式 
  * typeAsBlob(value)  
  * blobAsType(value)
* UUID 與 timeuuid 功能 
  * dateOf()  
  * now()  
  * minTimeuuid()  
  * unixTimestampOf()  
  * toDate(timeuuid)  
  * toTimestamp(timeuuid)  
  * toUnixTimestamp(timeuuid)  
  * toDate(timestamp)  
  * toUnixTimestamp(timestamp)  
  * toTimestamp(date)  
  * toUnixTimestamp(date)  


## <a name="cassandra-query-language-limits"></a>Cassandra 查詢語言限制

Azure Cosmos DB Cassandra API 在資料表中存放的資料大小沒限制。 可以儲存數百 TB 或 PB 的資料，同時確保遵守資料分割金鑰。 同樣地，每個實體或資料列對等項在資料行數目上沒有任何限制，不過，實體的總大小不應該超過 2 MB。

## <a name="tools"></a>工具 

Azure Cosmos DB Cassandra API 是受控服務平台。 它不會造成任何管理額外負荷，也不會使用記憶體回收行程、Java 虛擬機器 (JVM) 與 nodetool 等公用程式來管理叢集。 它支援 cqlsh 之類的工具，cqlsh 利用 Binary CQLv4 相容性。 

* Azure 入口網站的資料總管、計量、記錄診斷、PowerShell 與 cli 是其他支援的帳戶管理機制。

## <a name="cql-shell"></a>CQL Shell  

CQLSH 命令列公用程式隨附 Apache Cassandra 3.1.1，而且只要啟用下列環境變數即可運作，不需要進行額外的設定：

在執行下列命令之前，請[在 cacerts 存放區新增 Baltimore 根憑證](https://docs.microsoft.com/java/azure/java-sdk-add-certificate-ca-store?view=azure-java-stable#to-add-a-root-certificate-to-the-cacerts-store)。 

**Windows：** 

```bash
set SSL_VERSION=TLSv1_2 
SSL_CERTIFICATE=<path to Baltimore root ca cert>
set CQLSH_PORT=10350 
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> –ssl 
```
**Unix/Linux/Mac：**

```bash
export SSL_VERSION=TLSv1_2 
export SSL_CERTFILE=<path to Baltimore root ca cert>
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> –ssl 
```

## <a name="cql-commands"></a>CQL 命令

Azure Cosmos DB 支援在 Cassandra API 帳戶上使用下列資料庫命令。

* CREATE KEYSPACE 
* CREATE TABLE 
* ALTER TABLE 
* USE 
* INSERT 
* SELECT 
* UPDATE 
* BATCH - 只支援未記錄的命令 
* 刪除

透過 CQLV4 相容 SDK 執行的所有 crud 作業將會傳回有關錯誤、已耗用的要求單位、活動識別碼等額外資訊。 刪除及更新命令必須以考慮中的資源管控機制來處理，以避免過度使用佈建的資源。 
* 請注意，所指定的 gc_grace_seconds 值必須是零。

```csharp
var tableInsertStatement = table.Insert(sampleEntity); 
var insertResult = await tableInsertStatement.ExecuteAsync(); 
 
foreach (string key in insertResult.Info.IncomingPayload) 
        { 
            byte[] valueInBytes = customPayload[key]; 
            string value = Encoding.UTF8.GetString(valueInBytes); 
            Console.WriteLine($“CustomPayload:  {key}: {value}”); 
        } 
```

## <a name="consistency-mapping"></a>一致性對應 

Azure Cosmos DB Cassandra API 提供讀取作業的一致性選擇。 所有寫入作業，總是會使用寫入效能 SLA 來寫入 (不論帳戶一致性為何)。

## <a name="permission-and-role-management"></a>權限與角色管理

Azure Cosmos DB 支援角色型存取控制 (RBAC) 與讀寫和唯讀密碼/金鑰，可透過 [Azure 入口網站](https://portal.azure.com來取得。 Azure Cosmos DB 尚不支援資料平面活動的使用者與角色。 

## <a name="planned-support"></a>計劃的支援 
* 搭配使用時間戳記與 TTL  
* 目前忽略了建立 keyspace 命令中的區域名稱 - 資料的散佈會實作在基礎 Cosmos DB 平台中，並且會透過帳戶的入口網站或 powershell 加以公開。 





## <a name="next-steps"></a>後續步驟

- 開始使用 Java 應用程式來[建立 Cassandra API 帳戶、資料庫及資料表](create-cassandra-api-account-java.md)。

