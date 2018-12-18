---
title: 在 Azure Redis 快取中匯入與匯出資料 | Microsoft Docs
description: 了解如何使用進階 Azure Redis 快取執行個體將資料匯入至 blob 儲存體並從其中匯出資料
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 4a68ac38-87af-4075-adab-569d37d7cc9e
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 07/31/2017
ms.author: wesmc
ms.openlocfilehash: 6733891213f15e9ceaf08ef7fb50380db47a695f
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39259189"
---
# <a name="import-and-export-data-in-azure-redis-cache"></a>在 Azure Redis 快取中匯入與匯出資料
匯入/匯出是 Azure Redis 快取資料管理作業，可讓您將資料匯入 Azure Redis 快取或將資料從 Azure Redis 快取匯出，方法是從進階快取將 Redis 快取資料庫 (RDB) 快照匯入和匯出至 Azure 儲存體帳戶中的 blob。 

- **匯出** - 您可以將您的 Azure Redis 快取 RDB 快照集匯出至分頁 Blob。
- **匯入** - 您可以從分頁 Blob 或區塊 Blob 匯入您的 Redis 快取 RDB 快照集。

匯入/匯出可讓您在不同的 Azure Redis 快取執行個體之間移轉，或在使用前將資料填入快取。

本文提供使用 Azure Redis 快取匯入和匯出資料的指南，並提供常見問題的解答。

> [!IMPORTANT]
> 匯入/匯出僅供預覽，只供 [進階層](cache-premium-tier-intro.md) 快取使用。
>
>

## <a name="import"></a>Import
匯入可以用來從任何雲端或環境中執行的 Redis 伺服器 (包含在 Linux、Windows 上執行的 Redis，或任何雲端提供者，例如 Amazon Web Services 等) 引入 Redis 相容 RDB 檔案。 匯入資料是使用預先填入資料建立快取的輕鬆方式。 在匯入程序期間，Azure Redis 快取會從 Azure 儲存體將 RDB 檔案載入記憶體，然後將金鑰插入快取。

> [!NOTE]
> 開始匯入作業之前，請確定您的 Redis 資料庫 (RDB) 檔案已上傳到 Azure 儲存體中的分頁或區塊 blob，位於和 Azure Redis 快取執行個體相同的區域和訂用帳戶。 如需詳細資訊，請參閱 [開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 如果您使用 [Azure Redis 快取匯出](#export) 功能匯出 RDB 檔案，表示 RDB 檔案已儲存在分頁 blob，並且可供匯入。
>
>

1. 若要匯入一或多個匯出的快取 Blob，請[瀏覽至 Azure 入口網站中的快取](cache-configure.md#configure-redis-cache-settings)，然後從 [資源功能表] 按一下 [匯入資料]。

    ![匯入資料][cache-import-data]
2. 按一下 [選擇 Blob]  ，然後選取包含要匯入之資料的儲存體帳戶。

    ![Choose storage account][cache-import-choose-storage-account]
3. 按一下包含要匯入之資料的容器。

    ![選擇容器][cache-import-choose-container]
4. 按一下 blob 名稱左側的區域以選取一或多個 blob，即可加以匯入，然後按一下 [選取] 。

    ![選擇 blob][cache-import-choose-blobs]
5. 按一下 [匯入]  開始匯入程序。

   > [!IMPORTANT]
   > 快取用戶端無法在匯入程序期間存取快取，而且在快取中的所有現有資料都會刪除。
   >
   >

    ![Import][cache-import-blobs]

    您可以遵循 Azure 入口網站的通知，或檢視 [稽核記錄檔](../azure-resource-manager/resource-group-audit.md)中的事件來監視匯入作業的進度。

    ![匯入進度][cache-import-data-import-complete]

## <a name="export"></a>匯出
匯出可讓您將儲存在 Azure Redis 快取中的資料匯出至 Redis 相容 RDB 檔案。 您可以使用這項功能，將資料從一個 Azure Redis 快取執行個體移到另一個或其他 Redis 伺服器。 在匯出程序期間，會在裝載 Azure Redis 快取伺服器執行個體的 VM 上建立站存檔案，並將檔案上傳至指定的儲存體帳戶。 當匯出作業完成時的狀態為成功或失敗時，都會刪除暫存檔案。

1. 若要將目前的快取內容匯出至儲存體，請[瀏覽至 Azure 入口網站中的快取](cache-configure.md#configure-redis-cache-settings)，然後從 [資源功能表] 按一下 [匯出資料]。

    ![選擇儲存體容器][cache-export-data-choose-storage-container]
2. 按一下 [選擇儲存體容器]  ，然後選取所需的儲存體帳戶。 儲存體帳戶必須與您的快取位於相同的訂用帳戶和區域中。

   > [!IMPORTANT]
   > 匯出使用分頁 Blob，傳統和 Resource Manager 儲存體帳戶支援這些 Blob，但 [Blob 儲存體帳戶](../storage/common/storage-account-options.md#blob-storage-accounts)目前不支援。
   >
   >

    ![儲存體帳戶][cache-export-data-choose-account]
3. 選擇所需的 blob 容器，然後按一下 [選取] 。 若要使用新的容器，請按一下 [新增容器]  先加以新增，然後從清單中選取它。

    ![選擇儲存體容器][cache-export-data-container]
4. 輸入 [Blob 名稱前置詞]，然後按一下 [匯出] 以啟動匯出程序。 Blob 名稱前置詞可用來做為此匯出作業所產生之檔案名稱的前置詞。

    ![匯出][cache-export-data]

    您可以遵循 Azure 入口網站的通知，或檢視[稽核記錄檔](../azure-resource-manager/resource-group-audit.md)中的事件來監視匯出作業的進度。

    ![匯出資料完成][cache-export-data-export-complete]

    在匯出程序期間快取隨時可供使用。

## <a name="importexport-faq"></a>匯入/匯出常見問題集
本節包含匯入/匯出功能的常見問題集。

* [哪些定價層可以使用匯入/匯出？](#what-pricing-tiers-can-use-importexport)
* [我是否可以從任何 Redis 伺服器匯入資料？](#can-i-import-data-from-any-redis-server)
* [我可以匯入哪些 RDB 版本？](#what-rdb-versions-can-i-import)
* [在匯入/匯出作業期間，是否可以使用我的快取？](#is-my-cache-available-during-an-importexport-operation)
* [我可以使用匯入/匯出搭配 Redis 叢集嗎？](#can-i-use-importexport-with-redis-cluster)
* [匯入/匯出如何對自訂資料庫設定運作？](#how-does-importexport-work-with-a-custom-databases-setting)
* [匯入/匯出和 Redis 永續性有何不同？](#how-is-importexport-different-from-redis-persistence)
* [我可以使用 PowerShell、CLI 或其他管理用戶端自動化匯入/匯出嗎？](#can-i-automate-importexport-using-powershell-cli-or-other-management-clients)
* [我在匯入/匯出作業期間收到逾時錯誤。這代表什麼意思？](#i-received-a-timeout-error-during-my-importexport-operation-what-does-it-mean)
* [我將資料匯出至 Azure Blob 儲存體時收到錯誤。發生什麼情形？](#i-got-an-error-when-exporting-my-data-to-azure-blob-storage-what-happened)

### <a name="what-pricing-tiers-can-use-importexport"></a>哪些定價層可以使用匯入/匯出？
匯入/匯出僅適用於進階定價層。

### <a name="can-i-import-data-from-any-redis-server"></a>我是否可以從任何 Redis 伺服器匯入資料？
是，除了匯入從 Azure Redis 快取執行個體匯出的資料之外，您還可以從執行於雲端或環境 (例如 Linux、Windows 或雲端提供者，例如 Amazon Web Services) 中的任何 Redis 伺服器匯入 RDB 檔案。 若要這樣做，請將 RDB 檔案從所需的 Redis 伺服器上傳至 Azure 儲存體帳戶中的分頁或區塊 blob，然後將它匯入到您的進階 Azure Redis 快取執行個體。 例如，建議您從生產快取匯出資料，並將其匯入至快取，用於測試活移轉做為預備環境的一部分。

> [!IMPORTANT]
> 若要在使用分頁 blob 時成功匯入從非 Azure Redis 快取的 Redis 伺服器匯出的資料，分頁 blob 大小必須在 512 個位元組的界限上對齊。 如需執行任何所需的位元組填補的範例程式碼，請參閱[範例分頁 Blob 上傳 ](https://github.com/JimRoberts-MS/SamplePageBlobUpload) \(英文\)。
> 
> 

### <a name="what-rdb-versions-can-i-import"></a>我可以匯入哪些 RDB 版本？

Azure Redis 快取最多可支援到 RDB 第 7 版的 RDB 匯入。

### <a name="is-my-cache-available-during-an-importexport-operation"></a>在匯入/匯出作業期間，是否可以使用我的快取？
* **匯出** - 快取持續可供使用，而且您可以繼續在匯出作業期間使用快取。
* **匯入** - 匯入作業啟動時會無法使用快取，當匯入作業完成時，快取即可供使用。

### <a name="can-i-use-importexport-with-redis-cluster"></a>我可以使用匯入/匯出搭配 Redis 叢集嗎？
可以，您可以在叢集快取和非叢集快取之間匯入/匯出。 由於 Redis 叢集[僅支援資料庫 0](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)，因此不會匯入資料庫中 0 以外的任何資料。 匯入叢集快取資料時，金鑰會在叢集的分區之間重新分配。

### <a name="how-does-importexport-work-with-a-custom-databases-setting"></a>匯入/匯出如何對自訂資料庫設定運作？
某些定價層具有不同的[資料庫限制](cache-configure.md#databases)，因此，在匯入時，如果您在快取建立期間為 `databases` 設定設定了自訂值，則有一些考量。

* 匯入到與您從中匯出的階層具有較低 `databases` 限制的定價層時：
  * 如果您使用預設數字 `databases`，即所有定價層為 16，則不會遺失資料。
  * 如果您要使用的自訂數字 `databases` ，落在您要匯入之階層限制範圍內，則不會遺失資料。
  * 如果您的匯出資料包含位於超出新階層限制之資料庫中的資料，則不會匯入來自這些較高階層資料庫中的資料。

### <a name="how-is-importexport-different-from-redis-persistence"></a>匯入/匯出和 Redis 永續性有何不同？
Azure Redis 快取永續性讓您將儲存在 Redis 快取中的資料存留至 Azure 儲存體。 設定永續性後，Azure Redis 快取會依據可設定的備份頻率，在磁碟中保存一份 Redis 二進位格式的 Redis 快取快照。 如果發生同時停用主要和複本快取的災難性事件，即可使用最新的快照自動還原快取資料。 如需詳細資訊，請參閱 [如何設定進階 Azure Redis 快取的資料永續性](cache-how-to-premium-persistence.md)。

匯入/匯出可讓您將資料引入 Azure Redis 快取或從其中匯出。 它無法設定備份和還原使用 Redis 永續性。

### <a name="can-i-automate-importexport-using-powershell-cli-or-other-management-clients"></a>我可以使用 PowerShell、CLI 或其他管理用戶端自動化匯入/匯出嗎？
可以。如需 PowerShell 指示，請參閱[匯入 Redis 快取](cache-howto-manage-redis-cache-powershell.md#to-import-a-redis-cache)和[匯出 Redis 快取](cache-howto-manage-redis-cache-powershell.md#to-export-a-redis-cache)。

### <a name="i-received-a-timeout-error-during-my-importexport-operation-what-does-it-mean"></a>我在匯入/匯出作業期間收到逾時錯誤。 這代表什麼意思？
如果您在初始化作業之前停留在 [匯入資料] 或 [匯出資料] 刀鋒視窗超過 15 分鐘，您會收到類似下列範例錯誤訊息的錯誤：

    The request to import data into cache 'contoso55' failed with status 'error' and error 'One of the SAS URIs provided could not be used for the following reason: The SAS token end time (se) must be at least 1 hour from now and the start time (st), if given, must be at least 15 minutes in the past.

若要解決此問題，請在經過 15 分鐘之前起始匯入或匯出作業。

### <a name="i-got-an-error-when-exporting-my-data-to-azure-blob-storage-what-happened"></a>我將資料匯出至 Azure Blob 儲存體時收到錯誤。 發生什麼情形？
匯出只能使用儲存為分頁 blob 的 RDB 檔案。 目前不支援其他的 Blob 類型，包括經常存取及不常存取層的 Blob 儲存體帳戶。 如需詳細資訊，請參閱 [Blob 儲存體帳戶](../storage/common/storage-account-options.md#blob-storage-accounts)。

## <a name="next-steps"></a>後續步驟
了解如何使用更多進階快取功能。

* [Azure Redis Cache 高階層簡介](cache-premium-tier-intro.md)    

<!-- IMAGES -->
[cache-settings-import-export-menu]: ./media/cache-how-to-import-export-data/cache-settings-import-export-menu.png
[cache-export-data-choose-account]: ./media/cache-how-to-import-export-data/cache-export-data-choose-account.png
[cache-export-data-choose-storage-container]: ./media/cache-how-to-import-export-data/cache-export-data-choose-storage-container.png
[cache-export-data-container]: ./media/cache-how-to-import-export-data/cache-export-data-container.png
[cache-export-data-export-complete]: ./media/cache-how-to-import-export-data/cache-export-data-export-complete.png
[cache-export-data]: ./media/cache-how-to-import-export-data/cache-export-data.png
[cache-import-data]: ./media/cache-how-to-import-export-data/cache-import-data.png
[cache-import-choose-storage-account]: ./media/cache-how-to-import-export-data/cache-import-choose-storage-account.png
[cache-import-choose-container]: ./media/cache-how-to-import-export-data/cache-import-choose-container.png
[cache-import-choose-blobs]: ./media/cache-how-to-import-export-data/cache-import-choose-blobs.png
[cache-import-blobs]: ./media/cache-how-to-import-export-data/cache-import-blobs.png
[cache-import-data-import-complete]: ./media/cache-how-to-import-export-data/cache-import-data-import-complete.png
