---
title: 包含檔案
description: 包含檔案
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 4bfac9be5041fdf4ebfe7ea56f064b8b85806703
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98860112"
---
## <a name="supported-distributions-and-drivers"></a>支援的散發套件和驅動程式

### <a name="nvidia-cuda-drivers"></a>NVIDIA CUDA 驅動程式

只有下表所列的 Linux 發行版本才支援 NC、NCv2、NCv3、ND 及 NDv2 系列 VM (針對 NV 系列為選擇性) 的 NVIDIA CUDA 驅動程式。 CUDA 驅動程式為本文章發行時的最新資訊。 如需最新的 CUDA 驅動程式和支援的作業系統，請造訪 [NVIDIA](https://developer.nvidia.com/cuda-zone) 網站。 請確定您已安裝或升級為散發套件所適用的最新 CUDA 驅動程式。 

> [!TIP]
> 在 Linux VM 上手動安裝 CUDA 驅動程式的替代方案，就是部署 Azure [資料科學虛擬機器](../articles/machine-learning/data-science-virtual-machine/overview.md)映像。 適用於 Ubuntu 16.04 LTS 或 CentOS 7.4 的 DSVM 版本會預先安裝 NVIDIA CUDA 驅動程式、CUDA 深度類神經網路程式庫和其他工具。


### <a name="nvidia-grid-drivers"></a>NVIDIA GRID 驅動程式

Microsoft 會針對用來作為虛擬工作站或虛擬應用程式的 NV 和 NVv3 系列 Vm，轉散發 NVIDIA GRID 驅動程式安裝程式。 請僅將這些 GRID 驅動程式安裝在 Azure NV 虛擬機器上，且僅安裝在下表所列的作業系統上。 這些驅動程式包含在 Azure 的 GRID 虛擬 GPU 軟體的授權中。 您不需要設定 NVIDIA vGPU software 授權伺服器。

Azure 所轉散發的方格驅動程式無法在非 NV 系列 Vm （例如 NC、NCv2、NCv3、ND 和 NDv2 系列 Vm）上運作。

|散發|驅動程式|
| --- | -- |
|Ubuntu 18.04 LTS<br/><br/>Ubuntu 16.04 LTS<br/><br/>Red Hat Enterprise Linux 7.7 至7.9、8.0、8。1<br/><br/>SUSE Linux Enterprise Server 12 SP2 <br/><br/>SUSE Linux Enterprise Server 15 SP2 | NVIDIA GRID 12.0，驅動程式分支 [R460](https://go.microsoft.com/fwlink/?linkid=874272) ( .exe) |

如需所有先前的 Nvidia GRID 驅動程式連結的完整清單，請流覽 [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json) 。

> [!WARNING] 
> 在 Red Hat 產品上安裝第三方軟體可能會影響 Red Hat 支援條款。 請參閱 [Red Hat 知識庫文件 (英文)](https://access.redhat.com/articles/1067)。
>
