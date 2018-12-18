---
title: 關於 Microsoft Azure Windows VM 的非受控 (分頁 Blob) 和受控磁碟儲存體 | Microsoft Docs
description: 深入了解 Azure 中 Windows 虛擬機器的非受控 (分頁 Blob) 和受控磁碟儲存體的基本概念。
services: virtual-machines-windows,storage
author: roygara
ms.service: virtual-machines-windows
ms.tgt_pltfrm: windows
ms.topic: article
ms.date: 11/15/2017
ms.author: rogarana
ms.component: disks
ms.openlocfilehash: 8f5c33a63fd932bedd7f1de3d3ae47306b3ea3e4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954478"
---
# <a name="about-disks-storage-for-azure-windows-vms"></a>關於 Azure Windows VM 的磁碟儲存體

就像任何其他電腦，Azure 中的虛擬機器會使用磁碟做為儲存作業系統、應用程式和資料的位置。 所有 Azure 虛擬機器都至少有兩個磁碟 - 一個 Windows 作業系統磁碟和一個暫存磁碟。 作業系統磁碟是從映像建立，且作業系統磁碟與該映像都是儲存在 Azure 儲存體帳戶中的虛擬硬碟 (VHD)。 虛擬機器也可以有一或多個資料磁碟，而這些磁碟也會儲存成 VHD。 

在本文中，我們將討論磁碟的不同用法，接著討論您可以建立和使用的不同磁碟類型。 本文也適用於 [Linux 虛擬機器](../linux/about-disks-and-vhds.md)。

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="disks-used-by-vms"></a>VM 使用的磁碟

讓我們看看 VM 如何使用磁碟。

### <a name="operating-system-disk"></a>作業系統磁碟

每個虛擬機器都有一個連接的作業系統磁碟。 它會註冊為 SATA 磁碟機，並預設標示為 C: 磁碟機。 此磁碟的最大容量為 2048 GB。

### <a name="temporary-disk"></a>暫存磁碟

每個 VM 都包含一個暫存磁碟。 暫存磁碟為應用程式和處理程序提供短期的儲存空間，且僅供用來儲存分頁檔之類的資料。 暫存磁碟上的資料可能會在[維護事件](manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#understand-vm-reboots---maintenance-vs-downtime)期間或當您[重新佈署 VM](redeploy-to-new-node.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 時遺失。 在 VM 的成功標準重新開機期間，暫存磁碟上的資料將保留。 

暫存磁碟預設會標示為 D: 磁碟機，並用於儲存 pagefile.sys。 若要將此磁碟重新對應至不同的磁碟機代號，請參閱 [變更 Windows 暫存磁碟的磁碟機代號](change-drive-letter.md)。 暫存磁碟的大小會依據虛擬機器的大小而改變。 如需詳細資訊，請參閱 [Windows 虛擬機器的大小](sizes.md)。

如需有關 Azure 如何使用暫存磁碟的詳細資訊，請參閱 [Understanding the temporary drive on Microsoft Azure Virtual Machines (了解 Microsoft Azure 虛擬機器上的暫存磁碟機)](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

### <a name="data-disk"></a>資料磁碟

資料磁碟是連接至虛擬機器的 VHD，用來儲存應用程式資料或其他您需要保留的資料。 資料磁碟註冊為 SCSI 磁碟機，並以您選擇的字母標示。 每個資料磁碟都有 4,095 GB 的容量上限，受控磁碟都有 32,767 TiB 的容量上限。 虛擬機器的大小會決定您可以連接之磁碟的數量，以及您可以用來裝載磁碟的儲存體類型。

> [!NOTE]
> 如需有關虛擬機器容量的詳細資訊，請參閱 [Windows 虛擬機器的大小](sizes.md)。

當您從映像建立虛擬機器時，Azure 會建立作業系統磁碟。 如果您使用包含資料磁碟映像時，Azure 建立虛擬機器時也會建立資料磁碟。 若沒有，則您可以建立虛擬機器後再新增資料磁碟。

您可以隨時將資料磁碟「連接」  到虛擬機器來將該磁碟新增到虛擬機器中。 您可以使用已上傳或複製到儲存體帳戶的 VHD，或是使用 Azure 為您建立的空白 VHD。 附加資料磁碟時，會透過在 VHD 上加上「租約」，來將 VHD 檔案與 VM 關聯，如此一來，當它仍處於附加狀態時，就無法將它從儲存體中刪除。


[!INCLUDE [storage-about-vhds-and-disks-windows-and-linux](../../../includes/storage-about-vhds-and-disks-windows-and-linux.md)]

## <a name="one-last-recommendation-use-trim-with-unmanaged-standard-disks"></a>最後一個建議：對非受控標準磁碟使用 TRIM

如果您使用非受控標準儲存體 (HDD)，您應該啟用 TRIM。 TRIM 會捨棄磁碟上未使用的區塊，因此您只需支付實際使用的儲存體。 如果您建立大型檔案，然後將它們刪除，這便可節省成本。

您可以執行此命令來檢查 TRIM 設定。 在您的 Windows VM 上開啟命令提示字元並輸入︰

```
fsutil behavior query DisableDeleteNotify
```

如果命令傳回 0，則已正確啟用 TRIM。 如果傳回 1，請執行下列命令來啟用 TRIM：

```
fsutil behavior set DisableDeleteNotify 0
```

> [!NOTE]
> 注意：Trim 支援是從 Windows Server 2012 / Windows 8 和更新版本開始，請參閱[新 API 允許應用程式將 "TRIM and Unmap" 提示傳送給儲存媒體](https://msdn.microsoft.com/windows/compatibility/new-api-allows-apps-to-send-trim-and-unmap-hints)。
> 

<!-- Might want to match next-steps from overview of managed disks -->
## <a name="next-steps"></a>後續步驟
* [連接磁碟](attach-disk-portal.md) 來為您的 VM 新增額外的儲存空間。
* [建立快照集](snapshot-copy-managed-disk.md)。
* [轉換為受控磁碟](convert-unmanaged-to-managed-disks.md)。


