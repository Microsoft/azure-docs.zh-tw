---
title: Azure 檔案服務延展性和效能目標
description: 了解 Azure 檔案服務的延展性和效能目標，包括容量、要求率以及輸入和輸出頻寬限制。
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 10/16/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: e10f45af89e19f6fe62ff729f96d870e008c96ec
ms.sourcegitcommit: 8a74ab1beba4522367aef8cb39c92c1147d5ec13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98611095"
---
# <a name="azure-files-scalability-and-performance-targets"></a>Azure 檔案服務延展性和效能目標

[Azure 檔案服務](storage-files-introduction.md)可提供在雲端中完全受控的檔案共用，可透過業界標準 SMB 通訊協定加以存取。 本文討論 Azure 檔案服務和 Azure 檔案同步的延展性和效能目標。

此處所列的延展性和效能目標是高階的目標，但可能會受到部署中的其他變數影響。 例如，檔案的輸送量可能也受限於可用的網路頻寬，而不只是裝載 Azure 檔案服務的伺服器。 我們強烈建議您測試您的使用模式，以判斷 Azure 檔案服務的延展性和效能是否符合您的需求。 我們也保證會隨時間提高這些限制。 歡迎您在底下留言或前往 [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) \(英文\)，提供想要我們提高哪些限制的意見反應。

## <a name="azure-storage-account-scale-targets"></a>Azure 儲存體帳戶擴展目標

Azure 檔案共用的父資源是 Azure 儲存體帳戶。 儲存體帳戶代表 Azure 中可供多個儲存體服務 (包括 Azure 檔案服務) 儲存資料的儲存體集區。 將資料儲存在儲存體帳戶的其他服務有 Azure Blob 儲存體、Azure 佇列儲存體和 Azure 資料表儲存體。 下列目標適用於在儲存體帳戶中儲存資料的所有儲存體服務：

[!INCLUDE [azure-storage-account-limits-standard](../../../includes/azure-storage-account-limits-standard.md)]

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

> [!Important]  
> 其他儲存體服務的一般用途儲存體帳戶使用量，會影響您儲存體帳戶中的 Azure 檔案共用。 比方說，如果達到 Azure Blob 儲存體的最大儲存體帳戶容量，您將無法在 Azure 檔案共用上建立新檔案，即使您的 Azure 檔案共用低於最大共用大小也一樣。

## <a name="azure-files-scale-targets"></a>Azure 檔案擴展目標

Azure 檔案儲存體有三個要考慮的限制類別：儲存體帳戶、共用和檔案。

例如：使用 premium 檔案共用時，單一共用可以達到 100000 IOPS，而單一檔案最多可擴大至 5000 IOPS。 因此，如果一個共用中有三個檔案，您可以從該共用取得的最大 IOPS 為15000。

### <a name="standard-storage-account-limits"></a>標準儲存體帳戶限制

如需這些限制，請參閱 [Azure 儲存體帳戶擴展目標](#azure-storage-account-scale-targets) 一節。

### <a name="premium-filestorage-account-limits"></a>Premium FileStorage 帳戶限制

[!INCLUDE [azure-storage-limits-filestorage](../../../includes/azure-storage-limits-filestorage.md)]

> [!IMPORTANT]
> 儲存體帳戶限制適用于所有共用。 只有在每個 FileStorage 帳戶只有一個共用時，才能達到 FileStorage 帳戶的最大值。

### <a name="file-share-and-file-scale-targets"></a>檔案共用和檔案擴展目標

> [!NOTE]
> 超過 5 TiB 的標準檔案共用具有某些限制。 如需啟用較大檔案共用大小的限制和指示清單，請參閱規劃指南中的在 [標準檔案共用上啟用較大](storage-files-planning.md#enable-standard-file-shares-to-span-up-to-100-tib) 的檔案共用一節。

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

[!INCLUDE [storage-files-premium-scale-targets](../../../includes/storage-files-premium-scale-targets.md)]

## <a name="azure-file-sync-scale-targets"></a>Azure 檔案同步擴展目標

Azure 檔案同步的設計目標是無限制的使用方式，但無限制的使用方式不一定行得通。 下表指出 Microsoft 的測試界限，也指出哪些目標是固定限制：

[!INCLUDE [storage-sync-files-scale-targets](../../../includes/storage-sync-files-scale-targets.md)]

### <a name="azure-file-sync-performance-metrics"></a>Azure 檔案同步效能計量

由於 Azure 檔案同步代理程式會在連線至 Azure 檔案共用的 Windows Server 機器上執行，因此有效的同步效能將取決於基礎結構中的許多因素：Windows Server 和基礎磁碟組態、伺服器與 Azure 儲存體之間的網路頻寬、檔案大小、資料集大小總計，以及資料集的活動。 由於 Azure 檔案同步會在檔案層級上運作，因此 Azure 檔案同步解決方案的效能特性應以每秒處理的物件 (檔案和目錄) 數來測量，以獲得較精準的結果。

在下列兩個階段中，Azure 檔案同步必須達到高效能：

1. **初始一次性佈建**：若要讓初始佈建達到最佳效能，請參閱 [透過 Azure 檔案同步上架](storage-sync-files-deployment-guide.md#onboarding-with-azure-file-sync)以取得最佳部署的詳細資料。
2. **持續同步**：在 Azure 檔案共用中初次植入資料之後，Azure 檔案同步會將多個端點保持在同步狀態。

為了協助您規劃每個階段的部署，以下提供在採用某種組態的系統上進行內部測試期間所觀察到的結果

| 系統設定 | 詳細資料 |
|-|-|
| CPU | 具有 64 MiB L3 快取的 64 個虛擬核心 |
| 記憶體 | 128 GiB |
| 磁碟 | 具有電池供電式快取記憶體 RAID 10 的 SAS 磁碟 |
| 網路 | 1 Gbps 的網路 |
| 工作負載 | 一般用途檔案伺服器|

| 初始一次性佈建  | 詳細資料 |
|-|-|
| 物件數目 | 2500 萬個物件 |
| 資料集大小| ~ 4.7 TiB |
| 平均檔案大小 | ~ 200 KiB (最大的檔案： 100 GiB)  |
| 初始雲端變更列舉 | 每秒 20 個物件  |
| 上傳輸送量 | 每個同步處理群組每秒20個物件 |
| 命名空間下載輸送量 | 每秒 400 個物件 |

### <a name="initial-one-time-provisioning"></a>初始一次性佈建

**初始雲端變更列舉**：建立新的同步群組時，初始雲端變更列舉是將執行的第一個步驟。 在此程式中，系統會列舉 Azure 檔案共用中的所有專案。 在此過程中，將不會有任何同步活動，也就是沒有任何專案會從雲端端點下載到伺服器端點，而且不會從伺服器端點上傳任何專案到雲端端點。 完成初次雲端變更列舉之後，將會繼續同步活動。
效能的速率是每秒20個物件。 客戶可以透過判斷雲端共用中的專案數，並使用下列 homebrew 公式來取得時間（以天為單位），以預估完成初始雲端變更列舉所需要的時間。 

   **初始雲端列舉的時間 (天) = 雲端端點中的物件 (數目) / (20 * 60 * 60 * 24)**

**命名空間下載輸送量** 當新的伺服器端點新增至現有的同步處理群組時，Azure 檔案同步代理程式不會從雲端端點下載任何檔案內容。 它會先同步完整命名空間，然後再觸發背景回復以下載檔案；有可能是下載完整檔案，或者，如果已啟用雲端分層處理，則會根據伺服器端點上設定的雲端分層處理原則進行下載。


| 持續同步  | 詳細資料  |
|-|--|
| 已同步的物件數目| 125,000 個物件 (~1% 變換) |
| 資料集大小| 50 GiB |
| 平均檔案大小 | ~500 KiB |
| 上傳輸送量 | 每個同步處理群組每秒20個物件 |
| 完整下載輸送量* | 每秒 60 個物件 |

*如果雲端分層處理已啟用，您應該會發現效能有所提升，因為只會下載部分檔案資料。 只有在任何端點上的快取檔案資料有所變更時，Azure 檔案同步才會下載這些資料。 對於任何分層或新建的檔案，代理程式並不會下載檔案資料，而只會將命名空間同步至所有伺服器端點。 代理程式也支援在使用者存取分層的檔案時進行檔案的部分下載。 

> [!Note]  
> 上述數字不代表您將實際體驗到的效能。 如本節開頭所述，實際效能將取決於多項因素。

以下提供部署的一般指南，有幾件事您應謹記在心：

- 物件輸送量的消長大致上會與伺服器上的同步群組數目成正比。 在伺服器上將資料分割到多個同步群組時，會產生較佳的輸送量，但仍受限於伺服器和網路。
- 物件輸送量與每秒 MiB 輸送量成反比。 檔案較小時，在每秒處理的物件數方面會呈現較高的輸送量，但每秒的 MiB 輸送量則會降低。 相反地，若檔案較大，每秒處理的物件數將會降低，但每秒的 MiB 輸送量則會提高。 每秒的 MiB 輸送量會受限於 Azure 檔案擴展目標。

## <a name="see-also"></a>另請參閱

- [規劃 Azure 檔案服務部署](storage-files-planning.md) (機器翻譯)
- [規劃 Azure 檔案同步部署](storage-sync-files-planning.md)
