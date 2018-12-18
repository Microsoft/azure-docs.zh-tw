---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/09/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 3dfc72ff0347a93c6c6dce0e7ec763dd8241c55b
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/16/2018
ms.locfileid: "29958804"
---
## <a name="azure-backup"></a>Azure 備份

若要備份執行生產工作負載的 Azure VM，請使用 Azure 備份。 Azure 備份同時針對 Windows 和 Linux VM 支援應用程式一致備份。 Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或只還原特定檔案。 

如需 Azure VM 適用的簡單實際操作 Azure 備份簡介，請參閱適用於 [Linux](../articles/virtual-machines/linux/tutorial-backup-vms.md) 或 [Windows](../articles/virtual-machines/windows/tutorial-backup-vms.md) 的「備份 Azure 虛擬機器」教學課程。

如需有關 Azure 備份運作方式的詳細資訊，請參閱[在 Azure 中規劃您的 VM 備份基礎結構](../articles/backup/backup-azure-vms-introduction.md)


## <a name="azure-site-recovery"></a>Azure Site Recovery

當整個地區因重大天災或大規模服務中斷而發生中斷時，Azure Site Recovery 可保護您的 VM 免於遭受重大災害情節。 您可以為 VM 設定 Azure Site Recovery，如此一來，只要按一下花幾分鐘就能復原應用程式。 您可以複寫至所選擇的 Azure 區域，而不限於配對的區域。 

您可藉由隨選測試容錯移轉來執行災害復原演練，完全不影響生產工作負載或進行中的複寫。 建立復原計劃，對於在多個 VM 上執行的整個應用程式協調容錯移轉和容錯回復。 復原計劃功能會與 Azure 自動化 Runbook 整合。

您可以[複寫虛擬機器](https://aka.ms/a2a-getting-started)來開始進行。 

## <a name="managed-snapshots"></a>受控快照集 

在開發與測試環境中，快照集會提供快速簡單的選項來備份使用受控磁碟的 VM。 受控快照集是受控磁碟的唯讀完整複本。 快照集可在來源磁碟外獨立存在，還能用來建立新的受控磁碟以重建 VM。 它們是以使用的磁碟部分作為基礎來計費。 例如，如果建立佈建容量為 64 GB 的受控磁碟快照集，而實際使用資料大小為 10 GB，則只會對已使用的 10 GB 資料大小收取快照集費用。  

如需有關如何建立快照集的詳細資訊，請參閱：

* [在 Windows 中建立 VHD 複本並儲存為受控磁碟](../articles/virtual-machines/windows/snapshot-copy-managed-disk.md)
* [在 Linux 中建立 VHD 複本並儲存為受控磁碟](../articles/virtual-machines/linux/snapshot-copy-managed-disk.md)



## <a name="next-steps"></a>後續步驟
您可以試用 Azure 備份，方法是遵循適用於 [Linux](../articles/virtual-machines/linux/tutorial-backup-vms.md) 或 [Windows](../articles/virtual-machines/windows/tutorial-backup-vms.md) 的「備份 Windows 虛擬機器教學課程」。
