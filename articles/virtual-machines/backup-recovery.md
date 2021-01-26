---
title: 總覽 Vm 的備份選項
description: 概述 Azure 虛擬機器的備份選項。
author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 8/03/2020
ms.author: cynthn
ms.openlocfilehash: 5a093de0a27c8379cb6eff9c2bc3867dfdc20db5
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98787802"
---
# <a name="backup-and-restore-options-for-virtual-machines-in-azure"></a>Azure 中虛擬機器的備份和還原選項

您可以定期建立備份以保護您的資料。 可供 VM 使用的備份選項有數個，視您的使用案例而定。

## <a name="azure-backup"></a>Azure 備份

若要備份執行生產工作負載的 Azure VM，請使用 Azure 備份。 Azure 備份同時針對 Windows 和 Linux VM 支援應用程式一致備份。 Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或只還原特定檔案。 

如需 Azure Vm Azure 備份的簡單實際操作簡介，請參閱 [Azure 備份快速入門](../backup/quick-backup-vm-portal.md)。

如需有關 Azure 備份運作方式的詳細資訊，請參閱[在 Azure 中規劃您的 VM 備份基礎結構](../backup/backup-azure-vms-introduction.md)


## <a name="azure-site-recovery"></a>Azure Site Recovery

當整個地區因重大天災或大規模服務中斷而發生中斷時，Azure Site Recovery 可保護您的 VM 免於遭受重大災害情節。 您可以為 VM 設定 Azure Site Recovery，如此一來，只要按一下花幾分鐘就能復原應用程式。 您可以複寫至所選擇的 Azure 區域，而不限於配對的區域。 

您可藉由隨選測試容錯移轉來執行災害復原演練，完全不影響生產工作負載或進行中的複寫。 建立復原計劃，對於在多個 VM 上執行的整個應用程式協調容錯移轉和容錯回復。 復原計劃功能會與 Azure 自動化 Runbook 整合。

您可以[複寫虛擬機器](../site-recovery/azure-to-azure-quickstart.md)來開始進行。 

## <a name="managed-snapshots"></a>受控快照集 

在開發與測試環境中，快照集會提供快速簡單的選項來備份使用受控磁碟的 VM。 受控快照集是受控磁碟的唯讀完整複本。 快照集可在來源磁碟外獨立存在，還能用來建立新的受控磁碟以重建 VM。 它們是以使用的磁碟部分作為基礎來計費。 例如，如果建立佈建容量為 64 GB 的受控磁碟快照集，而實際使用資料大小為 10 GB，則只會對已使用的 10 GB 資料大小收取快照集費用。  

如需有關如何建立快照集的詳細資訊，請參閱：

* [在 Windows 中建立 VHD 複本並儲存為受控磁碟](./windows/snapshot-copy-managed-disk.md)
* [使用 Linux 中的快照集建立 VHD 的複本並儲存為受控磁片](./linux/snapshot-copy-managed-disk.md)



## <a name="next-steps"></a>後續步驟
您可以遵循 [Azure 備份快速入門](../backup/quick-backup-vm-portal.md)來嘗試 Azure 備份。