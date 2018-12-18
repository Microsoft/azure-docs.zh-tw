---
title: 資料庫待用加密：Azure Cosmos DB | Microsoft Docs
description: 了解 Azure Cosmos DB 如何提供所有資料的預設加密。
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: rafats
ms.openlocfilehash: 2b54f8c7d9f6427f3104d3c64c65cc555f68738a
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037982"
---
# <a name="azure-cosmos-db-database-encryption-at-rest"></a>Azure Cosmos DB 資料庫待用加密

「待用加密」是一個用來描述加密靜態儲存裝置 (例如固態硬碟 (SSD) 和硬碟 (HDD)) 上資料的常見用語。 Cosmos DB 會在 SSD 上儲存其主要資料庫。 其媒體附件和備份會儲存在通常由 HDD 備份的 Azure Blob 儲存體中。 在針對 Cosmos DB 發行待用加密之後，您所有的資料庫、媒體附件及備份都會進行加密。 您的資料現在會在傳輸過程 (透過網路) 和待用 (位於靜態儲存裝置上) 期間進行加密，提供您端對端的加密。

Cosmos DB 是一種 PaaS 服務，使用起來非常容易。 由於儲存在 Cosmos DB 中的所有使用者資料都會在待用和傳輸過程期間進行加密，因此您不必採取任何動作。 換句話說，待用加密預設便會「開啟」。 沒有關閉或開啟的控制項。 在提供這項功能的同時，我們會繼續符合[可用性和效能 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db)。

## <a name="implementation-of-encryption-at-rest-for-azure-cosmos-db"></a>Azure Cosmos DB 待用加密實作

待用加密是使用數種安全性技術來實作的，這些技術包括安全金鑰儲存體系統、加密的網路，以及密碼編譯 API。 負責解密及處理資料的系統，必須和負責管理金鑰的系統進行通訊。 下圖顯示加密資料儲存體和金鑰管理的分隔方式。 

![設計圖表](./media/database-encryption-at-rest/design-diagram.png)

使用者要求的基本流程如下：
- 準備好使用者資料庫帳戶，並透過向管理服務資源提供者提出要求來擷取儲存體金鑰。
- 使用者透過 HTTPS/安全傳輸建立與 Cosmos DB 的連線 (SDK 會對詳細資料進行抽象處理)。
- 使用者透過先前所建立的安全連線，傳送要儲存的 JSON 文件。
- 為 JSON 文件編製索引 (除非使用者已關閉編製索引)。
- JSON 文件和索引資料皆寫入安全儲存體。
- 系統會定期自安全儲存體讀取資料，並備份到 Azure 加密 Blob 存放區。

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="q-how-much-more-does-azure-storage-cost-if-storage-service-encryption-is-enabled"></a>問︰如果已啟用儲存體服務加密，Azure 儲存體的成本會多出多少？
答：沒有任何額外成本。

### <a name="q-who-manages-the-encryption-keys"></a>問︰誰負責管理加密金鑰？
答︰金鑰是由 Microsoft 管理。

### <a name="q-how-often-are-encryption-keys-rotated"></a>問︰加密金鑰多久輪替一次？
答：Microsoft 針對加密金鑰輪替有一套 Cosmos DB 會遵循的內部方針。 不會發佈特定方針。 Microsoft 會發佈[安全性開發週期 (SDL)](https://www.microsoft.com/sdl/default.aspx)，這會視為內部指導方針的子集，其中包含對開發人員很實用的最佳做法。

### <a name="q-can-i-use-my-own-encryption-keys"></a>問︰我是否可以使用自己的加密金鑰？
答：Cosmos DB 是一種 PaaS 服務，我們也很努力地讓這個服務容易使用。 我們注意到會詢問此問題的使用者，通常是想詢問有關符合如 PCI-DSS 等合規性需求的 Proxy 問題。 建置這項功能時，我們已與合規性稽核人員合作，以確保使用 Cosmos DB 的客戶能夠在不需要自行管理金鑰的情況下滿足他們的需求。

### <a name="q-what-regions-have-encryption-turned-on"></a>問：有哪些區域已開啟加密？
答：所有的 Azure Cosmos DB 區域皆已針對所有使用者資料開啟加密。

### <a name="q-does-encryption-affect-the-performance-latency-and-throughput-slas"></a>問︰加密會影響效能延遲和輸送量 SLA 嗎？
答︰現在所有現有和新的帳戶都啟用待用加密，但是效能 SLA 沒有任何影響或變化。 您可以在 [Cosmos DB SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) 頁面閱讀更多資訊以查看最新的保證。

### <a name="q-does-the-local-emulator-support-encryption-at-rest"></a>問︰本機模擬器支援待用加密嗎？
答︰模擬器是獨立的開發/測試工具，不使用受控 Cosmos DB 服務所使用的金鑰管理服務。 我們建議您在儲存機密模擬器測試資料的磁碟機上啟用 BitLocker。 [模擬器支援變更預設資料目錄](local-emulator.md)，也支援使用已知位置。

## <a name="next-steps"></a>後續步驟

如需 Cosmos DB 安全性及最新改進的概觀，請參閱 [Azure Cosmos DB 資料庫安全性](database-security.md)。
如需 Microsoft 認證的詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/)。
