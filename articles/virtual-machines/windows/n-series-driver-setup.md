---
title: 適用於 Windows 的 Azure N 系列 GPU 驅動程式設定 | Microsoft Docs
description: 如何針對 Azure 中執行 Windows Server 或 Windows 的 N 系列虛擬機器設定 NVIDIA GPU 驅動程式
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: f3950c34-9406-48ae-bcd9-c0418607b37d
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a4d259c7f9a139b3c31d96e75d588c7be162189c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033240"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-windows"></a>在執行 Windows 的 N 系列 VM 上安裝 NVIDIA GPU 驅動程式 

若要利用 Azure N 系列 VM (執行 Windows) 的 GPU 功能，您必須安裝 NVIDIA GPU 驅動程式。 [NVIDIA GPU 驅動程式擴充功能](../extensions/hpccompute-gpu-windows.md)會在 N 系列 VM 上安裝適當的 NVIDIA CUDA 或 GRID 驅動程式。 使用 Azure 入口網站或者 Azure PowerShell 或 Azure Resource Manager 範本之類的工具，安裝或管理擴充功能。 如需支援的作業系統和部署步驟，請參閱 [NVIDIA GPU 驅動程式擴充功能文件](../extensions/hpccompute-gpu-windows.md)。

如果您選擇手動安裝 GPU 驅動程式，本文提供支援的作業系統、驅動程式，以及安裝和驗證步驟。 驅動程式手動設定資訊也適用於 [Linux VM](../linux/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

如需基本規格、儲存體容量與磁碟的詳細資料，請參閱 [GPU Windows VM 大小](sizes-gpu.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 

[!INCLUDE [virtual-machines-n-series-windows-support](../../../includes/virtual-machines-n-series-windows-support.md)]

## <a name="driver-installation"></a>驅動程式安裝

1. 由遠端桌面連接至每個 N 系列 VM。

2. 下載、擷取及安裝 Windows 作業系統支援的驅動程式。

在 VM 上安裝 GRID 驅動程式之後，必須重新啟動。 在安裝 CUDA 驅動程式之後，不需要重新啟動。

## <a name="verify-driver-installation"></a>確認驅動程式安裝

您可以在 [裝置管理員] 中確認驅動程式安裝。 下列範例會顯示 Azure NC VM 上成功的 Tesla K80 卡組態。

![GPU 驅動程式屬性](./media/n-series-driver-setup/GPU_driver_properties.png)

若要查詢 GPU 裝置狀態，請執行與驅動程式一起安裝的 [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface)命令列公用程式。

1. 開啟命令提示字元然後變更位置到 **C:\Program Files\NVIDIA Corporation\NVSMI** 目錄中。

2. 執行 `nvidia-smi`。 如果已安裝驅動程式，您會看到類以以下的輸出。 請注意，除非您正在 VM上執行 GPU 工作負載，否則 [GPU-Util] 會顯示 **0%**。 您的驅動程式版本和 GPU 詳細資料可能會與顯示的不同。

![NVIDIA 裝置狀態](./media/n-series-driver-setup/smi.png)  

## <a name="rdma-network-connectivity"></a>RDMA 網路連線

可以在支援 RDMA 的 N 系列 VM (例如部署在同一個可用性設定組或 VM 擴展集的單一放置群組中的 NC24r) 上啟用 RDMA 網路連線能力。 在具備 RDMA 功能的 VM 上，HpcVmDrivers 擴充必須新增以安裝 Windows 網路裝置驅動程式，該驅動程式會啟用 RDMA 連線能力。 若要將 VM 擴充功能新增至 RDMA 啟用的 N 系列 VM，請針對 Azure Resource Manager 使用 [Azure PowerShell](/powershell/azure/overview) Cmdlet。

若要在美國西部區域中名為 myVM 的現有具備 RDMA 功能的 VM 上安裝最新版本 1.1 HpcVMDrivers 延伸模組：
  ```PowerShell
  Set-AzureRmVMExtension -ResourceGroupName "myResourceGroup" -Location "westus" -VMName "myVM" -ExtensionName "HpcVmDrivers" -Publisher "Microsoft.HpcCompute" -Type "HpcVmDrivers" -TypeHandlerVersion "1.1"
  ```
  如需詳細資訊，請參閱[適用於 Windows 的虛擬機器擴充功能和功能](extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

RDMA 網路可針對使用 [Microsoft MPI](https://docs.microsoft.com/message-passing-interface/microsoft-mpi) 或 Intel MPI 5.x 執行的應用程式，支援訊息傳遞介面 (MPI) 流量。 


## <a name="next-steps"></a>後續步驟

* 針對 NVIDIA Tesla GPU 組建 GPU 加速應用程式的開發人員也可以下載及安裝最新 [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)。 如需詳細資訊，請參閱 [CUDA 安裝指南](http://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html#axzz4ZcwJvqYi)。


