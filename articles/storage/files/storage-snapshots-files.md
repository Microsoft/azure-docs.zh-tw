---
title: 適用於 Azure 檔案的共用快照集概觀 | Microsoft Docs
description: Azure 共用快照集是在某個時間點拍攝的 Azure 檔案共用唯讀版本，是備份共用的一個方法。
services: storage
author: RenaShahMSFT
ms.service: storage
ms.topic: article
ms.date: 01/17/2018
ms.author: renash
ms.component: files
ms.openlocfilehash: f98817802478853e8d176223a9a9a3081531897f
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528369"
---
# <a name="overview-of-share-snapshots-for-azure-files"></a>Azure 檔案的共用快照集概觀 
Azure 檔案提供為檔案共用拍攝共用快照集的功能。 共用快照集能擷取該時間點的共用狀態。 在本文中，我們說明共用快照集提供哪些功能，以及如何在您的自訂使用案例中運用這些功能。

## <a name="when-to-use-share-snapshots"></a>使用共用快照集的時機

### <a name="protection-against-application-error-and-data-corruption"></a>預防應用程式錯誤和資料損毀
使用檔案共用執行各種作業 (如寫入、讀取、儲存、傳輸和處理) 的應用程式。 設定錯誤的應用程式或導入的非蓄意錯誤 (bug)，可能會導致意外覆寫或毀損幾個區塊。 為防範這些狀況，您可以先建立共用快照集，再部署新的應用程式程式碼。 如果錯誤 (bug) 或應用程式錯誤是因為新部署所引起，您可以將您的資料還原成該檔案共用上的先前版本。 

### <a name="protection-against-accidental-deletions-or-unintended-changes"></a>預防意外刪除或非預期的變更
假設您正在使用檔案共用中的某個文字檔。 在該文字檔關閉後，您就無法復原變更。 在這些情況下，您必須復原該檔案先前的版本。 如果檔案不小心遭到重新命名或刪除，您可使用共用快照集來復原先前版本的檔案。

### <a name="general-backup-purposes"></a>一般備份用途
建立檔案共用之後，您可以定期建立檔案共用的共用快照集，以供資料備份之用。 定期建立的共用快照集，有助於維護先前版本的檔案，因應未來的稽核需求或災害復原之需。

## <a name="capabilities"></a>功能
共用快照集是某個時間點的唯讀資料複本。 您可以使用 REST API 來建立、刪除及管理快照集。 用戶端程式庫、Azure CLI 和 Azure 入口網站亦提供相同的功能。 

您可以使用 REST API 和 SMB 兩者來檢視共用的快照集。 您可以擷取目錄或檔案的版本清單，也可以將特定版本當做磁碟機直接掛接。 

建立共用快照集之後，即可加以讀取、複製或刪除，但不能修改。 您無法將整個共用快照集複製到另一個儲存體帳戶。 您必須使用 AzCopy 或其他複製機制逐一複製每個檔案。

共用快照集功能會於檔案共用層級提供。 擷取會於個別的檔案層級提供，以允許還原個別檔案。 您可以使用 SMB、REST API、入口網站、用戶端程式庫或 PowerShell/CLI 工具來還原完整的檔案共用。

檔案共用的共用快照集和其基底檔案共用相同。 唯一的差別在於，**DateTime** 值會附加至共用 URI，以表示建立共用快照集的時間。 例如，如果檔案共用 URI 為 http://storagesample.core.file.windows.net/myshare，則共用快照集 URI 會類似於：
```
http://storagesample.file.core.windows.net/myshare?snapshot=2011-03-09T01:42:34.9360000Z
```

系統會保存共用快照集，直到您將它們明確刪除為止。 共用快照集必須有其基底檔案共用才能存在。 您可以列舉與基底檔案共用相關聯的快照集，以追蹤目前的快照集。 

當您建立檔案共用的共用快照集時，系統會將位於共用系統屬性中的檔案複製到值相同的共用快照集。 除非您在建立共用快照集時為其指定個別的中繼資料，否則，基底檔案和檔案共用的中繼資料也會複製到共用快照集。

您無法刪除具有共用快照集的共用，除非先刪除所有共用快照集。

## <a name="space-usage"></a>空間使用量 
共用快照集具有累加性質。 系統只會儲存最新共用快照集之後變更的資料。 這樣能縮短建立共用快照集所需的時間，也能節省儲存成本。 物件或屬性的任何寫入作業或中繼資料更新作業都算是「變更內容」，因此會儲存在共用快照集內。 

為了節省空間，您可以在變換率最高的期間內刪除共用快照集。

即使共用快照集以累加方式儲存，您只需要保留最新的共用快照集，以便還原共用。 當您刪除共用快照集時，只會移除該共用快照集的唯一資料。 使用中的快照集包含所有瀏覽及還原資料 (從共用快照集建立時間開始) 到原始或替代位置所需的資訊。 您可以在項目層級來進行還原。

快照集不計入 5 TB 的共用限制。 共用快照集佔用的空間總數沒有限制。 但儲存體帳戶限制依然有效。

## <a name="limits"></a>限制
Azure 檔案服務目前允許的共用快照集最大數目為 200。 保留 200 個共用快照集之後，必須先刪除舊的共用快照集，才能建立新的共用快照集。 

建立共用快照集的同時呼叫數目沒有限制。 特定檔案共用之共用快照集耗用的空間數量沒有限制。 

## <a name="copying-data-back-to-a-share-from-share-snapshot"></a>從共用快照集將資料複製回共用
涉及檔案和共用快照集的複製作業須遵循下列規則：

您可以將檔案共用快照集內的個別檔案複製到其基底共用或其他任何位置。 您可以從共用快照集還原舊版的檔案，或利用逐一複製檔案的方式還原整個檔案共用。 共用快照集不會提升為基底共用。 

複製過後共用快照集會保持不變，不過系統會將先前共用快照集提供的資料複本覆寫到基底檔案共用。 所有還原的檔案都算是「變更內容」。

您可以將共用快照集內的檔案複製到不同名稱的目的地。 產生的目的地檔案會是一個可寫入的檔案，而不是共用快照集。

使用某個複本覆寫目的地檔案時，與原始目的地檔案相關聯的所有共用快照集都會保持不變。

## <a name="general-best-practices"></a>一般最佳作法 
在 Azure 上執行基礎結構時，請盡可能將資料復原所需的備份作業自動化。 自動化的動作比手動程序來得可靠，能協助您加強資料保護和復原能力。 您可以使用 REST API、用戶端 SDK 或指令碼來進行自動化。

在部署共用快照集排程器之前，請審慎考慮共用快照集頻率和保留設定，以避免產生不必要的共用快照集費用。

共用快照集只提供檔案層級的保護。 共用快照集無法避免檔案共用或儲存體帳戶的誤刪情況。 為了預防儲存體帳戶遭到意外刪除，您可以鎖定儲存體帳戶或資源群組。

## <a name="next-steps"></a>後續步驟
- 在下列位置使用共用快照集：
    - [入口網站](storage-how-to-use-files-portal.md#create-and-modify-share-snapshots)
    - [PowerShell](storage-how-to-use-files-powershell.md#create-and-modify-share-snapshots)
    - [CLI](storage-how-to-use-files-cli.md#create-and-modify-share-snapshots)
- [共用快照集常見問題集](storage-files-faq.md#share-snapshots)
