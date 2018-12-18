---
title: Azure 中的 Windows HPC Pack 叢集選項 | Microsoft Docs
description: 了解在 Azure 雲端使用 Microsoft HPC Pack 建立及管理 Windows 高效能運算 (HPC) 叢集的選項
services: virtual-machines-windows,cloud-services,batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management,hpc-pack
ms.assetid: 02c5566d-2129-483c-9ecf-0d61030442d7
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 05/14/2018
ms.author: danlep
ms.openlocfilehash: 1232228d5e2fcd05f41a096a933072dffcca2119
ms.sourcegitcommit: d78bcecd983ca2a7473fff23371c8cfed0d89627
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/14/2018
ms.locfileid: "34165752"
---
# <a name="options-with-hpc-pack-to-create-and-manage-a-cluster-for-windows-hpc-workloads-in-azure"></a>使用 HPC Pack 在 Azure 中建立及管理適用於 Windows HPC 工作負載之叢集的選項
[!INCLUDE [virtual-machines-common-hpcpack-cluster-options](../../../includes/virtual-machines-common-hpcpack-cluster-options.md)]

本文著重於可建立 HPC Pack 叢集以執行 Windows 工作負載的 Azure 選項。 此外，也有建立 HPC Pack 叢集以執行 [Linux HPC 工作負載](../linux/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)的選項。


## <a name="hpc-pack-cluster-in-azure-vms-and-vm-scale-sets"></a>Azure VM 和 VM 擴展集中的 HPC Pack 叢集
### <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本
* (GitHub) [HPC Pack 2016 Update 1 叢集範本](https://github.com/MsHpcPack/HPCPack2016)
* (GitHub) [HPC Pack 2012 R2 叢集範本](https://github.com/MsHpcPack/HPCPack2012R2)
* (快速入門) [建立 HPC Pack 2012 R2 叢集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster)
* (快速入門) [使用自訂的計算節點映像建立 HPC Pack 2012 R2 叢集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster-custom-image)

### <a name="azure-vm-images"></a>Azure VM 映像
如果您想要在 Azure 中建置自己的 HPC Pack 叢集，請瀏覽 [Azure Marketplace 中的 HPC Pack 映像](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?page=1&search=%22HPC%20%20Pack%22)。


### <a name="tutorials"></a>教學課程
* [教學課程：在 Azure 中部署 HPC Pack 2016 叢集](hpcpack-2016-cluster.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 Azure Active Directory 管理 Azure 中的 HPC Pack 2016 叢集](hpcpack-cluster-active-directory.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)


## <a name="hpc-pack-2012-r2-cluster-deployment-in-the-classic-deployment-model"></a>傳統部署模型中的 HPC Pack 2012 R2 叢集部署
* [使用 HPC Pack IaaS 部署指令碼建立 HPC 叢集](classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [教學課程：開始使用 Azure 中的 HPC Pack 叢集執行 Excel 和 SOA 工作負載](excel-cluster-hpcpack.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [在 Azure 中管理 HPC Pack 叢集的運算節點](classic/hpcpack-cluster-node-manage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [增加及縮減 HPC Pack 叢集中的 Azure 運算資源](classic/hpcpack-cluster-node-autogrowshrink.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [使用 HPC Pack 設定 Windows RDMA 叢集以執行 MPI 應用程式](classic/hpcpack-rdma-cluster.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)


## <a name="burst-to-azure-from-hpc-pack-2012-r2"></a>從 HPC Pack 2012 R2 將高載工作負載移至 Azure
* [使用 Microsoft HPC Pack 將暴增的工作負載移至 Azure 背景工作執行個體](https://technet.microsoft.com/library/gg481749.aspx)
* [使用 HPC Pack 將暴增的工作負載移至 Azure Batch](https://technet.microsoft.com/library/mt612877.aspx)
* [教學課程：在 Azure 中使用 HPC Pack 設定混合式叢集](../../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md)

## <a name="job-submission"></a>作業提交

* [將作業送出至 Azure 的 HPC Pack 叢集](hpcpack-cluster-submit-jobs.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)






