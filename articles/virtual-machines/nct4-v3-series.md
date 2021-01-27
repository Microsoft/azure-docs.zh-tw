---
title: NCas T4 v3 系列
description: NCas T4 v3 系列 Vm 的規格。
services: virtual-machines
ms.subservice: sizes
author: vikancha-MSFT
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: vikancha
ms.openlocfilehash: 5f31148a811ac1a7789cb81d744b46b847105d5c
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98917398"
---
# <a name="ncast4_v3-series"></a>NCasT4_v3 系列 

NCasT4_v3 系列虛擬機器支援 [Nvidia Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) GPU 和 AMD EPYC 7V12 (羅馬) cpu。 Vm 最多可以有4個 NVIDIA T4 Gpu （每個都有 16 GB 的記憶體），最多64個非多執行緒 AMD EPYC 7V12 (羅馬) 處理器核心和 440 GiB 的系統記憶體。 這些虛擬機器適用于部署 AI 服務，例如即時推斷使用者產生的要求，或使用 NVIDIA 的方格驅動程式和虛擬 GPU 技術的互動式圖形和視覺效果工作負載。 以 CUDA、TensorRT、Caffe、ONNX 和其他架構為基礎的標準 GPU 計算工作負載，或是以 GPU 加速圖形應用程式為基礎的 OpenGL 和 DirectX，可在 NCasT4_v3 系列上以經濟實惠的方式進行部署，並接近使用者的距離。

<br>

[ACU](acu.md)：230-260<br>
[進階儲存體](premium-storage-performance.md)：支援<br>
[進階儲存體](premium-storage-performance.md)快取：支援<br>
[即時移轉](maintenance-and-updates.md)：不支援<br>
[記憶體保留更新](maintenance-and-updates.md)：不支援<br>
[VM 世代支援](generation-2.md)：第1代和第2代<br>
[加速網路](../virtual-network/create-vm-accelerated-networking-cli.md)：支援<br>
Nvidia NVLink 互連：不支援<br>
<br>

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | GPU 記憶體：GiB | 最大資料磁碟 | 最大 NIC/預期的網路頻寬 (Mbps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_T4_v3 |4 |28 |180 | 1 | 16 | 8 | 2 / 8000 |
| Standard_NC8as_T4_v3 |8 |56 |360 | 1 | 16 | 16 | 4 / 8000  |
| Standard_NC16as_T4_v3 |16 |110 |360 | 1 | 16 | 32 | 8 / 8000  |
| Standard_NC64as_T4_v3 |64 |440 |2880 | 4 | 64 | 32 | 8 / 32000  |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>支援的作業系統和驅動程式

若要利用 Azure NCasT4_v3 系列 Vm （執行 Windows 或 Linux）的 GPU 功能，必須安裝 Nvidia GPU 驅動程式。

若要手動安裝 Nvidia GPU 驅動程式，請參閱適用于 [Windows 的 N 系列 GPU 驅動程式設定](./windows/n-series-driver-setup.md) ，以取得支援的作業系統、驅動程式、安裝和驗證步驟。

## <a name="other-sizes"></a>其他大小

- [一般用途](sizes-general.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [GPU 最佳化](sizes-gpu.md)
- [高效能計算](sizes-hpc.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟

深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。
