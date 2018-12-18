---
title: 升級至一般用途 v2 儲存體帳戶 - Azure 儲存體 | Microsoft Docs
description: 升級至一般用途 v2 儲存體帳戶。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/11/2018
ms.author: tamram
ms.openlocfilehash: 6e77c4836531a7efd0b52b9a411ac40ff6a613fa
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224489"
---
# <a name="upgrade-to-a-general-purpose-v2-storage-account"></a>升級至一般用途 v2 儲存體帳戶

一般用途 v2 儲存體帳戶能支援最新的 Azure 儲存體功能，而且包含一般用途 v1 與 Blob 儲存體帳戶的所有功能。 針對大部分的儲存體案例，皆建議使用一般用途 v2 帳戶。 一般用途 v2 帳戶能針對 Azure 儲存體提供最低的每 GB 容量價格，以及極具業界競爭力的交易價格。

從一般用途 v1 或 Blob 儲存體帳戶升級至一般用途 v2 儲存體帳戶很簡單。 您可以使用 Azure 入口網站、PowerShell 或 Azure CLI 來執行升級。 升級帳戶的動作無法復原，而且可能會產生計費費用。

## <a name="upgrade-using-the-azure-portal"></a>使用 Azure 入口網站升級

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至儲存體帳戶。
3. 在 [設定] 區段中，按一下 [組態]。
4. 在 [帳戶類型] 下，按一下 [升級]。
5. 在 [確認升級] 下，輸入您的帳戶名稱。 
6. 按一下刀鋒視窗底部的 [升級]。

## <a name="upgrade-with-powershell"></a>使用 PowerShell 升級

若要使用 PowerShell 將一般用途 v1 帳戶升級至一般用途 v2 帳戶，請先更新 PowerShell 以使用最新版的 **AzureRm.Storage** 模組。 如需安裝 PowerShell 的相關資訊，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。 

接下來，請呼叫下列命令，並取代為您的資源群組與儲存體帳戶的名稱，以升級帳戶：

```powershell
Set-AzureRmStorageAccount -ResourceGroupName <resource-group> -AccountName <storage-account> -UpgradeToStorageV2
```

## <a name="upgrade-with-azure-cli"></a>使用 Azure CLI 升級

若要使用 Azure CLI 將一般用途 v1 帳戶升級至一般用途 v2 帳戶，請先安裝最新版的 Azure CLI。 如需安裝 CLI 的相關資訊，請參閱[安裝 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。 

接下來，請呼叫下列命令，並取代為您的資源群組與儲存體帳戶的名稱，以升級帳戶：

```cli
az storage account update -g <resource-group> -n <storage-account> --set kind=StorageV2
``` 

## <a name="specify-an-access-tier-for-blob-data"></a>指定 Blob 資料的存取層

一般用途 v2 帳戶能支援所有 Azure 儲存體服務與資料物件，但存取層僅適用於 Blob 儲存體中的區塊 Blob。 當您升級至一般用途 v2 儲存體帳戶時，您可以指定 Blob 資料的存取層。 

存取層可讓您根據預期的使用模式，選擇最符合成本效益的儲存體。 區塊 Blob 可以儲存在經常性存取層、非經常性存取層，或封存存取層。 如需存取層的詳細資訊，請參閱 [Azure Blob 儲存體︰經常性存取、非經常性存取和封存儲存層](../blobs/storage-blob-storage-tiers.md)。

根據預設，系統會在經常性存取層中建立新的儲存體帳戶，而一般用途 v1 儲存體帳戶則會升級至經常性存取層。 如果您是在已升級的情況下決定要針對資料使用哪個存取層，請考慮您的案例。 有兩個適用於移轉至一般用途 v2 帳戶的典型使用者案例：

* 您有現有一般用途 v1 儲存體帳戶，而且想要針對變更成具有正確 Blob 資料儲存層的一般用途 v2 儲存體帳戶進行評估。
* 您已決定使用一般用途 v2 儲存體帳戶或是已有一個帳戶，並想要評估應該針對 Blob 資料使用經常性或非經常性儲存層。

在這兩種情況下，您的第一要務都是估計儲存、存取及操作儲存在一般用途 v2 儲存體帳戶中之資料的成本，並與您目前的成本進行比較。

### <a name="estimate-costs-for-your-current-usage-patterns"></a>針對目前的使用模式估計成本

若要估計儲存及存取特定存取層的一般用途 v2 儲存體帳戶中 Blob 資料的成本，您必須評估現有的使用模式或估計預期的使用模式。 一般而言，您想要知道︰

* 您的 Blob 儲存體耗用量 (以 GB 為單位)，包括：
    - 有多少資料儲存在儲存體帳戶中？
    - 每月的資料量變更為何？新資料是否會不斷取代舊資料？
* 您 Blob 儲存體資料的主要存取模式，包括：
    - 儲存體帳戶的讀取和寫入資料量有多少？ 
    - 在儲存體帳戶中的資料上發生多少讀取作業和寫入作業？

為了協助決定符合您需求的最佳存取層，您應該判斷 Blob 資料目前所使用的容量，以及該資料的使用狀況。 

若要在移轉前收集儲存體帳戶的使用量資料，您可以使用 [Azure 監視器](../../monitoring-and-diagnostics/monitoring-overview-azure-monitor.md)來監視儲存體帳戶。 Azure 監視器能執行記錄並提供各種 Azure 服務的計量資料，包括 Azure 儲存體。 

若要監視儲存體帳戶中 Blob 的耗用量資料，請啟用 Azure 監視器中的容量計量。 容量計量會記錄有關帳戶中的 Blob 每天所使用之儲存體容量的資料。 容量計量可以用來估計將資料儲存在儲存體帳戶中的成本。 若要了解 Blob 儲存體容量針對每個帳戶類型的價格，請參閱[區塊 Blob 價格](https://azure.microsoft.com/pricing/details/storage/blobs/)。

若要監視 Blob 儲存體的資料存取模式，請啟用 Azure 監視器中的交易計量。 您可以篩選不同的 Azure 儲存體作業，以估計呼叫每個作業的頻率。 若要了解每個類型的帳戶針對不同類型交易的區塊及附加區塊價格，請參閱[區塊 Blob 價格](https://azure.microsoft.com/pricing/details/storage/blobs/)。  

如需從 Azure 監視器收集計量的詳細資訊，請參閱 [Azure 監視器中的 Azure 儲存體計量](storage-metrics-in-azure-monitor.md)。

## <a name="next-steps"></a>後續步驟

- [建立儲存體帳戶](storage-quickstart-create-account.md)
- [管理 Azure 儲存體帳戶](storage-account-manage.md)