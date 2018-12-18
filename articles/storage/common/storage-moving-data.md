---
title: 從 Azure 中的雲端儲存體來回移動大量資料 | Microsoft Docs
description: 從 Azure 儲存體來回移動資料之不同方法的概觀。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/26/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 1df237a65a8b5312b20de19a99399b3a3dd075ff
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049605"
---
# <a name="moving-data-to-and-from-azure-storage"></a>從 Azure 儲存體來回移動資料
如果您想要將內部部署資料移至 Azure 儲存體 (或相反)，有各種不同的方法可以辦到。 最適合的方法將取決於您的案例。 本文將提供不同案例的快速概觀和每個案例的適當供應項目。

## <a name="building-applications"></a>建置應用程式
如果您要建置應用程式，針對 REST API 或許多用戶端程式庫進行開發是從 Azure 儲存體來回移動資料的好方法。

Azure 儲存體為許多熱門語言提供豐富的用戶端程式庫，這些語言包含 .NET、Java、Android、Go、Xamarin、C++、Node.JS、PHP、Ruby、Python 和 iOS。 這些用戶端程式庫提供多種進階功能，例如大重試邏輯、記錄與並行上傳等等。 您也可以直接透過 REST API 開發，它可以透過提出 HTTP/HTTPS 要求的任何語言進行呼叫。

若要深入了解，請參閱 [開始使用 Azure Blob 儲存體](../blobs/storage-dotnet-how-to-use-blobs.md) 。

此外，我們也提供 [Azure 儲存體資料移動程式庫](https://www.nuget.org/packages/Microsoft.Azure.Storage.DataMovement) ，這是專為在 Azure 中高效能來回複製資料所設計。 若要深入了解，請參閱我們的資料移動程式庫 [文件](https://github.com/Azure/azure-storage-net-data-movement) 。 

## <a name="quickly-viewinginteracting-with-your-data"></a>快速檢視/和資料互動
如果您想要輕鬆地檢視 Azure 儲存體資料，同時有上傳和下載資料的能力，請考慮使用 Azure 儲存體總管。

若要深入了解，請參閱我們的 [Azure 儲存體總管](../storage-explorers.md) 清單。

## <a name="system-administration"></a>系統管理
如果您需要或比較熟悉命令列公用程式 (例如系統管理員)，以下是您應考慮的幾個選項︰

### <a name="azcopy"></a>AzCopy
AzCopy 是命令列公用程式，可以極高效能將資料複製到 Azure 儲存體，以及從 Azure 儲存體複製資料。 您也可以複製儲存體帳戶內或不同儲存體帳戶之間的資料。 AzCopy 可於 [Windows](storage-use-azcopy.md) 和 [Linux](storage-use-azcopy-linux.md) 中取得。

若要深入了解，請參閱[使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)或[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

### <a name="azure-powershell"></a>Azure PowerShell
Azure PowerShell 模組提供 Cmdlet 讓您用於管理 Azure 上的服務。 此模組為工作型的命令列 Shell 與指令碼語言，專為系統管理所設計。

若要深入了解，請參閱 [搭配使用 Azure PowerShell 與 Azure 儲存體](storage-powershell-guide-full.md) 。

### <a name="azure-cli"></a>Azure CLI
Azure CLI 提供您一組開放原始碼的跨平台命令，供您使用 Azure 服務。 Azure CLI 可在 Windows、OSX 和 Linux 上使用。

若要深入了解，請參閱 [使用 Azure CLI 搭配 Azure 儲存體](../storage-azure-cli.md) 。

## <a name="moving-large-amounts-of-data-with-a-slow-network"></a>以較慢的網路移動大量資料
與移動大量資料相關聯的最大挑戰之一是傳輸時間。 如果您想要從 Azure 儲存體來回取得資料，而不需擔心網路成本或撰寫程式碼，Azure 匯入/匯出會是適當的解決方案。

若要深入了解，請參閱 [Azure 匯入/匯出](../storage-import-export-service.md) 。

## <a name="backing-up-your-data"></a>備份您的資料
如果您只需要將資料備份到 Azure 儲存體，Azure 備份是最好的選擇。 對於備份內部部署資料和 Azure VM，這是功能強大的解決方案。

若要深入了解，請參閱 [Azure 備份](../../backup/backup-introduction-to-azure-backup.md) 。

## <a name="accessing-your-data-on-premises-and-from-the-cloud"></a>存取內部部署資料和雲端的資料
如果您需要解決方案來存取內部部署資料和雲端的資料，您應該考慮使用 Azure 的混合式雲端儲存體解決方案，StorSimple。 此解決方案包含實體 StorSimple 裝置，以智慧方式將經常使用的資料儲存在 SSD 上，將偶爾使用的資料儲存在 HDD 上，並且將非作用中/備份/封存資料儲存在 Azure 儲存體上。

若要深入了解，請參閱 [StorSimple](../../storsimple/storsimple-overview.md) 。

## <a name="recovering-your-data"></a>復原您的資料
當您擁有內部部署工作負載和應用程式，您需要可讓您在發生災害事件時繼續執行業務的解決方案。 Azure Site Recovery 可處理虛擬機器和實體伺服器的複寫、容錯移轉及復原作業。 複寫的資料會儲存在 Azure 儲存體，讓您不需要次要本地資料中心。

若要深入了解，請參閱 [Azure Site Recovery](../../site-recovery/site-recovery-overview.md) 。
### <a name="moving-data-faq"></a>移動資料常見問題集︰
## <a name="can-i-migrate-vhds-from-one-region-to-another-without-copying"></a>我可以在兩個區域之間移轉 VHD 而不要複製嗎？
區域之間複製 VHD 的唯一方法是在每個區域的儲存體帳戶之間複製資料。 您可以使用 AZCopy 這樣做。 若要深入了解，請參閱「使用 AzCopy 命令列公用程式傳輸資料」。 針對極大的資料量，您也可以使用 Azure 匯入/匯出。 若要深入了解，請參閱 [Azure 匯入/匯出](https://docs.microsoft.com/azure/storage/storage-import-export-service) 。
