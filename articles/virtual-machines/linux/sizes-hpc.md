---
title: Azure Linux VM 大小 - HPC | Microsoft Docs
description: 列出 Azure 中可用的不同 Linux 高效能運算虛擬機器大小。 列出 vCPU 數目、資料磁碟和 NIC 的相關資訊，以及此服務中各種大小之儲存體輸送量和網路頻寬的相關資訊。
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
ms.openlocfilehash: 748cb4612b2b5aed26ba8197cfad0782f2645e1e
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37902124"
---
# <a name="high-performance-compute-virtual-machine-sizes"></a>高效能運算的虛擬機器大小

[!INCLUDE [virtual-machines-common-sizes-hpc](../../../includes/virtual-machines-common-sizes-hpc.md)]

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../../includes/virtual-machines-common-sizes-table-defs.md)]

[!INCLUDE [virtual-machines-common-a8-a9-a10-a11-specs](../../../includes/virtual-machines-common-a8-a9-a10-a11-specs.md)]


### <a name="mpi"></a>MPI 

只支援 Intel MPI 5.x 版本。 Intel MPI 執行階段程式庫的較新版本 (2017 和 2018) 與 Azure Linux RDMA 驅動程式不相容。


### <a name="distributions"></a>散發
 
從 Azure Marketplace 中支援 RDMA 連線的其中一個映像，部署計算密集的 VM：
  
* **Ubuntu**：Ubuntu Server 16.04 LTS。 設定 VM 上的 RDMA 驅動程式，並向 Intel 註冊以下載 Intel MPI：

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **SUSE Linux Enterprise Server**：SLES 12 SP3 for HPC、SLES 12 SP3 for HPC (Premium)、SLES 12 SP1 for HPC、SLES 12 SP1 for HPC (Premium)。 已在 VM 上安裝 RDMA 驅動程式並散發 Intel MPI 套件。 執行下列命令來安裝 MPI：

  ```bash
  sudo rpm -v -i --nodeps /opt/intelMPI/intel_mpi_packages/*.rpm
  ```
    
* **以 CentOS 為基礎的 HPC** - 以 CentOS 為基礎的 6.5 HPC 或更新版本 (針對 H 系列，建議使用 7.1 版或更新版本)。 已在 VM 上安裝 RDMA 驅動程式和 Intel MPI 5.1。  
 
  > [!NOTE]
  > 在 CentOS 型 HPC 映像上， **yum** 組態檔中已停用核心更新。 這是因為 Linux RDMA 驅動程式以 RPM 封裝散發，如果更新核心，驅動程式更新可能無法運作。
  > 
 
### <a name="cluster-configuration"></a>叢集組態 
    
必須進行額外的系統設定，才能在叢集 VM 上執行 MPI 作業。 例如，在 VM 叢集上，您必須在計算節點之間建立信任關係。 如需了解一般設定，請參閱[設定 Linux RDMA 叢集以執行 MPI 應用程式](classic/rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)。

### <a name="network-topology-considerations"></a>網路拓撲考量
* 在 Azure 中具備 RDMA 功能的 Linux VM 上，Eth1 會保留給 RDMA 網路流量使用。 請勿變更任何 Eth1 設定或參考到此網路之組態檔中的任何資訊。 Eth0 會保留給一般 Azure 網路流量。

* Azure 中的 RDMA 網路會保留位址空間 172.16.0.0/16。 


## <a name="using-hpc-pack"></a>使用 HPC Pack
[HPC Pack](https://technet.microsoft.com/library/jj899572.aspx)是 Microsoft 的免費 HPC 叢集和作業管理解決方案，提供您一個搭配 Linux 使用計算密集型執行個體的選項。 HPC Pack 的最新版本支援讓數個 Linux 散發套件在部署於 Azure VM 中、由 Windows Server 前端節點管理的計算節點上執行。 搭配支援 RDMA 且執行 Intel MPI 的 Linux 計算節點時，HPC Pack 可以排定及執行存取 RDMA 網路的 Linux MPI 應用程式。 請參閱[開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](classic/hpcpack-cluster.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)。

## <a name="other-sizes"></a>其他大小
- [一般用途](sizes-general.md)
- [計算最佳化](sizes-compute.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [GPU](../windows/sizes-gpu.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟

- 若要開始在 Linux 上部署及使用具有 RDMA 的計算密集型大小，請參閱[設定 Linux RDMA 叢集以執行 MPI 應用程式](classic/rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)。

- 深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。




