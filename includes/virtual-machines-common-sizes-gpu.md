---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows, virtual-machines-linux
author: dlepow
ms.service: multiple
ms.topic: include
ms.date: 09/10/2018
ms.author: danlep;azcspmt;jonbeck
ms.custom: include file
ms.openlocfilehash: e9f88a018ba0f40d41bf7ad17a74850a9edf8ae6
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47043842"
---
GPU 最佳化的 VM 大小，為搭配單一或多個 NVIDIA GPU 提供的特製化虛擬機器。 這些大小是專門針對計算密集型、圖形密集型及視覺效果的工作負載所設計。 本文章提供有關 GPU、vCPU、資料磁碟及 NIC 之數量和類型的資訊。 另說明此群組中每個大小的輸送量和網路頻寬。 

* **NC、NCv2、NCv3 及 ND** 等大小最適合計算密集和高網路用量的應用程式與演算法。 範例包括以 CUDA 和 OpenCL 為基礎的應用程式，以及模擬、AI、深度學習等。 NCv3 系列著重於高效能運算工作負載，採用 NVIDIA 的 Tesla V100 GPU。  ND 系列的目標為深度學習的訓練與推斷案例。 該系列使用了 NVIDIA Tesla P40 GPU。
* **NV 和 NVv2** 大小則會針對遠端視覺效果、串流、遊戲、編碼及利用 OpenGL 和 DirectX 這類架構的 VDI 案例進行最佳化和設計。  這些 VM 是由 NVIDIA Tesla M60 GPU 提供支援。


## <a name="nc-series"></a>NC 系列

進階儲存體：不支援

進階儲存體快取：不支援

NC 系列 VM 是由 [NVIDIA Tesla K80](http://images.nvidia.com/content/pdf/kepler/Tesla-K80-BoardSpec-07317-001-v05.pdf) \(英文\) 卡提供技術支援。 使用者可以藉由將 CUDA 用於能源探勘應用程式、當機模擬、光線追蹤轉譯、深度學習等等，更快速地處理資料。 NC24r 設定提供低延遲且高輸送量網路介面，最適合用於緊密結合的平行計算工作負載。


| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_NC6 |6 |56 | 340 | 1 | 24 | 1 |
| Standard_NC12 |12 |112 | 680 | 2 | 48 | 2 |
| Standard_NC24 |24 |224 | 1440 | 4 | 64 | 4 |
| Standard_NC24r* |24 |224 | 1440 | 4 | 64 | 4 |

1 GPU = 1/2 K80 卡。

*支援 RDMA

## <a name="ncv2-series"></a>NCv2 系列

進階儲存體：支援

進階儲存體快取：支援

NCv2 系列 VM 是由 [NVIDIA Tesla P100](http://images.nvidia.com/content/tesla/pdf/nvidia-tesla-p100-datasheet.pdf) GPU 提供技術支援。 這些 GPU 可提供 NC 系列 2 倍以上的計算效能。 客戶可針對儲槽模型、DNA 定序、蛋白質分析、蒙地卡羅模擬等傳統 HPC 工作負載，善用這些更新過的 GPU。 NC24rs v2 組態提供低延遲且高輸送量網路介面，最適合用於緊密結合的平行計算工作負載。

> [!IMPORTANT]
> 對於這個大小的系列，一開始會在您的訂用帳戶中，將每個區域中的 vCPU (核心) 配額設為 0。 在[可用區域](https://azure.microsoft.com/regions/services/)中，針對這個系列[要求增加 vCPU 配額](../articles/azure-supportability/resource-manager-core-quotas-request.md)。
>

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC |
| --- | --- | --- | --- | --- | --- | ---  |
| Standard_NC6s_v2 |6 |112 | 736 | 1 | 12 | 4 |
| Standard_NC12s_v2 |12 |224 | 1474 | 2 | 24 | 8 |
| Standard_NC24s_v2 |24 |448 | 2948 | 4 | 32 | 8 |
| Standard_NC24rs_v2* |24 |448 | 2948 | 4 | 32 | 8 |

1 GPU = 一張 P100 卡。

*支援 RDMA

## <a name="ncv3-series"></a>NCv3 系列

進階儲存體：支援

進階儲存體快取：支援

NCv3 系列 VM 是由 [NVIDIA Tesla V100](http://www.nvidia.com/content/PDF/Volta-Datasheet.pdf) GPU 提供技術支援。 這些 GPU 可提供 NCv2 系列 1.5 倍的計算效能。 客戶可針對儲槽模型、DNA 定序、蛋白質分析、蒙地卡羅模擬等傳統 HPC 工作負載，善用這些更新過的 GPU。 NC24rs v3 組態提供低延遲且高輸送量網路介面，最適合用於緊密結合的平行計算工作負載。

> [!IMPORTANT]
> 對於這個大小的系列，一開始會在您的訂用帳戶中，將每個區域中的 vCPU (核心) 配額設為 0。 在[可用區域](https://azure.microsoft.com/regions/services/)中，針對這個系列[要求增加 vCPU 配額](../articles/azure-supportability/resource-manager-core-quotas-request.md)。
>

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_NC6s_v3 |6 |112 | 736 | 1 | 12 | 4 |
| Standard_NC12s_v3 |12 |224 | 1474 | 2 | 24 | 8 |
| Standard_NC24s_v3 |24 |448 | 2948 | 4 | 32 | 8 | 
| Standard_NC24rs_v3* |24 |448 | 2948 | 4 | 32 | 8 |

1 GPU = 一張 V100 卡。

*支援 RDMA

## <a name="nd-series"></a>ND 系列

進階儲存體：支援

進階儲存體快取：支援

ND 系列的虛擬機器是 GPU 系列的新成員，專為 AI 和深度學習工作負載所設計。 它們能為訓練和推斷提供絕佳效能。 ND 執行個體是由 [NVIDIA Tesla P40](http://images.nvidia.com/content/pdf/tesla/184427-Tesla-P40-Datasheet-NV-Final-Letter-Web.pdf) \(英文\) GPU 提供技術支援。 這些執行個體為單精確度浮點數作業、使用 Microsoft Cognitive Toolkit 的 AI 工作負載、TensorFlow、Caffe 及其他架構，提供絕佳的效能。 ND 系列還能提供更大的 GPU 記憶體大小 (24 GB)，能夠用於更大的類神經網路模型。 如同 NC 系列，ND 系列透過 RDMA 提供具有次要低延遲且高輸送量網路的設定，以及 InfiniBand 連線能力，讓您能夠執行使用橫跨數個 GPU 的大規模訓練作業。

> [!IMPORTANT]
> 對於這個大小的系列，一開始會在您的訂用帳戶中，將每個區域中的 vCPU (核心) 配額設為 0。 在[可用區域](https://azure.microsoft.com/regions/services/)中，針對這個系列[要求增加 vCPU 配額](../articles/azure-supportability/resource-manager-core-quotas-request.md)。
>

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_ND6s |6 |112 | 736 | 1 | 12 | 4 |
| Standard_ND12s |12 |224 | 1474 | 2 | 24 | 8 | 
| Standard_ND24s |24 |448 | 2948 | 4 | 32 | 8 |
| Standard_ND24rs* |24 |448 | 2948 | 4 | 32 | 8 |

1 GPU = 一張 P40 卡。

*支援 RDMA

## <a name="nv-series"></a>NV 系列

進階儲存體：不支援

進階儲存體快取：不支援

NV 系列虛擬機器是由 [NVIDIA Tesla M60](http://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) \(英文\) GPU 和 NVIDIA GRID 技術提供技術支援，適用於桌面加速應用程式和虛擬桌面，可供客戶將其資料或模擬視覺化。 使用者能夠在 NV 執行個體上，將其圖形密集型工作流程視覺化以獲得較佳的圖形功能，此外還能夠執行單精確度工作負載，例如編碼和轉譯。 

NV 執行個體中的每個 GPU 均隨附 GRID 授權。 此授權可讓您彈性地使用 NV 執行個體作為單一使用者的虛擬工作站，或讓 25 位並行使用者可以針對某個虛擬應用程式案例連線至 VM。

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC | 虛擬工作站 | 虛擬應用程式 | 
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NV6 |6 |56 |340 | 1 | 24 | 1 | 1 | 25 |
| Standard_NV12 |12 |112 |680 | 2 | 48 | 2 | 2 | 50 |
| Standard_NV24 |24 |224 |1440 | 4 | 64 | 4 | 4 | 100 |

1 GPU = 1/2 M60 卡。

## <a name="nvv2-series-preview"></a>NVv2 系列 (預覽)

進階儲存體：支援

進階儲存體快取：支援

NVv2 系列的虛擬機器採用 [NVIDIA Tesla M60](http://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) GPU 以及具備 Intel Broadwell CPU 的 NVIDIA GRID 技術。 這些虛擬機器專用於 GPU 高速圖形應用程式以及虛擬桌面，客戶可用於將其資料視覺化、將結果模擬到檢視中、運用於 CAD 之上，或是轉譯內容及串流內容。 除此之外，這些虛擬機器可以執行單一的精密工作負載，像是編碼及轉譯。 NVv2 虛擬機器支援進階儲存體，同時相較於之前的 NV 系列，配備兩倍的系統記憶體 (RAM)。  

NVv2 執行個體中的每個 GPU 均隨附 GRID 授權。 此授權可讓您彈性地使用 NV 執行個體作為單一使用者的虛擬工作站，或讓 25 位並行使用者可以針對某個虛擬應用程式案例連線至 VM。

| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | 最大資料磁碟 | 最大 NIC | 虛擬工作站 | 虛擬應用程式 | 
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NV6s_v2 |6 |112 |320 | 1 | 12 | 4 | 1 | 25 |
| Standard_NV12s_v2 |12 |224 |640 | 2 | 24 | 8 | 2 | 50 |
| Standard_NV24s_v2 |24 |448 |1280 | 4 | 32 | 8 | 4 | 100 |

1 GPU = 1/2 M60 卡。

 
