---
title: 將 Azure VM 移轉至受控磁碟 | Microsoft Docs
description: 將使用儲存體帳戶中非受控磁碟建立的 Azure 虛擬機器移轉成使用受控磁碟。
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
ms.date: 01/03/2018
ms.author: cynthn
ms.openlocfilehash: d280ad1180949167bb8ebfc6b21521736db0f55d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777139"
---
# <a name="migrate-azure-vms-to-managed-disks-in-azure"></a>將 Azure VM 移轉至 Azure 中的受控磁碟

Azure 受控磁碟可免除個別管理儲存體帳戶的需求，進而簡化儲存體管理。  您也可以將現有的 Azure VM 移轉至受控磁碟，以便受惠於可用性設定組中更佳的 VM 可靠性。 它可確保可用性設定組中不同 VM 的磁碟完全彼此隔離，以避免單一失敗點。 它會自動以不同的儲存體縮放單位 (戳記) 將不同 VM 的磁碟放在一個可用性設定組中，以限制硬體和軟體失敗所引起之單一儲存體縮放單位失敗的影響。
根據您的需求，您可選擇兩種類型的儲存體選項︰

- [進階受控磁碟](premium-storage.md)是固態硬碟 (SSD) 式儲存媒體，可針對執行大量 I/O 工作負載的虛擬機器，提供高效能、低延遲的磁碟支援。 您可以將這類磁碟移轉至進階受控磁碟，以利用這類磁碟的速度和效能。

- [標準受控磁碟](standard-storage.md)使用固態硬碟 (SSD) 式儲存媒體，最適合用於開發/測試及其他較不容易受效能變異影響的不常用工作負載。

您可以在下列案例中移轉到受控磁碟︰

| 移轉...                                            | 文件連結                                                                                                                                                                                                                                                                  |
|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 將獨立 VM 和可用性設定組中的 VM 轉換成受控磁碟   | [將 VM 轉換成使用受控磁碟](convert-unmanaged-to-managed-disks.md) |
| 將單一 VM 由受控磁碟上的傳統部署移轉到 Resource Manager 部署     | [移轉單一 VM](migrate-single-classic-to-resource-manager.md)  | 
| 將 vNet 中的所有 VM 由受控磁碟上的傳統部署移轉到 Resource Manager 部署     | [將 IaaS 資源從傳統部署移轉至 Resource Manager 部署](migration-classic-resource-manager-ps.md)，然後[將 VM 從非受控磁碟轉換為受控磁碟](convert-unmanaged-to-managed-disks.md) | 






## <a name="plan-for-the-conversion-to-managed-disks"></a>規劃轉換為受控磁碟

本節可協助您做出最佳的 VM 和磁碟類型決策。


## <a name="location"></a>位置

挑選 Azure 受控磁碟可用的位置。 如果您要移至進階受控磁碟，也請確保進階儲存體可用於您打算移至的區域。 如需可使用 Azure 服務之地點的最新資訊，請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services)。

## <a name="vm-sizes"></a>VM 大小

如果您要移轉至進階受控磁碟，您必須將 VM 大小更新為 VM 所在區域中進階儲存體可支援的大小。 檢閱進階儲存體可支援的 VM 大小。 Azure VM 大小的規格已列在 [虛擬機器的大小](sizes.md)一文中。
請檢閱使用於進階儲存體的虛擬機器效能特性，然後選擇最適合您的工作負載的 VM 大小。 確定 VM 上有足夠的磁碟流量頻寬。

## <a name="disk-sizes"></a>磁碟大小

**進階受控磁碟**

有七種型別的進階受控磁碟可以搭配 VM 使用，而且每種都有特定的 IOP 和輸送量限制。 為您的 VM 選擇進階磁碟類型時，請根據應用程式在容量、效能、延展性以及尖峰負載方面的需求，將這些限制納入考量。

| 進階磁碟類型  | P4    | P6    | P10   | P15   | P20   | P30   | P40   | P50   | 
|---------------------|-------|-------|-------|-------|-------|-------|-------|-------|
| 磁碟大小           | 32 GB| 64 GB| 128 GB| 256 GB|512 GB | 1024 GB (1 TB)    | 2048 GB (2 TB)    | 4095 GB (4 TB)    | 
| 每一磁碟的 IOPS       | 120   | 240   | 500   | 1100  |2300              | 5000              | 7500              | 7500              | 
| 每一磁碟的輸送量 | 每秒 25 MB  | 每秒 50 MB  | 每秒 100 MB | 每秒 125 MB |每秒 150 MB | 每秒 200 MB | 每秒 250 MB | 每秒 250 MB |

**標準受控磁碟**

有七種型別的標準受控磁碟可搭配 VM 使用。 每種類型的容量各不相同，但其 IOPS 和輸送量限制相同。 根據您應用程式的容量需求，選擇標準受控磁碟的類型。

| 標準磁碟類型  | S4               | S6               | S10              | S15              | S20              | S30              | S40              | S50              | 
|---------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------| 
| 磁碟大小           | 30 GB            | 64 GB            | 128 GB           | 256 GB           |512 GB           | 1024 GB (1 TB)   | 2048 GB (2TB)    | 4095 GB (4 TB)   | 
| 每一磁碟的 IOPS       | 500              | 500              | 500              | 500              |500              | 500              | 500             | 500              | 
| 每一磁碟的輸送量 | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB |每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 每秒 60 MB | 

## <a name="disk-caching-policy"></a>磁碟快取原則

**進階受控磁碟**

根據預設，所有 Premium 資料磁碟的磁碟快取原則都是*唯讀*，而連接至 VM 的 Premium 作業系統磁碟的磁碟快取原則則是*讀寫*。 為使應用程式的 IO 達到最佳效能，建議使用此組態設定。 對於頻繁寫入或唯寫的資料磁碟 (例如 SQL Server 記錄檔)，停用磁碟快取可獲得更佳的應用程式效能。

## <a name="pricing"></a>價格

請檢閱[受控磁碟的價格](https://azure.microsoft.com/pricing/details/managed-disks/)。 進階受控磁碟與進階非受控磁碟的價格相同。 但標準受控磁碟與標準非受控磁碟的價格不同。



## <a name="next-steps"></a>後續步驟

- 深入了解[受控磁碟](managed-disks-overview.md)
