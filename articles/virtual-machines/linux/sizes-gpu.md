---
title: Azure Linux VM 大小 - GPU | Microsoft Docs
description: 列出 Azure 中可用的不同 Linux 虛擬機器 GPU 最佳化大小。 列出 vCPU 數目、資料磁碟和 NIC 的相關資訊，以及此服務中各種大小之儲存體輸送量和網路頻寬的相關資訊。
services: virtual-machines-linux
documentationcenter: ''
author: jonbeck7
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/06/2018
ms.author: jonbeck
ms.openlocfilehash: 435dc3a4815d9fe554ccba57a49ed9a772b53529
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37904074"
---
# <a name="gpu-optimized-virtual-machine-sizes"></a>GPU 最佳化的虛擬機器大小

[!INCLUDE [virtual-machines-common-sizes-gpu](../../../includes/virtual-machines-common-sizes-gpu.md)]


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-distributions-and-drivers"></a>支援的散發套件和驅動程式

若要利用 Azure N 系列 VM (執行 Linux) 的 GPU 功能，您必須安裝 NVIDIA GPU 驅動程式。 [NVIDIA GPU 驅動程式擴充功能](../extensions/hpccompute-gpu-linux.md)會在 N 系列 VM 上安裝適當的 NVIDIA CUDA 或 GRID 驅動程式。 使用 Azure 入口網站或者 Azure CLI 或 Azure Resource Manager 範本之類的工具，安裝或管理擴充功能。 如需支援的發佈和部署步驟，請參閱 [NVIDIA GPU 驅動程式擴充功能文件](../extensions/hpccompute-gpu-linux.md)。 如需有關虛擬機器擴充功能的一般資訊，請參閱 [Azure 虛擬機器擴充功能和功能](../extensions/overview.md)。

如果您選擇手動安裝 NVIDIA GPU 驅動程式，請參閱[適用於 Linux 的 N 系列 GPU 驅動程式設定](n-series-driver-setup.md)，以了解支援的發佈、驅動程式，以及安裝和驗證步驟。


[!INCLUDE [virtual-machines-n-series-considerations](../../../includes/virtual-machines-n-series-considerations.md)]

* 您不應該在 Ubuntu NC VM 上安裝 X 伺服器或其他使用 `Nouveau` 驅動程式的系統。 安裝 NVIDIA GPU 驅動程式之前，必須停用 `Nouveau` 驅動程式。  

## <a name="other-sizes"></a>其他大小
- [一般用途](sizes-general.md)
- [計算最佳化](sizes-compute.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [高效能計算](sizes-hpc.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟
深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。