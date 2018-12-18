---
title: Azure Windows VM 大小 - HPC | Microsoft Docs
description: 列出 Azure 中可用的不同 Windows 高效能運算虛擬機器大小。 列出 vCPU 數目、資料磁碟和 NIC 的相關資訊，以及此服務中各種大小之儲存體輸送量和網路頻寬的相關資訊。
services: virtual-machines-windows
documentationcenter: ''
author: jonbeck7
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/06/2018
ms.author: jonbeck
ms.openlocfilehash: 31e81741d2a627888e478b3871bdbab4e6b6d6f5
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37902634"
---
# <a name="high-performance-compute-vm-sizes"></a>高效能運算 VM 大小

[!INCLUDE [virtual-machines-common-sizes-hpc](../../../includes/virtual-machines-common-sizes-hpc.md)]

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../../includes/virtual-machines-common-sizes-table-defs.md)]

[!INCLUDE [virtual-machines-common-a8-a9-a10-a11-specs](../../../includes/virtual-machines-common-a8-a9-a10-a11-specs.md)]


* **作業系統** - Windows Server 2016、Windows Server 2012 R2、Windows Server 2012

* **MPI** - Microsoft MPI (MS-MPI) 2012 R2 或更新版本、Intel MPI Library 5.x

  支援的 MPI 實作使用 Microsoft Network Direct 介面在執行個體之間進行通訊。 

* **RDMA 網路位址空間** - Azure 中的 RDMA 網路會保留位址空間 172.16.0.0/16。 若要在 Azure 虛擬網路中已部署的執行個體上執行 MPI 應用程式，請確定虛擬網路位址空間不會與 RDMA 網路重疊。

* **HpcVmDrivers VM 擴充功能** - 在支援 RDMA 的 VM 上，新增 HpcVmDrivers 擴充功能，以便安裝 RDMA 連線的 Windows 網路裝置驅動程式。 (在某些 A8 和 A9 執行個體部署中，會自動新增 HpcVmDrivers 擴充功能)。若要將 VM 擴充功能新增至 VM，您可以使用 [Azure PowerShell](/powershell/azure/overview) Cmdlet。 

  
  下列命令會在支援 RDMA 的虛擬機器 (名稱為 *myVM*，部署在*美國西部*區域中名稱為 *myResourceGroup* 的資源群組) 上安裝最新 1.1 版 HpcVMDrivers 擴充功能：

  ```PowerShell
  Set-AzureRmVMExtension -ResourceGroupName "myResourceGroup" -Location "westus" -VMName "myVM" -ExtensionName "HpcVmDrivers" -Publisher "Microsoft.HpcCompute" -Type "HpcVmDrivers" -TypeHandlerVersion "1.1"
  ```
  
  如需詳細資訊，請參閱[虛擬機器擴充功能和功能](extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 您也可以針對已在[傳統部署模型](classic/manage-extensions.md)中部署的 VM 使用擴充功能。


## <a name="using-hpc-pack"></a>使用 HPC Pack

[Microsoft HPC Pack](https://technet.microsoft.com/library/jj899572.aspx) 是 Microsoft 的免費 HPC 叢集和作業管理解決方案，可為您提供一個選項，讓您能夠在 Azure 中建立計算叢集來執行 Windows 型 MPI 應用程式及其他 HPC 工作負載。 HPC Pack 2012 R2 和更新版本包含 MS-MPI 的執行階段環境，此 MS-MPI 如果部署在支援 RDMA 的 VM 上，即可使用 Azure RDMA 網路。



## <a name="other-sizes"></a>其他大小
- [一般用途](sizes-general.md)
- [計算最佳化](sizes-compute.md)
- [記憶體最佳化](../virtual-machines-windows-sizes-memory.md)
- [儲存體最佳化](../virtual-machines-windows-sizes-storage.md)
- [GPU 最佳化](sizes-gpu.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟

- 如需在 Windows 伺服器上使用大量計算執行個體和 HPC Pack 的檢查清單，請參閱[使用 HPC Pack 設定 Windows RDMA 叢集以執行 MPI 應用程式](classic/hpcpack-rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

- 若要在以 Azure Batch 執行 MPI 應用程式時使用計算密集型執行個體，請參閱[在 Azure Batch 中使用多重執行個體工作來執行訊息傳遞介面 (MPI) 應用程式](../../batch/batch-mpi.md)。

- 深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。




