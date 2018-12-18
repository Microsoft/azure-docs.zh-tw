---
title: Azure 中的 Linux HPC Pack 叢集選項 | Microsoft Docs
description: 了解在 Azure 雲端使用 Microsoft HPC Pack 建立及管理 Linux 高效能運算 (HPC) 叢集的選項
services: virtual-machines-linux,cloud-services
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management,hpc-pack
ms.assetid: ac60624e-aefa-40c3-8bc1-ef6d5c0ef1a2
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 05/14/2018
ms.author: danlep
ms.openlocfilehash: 79600909387b1876b112219b5b9b1e45ba4054b7
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340070"
---
# <a name="options-with-hpc-pack-to-create-and-manage-a-cluster-for-linux-hpc-workloads-in-azure"></a>使用 HPC Pack 在 Azure 中建立及管理適用於 Linux HPC 工作負載之叢集的選項
[!INCLUDE [virtual-machines-common-hpcpack-cluster-options](../../../includes/virtual-machines-common-hpcpack-cluster-options.md)]

本文著重於使用 HPC Pack 來執行 Linux 工作負載的 Azure 選項。 另有[使用 HPC Pack 執行 Windows HPC 工作負載](../windows/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)的選項。

## <a name="hpc-pack-cluster-in-azure-vms-and-vm-scale-sets"></a>Azure VM 和 VM 擴展集中的 HPC Pack 叢集
### <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本
* (GitHub) [HPC Pack 2016 Update 1 叢集範本](https://github.com/MsHpcPack/HPCPack2016)
* (GitHub) [HPC Pack 2012 R2 叢集範本](https://github.com/MsHpcPack/HPCPack2012R2)
* (快速入門) [使用 Linux 計算節點建立 HPC Pack 2012 R2 叢集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster-linux-cn)

### <a name="azure-vm-images"></a>Azure VM 映像
如果您想要在 Azure 中建置自己的 HPC Pack 叢集，請瀏覽 [Azure Marketplace 中的 HPC Pack 映像](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?page=1&search=%22HPC%20%20Pack%22)。

## <a name="hpc-pack-2012-r2-cluster-in-classic-deployment-model"></a>傳統部署模型中的 HPC Pack 2012 R2 叢集
* [使用 HPC Pack IaaS 部署指令碼建立 Linux HPC 叢集](../windows/classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)
* [教學課程：開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](classic/hpcpack-cluster.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)
* [教學課程：在 Azure 中的 Linux 計算節點以 Microsoft HPC Pack 執行 NAMD](classic/hpcpack-cluster-namd.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)
* [教學課程：在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 OpenFOAM](classic/hpcpack-cluster-openfoam.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)
* [教學課程：在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 STAR-CCM+](classic/hpcpack-cluster-starccm.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

## <a name="job-submisssion"></a>作業提交
* [將作業送出至 Azure 的 HPC Pack 叢集](../windows/hpcpack-cluster-submit-jobs.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)




