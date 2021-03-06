---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 5413a6acafa0ea54f98383fc8140a34aff0cf840
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98859725"
---
## <a name="supported-operating-systems-and-drivers"></a>支援的作業系統和驅動程式

### <a name="nvidia-tesla-cuda-drivers"></a>NVIDIA Tesla (CUDA) 驅動程式

適用于 NC、NCv2、NCv3、NCasT4_v3、ND 和 NDv2 系列 Vm 的 NVIDIA Tesla (CUDA) 驅動程式 (只有下表所列的作業系統才支援 NV 系列) 的選用。 以下是本文章發行時的最新驅動程式下載連結。 如需最新的驅動程式，請瀏覽 [NVIDIA](https://www.nvidia.com/) 網站。

> [!TIP]
> 在 Windows Server VM 上手動安裝 CUDA 驅動程式的替代方案，就是部署 Azure [資料科學虛擬機器](../articles/machine-learning/data-science-virtual-machine/overview.md)映像。 適用於 Windows Server 2016 的 DSVM 版本會預先安裝 NVIDIA CUDA 驅動程式、CUDA 深度類神經網路程式庫和其他工具。


| OS | 驅動程式 |
| -------- |------------- |
| Windows Server 2019 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) ( .exe)  |
| Windows Server 2016 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) ( .exe)  |

### <a name="nvidia-grid-drivers"></a>NVIDIA GRID 驅動程式

Microsoft 會針對用來作為虛擬工作站或虛擬應用程式的 NV 和 NVv3 系列 Vm，轉散發 NVIDIA GRID 驅動程式安裝程式。 僅在下表所列的作業系統上安裝 Azure NV 系列 Vm 上的這些方格驅動程式。 這些驅動程式包含在 Azure 的 GRID 虛擬 GPU 軟體的授權中。 您不需要設定 NVIDIA vGPU software 授權伺服器。

Azure 所轉散發的方格驅動程式無法在非 NV 系列 Vm （例如 NCv2、NCv3、ND 和 NDv2 系列 Vm）上運作。 其中一個例外狀況是 NCas_T4_V3 VM 系列，格線驅動程式會在其中啟用類似于 NV 系列的圖形功能。

具有 Nvidia K80 Gpu 的 NC-Series 不支援方格/圖形應用程式。  

請注意，Nvidia 延伸模組一律會安裝最新的驅動程式。 針對相依于較舊版本的客戶，我們提供了先前版本的連結。

若為 Windows Server 2019、Windows Server 2016 1607、1709和 Windows 10 () 建立20H2：
- [方格 12.0 (461.09) ](https://go.microsoft.com/fwlink/?linkid=874181) ( .exe) 
- [方格 11.3 (452.77) ](https://download.microsoft.com/download/f/d/5/fd5ad39b-89cb-4990-ae85-a6fd30475584/452.77_grid_win10_server2016_server2019_64bit_azure_swl.exe) ( .exe)  

若為 Windows Server 2012 R2： 
- [方格 12.0 (461.09) ](https://download.microsoft.com/download/c/5/e/c5e7df99-364d-45f5-bff7-c253d59121f1/461.09_grid_server2012R2_64bit_azure_swl.exe) ( .exe) 
- [方格 11.3 (452.77) ](https://download.microsoft.com/download/5/4/3/54323644-3c84-4aa1-97ec-35491f94c866/452.77_grid_server2012R2_64bit_azure_swl.exe) ( .exe)  


如需所有先前的 Nvidia GRID 驅動程式連結的完整清單，請造訪 [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json)
