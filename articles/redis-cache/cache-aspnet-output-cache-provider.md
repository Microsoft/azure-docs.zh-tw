---
title: 快取 ASP.NET 輸出快取提供者
description: 了解如何使用 Azure Redis 快取進行 ASP.NET 頁面輸出快取
services: redis-cache
documentationcenter: na
author: wesmc7777
manager: cfowler
editor: tysonn
ms.assetid: 78469a66-0829-484f-8660-b2598ec60fbf
ms.service: cache
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: cache-redis
ms.workload: tbd
ms.date: 02/14/2017
ms.author: wesmc
ms.openlocfilehash: a6c3314a981b46aa6f1cbca1f34392d1e1ae6c9a
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47431639"
---
# <a name="aspnet-output-cache-provider-for-azure-redis-cache"></a>Azure Redis 快取的 ASP.NET 輸出快取提供者
Redis 輸出快取提供者為輸出快取資料的程序外儲存體機制。 此資料特別適用於完整 HTTP 回應 (頁面輸出快取)。 提供者插入 ASP.NET 4 中導入的新輸出快取提供者擴充點。

若要使用 Redis 輸出快取提供者，請先設定您的快取，然後使用「Redis 輸出快取提供者 NuGet 封裝」設定 ASP.NET 應用程式。 本主題提供為使用 Redis 輸出快取提供者而進行應用程式設定的相關指引。 如需建立和設定 Azure Redis 快取執行個體的相關詳細資訊，請參閱 [建立快取](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache)。

## <a name="store-aspnet-page-output-in-the-cache"></a>將 ASP.NET 頁面輸出儲存在快取中
若要在 Visual Studio 中使用「Redis 快取工作階段狀態 NuGet 套件」來設定用戶端應用程式，請依序按一下 [工具] 功能表中的 [NuGet 套件管理員] 和 [套件管理器主控台]。

從 `Package Manager Console` 視窗執行下列命令。
    
```
Install-Package Microsoft.Web.RedisOutputCacheProvider
```

「Redis 輸出快取提供者 NuGet 封裝」對「StackExchange.Redis.StrongName 封裝」有相依性。 如果 StackExchange.Redis.StrongName 封裝不在專案中，代表已經安裝。 如需「Redis 輸出快取提供者 NuGet 套件」的詳細資訊，請參閱 [RedisOutputCacheProvider (英文)](https://www.nuget.org/packages/Microsoft.Web.RedisOutputCacheProvider/) NuGet 頁面。

>[!NOTE]
>除了強式名稱的 StackExchange.Redis.StrongName 封裝外，另外還有非強式名稱版本的 StackExchange.Redis。 如果您的專案使用非強式名稱的 StackExchange.Redis 版本，您必須將它解除安裝，否則專案中會發生命名衝突。 如需這些封裝的相關詳細資訊，請參閱 [設定 .NET 快取用戶端](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients)。
>
>

NuGet 封裝會下載和加入必要的組件參考，並將下列區段加入您的 web.config 檔案。 此區段包含讓您的 ASP.NET 應用程式使用 Redis 輸出快取提供者所需的設定。

```xml
<caching>
  <outputCache defaultProvider="MyRedisOutputCache">
    <providers>
      <!-- For more details check https://github.com/Azure/aspnet-redis-providers/wiki -->
      <!-- Either use 'connectionString' OR 'settingsClassName' and 'settingsMethodName' OR use 'host','port','accessKey','ssl','connectionTimeoutInMilliseconds' and 'operationTimeoutInMilliseconds'. -->
      <!-- 'databaseId' and 'applicationName' can be used with both options. -->
      <!--
      <add name="MyRedisOutputCache" 
        host = "127.0.0.1" [String]
        port = "" [number]
        accessKey = "" [String]
        ssl = "false" [true|false]
        databaseId = "0" [number]
        applicationName = "" [String]
        connectionTimeoutInMilliseconds = "5000" [number]
        operationTimeoutInMilliseconds = "1000" [number]
        connectionString = "<Valid StackExchange.Redis connection string>" [String]
        settingsClassName = "<Assembly qualified class name that contains settings method specified below. Which basically return 'connectionString' value>" [String]
        settingsMethodName = "<Settings method should be defined in settingsClass. It should be public, static, does not take any parameters and should have a return type of 'String', which is basically 'connectionString' value.>" [String]
        loggingClassName = "<Assembly qualified class name that contains logging method specified below>" [String]
        loggingMethodName = "<Logging method should be defined in loggingClass. It should be public, static, does not take any parameters and should have a return type of System.IO.TextWriter.>" [String]
        redisSerializerType = "<Assembly qualified class name that implements Microsoft.Web.Redis.ISerializer>" [String]
      />
      -->
      <add name="MyRedisOutputCache" type="Microsoft.Web.Redis.RedisOutputCacheProvider"
           host=""
           accessKey=""
           ssl="true" />
    </providers>
  </outputCache>
</caching>
```

標示註解的區段可提供屬性的範例和每個屬性的範例設定。

以來自 Microsoft Azure 入口網站之快取刀鋒視窗的值來設定屬性，並視需要設定其他值。 如需存取快取屬性的指示，請參閱 [設定 Redis 快取設定](cache-configure.md#configure-redis-cache-settings)。

* **主機** – 指定您的快取端點。
* **連接埠** – 使用您的非 SSL 連接埠或 SSL 連接埠，依 ssl 設定而定。
* **accessKey** – 用於快取的主要或次要金鑰。
* **ssl** – 如果您想要使用 ssl 保護快取/用戶端通訊則為 true，否則為 false。 請務必指定正確的連接埠。
  * 預設會為新快取停用非 SSL 連接埠。 請於此設定指定為 true，使用 SSL 連接埠。 如需啟用非 SSL 連接埠的相關詳細資訊，請參閱[設定快取](cache-configure.md)主題中的[存取連接埠](cache-configure.md#access-ports)一節。
* **databaseId** – 指定快取輸出資料所使用的資料庫。 若未指定，就會使用預設值 0。
* **applicationName** – 金鑰在 redis 中會儲存為 `<AppName>_<SessionId>_Data`。 這個命名配置可讓多個應用程式共用同一金鑰。 此參數是選擇性的，如果您未提供，將會使用預設值。
* **connectionTimeoutInMilliseconds** – 此設定可讓您覆寫 StackExchange.Redis 用戶端中的 connectTimeout 設定。 若未指定，將會使用預設的 connectTimeout 設定為 5000。 如需詳細資訊，請參閱 [StackExchange.Redis 設定模型](http://go.microsoft.com/fwlink/?LinkId=398705)(英文)。
* **operationTimeoutInMilliseconds** – 此設定可讓您覆寫 StackExchange.Redis 用戶端中的 syncTimeout 設定。 若未指定，將會使用預設的 syncTimeout 設定為 1000。 如需詳細資訊，請參閱 [StackExchange.Redis 設定模型](http://go.microsoft.com/fwlink/?LinkId=398705)(英文)。

將 OutputCache 指示詞新增至每一個您要快取輸出的頁面。

```
<%@ OutputCache Duration="60" VaryByParam="*" %>
```

在上一個範例中，已快取的頁面資料會留在快取中 60 秒，而且會對每一個參數組合快取不同版本的頁面。 如需 OutputCache 指示詞的相關詳細資訊，請參閱 [@OutputCache](http://go.microsoft.com/fwlink/?linkid=320837)。

一旦執行這些步驟，您的應用程式將設定為使用 Redis 輸出快取提供者。

## <a name="next-steps"></a>後續步驟
請查看 [Azure Redis 快取的 ASP.NET 工作階段狀態提供者](cache-aspnet-session-state-provider.md)。

