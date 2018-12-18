---
title: 用來部署 Linux HPC 叢集的 PowerShell 指令碼 | Microsoft Docs
description: 執行 PowerShell 指令碼，以在 Azure 虛擬機器中部署 Linux HPC Pack 2012 R2 叢集
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,hpc-pack
ms.assetid: 73041960-58d3-4ecf-9540-d7e1a612c467
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 12/29/2016
ms.author: danlep
ms.openlocfilehash: 66affb47190ba0c6fccaae8e8267b310682aee46
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841838"
---
# <a name="create-a-linux-high-performance-computing-hpc-cluster-with-the-hpc-pack-iaas-deployment-script"></a>使用 HPC Pack IaaS 部署指令碼建立 Linux 高效能運算 (HPC) 叢集
執行 HPC Pack IaaS 部署 PowerShell 指令碼，以在 Azure 虛擬機器中為 Linux 工作負載部署完整的 HPC Pack 2012 R2 叢集。 叢集是由加入 Active Directory、且執行 Windows Server 和 Microsoft HPC Pack 的前端節點，以及執行其中一個 HPC Pack 所支援的 Linux 散發套件的計算節點所組成。 如果您想要在 Azure 中為 Windows 工作負載部署 HPC Pack 叢集，請參閱[使用 HPC Pack IaaS 部署指令碼建立 Windows HPC 叢集](../../windows/classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
> [!IMPORTANT] 
> 本文中所述的 PowerShell 指令碼會使用傳統部署模型，在 Azure 中建立 Microsoft HPC Pack 2012 R2 叢集。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> 此外，本文中所述的指令碼不支援 HPC Pack 2016。 如需適用於 HPC Pack 2012 R2 和 HPC Pack 2016 之 Resource Manager 範本的詳細資訊，請參閱 [Azure 中的 HPC Pack 叢集部署選項](../hpcpack-cluster-options.md)。


[!INCLUDE [virtual-machines-common-classic-hpcpack-cluster-powershell-script](../../../../includes/virtual-machines-common-classic-hpcpack-cluster-powershell-script.md)]

## <a name="example-configuration-file"></a>範例組態檔
下列組態檔會建立網域控制站和網域樹系並部署 HPC Pack 叢集，此叢集包含 1 個具有本機資料庫的前端節點和 10 個 Linux 計算節點。 所有雲端服務都直接建立在「東亞」位置中。 Linux 計算節點會建立在 2 個雲端服務和 2 個儲存體帳戶中 (亦即 *MyLnxCNService01* 和 *mylnxstorage01* 中的 *MyLnxCN-0001* 至 *MyLnxCN-0005*，以及 *MyLnxCNService02* 和 *mylnxstorage02* 中的 *MyLnxCN-0006* 至 *MyLnxCN-0010*)。 計算節點會從 OpenLogic CentOS 7.0 版 Linux 映像建立。 

請將訂用帳戶名稱和服務及服務名稱取代為您自己的值。

```Xml
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
  <Location>East Asia</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>NewDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
    <DomainController>
      <VMName>MyDCServer</VMName>
      <ServiceName>MyHPCService</ServiceName>
      <VMSize>Large</VMSize>
    </DomainController>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>MyHeadNode</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>ExtraLarge</VMSize>
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>MyLnxCN-%0001%</VMNamePattern>
    <ServiceNamePattern>MyLnxCNService%01%</ServiceNamePattern>
    <MaxNodeCountPerService>5</MaxNodeCountPerService>
    <StorageAccountNamePattern>mylnxstorage%01%</StorageAccountNamePattern>
    <VMSize>Medium</VMSize>
    <NodeCount>10</NodeCount>
    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150325 </ImageName>
  </LinuxComputeNodes>
</IaaSClusterConfig>
```
## <a name="troubleshooting"></a>疑難排解
* **「VNet 不存在」錯誤**。 如果您執行 HPC Pack IaaS 部署指令碼來將多個叢集同時部署在 Azure 中的一個訂用帳戶底下，可能會有一或多個部署因發生「VNet *VNet\_Name* 不存在」錯誤而失敗。
  如果發生此錯誤，請針對失敗的部署重新執行指令碼。
* **從 Azure 虛擬網路存取網際網路時發生問題**。 如果您藉由使用部署指令碼來建立具有新網域控制站的 HPC Pack 叢集，或是手動將前端節點 VM 升級到網域控制站，則在將 Azure 虛擬網路中的 VM 連接到網際網路時，可能會發生問題。 如果在網域控制站上自動設定轉寄站 DNS 伺服器，且這個轉寄站 DNS 伺服器未正確解析，就可能出現這種狀況。
  
    若要解決此問題，請登入網域控制站，並選擇移除轉寄站組態設定，或設定有效的轉寄站 DNS 伺服器。 若要這樣做，請在伺服器管理員中按一下 [工具]  >  [DNS] 以開啟 DNS 管理員，然後按兩下 [轉寄站]。

## <a name="next-steps"></a>後續步驟
* 有關支援的 Linux 散發套件、移動資料，以及使用 Linux 計算節點將工作提交至 HPC Pack 叢集，如需詳細資訊請參閱[開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](hpcpack-cluster.md)。
* 如需有關使用指令碼來建立叢集和執行 Linux HPC 工作負載的教學課程，請參閱︰
  * [在 Azure 中的 Linux 運算節點以 Microsoft HPC Pack 執行 NAMD](hpcpack-cluster-namd.md)
  * [在 Azure 中的 Linux 計算節點以 Microsoft HPC Pack 執行 OpenFOAM](hpcpack-cluster-openfoam.md)
  * [在 Azure 中的 Linux 計算節點以 Microsoft HPC Pack 執行 STAR-CCM+](hpcpack-cluster-starccm.md)

