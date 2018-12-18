---
title: 從 AWS 和其他平台移轉至 Azure 中的受控磁碟 | Microsoft Docs
description: 在 Azure 中使用從其他雲端 (如 AWS) 或其他虛擬化平台上傳的 VHD 建立 VM，並充分利用 Azure 受控磁碟。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/07/2017
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0440eccbdf76fc58fadf46de2508d362c0de6cc3
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32190780"
---
# <a name="migrate-from-amazon-web-services-aws-and-other-platforms-to-managed-disks-in-azure"></a>從 Amazon Web Services (AWS) 和其他平台移轉至 Azure 中的受控磁碟

您可以將 VHD 檔案從 AWS 或內部部署虛擬化解決方案上傳至 Azure，以建立利用受控磁碟的 VM。 使用 Azure 受控磁碟就不需要為 Azure IaaS VM 管理儲存體帳戶。 您只需要指定類型 (進階或標準)，還有您需要的磁碟大小，Azure 就會替您建立並管理磁碟。 

您可以上傳一般化和特製化 VHD。 
- **一般化 VHD** - 已使用 Sysprep 將您所有的個人帳戶資訊移除。 
- **特製化 VHD**：會維護來自原始 VM 的使用者帳戶、應用程式和其他狀態資料。 

> [!IMPORTANT]
> 將任何 VHD 上傳至 Azure 之前，您應該遵循[準備 Windows VHD 或 VHDX 以上傳至 Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
>
>


| 案例                                                                                                                         | 文件                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| 您具有想要使用受控磁碟移轉至 Azure VM 的 AWS EC2 執行個體                              | [從 Amazon Web Services (AWS) 將 VM 移至 Azure](aws-to-azure.md)                           |
| 您具有來自其他虛擬化平台的 VM，且想要將它作為映像使用，來建立多個 Azure VM。 | [將一般化 VHD 上傳，並使用它在 Azure 中建立新的 VM](upload-generalized-managed.md) |
| 您想要在 Azure 中重新建立唯一自訂的 VM。                                                      | [將特製化 VHD 上傳至 Azure，並建立新的 VM](create-vm-specialized.md)         |


## <a name="overview-of-managed-disks"></a>受控磁碟概觀

Azure 受控磁碟可免除管理儲存體帳戶的需求，進而簡化 VM 管理。 受控磁碟也受惠於可用性設定組中更佳的 VM 可靠性。 它可確保可用性設定組中不同 VM 的磁碟會確實地和彼此隔離，以避免發生單一失敗點。 它會自動以不同的儲存體縮放單位 (戳記) 將不同 VM 的磁碟放在一個可用性設定組中，以限制硬體和軟體失敗所引起之單一儲存體縮放單位失敗的影響。 根據您的需求，您可選擇兩種類型的儲存體選項︰ 
 
- [進階受控磁碟](premium-storage.md)是固態硬碟 (SSD) 式儲存媒體，可針對執行大量 I/O 工作負載的虛擬機器，提供高效能、低延遲的磁碟支援。 您可以將這類磁碟移轉至進階受控磁碟，以利用這類磁碟的速度和效能。  

- [標準受控磁碟](standard-storage.md)使用固態硬碟 (SSD) 式儲存媒體，最適合用於開發/測試及其他較不容易受效能變異影響的不常用工作負載。  

## <a name="plan-for-the-migration-to-managed-disks"></a>規劃移轉至受控磁碟

本節可協助您做出最佳的 VM 和磁碟類型決策。

如果您打算從非受控磁碟移轉至受控磁碟，您應注意到具有[虛擬機器參與者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)角色的使用者將無法變更 VM 大小 (因為它們可能為轉換前)。 這是因為具有受控磁碟的 VM 需要使用者具有 OS 磁碟上的 Microsoft.Compute/disks/write 權限。

### <a name="location"></a>位置

挑選 Azure 受控磁碟可用的位置。 如果您要移轉至進階受控磁碟，也請確保進階儲存體可用於您打算移轉至的區域。 如需可使用 Azure 服務之地點的最新資訊，請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services)。

### <a name="vm-sizes"></a>VM 大小

如果您要移轉至進階受控磁碟，您必須將 VM 大小更新為 VM 所在區域中進階儲存體可支援的大小。 檢閱進階儲存體可支援的 VM 大小。 Azure VM 大小的規格已列在 [虛擬機器的大小](sizes.md)一文中。
請檢閱使用於進階儲存體的虛擬機器效能特性，然後選擇最適合您的工作負載的 VM 大小。 確定 VM 上有足夠的磁碟流量頻寬。

### <a name="disk-sizes"></a>磁碟大小

**進階受控磁碟**

有三種類型的進階受控磁碟可以搭配您的 VM 使用，而且每種都有特定的 IOP 和輸送量限制。 為您的 VM 選擇進階磁碟類型時，請根據應用程式在容量、效能、延展性以及尖峰負載方面的需求，將這些限制納入考量。

| 進階磁碟類型  | P10               | P20               | P30               |
|---------------------|-------------------|-------------------|-------------------|
| 磁碟大小           | 128 GB            | 512 GB            | 1024 GB (1 TB)    |
| 每一磁碟的 IOPS       | 500               | 2300              | 5000              |
| 每一磁碟的輸送量 | 每秒 100 MB | 每秒 150 MB | 每秒 200 MB |

**標準受控磁碟**

有五種類型的標準受控磁碟可搭配您的 VM 使用。 每種類型的容量各不相同，但其 IOPS 和輸送量限制相同。 根據您應用程式的容量需求，選擇標準受控磁碟的類型。

| 標準磁碟類型  | S4               | S6               | S10              | S20              | S30              |
|---------------------|------------------|------------------|------------------|------------------|------------------|
| 磁碟大小           | 30 GB            | 64 GB            | 128 GB           | 512 GB           | 1024 GB (1 TB)   |
| 每一磁碟的 IOPS       | 500              | 500              | 500              | 500              | 500              |
| 每一磁碟的輸送量 | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB |

### <a name="disk-caching-policy"></a>磁碟快取原則 

**進階受控磁碟**

根據預設，所有 Premium 資料磁碟的磁碟快取原則都是*唯讀*，而連接至 VM 的 Premium 作業系統磁碟的磁碟快取原則則是*讀寫*。 為使應用程式的 IO 達到最佳效能，建議使用此組態設定。 對於頻繁寫入或唯寫的資料磁碟 (例如 SQL Server 記錄檔)，停用磁碟快取可獲得更佳的應用程式效能。

### <a name="pricing"></a>價格

請檢閱[受控磁碟的價格](https://azure.microsoft.com/pricing/details/managed-disks/)。 進階受控磁碟與進階非受控磁碟的價格相同。 但標準受控磁碟與標準非受控磁碟的價格不同。


## <a name="next-steps"></a>後續步驟

- 將任何 VHD 上傳至 Azure 之前，您應該遵循[準備 Windows VHD 或 VHDX 以上傳至 Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
