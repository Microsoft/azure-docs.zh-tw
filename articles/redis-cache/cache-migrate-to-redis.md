---
title: 將受控快取服務應用程式移轉至 Redis - Azure | Microsoft Docs
description: 了解如何將受控快取服務和 In-Role Cache 應用程式移轉至 Azure Redis Cache
services: redis-cache
documentationcenter: na
author: wesmc7777
manager: cfowler
editor: tysonn
ms.assetid: 041f077b-8c8e-4d7c-a3fc-89d334ed70d6
ms.service: cache
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: cache-redis
ms.workload: tbd
ms.date: 05/30/2017
ms.author: wesmc
ms.openlocfilehash: f499925ecea8ca127c90691f7d92e74e8df68cf9
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38697334"
---
# <a name="migrate-from-managed-cache-service-to-azure-redis-cache"></a>從受控快取服務移轉至 Azure Redis 快取
若想將使用 Azure 受控快取服務的應用程式移轉至 Azure Redis 快取，您幾乎不需要變更應用程式就可達成，詳細情形取決於快取應用程式所使用的受控快取服務功能。 API 雖非完全相同，但卻極為類似，而且您現有使用受控快取服務來存取快取的程式碼，大多只需要略做變更即可重複使用。 本文說明如何對設定和應用程式進行必要的變更，以將受控快取服務應用程式移轉為使用 Azure Redis 快取，並說明如何使用 Azure Redis 快取的某些功能，來實作受控快取服務快取的功能。

>[!NOTE]
>受控快取服務及 In-Role Cache 已於 2016 年 11 月 30 日起[淘汰](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)。 如果您有任何 In-Role Cache 部署想要移轉到 Azure Redis Cache，您可以依照這篇文章中的步驟作業。

## <a name="migration-steps"></a>移轉步驟
要將受控快取服務應用程式移轉為使用 Azure Redis 快取，需要執行下列步驟。

* 將受控快取服務功能對應至 Azure Redis 快取
* 選擇快取供應項目
* 建立快取
* 設定快取用戶端
  * 移除受控快取服務設定
  * 設定使用 StackExchange.Redis NuGet 封裝的快取用戶端
* 移轉受控快取服務程式碼
  * 使用 ConnectionMultiplexer 類別連接到快取
  * 存取快取中的基本資料型別
  * 使用快取中的 .NET 物件
* 將 ASP.NET 工作階段狀態和輸出快取移轉至 Azure Redis 快取 

## <a name="map-managed-cache-service-features-to-azure-redis-cache"></a>將受控快取服務功能對應至 Azure Redis 快取
Azure 受控快取服務與 Azure Redis 快取類似，但兩者在實作某些功能時會使用不同的方式。 本節將描述其中的一些差異，並提供有關在 Azure Redis 快取中實作受控快取服務功能的指引。

| 受控快取服務功能 | 受控快取服務支援 | Azure Redis 快取支援 |
| --- | --- | --- |
| 具名快取 |會設定預設快取，而在標準版和進階版快取供應項目中，還可以視需要額外設定多達 9 個具名快取。 |Azure Redis 快取具有可供用來對具名快取實作類似功能的可設定數目資料庫 (預設值為 16 個)。 如需詳細資訊，請參閱[什麼是 Redis 資料庫？](cache-faq.md#what-are-redis-databases)和[預設 Redis 伺服器組態](cache-configure.md#default-redis-server-configuration)。 |
| 高可用性 |在標準版和進階版快取供應項目中，會針對快取中的項目提供高可用性。 如果因為失敗而導致項目遺失，快取中的項目仍有備份複本可供使用。 次要快取的寫入作業是以同步方式進行。 |標準版和進階版快取供應項目有提供高可用性，其具有雙節點的主要/複本設定 (進階版快取的每個分區都有主要/複本配對)。 複本的寫入作業是以非同步方式進行。 如需詳細資訊，請參閱 [Azure Redis 快取定價](https://azure.microsoft.com/pricing/details/cache/)。 |
| 通知 |當具名快取上發生各種快取作業時，允許用戶端接收非同步通知。 |用戶端應用程式可以使用 Redis 發行/訂閱或 [Keyspace 通知](cache-configure.md#keyspace-notifications-advanced-settings) 來達成和通知類似的功能。 |
| 本機快取 |在用戶端本機上儲存快取物件的複本，以利快速存取。 |用戶端應用程式必須使用字典或類似的資料結構來實作這項功能。 |
| 收回原則 |無或 LRU。 預設原則是 LRU。 |Azure Redis 快取支援下列收回原則：volatile-lru、allkeys-lru、volatile-random、allkeys-random、volatile-ttl、noeviction。 預設原則是 volatile-lru。 如需詳細資訊，請參閱 [預設 Redis 伺服器組態](cache-configure.md#default-redis-server-configuration)。 |
| 到期原則 |預設的到期原則為「絕對」，預設的到期間隔為 10 分鐘。 另外也提供 [滑動] 和 [永不] 原則。 |依預設，快取中的項目不會到期，但可以使用快取集多載，對每筆寫入作業設定到期時間。 |
| 區域和標記 |區域是快取項目的子群組。 區域也支援以稱為標記的額外描述性字串，來為快取項目加上註解。 區域支援對該區域內的任何標記項目執行搜尋作業的能力。 區域內的所有項目都位於單一快取叢集節點內。 |Redis 快取是由單一節點所組成 (除非已啟用 Redis 叢集)，因此不適用受控快取服務區域的概念。 Redis 支援在擷取索引鍵時執行搜尋和萬用字元作業，讓描述性標記可以內嵌在索引鍵名稱內並於稍後用來擷取項目。 如需使用 Redis 實作標記解決方案的範例，請參閱 [使用 Redis 實作快取標記](http://stackify.com/implementing-cache-tagging-redis/)。 |
| 序列化 |受控快取支援 NetDataContractSerializer 和 BinaryFormatter，也支援使用自訂序列化程式。 預設值為 NetDataContractSerializer。 |由用戶端應用程式負責先將 .NET 物件序列化再將它們放入快取中，至於要選擇使用哪個序列化程式則由用戶端應用程式的開發人員決定。 如需詳細資訊和範例程式碼，請參閱 [在快取中使用 .NET 物件](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache)。 |
| 快取模擬器 |受控快取提供本機快取模擬器。 |Azure Redis 快取沒有模擬器，但您可以 [在本機執行 redis-server.exe 的 MSOpenTech 組建](cache-faq.md#cache-emulator) 提供模擬器體驗。 |

## <a name="choose-a-cache-offering"></a>選擇快取供應項目
Microsoft Azure Redis 快取有下列階層：

* **基本** - 單一節點。 多種大小，最高為 53 GB。
* **標準** – 兩個節點 (主要/從屬)。 多種大小，最高為 53 GB。 99.9% SLA。
* **進階** – 兩個節點的主要/從屬，最多具有 10 個分區。 提供多種大小，範圍從 6 GB 到 530 GB。 所有「標準」層級的功能以及更多功能，可支援 [Redis 叢集](cache-how-to-premium-clustering.md)、[Redis 持續性](cache-how-to-premium-persistence.md)和 [Azure 虛擬網路](cache-how-to-premium-vnet.md)。 99.9% SLA。

每一個階層都有不同的功能和價格。 本指南稍後將探討這些功能，如需定價的詳細資訊，請參閱 [快取定價詳細資料](https://azure.microsoft.com/pricing/details/cache/)。

移轉作業的第一步是挑選符合先前受控快取服務快取大小的容量，然後再依據應用程式的需求擴大或縮小規模。 如需選擇合適的 Azure Redis 快取供應項目的詳細資訊，請參閱[應該使用哪個 Redis 快取供應項目和大小](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)。

## <a name="create-a-cache"></a>建立快取
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="configure-the-cache-clients"></a>設定快取用戶端
一旦建立並設定快取，下一步是移除受控快取服務設定，並新增 Azure Redis 快取設定和參考，讓快取用戶端可以存取快取。

* 移除受控快取服務設定
* 設定使用 StackExchange.Redis NuGet 封裝的快取用戶端

### <a name="remove-the-managed-cache-service-configuration"></a>移除受控快取服務設定
要將用戶端應用程式設定為使用 Azure Redis 快取，必須先解除安裝受控快取服務 NuGet 封裝，以便移除現有受控快取服務的設定和組件參考。

若要解除安裝受控快取服務 NuGet 封裝，請在 [方案總管] 中的用戶端專案上按一下滑鼠右鍵，然後選擇 [管理 NuGet 封裝]。 選取 [已安裝的封裝] 節點，然後在 [搜尋已安裝的封裝] 方塊中輸入 **WindowsAzure.Caching**。 選取 [Windows Azure 快取 (Windows Azure Cache)] \(或 [Windows Azure 快取 (Windows Azure Caching)]，視 NuGet 封裝的版本而定)，按一下 [解除安裝]，然後按一下 [關閉]。

![解除安裝 Azure 受控快取服務 NuGet 套件](./media/cache-migrate-to-redis/IC757666.jpg)

解除安裝受控快取服務 NuGet 封裝，會移除用戶端應用程式的 app.config 或 web.config 中的受控快取服務組件和受控快取服務項目。 解除安裝 NuGet 套件時可能不會移除部分自訂的設定，因此請開啟 web.config 或 app.config，確定已移除下列項目。

確定已從 `configSections` 項目移除 `dataCacheClients` 項目。 請勿移除整個 `configSections` 項目，只需移除 `dataCacheClients` 項目 (若存在)。

```xml
<configSections>
  <!-- Existing sections omitted for clarity. -->
  <section name="dataCacheClients"type="Microsoft.ApplicationServer.Caching.DataCacheClientsSection, Microsoft.ApplicationServer.Caching.Core" allowLocation="true" allowDefinition="Everywhere"/>
</configSections>
```

確定已移除 `dataCacheClients` 區段。 `dataCacheClients` 區段會類似下列範例。

```xml
<dataCacheClients>
  <dataCacheClientname="default">
    <!--To use the in-role flavor of Azure Cache, set identifier to be the cache cluster role name -->
    <!--To use the Azure Managed Cache Service, set identifier to be the endpoint of the cache cluster -->
    <autoDiscoverisEnabled="true"identifier="[Cache role name or Service Endpoint]"/>

    <!--<localCache isEnabled="true" sync="TimeoutBased" objectCount="100000" ttlValue="300" />-->
    <!--Use this section to specify security settings for connecting to your cache. This section is not required if your cache is hosted on a role that is a part of your cloud service. -->
    <!--<securityProperties mode="Message" sslEnabled="true">
      <messageSecurity authorizationInfo="[Authentication Key]" />
    </securityProperties>-->
  </dataCacheClient>
</dataCacheClients>
```

一旦移除受控快取服務設定，您就可以如下一節所述設定快取用戶端。

### <a name="configure-a-cache-client-using-the-stackexchangeredis-nuget-package"></a>設定使用 StackExchange.Redis NuGet 封裝的快取用戶端
[!INCLUDE [redis-cache-configure](../../includes/redis-cache-configure-stackexchange-redis-nuget.md)]

## <a name="migrate-managed-cache-service-code"></a>移轉受控快取服務程式碼
StackExchange.Redis 快取用戶端的 API 類似受控快取服務。 本節將概述兩者的差異。

### <a name="connect-to-the-cache-using-the-connectionmultiplexer-class"></a>使用 ConnectionMultiplexer 類別連接到快取
在受控快取服務中，快取連線是由 `DataCacheFactory` 和 `DataCache` 類別負責處理。 在 Azure Redis 快取中，這些連接則是由 `ConnectionMultiplexer` 類別進行管理。

請在您要用來存取快取的檔案頂端加入下列 using 陳述式。

```csharp
using StackExchange.Redis
```

如果此命名空間並未解析，請確定您已如[快速入門：搭配使用 Azure Redis 快取與 .NET 應用程式](cache-dotnet-how-to-use-azure-redis-cache.md)所述新增 StackExchange.Redis NuGet 套件。

> [!NOTE]
> 請注意，StackExchange.Redis 用戶端需要 .NET Framework 4 或更高版本。
> 
> 

若要連接至 Azure Redis 快取執行個體，請呼叫靜態 `ConnectionMultiplexer.Connect` 方法並傳入端點和金鑰。 在您的應用程式中共用 `ConnectionMultiplexer` 執行個體的其中一種方法，就是擁有可傳回已連接執行個體的靜態屬性，類似下列範例。 此方法提供安全執行緒方式，初始化已連接的單一 `ConnectionMultiplexer` 執行個體。 在此範例中， `abortConnect` 已設為 false，這表示即使無法建立與快取的連接，呼叫也會成功。 `ConnectionMultiplexer` 的主要功能之一，就是一旦網路問題或其他原因獲得解決，它就會自動恢復與快取的連接。

```csharp
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    return ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");
});

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

快取端點、金鑰和連接埠可自您快取執行個體的 [Redis 快取]  刀鋒視窗取得。 如需詳細資訊，請參閱 [Redis 快取屬性](cache-configure.md#properties)。

一旦建立連接，即會透過呼叫 `ConnectionMultiplexer.GetDatabase` 方法傳回 Redis 快取資料庫的參考。 透過 `GetDatabase` 方法傳回的物件是輕量型傳遞物件，而且不需要儲存。

```csharp
IDatabase cache = Connection.GetDatabase();

// Perform cache operations using the cache object...
// Simple put of integral data types into the cache
cache.StringSet("key1", "value");
cache.StringSet("key2", 25);

// Simple get of data types from the cache
string key1 = cache.StringGet("key1");
int key2 = (int)cache.StringGet("key2");
```

StackExchange.Redis 用戶端會使用 `RedisKey` 和 `RedisValue` 型別來對快取存取和儲存項目。 這些型別會對應到最基本的語言型別 (包括字串)，但通常不會直接使用。 Redis 字串是最基本的一種 Redis 值，可包含許多型別的資料 (包括序列化的二進位資料流)，您可能不會直接使用此型別，但您會使用到名稱中包含 `String` 的方法。 對於最基本的資料型別，您會使用 `StringSet` 和 `StringGet` 方法對快取儲存和擷取項目，除非您是要在快取中儲存集合或其他 Redis 資料型別。 

`StringSet` 和 `StringGet` 類似受控快取服務的 `Put` 和 `Get` 方法，其中最主要的不同點在於，若要設定 .NET 物件並將其放到快取中，您必須先將其序列化。 

呼叫 `StringGet` 時，如果物件已存在，即會傳回，如果物件不存在，則會傳回 Null。 在此情況下，您可以從需要的資料來源中擷取值，並將它儲存在快取中供後續使用。 這種模式稱為另行快取模式。

若要指定快取中項目的到期時間，請使用 `StringSet` 的 `TimeSpan` 參數。

```csharp
cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));
```

Azure Redis 快取可以使用 .NET 物件及基本資料型別，但必須先將 .NET 物件序列化，才能加以快取。 此序列化是應用程式開發人員的責任，同時賦與開發人員選擇序列化程式的彈性。 如需詳細資訊和範例程式碼，請參閱 [在快取中使用 .NET 物件](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache)。

## <a name="migrate-aspnet-session-state-and-output-caching-to-azure-redis-cache"></a>將 ASP.NET 工作階段狀態和輸出快取移轉至 Azure Redis 快取
Azure Redis 快取有適用於 ASP.NET 工作階段狀態和頁面輸出快取的提供者。 若要移轉使用這些提供者之受控快取服務版本的應用程式，請先從您的 web.config 移除現有的區段，然後設定這些提供者的 Azure Redis 快取版本。 如需使用 Azure Redis 快取 ASP.NET 提供者的指示，請參閱 [Azure Redis 快取的 ASP.NET 工作階段狀態供應器](cache-aspnet-session-state-provider.md)和 [Azure Redis 快取的 ASP.NET 輸出快取提供者](cache-aspnet-output-cache-provider.md)。

## <a name="next-steps"></a>後續步驟
瀏覽 [Azure Redis 快取文件](https://azure.microsoft.com/documentation/services/cache/) 中的教學課程、範例、影片及其他資訊。

