---
title: 用來部署 Windows HPC 叢集的 PowerShell 指令碼 | Microsoft Docs
description: 執行 PowerShell 指令碼，以在 Azure 虛擬機器中部署 Windows HPC Pack 2012 R2 叢集
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,hpc-pack
ms.assetid: 286b2be8-2533-40df-b02a-26156b1f1133
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 12/29/2016
ms.author: danlep
ms.openlocfilehash: e05562aeac0ea89ec1c3d80d2967c8f59c68d332
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30914835"
---
# <a name="create-a-windows-high-performance-computing-hpc-cluster-with-the-hpc-pack-iaas-deployment-script"></a>使用 HPC Pack IaaS 部署指令碼建立 Windows 高效能運算 (HPC) 叢集
執行 HPC Pack IaaS 部署 PowerShell 指令碼，以在 Azure 虛擬機器中為 Windows 工作負載部署完整的 HPC Pack 2012 R2 叢集。 叢集是由加入 Active Directory、且執行 Windows Server 和 Microsoft HPC Pack 的前端節點，以及您指定的其他 Windows 計算資源所組成。 如果您想要在 Azure 中為 Linux 工作負載部署 HPC Pack 叢集，請參閱 [使用 HPC Pack IaaS 部署指令碼建立 Linux HPC 叢集](../../linux/classic/hpcpack-cluster-powershell-script.md)。 

> [!IMPORTANT] 
> 本文中所述的 PowerShell 指令碼會使用傳統部署模型，在 Azure 中建立 Microsoft HPC Pack 2012 R2 叢集。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> 此外，本文中所述的指令碼不支援 HPC Pack 2016。 如需適用於 HPC Pack 2012 R2 和 HPC Pack 2016 之 Resource Manager 範本的詳細資訊，請參閱 [Azure 中的 HPC Pack 叢集部署選項](../hpcpack-cluster-options.md)。

[!INCLUDE [virtual-machines-common-classic-hpcpack-cluster-powershell-script](../../../../includes/virtual-machines-common-classic-hpcpack-cluster-powershell-script.md)]

## <a name="example-configuration-files"></a>範例組態檔
在下列範例中，請將訂用帳戶 ID 或名稱及帳戶和服務名稱取代為您自己的值。

### <a name="example-1"></a>範例 1
下列組態檔會部署一個 HPC Pack 叢集，其中包含一個具有本機資料庫的前端節點，以及五個執行 Windows Server 2012 R2 作業系統的計算節點。 所有雲端服務都直接建立在「美國西部」位置。 前端節點會做為網域樹系的網域控制站。

```Xml
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionId>08701940-C02E-452F-B0B1-39D50119F267</SubscriptionId>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
  <Location>West US</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>MyHeadNode</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>ExtraLarge</VMSize>
  </HeadNode>
  <ComputeNodes>
    <VMNamePattern>MyHPCCN-%1000%</VMNamePattern>
    <ServiceName>MyHPCCNService</ServiceName>
    <VMSize>Medium</VMSize>
    <NodeCount>5</NodeCount>
    <OSVersion>WindowsServer2012R2</OSVersion>
  </ComputeNodes>
</IaaSClusterConfig>
```

### <a name="example-2"></a>範例 2
下列組態檔會在現有的網域樹系中部署 HPC Pack 叢集。 叢集中有 1 個具有本機資料庫的前端節點，和 12 個套用了 BGInfo VM 延伸模組的計算節點。
對於網域樹系中的所有 VM，都會停用 Windows 更新的自動安裝。 所有雲端服務都直接建立在「東亞」位置中。 計算節點會建立在三個雲端服務和三個儲存體帳戶中：*MyHPCCNService01* 和 *mycnstorage01* 中的 *MyHPCCN-0001* 至*MyHPCCN-0005*；*MyHPCCNService02* 和 *mycnstorage02* 中的 *MyHPCCN-0006* 至 *MyHPCCN0010*；以及 *MyHPCCNService03* 和 *mycnstorage03* 中的 *MyHPCCN-0011* 至 *MyHPCCN-0012*)。 計算節點會從擷取自雲端節點的現有私人映像建立。 自動增加和縮減服務會根據預設的增加和縮減間隔來啟用。

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
     <NoWindowsAutoUpdate />
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>MyHeadNode</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>ExtraLarge</VMSize>
  </HeadNode>
  <Certificates>
    <Certificate>
      <Id>1</Id>
      <PfxFile>d:\mytestcert1.pfx</PfxFile>
      <Password>MyPsw!!2</Password>
    </Certificate>
  </Certificates>
  <ComputeNodes>
    <VMNamePattern>MyHPCCN-%0001%</VMNamePattern>
<ServiceNamePattern>MyHPCCNService%01%</ServiceNamePattern>
<MaxNodeCountPerService>5</MaxNodeCountPerService>
<StorageAccountNamePattern>mycnstorage%01%</StorageAccountNamePattern>
    <VMSize>Medium</VMSize>
    <NodeCount>12</NodeCount>
    <ImageName HPCPackInstalled=”true”>MyHPCComputeNodeImage</ImageName>
    <VMExtensions>
       <VMExtension>
          <ExtensionName>BGInfo</ExtensionName>
          <Publisher>Microsoft.Compute</Publisher>
          <Version>1.*</Version>
       </VMExtension>
    </VMExtensions>
  </ComputeNodes>
  <AutoGrowShrink>
    <CertificateId>1</CertificateId>
  </AutoGrowShrink>
</IaaSClusterConfig>

```

### <a name="example-3"></a>範例 3
下列組態檔會在現有的網域樹系中部署 HPC Pack 叢集。 叢集中包含 1 個前端節點、1 個具有 500 GB 資料磁碟的資料庫伺服器、2 個執行 Windows Server 2012 R2 作業系統的訊息代理程式節點，以及 5 個執行 Windows Server 2012 R2 作業系統的計算節點。 MyHPCCNService 雲端服務會建立在同質群組 *MyIBAffinityGroup* 中，其他雲端服務則是建立在同質群組 *MyAffinityGroup* 中。 前端節點上會啟用 HPC 工作排程器 REST API 和 HPC Web 入口網站。

```Xml
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
  <AffinityGroup>MyAffinityGroup</AffinityGroup>
  <Location>East Asia</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>    
  <Domain>
    <DCOption>ExistingDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>NewRemoteDB</DBOption>
    <DBVersion>SQLServer2014_Enterprise</DBVersion>
    <DBServer>
      <VMName>MyDBServer</VMName>
      <ServiceName>MyHPCService</ServiceName>
      <VMSize>ExtraLarge</VMSize>
      <DataDiskSizeInGB>500</DataDiskSizeInGB>
    </DBServer>
  </Database>
  <HeadNode>
    <VMName>MyHeadNode</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>ExtraLarge</VMSize>
    <EnableRESTAPI />
    <EnableWebPortal />
  </HeadNode>
  <ComputeNodes>
    <VMNamePattern>MyHPCCN-%0000%</VMNamePattern>
    <ServiceName>MyHPCCNService</ServiceName>
    <VMSize>A8</VMSize>
<NodeCount>5</NodeCount>
<AffinityGroup>MyIBAffinityGroup</AffinityGroup>
  </ComputeNodes>
  <BrokerNodes>
    <VMNamePattern>MyHPCBN-%0000%</VMNamePattern>
    <ServiceName>MyHPCBNService</ServiceName>
    <VMSize>Medium</VMSize>
    <NodeCount>2</NodeCount>
  </BrokerNodes>
</IaaSClusterConfig>
```



### <a name="example-4"></a>範例 4
下列組態檔會在現有的網域樹系中部署 HPC Pack 叢集。 叢集中包含兩個具有本機資料庫的前端節點、建立了兩個 Azure 節點範本，且針對 Azure 節點範本 *AzureTemplate1*建立了 3 個中型大小的 Azure 節點。 在設定完前端節點之後，指令檔會在前端節點上執行。

```Xml
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
  <AffinityGroup>MyAffinityGroup</AffinityGroup>
  <Location>East Asia</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>ExistingDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>MyHeadNode</VMName>
    <ServiceName>MyHPCService</ServiceName>
<VMSize>ExtraLarge</VMSize>
    <PostConfigScript>c:\MyHNPostActions.ps1</PostConfigScript>
  </HeadNode>
  <Certificates>
    <Certificate>
      <Id>1</Id>
      <PfxFile>d:\mytestcert1.pfx</PfxFile>
      <Password>MyPsw!!2</Password>
    </Certificate>
    <Certificate>
      <Id>2</Id>
      <PfxFile>d:\mytestcert2.pfx</PfxFile>
    </Certificate>    
  </Certificates>
  <AzureBurst>
    <AzureNodeTemplate>
      <TemplateName>AzureTemplate1</TemplateName>
      <SubscriptionId>bb9252ba-831f-4c9d-ae14-9a38e6da8ee4</SubscriptionId>
      <CertificateId>1</CertificateId>
      <ServiceName>mytestsvc1</ServiceName>
      <StorageAccount>myteststorage1</StorageAccount>
      <NodeCount>3</NodeCount>
      <RoleSize>Medium</RoleSize>
    </AzureNodeTemplate>
    <AzureNodeTemplate>
      <TemplateName>AzureTemplate2</TemplateName>
      <SubscriptionId>ad4b9f9f-05f2-4c74-a83f-f2eb73000e0b</SubscriptionId>
      <CertificateId>1</CertificateId>
      <ServiceName>mytestsvc2</ServiceName>
      <StorageAccount>myteststorage2</StorageAccount>
      <Proxy>
        <UsesStaticProxyCount>false</UsesStaticProxyCount>     
        <ProxyRatio>100</ProxyRatio>
        <ProxyRatioBase>400</ProxyRatioBase>
      </Proxy>
      <OSVersion>WindowsServer2012</OSVersion>
    </AzureNodeTemplate>
  </AzureBurst>
</IaaSClusterConfig>
```

## <a name="troubleshooting"></a>疑難排解
* **「VNet 不存在」錯誤** - 如果您執行指令碼將多個叢集同時部署在 Azure 中的一個訂用帳戶底下，可能會有一或多個部署因發生「VNet *VNet\_Name* 不存在」錯誤而失敗。
  如果發生此錯誤，請針對失敗的部署重新執行指令碼。
* **從 Azure 虛擬網路存取網際網路時發生問題** - 如果您使用部署指令碼建立具有新網域控制站的叢集，或手動將前端節點 VM 升級到網域控制站，則在將 VM 連接到網際網路時，可能會發生問題。 如果網域控制站上自動設定了轉寄站 DNS 伺服器，而此轉寄站 DNS 伺服器未正確解析，就可能發生這種問題。
  
    若要解決此問題，請登入網域控制站，並選擇移除轉寄站組態設定，或設定有效的轉寄站 DNS 伺服器。 若要設定這項設定，請在「伺服器管理員」中按一下 [工具] >
    [DNS] 以開啟 [DNS 管理員]，然後按兩下 [轉寄站]。
* **從計算密集型 VM 存取 RDMA 網路時發生問題** - 如果您使用支援 RDMA 大小 (例如 A8 或 A9) 的 VM 來新增 Windows Server 計算節點 VM 或訊息代理程式節點 VM，則在將這些 VM 連接到 RDMA 應用程式網路時，可能會發生問題。 之所以會發生此問題，其中一個原因是在將 VM 新增到叢集時，未正確安裝 HpcVmDrivers 擴充功能。 比方說，延伸模組可能卡在安裝中狀態。
  
    若要解決這個問題，請先檢查 VM 中的延伸模組狀態。 如果延伸模組未正確安裝，請嘗試從 HPC 叢集中移除節點，然後重新新增節點。 例如，您可以在前端節點上執行 Add-HpcIaaSNode.ps1 指令碼，以新增計算節點 VM。

## <a name="next-steps"></a>後續步驟
* 嘗試在叢集上執行測試工作負載。 如需範例，請參閱 HPC Pack [快速入門指南](https://technet.microsoft.com/library/jj884144)。
* 如需編寫叢集部署指令碼及執行 HPC 工作負載的教學課程，請參閱 [開始使用 Azure 中的 HPC Pack 叢集執行 Excel 和 SOA 工作負載](../../virtual-machines-windows-excel-cluster-hpcpack.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 嘗試以 HPC Pack 的工具啟動、停止、新增和移除您所建立之叢集中的計算節點。 請參閱 [在 Azure 中管理 HPC Pack 叢集的計算節點](hpcpack-cluster-node-manage.md)。
* 若要設定將工作從本機電腦提交至叢集，請參閱 [將 HPC 工作從內部部署電腦提交至 Azure 中的 HPC Pack 叢集](../../virtual-machines-windows-hpcpack-cluster-submit-jobs.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

