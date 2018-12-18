---
title: 何謂 Azure Redis 快取？ | Microsoft Docs
description: 了解 Azure Redis 快取是什麼及其常見的使用方式。
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: overview
ms.date: 03/26/2018
ms.author: wesmc
ms.custom: mvc
ms.openlocfilehash: 585dcd120c42562b1520d4454f9d04e445553101
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37096187"
---
# <a name="what-is-azure-redis-cache"></a>何謂 Azure Redis 快取

Azure Redis 快取是以常用的開放原始碼 [Redis 快取](https://redis.io/)為基礎。 通常用來作為快取，可為高度依賴後端資料存放區的系統改善效能及延展性。 藉由暫時將經常存取的資料複製到靠近應用程式的快速儲存體中，進而改善效能。 透過 [Redis 快取](https://redis.io/)，此快速儲存體會與 Redis 快取同樣位於記憶體中，而不會由資料庫從磁碟載入。

Azure Redis 快取也可用來作為記憶體中的資料結構存放區、分散式的非關聯資料庫和訊息代理程式。 利用 Redis 引擎的低延遲與高輸送量效能可提升應用程式效能。

Azure Redis 快取可讓您從 Azure 內的任何應用程式，存取由 Microsoft 管理且裝載於 Azure 內部或外部的安全、專用 Redis 快取。

## <a name="why-use-azure-redis-cache"></a>使用 Azure Redis 快取的理由

Redis 快取用來支援應用程式架構，或改善應用程式效能的常見模式有很多。 最常見的一些模式包括：

| 模式      | 說明                                        |
| ------------ | -------------------------------------------------- |
| [另行快取](cache-web-app-cache-aside-leaderboard.md) | 由於資料庫可能很大，所以將整個資料庫載入快取並不是建議的方法。 常見的方法是使用[另行快取](https://docs.microsoft.com/azure/architecture/patterns/cache-aside)模式，只在需要時，將資料項目載入至快取。 系統變更後端資料時，也會同時更新分佈到其他用戶端的快取。 此外，系統可以在資料項目上設定到期日，或使用收回原則，讓資料更新重新載入至快取。|
| [內容快取](cache-aspnet-output-cache-provider.md) | 大部分的網頁都是從範本產生，且具有頁首、頁尾、工具列、功能表等項目。這些網頁實際上不常變更，而且不應該以動態方式產生。 相較於後端存放區，使用 Azure Redis 這類記憶體中的快取，可讓 Web 伺服器快速存取此類型的靜態內容。 此模式可減少動態產生內容時所需的處理時間和伺服器負載。 這可讓 Web 伺服器的回應速度更快，並可讓您減少需要處理負載的伺服器數目。 Azure Redis 快取會提供「Redis 輸出快取提供者」，以協助使用 ASP.NET. 來支援此模式。|
| [使用者工作階段快取](cache-aspnet-session-state-provider.md) | 此模式通常會與購物車和使用者歷程記錄這類資訊搭配使用，而 Web 應用程式可能會想要讓使用者 cookie 與這些資訊產生關聯。 在 cookie 中儲存太多資料可能會對效能產生負面影響，因為 cookie 的大小會增加，且會與每個要求一起傳遞和驗證。 一般解決方案是使用 cookie 作為索引鍵，來查詢後端資料庫中的資料。 若使用 Azure Redis 快取這類記憶體中的快取，來讓資訊與使用者相關聯，則速度會比與完整關聯式資料庫互動快很多。 |
| 作業與訊息佇列 | 當應用程式收到要求時，執行與要求相關聯的作業通常會需要額外時間。 常見的模式是將執行時間長的作業新增至佇列，以延後處理這些應用程式，或可能讓另一個伺服器來處理。 此延後工作的方法稱為工作佇列。 有許多軟體元件的設計都支援工作佇列。 Redis 快取也會以分散式佇列的形式來達到此目的。|
| 分散式交易 | 常見的應用程式需求是要能以單一作業的形式 (自動)，執行一系列對後端資料存放區執行的命令。 所有命令都必須都成功，或所有命令必須回復為初始狀態。 Redis 快取支援會以[交易](https://redis.io/topics/transactions)形式，將命令批次作為單一作業執行。 |

## <a name="azure-redis-cache-offerings"></a>Azure Redis 快取支供應項目

Azure Redis 快取服務可在以下層級使用：

| 層 | 說明 |
|---|---|
基本 | 單一節點快取。 本層支援多種記憶體大小 (250 MB - 53 GB)。 本層適用於開發/測試和非重大的工作量。 基本層沒有服務等級協定 (SLA) |
| 標準 | 複寫的快取是兩節點 (主要/次要) 組態，由 Microsoft 管理且具高可用 SLA (99.9%) |
| 進階 | 進階層是企業就緒層。 進階層快取支援更多功能，而且具有較高的輸送量和較低的延遲。 進階層中的快取是部署在更強大的硬體上，因此效能優於基本或標準層。 這項優勢表示，針對大小相同的快取，其輸送量在進階層中會比在標準層中高 |

> [!TIP]
> 如需進階快取的大小、輸送量和頻寬的詳細資訊，請參閱 [Azure Redis 快取常見問題集](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)。
>

快取建立好之後，您可以將快取提升至更高的層級。 但不支援向下調整至較低層級。 如需調整階層的逐步指示，請參閱[如何調整 Azure Redis 快取](cache-how-to-scale.md)和[如何自動化調整作業](cache-how-to-scale.md#how-to-automate-a-scaling-operation)。

### <a name="feature-comparision"></a>功能比較

[Redis 快取定價](https://azure.microsoft.com/pricing/details/cache/)頁面會提供每一階層的詳細比較。 下表可協助說明階層所支援的一些功能：

| 功能描述 | 進階 | 標準 | 基本 |
| ------------------- | :-----: | :------: | :---: |
| [服務等級協定 (SLA)](https://azure.microsoft.com/support/legal/sla/cache/v1_0/) |✔|✔|-|
| [Redis 資料永續性](cache-how-to-premium-persistence.md) |✔|-|-|
| [Redis 叢集](cache-how-to-premium-clustering.md) |✔|-|-|
| [透過防火牆規則保護的安全性](cache-configure.md#firewall) |✔|✔|✔|
| [透過 VNet 增強的安全性和隔離](cache-how-to-premium-vnet.md) |✔|-|-|
| [匯入/匯出](cache-how-to-import-export-data.md) |✔|-|-|
| [排程更新](cache-administration.md#schedule-updates) |✔|-|-|
| [異地複寫](cache-how-to-geo-replication.md) |✔|-|-|
| [重新啟動](cache-administration.md#reboot) |✔|✔|✔|

## <a name="next-steps"></a>後續步驟

* [ASP.NET Web 應用程式快速入門](cache-web-app-howto.md) 建立可使用 Azure Redis 快取的簡單 ASP.NET Web 應用程式。
* [.NET 快速入門](cache-dotnet-how-to-use-azure-redis-cache.md) 建立可使用 Azure Redis 快取的 .NET 應用程式。
* [.NET Core 快速入門](cache-dotnet-core-quickstart.md) 建立可使用 Azure Redis 快取的 .NET Core 應用程式。
* [Node.js 快速入門](cache-nodejs-get-started.md) 建立可使用 Azure Redis 快取的簡單 Node.js 應用程式。
* [Java 快速入門](cache-java-get-started.md) 建立可使用 Azure Redis 快取的簡單 Java 應用程式。
* [Python 快速入門](cache-python-get-started.md) 建立可使用 Azure Redis 快取的 Python 應用程式。
