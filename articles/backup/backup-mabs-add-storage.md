---
title: 在 Azure 備份伺服器 v2 中使用新式備份儲存體
description: 了解 Azure 備份伺服器 v2 中的新功能。 本文說明如何升級您的備份伺服器安裝。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 05/15/2017
ms.author: markgal
ms.openlocfilehash: 7c583ea048ed1837c662869c62039165aaa3c024
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606749"
---
# <a name="add-storage-to-azure-backup-server-v2"></a>在 Azure 備份伺服器 v2 中新增儲存體

Azure 備份伺服器 v2 隨附 System Center 2016 Data Protection Manager 新式備份儲存體。 新式備份儲存體可節省 50% 的儲存空間、備份速度快三倍，且更具儲存效率。 它也提供可感知工作負載的儲存體。 

> [!NOTE]
> 若要使用新式備份儲存體，您必須在 Windows Server 2016 上執行備份伺服器 v2。 如果您在舊版 Windows Server 上執行備份伺服器 v2，Azure 備份伺服器將無法利用新式備份儲存體。 相反地，它保護工作負載的方式會和備份伺服器 v1 一樣。 如需詳細資訊，請參閱備份伺服器版本[保護對照表](backup-mabs-protection-matrix.md)。

## <a name="volumes-in-backup-server-v2"></a>備份伺服器 v2 中的磁碟區

備份伺服器 v2 接受儲存體磁碟區。 當您新增磁碟區時，備份伺服器會將磁碟區格式化為新式備份儲存體所需要的復原檔案系統 (ReFS)。 若要新增磁碟區，並於稍後需要時加以擴充，建議您使用此工作流程：

1.  在 VM 上設定備份伺服器 v2。
2.  在儲存集區的虛擬磁碟上建立磁碟區：
    1.  在儲存集區中新增磁碟，並建立簡單配置的虛擬磁碟。
    2.  新增其他磁碟，並擴充虛擬磁碟。
    3.  在虛擬磁碟上建立磁碟區。
3.  在備份伺服器中新增磁碟區。
4.  設定可感知工作負載的儲存體。

## <a name="create-a-volume-for-modern-backup-storage"></a>為新式備份儲存體建立磁碟區

以磁碟區作為磁碟儲存體來使用備份伺服器 v2 可協助您掌控儲存體。 磁碟區可以是單一磁碟。 不過，如果您日後想要擴充儲存體，請從使用儲存體空間所建立的磁碟中建立磁碟區。 如果您想要擴充磁碟區以供儲存備份，這麼做會有所幫助。 本節會提供最佳做法，讓您了解如何建立具有此設定的磁碟區。

1. 在 [伺服器管理員] 中，選取 [檔案和存放服務] > [磁碟區] > [儲存集區]。 在 [實體磁碟] 底下，選取 [新增儲存集區]。 

    ![建立新的儲存集區](./media/backup-mabs-add-storage/mabs-add-storage-1.png)

2. 在 [工作] 下拉式方塊中，選取 [新增虛擬磁碟]。

    ![新增虛擬磁碟](./media/backup-mabs-add-storage/mabs-add-storage-2.png)

3. 選取儲存集區，然後選取 [新增實體磁碟]。

    ![新增實體磁碟](./media/backup-mabs-add-storage/mabs-add-storage-3.png)

4. 選取實體磁碟，然後選取 [擴充虛擬磁碟]。

    ![擴充虛擬磁碟](./media/backup-mabs-add-storage/mabs-add-storage-4.png)

5. 選取虛擬磁碟，然後選取 [新增磁碟區]。

    ![建立新的磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-5.png)

6. 在 [選取伺服器和磁碟] 對話方塊中，選取伺服器和新的磁碟。 然後，選取 [下一步]。

    ![選取伺服器和磁碟](./media/backup-mabs-add-storage/mabs-add-storage-6.png)

## <a name="add-volumes-to-backup-server-disk-storage"></a>在備份伺服器磁碟儲存體中新增磁碟區

若要在備份伺服器中新增磁碟區，請於 [管理] 窗格重新掃描儲存體，然後選取 [新增]。 隨即會出現可供為備份伺服器儲存體新增的所有磁碟區清單。 在可用的磁碟區新增到已選取的磁碟區清單後，您可以為他們提供易記名稱，以方便您管理這些磁碟區。 若要將這些磁碟區格式化為 ReFS，讓備份伺服器可以利用新式備份儲存體的好處，請選取 [確定]。

![新增可用的磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-7.png)

## <a name="set-up-workload-aware-storage"></a>設定可感知工作負載的儲存體

在可感知工作負載的儲存體中，您可以選取會優先儲存某些工作負載類型的磁碟區。 例如，您可以將支援大量每秒輸入/輸出作業 (IOPS) 的高度耗費資源磁碟區，設定為僅儲存需要頻繁且大量備份的工作負載。 舉例來說，具有交易記錄的 SQL Server。 備份頻率不高的其他工作負載 (像是 VM)，則可備份到低成本磁碟區。

### <a name="update-dpmdiskstorage"></a>Update-DPMDiskStorage

您可以使用 PowerShell Cmdlet Update-DPMDiskStorage 來設定可感知工作負載的儲存體，此 Cmdlet 會更新 Data Protection Manager 伺服器儲存集區的磁碟區屬性。

語法：

`Parameter Set: Volume`

```
Update-DPMDiskStorage [-Volume] <Volume> [[-FriendlyName] <String> ] [[-DatasourceType] <VolumeTag[]> ] [-Confirm] [-WhatIf] [ <CommonParameters>]
```
下列螢幕擷取畫面會顯示 PowerShell 視窗中的 Update-DPMDiskStorage Cmdlet。

![PowerShell 視窗中的 Update-DPMDiskStorage 命令](./media/backup-mabs-add-storage/mabs-add-storage-8.png)

您使用 PowerShell 所進行的變更會反映在備份伺服器管理員主控台中。

![管理員主控台中的磁碟和磁碟區](./media/backup-mabs-add-storage/mabs-add-storage-9.png)

## <a name="next-steps"></a>後續步驟
在安裝備份伺服器之後，請了解如何準備您的伺服器或開始保護工作負載。

- [準備備份伺服器工作負載](backup-azure-microsoft-azure-backup.md)
- [使用備份伺服器來備份 VMware 伺服器](backup-azure-backup-server-vmware.md)
- [使用備份伺服器來備份 SQL Server](backup-azure-sql-mabs.md)

