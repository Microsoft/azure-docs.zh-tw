---
title: NC 系列-Azure 虛擬機器
description: NC 系列 Vm 的規格。
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 40eed41dbaa3314c00faebf1c2452aaa6fb46dc9
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98920083"
---
# <a name="nc-series"></a>NC 系列

NC 系列 Vm 是由 [NVIDIA Tesla K80](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-product-literature/Tesla-K80-BoardSpec-07317-001-v05.pdf) 卡和 Intel 2690 E5-v3 所支援， (Haswell) 處理器。 使用者可以藉由將 CUDA 用於能源探勘應用程式、當機模擬、光線追蹤轉譯、深度學習等等，更快速地處理資料。 NC24r 設定提供低延遲且高輸送量網路介面，最適合用於緊密結合的平行計算工作負載。

[進階儲存體](premium-storage-performance.md)：不支援<br>
[進階儲存體](premium-storage-performance.md)快取：不支援<br>
[即時移轉](maintenance-and-updates.md)：不支援<br>
[記憶體保留更新](maintenance-and-updates.md)：不支援<br>
[VM 世代支援](generation-2.md)：第1代<br>
[加速網路](../virtual-network/create-vm-accelerated-networking-cli.md)：不支援<br>
Nvidia NVLink 互連：不支援<br>
<br>

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | GPU 記憶體：GiB | 最大資料磁碟 | 最大 NIC |
|---|---|---|---|---|---|---|---|
| Standard_NC6    | 6  | 56  | 340  | 1 | 12 | 24 | 1 |
| Standard_NC12   | 12 | 112 | 680  | 2 | 24 | 48 | 2 |
| Standard_NC24   | 24 | 224 | 1440 | 4 | 48 | 64 | 4 |
| Standard_NC24r* | 24 | 224 | 1440 | 4 | 48 | 64 | 4 |

1 GPU = 1/2 K80 卡。

*支援 RDMA

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>支援的作業系統和驅動程式

若要利用 Azure N 系列 Vm 的 GPU 功能，必須安裝 NVIDIA GPU 驅動程式。

[NVIDIA GPU 驅動程式擴充功能](./extensions/hpccompute-gpu-windows.md)會在 N 系列 VM 上安裝適當的 NVIDIA CUDA 或 GRID 驅動程式。 使用 Azure 入口網站或者 Azure PowerShell 或 Azure Resource Manager 範本之類的工具，安裝或管理擴充功能。 如需支援的作業系統和部署步驟，請參閱 [NVIDIA GPU 驅動程式擴充功能文件](./extensions/hpccompute-gpu-windows.md)。 如需有關虛擬機器擴充功能的一般資訊，請參閱 [Azure 虛擬機器擴充功能和功能](./extensions/overview.md)。

如果您選擇手動安裝 NVIDIA GPU 驅動程式，請參閱適用于 [Windows 的 n 系列 gpu 驅動程式設定](./windows/n-series-driver-setup.md) ，或適用于 [Linux 的 n 系列 gpu 驅動程式設定](./linux/n-series-driver-setup.md) ，以取得支援的作業系統、驅動程式、安裝和驗證步驟。

## <a name="other-sizes"></a>其他大小

- [一般用途](sizes-general.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [GPU 最佳化](sizes-gpu.md)
- [高效能計算](sizes-hpc.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟

深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。
