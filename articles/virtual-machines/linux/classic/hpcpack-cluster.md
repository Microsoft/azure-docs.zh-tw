---
title: HPC Pack 叢集中的 Linux 計算 VM | Microsoft Docs
description: 了解如何在 Azure 中建立及使用 HPC Pack 叢集，來處理 Linux 高效能運算 (HPC) 工作負載
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager,hpc-pack
ms.assetid: 4d080fdd-5ffe-4f54-a78d-4c818f6eb3fb
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 10/12/2016
ms.author: danlep
ms.openlocfilehash: 2d4091d8ad6a778405ee6bb916c399e0b144f21d
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39441522"
---
# <a name="get-started-with-linux-compute-nodes-in-an-hpc-pack-cluster-in-azure"></a>開始在 Azure 中的 HPC Pack 叢集使用 Linux 運算節點
在 Azure 中設定 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029.aspx) 叢集，其中包含一個執行 Windows Server 的前端節點，以及數個執行支援之 Linux 散發套件的計算節點。 探索叢集的 Linux 節點與 Windows 前端節點之間移動資料的一些選項。 了解如何將 Linux HPC 作業提交至叢集。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

下圖概略顯示您所建立及使用的 HPC Pack 叢集。

![具有 Linux 節點的 HPC Pack 叢集][scenario]

如需在 Azure 上執行 HPC 和批次工作負載的其他選項，請參閱[批次和高效能運算的技術資源](../../../batch/big-compute-resources.md)。

## <a name="deploy-an-hpc-pack-cluster-with-linux-compute-nodes"></a>部署具有 Linux 運算節點的 HPC Pack 叢集
本文將說明使用 Linux 運算節點在 Azure 中部署 HPC Pack 叢集的兩個選項。 這兩種方法使用 Windows Server 的 Marketplace 影像和 HPC Pack 建立前端節點。 

* **Azure Resource Manager 範本** - 使用來自 Azure Marketplace 的範本，或是來自社群的快速入門範本，在 Resource Manager 部署模型中自動建立叢集。 例如，Azure Marketplace 中的 [HPC Pack cluster for Linux workloads (處理 Linux 工作負載的 HPC Pack 叢集)](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/) 範本，會針對 Linux HPC 工作負載建立完整的 HPC Pack 叢集基礎結構。
* **PowerShell 指令碼** - 使用 [Microsoft HPC Pack IaaS 部署指令碼](../../windows/classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json) (**New-HpcIaaSCluster.ps1**)，在傳統部署模型中自動化完整叢集部署作業。 此 Azure PowerShell 指令碼使用 Azure Marketplace 中的 HPC Pack VM 映像進行快速部署，並提供一組完整的組態參數以部署 Linux 計算節點。

如需 Azure 中的 HPC Pack 叢集部署選項有關的詳細資訊，請參閱[使用 Microsoft HPC Pack 在 Azure 中建立及管理高效能運算 (HPC) 叢集的選項](../hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

### <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶** - 您可以使用「Azure 全球」或「Azure 中國」服務中的訂用帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。
* **核心配額** - 您可能需要增加核心的配額，特別是如果您選擇部署多核心 VM 大小的數個叢集節點。 若要增加配額，您可以免費開立線上客戶支援要求。
* **Linux 散發** - 目前 HPC Pack 針對計算節點支援下列 Linux 發佈。 您可以使用這些發佈的 Marketplace 版本，或提供您自己的版本。
  
  * **CentOS 型**：6.5、6.6、6.7、7.0、7.1、7.2、6.5 HPC、7.1 HPC
  * **Red Hat Enterprise Linux**：6.7、6.8、7.2
  * **SUSE Linux Enterprise Server**：SLES 12、SLES 12 (Premium)、SLES 12 SP1、SLES 12 SP1 (Premium)、SLES 12 for HPC、SLES 12 for HPC (Premium)
  * **Ubuntu Server**：14.04 LTS、16.04 LTS
    
    > [!TIP]
    > 若要使用 Azure RDMA 網路搭配其中一個支援 RDMA 的 VM 大小，請從 Azure Marketplace 指定 SUSE Linux Enterprise Server 12 HPC 或 CentOS 型 HPC 映像。 如需詳細資訊，請參閱[高效能運算 VM 大小](../sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
    > 
    > 

使用 HPC Pack IaaS 部署指令碼部署叢集的其他必要條件：

* **用戶端電腦** - 您需要有 Windows 型用戶端電腦，才能執行叢集部署指令碼。
* **Azure PowerShell** - [安裝和設定 Azure PowerShell](/powershell/azure/overview) (0.8.10 版或更新版本)。
* **HPC Pack IaaS 部署指令碼** - 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=44949)下載並解壓縮最新版的指令碼。 執行 `.\New-HPCIaaSCluster.ps1 –Version`即可檢查指令碼的版本。 這篇文章根據 4.4.1 版或更新版本的指令碼。

### <a name="deployment-option-1-use-a-resource-manager-template"></a>部署選項 1。 使用 Resource Manager 範本
1. 前往 Azure Marketplace 的 [HPC Pack cluster for Linux workloads (處理 Linux 工作負載的 HPC Pack 叢集)](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/) 範本，按一下 [部署]。
1. 檢閱 Azure 入口網站中的資訊，然後按一下 [建立] 。
   
    ![建立入口網站][portal]
1. 在 [基本]  刀鋒視窗中，輸入叢集的名稱，這也會指定前端節點 VM 的名稱。 您可以選擇現有的資源群組或建立新的資源群組，以便在您可用的位置進行部署。 位置會影響特定 VM 大小和其他 Azure 服務的可用性 (請參閱[依區域提供的產品](https://azure.microsoft.com/regions/services/))。
1. 如果是第一次部署，通常可接受 [Head node settings] \(前端節點設定)  刀鋒視窗中的預設值。 
   
   > [!NOTE]
   > **[Post-configuration script URL]** \(組態後指令碼 URL) 是選擇性的設定，可指定您想要在前端節點 VM 執行後，於其上執行的公開的 Windows PowerShell 指令碼。 
   > 
   > 
1. 在 [計算節點設定]  刀鋒視窗中，選取節點的命名模式、節點數目與大小，以及要部署的 Linux 散發。
1. 在 [Infrastructure settings] \(基礎結構設定)  刀鋒視窗中，輸入虛擬網路和 Active Directory 網域的名稱、網域和 VM 系統管理員認證，以及儲存體帳戶的命名模式。
   
   > [!NOTE]
   > HPC Pack 會使用 Active Directory 網域來驗證叢集使用者。 
   > 
   > 
1. 當驗證測試執行完畢，且您已檢閱使用條款之後，按一下 [購買] 。

### <a name="deployment-option-2-use-the-iaas-deployment-script"></a>部署選項 2。 使用 IaaS 部署指令碼
使用 HPC Pack IaaS 部署指令碼部署叢集的其他必要條件如下：

* **用戶端電腦** - 您需要有 Windows 型用戶端電腦，才能執行叢集部署指令碼。
* **Azure PowerShell** - [安裝和設定 Azure PowerShell](/powershell/azure/overview) (0.8.10 版或更新版本)。
* **HPC Pack IaaS 部署指令碼** - 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=44949)下載並解壓縮最新版的指令碼。 執行 `.\New-HPCIaaSCluster.ps1 –Version`即可檢查指令碼的版本。 這篇文章根據 4.4.1 版或更新版本的指令碼。

**XML 組態檔**

HPC Pack IaaS 部署指令碼會使用 XML 組態檔作為輸入，以描述 HPC 叢集。 下列範例組態檔會指定一個小型的叢集，由 HPC Pack 前端節點和兩個大小為 A7 的 CentOS 7.0 Linux 計算節點所組成。 

請根據環境和所要的叢集組態視需要修改檔案，並將它以 HPCDemoConfig.xml 之類的名稱儲存。 例如，您需要提供您的訂用帳戶名稱、唯一儲存體帳戶名稱，以及雲端服務名稱。 此外，您可能會想要為計算節點選擇支援的不同 Linux 映像。 如需組態檔中元素的詳細資訊，請參閱指令碼資料夾中的 Manual.rtf 檔案與 [使用 HPC Pack IaaS 部署指令碼建立 HPC 叢集](../../windows/classic/hpcpack-cluster-powershell-script.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

```
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>allvhdsje</StorageAccount>
  </Subscription>
  <Location>Japan East</Location>  
  <VNet>
    <VNetName>centos7rdmavnetje</VNetName>
    <SubnetName>CentOS7RDMACluster</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>CentOS7RDMA-HN</VMName>
    <ServiceName>centos7rdma-je</ServiceName>
  <VMSize>ExtraLarge</VMSize>
  <EnableRESTAPI />
  <EnableWebPortal />
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>CentOS7RDMA-LN%1%</VMNamePattern>
    <ServiceName>centos7rdma-je</ServiceName>
    <VMSize>A7</VMSize>
    <NodeCount>2</NodeCount>
    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150325</ImageName>
  </LinuxComputeNodes>
</IaaSClusterConfig>
```

**執行 HPC Pack IaaS 部署指令碼**

1. 在用戶端電腦上以系統管理員身分開啟 Windows PowerShell。
1. 將目錄變更為安裝指令碼的資料夾 (在本範例中為 E:\IaaSClusterScript)。
   
    ```powershell
    cd E:\IaaSClusterScript
    ```
1. 執行下列命令來部署 HPC Pack 叢集。 這個範例假設組態檔位於 E:\HPCDemoConfig.xml
   
    ```powershell
    .\New-HpcIaaSCluster.ps1 –ConfigFile E:\HPCDemoConfig.xml –AdminUserName MyAdminName
    ```
   
    a. 由於在先前的命令中並未指定 **AdminPassword**，因此系統會提示您輸入使用者 *MyAdminName* 的密碼。
   
    b. 指令碼會接著開始驗證組態檔。 根據網路連線而定，這可能會花費數分鐘的時間。
   
    ![驗證][validate]
   
    c. 通過驗證之後，指令碼會列出要建立的叢集資源。 輸入 *Y* 繼續。
   
    ![資源][resources]
   
    d. 此指令碼會開始部署 HPC Pack 叢集並完成設定，而不需要進一步的手動步驟。 指令碼的執行時間可能會長達數分鐘。
   
    ![部署][deploy]
   
   > [!NOTE]
   > 在此範例中，因為未指定 **-LogFile** 參數，所以指令碼會自動產生記錄檔。 記錄檔並非即時寫入，但在驗證和部署結束時會加以收集。 如果 PowerShell 程序在指令碼執行中時停止，將會遺失某些記錄檔。
   > 
   > 

## <a name="connect-to-the-head-node"></a>連接至前端節點
在 Azure 部署 HPC Pack 叢集之後，請使用您部署叢集時所提供的網域認證 (例如，*hpc\\clusteradmin*) [由遠端桌面連線](../../windows/connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)到前端節點 VM。 您可以從前端節點管理叢集。

在前端節點上，啟動 HPC 叢集管理員來查看 HPC Pack 叢集的狀態。 您可以使用您處理 Windows 運算節點的相同方式，管理和監視 Linux 運算節點。 例如，您會看到 [資源管理] 中列出 Linux 節點 (這些節點都是以 **LinuxNode** 範本部署的)。

![節點管理][management]

您也可以在 [熱度圖]  檢視中看到 Linux 節點。

![熱度圖][heatmap]

## <a name="how-to-move-data-in-a-cluster-with-linux-nodes"></a>如何在具有 Linux 節點的叢集中移動資料
您有數個選擇可在 Linux 節點與叢集的 Windows 前端節點間移動資料。 以下是三種常見的方法，下列各節將詳細說明︰

* **Azure 檔案** - 公開受控 SMB 檔案共用，以在 Azure 儲存體中儲存資料檔案。 Windows 節點和 Linux 節點可同時掛接 Azure 檔案共用作為磁碟機或資料夾，即使它們部署在不同的虛擬網路中。
* **前端節點 SMB 共用** - 在 Linux 節點上裝載前端節點的標準 Windows 共用資料夾。
* **前端節點 NFS 伺服器** - 為混合式 Windows 與 Linux 環境提供檔案共用解決方案。

### <a name="azure-file-storage"></a>Azure 檔案儲存體
[Azure 檔案](https://azure.microsoft.com/services/storage/files/) 服務會公開使用標準 SMB 2.1 通訊協定的檔案共用。 Azure VM 和雲端服務可以透過掛接的共用，在應用程式元件之間共用檔案資料，而內部部署應用程式可以透過檔案儲存體 API，存取共用中的檔案資料。 

如需建立 Azure 檔案共用以及將其裝載於前端節點上的詳細步驟，請參閱 [開始使用 Windows 上的 Azure 檔案儲存體](../../../storage/files/storage-how-to-use-files-windows.md)。 若要在 Linux 節點上裝載 Azure 檔案共用，請參閱[如何搭配使用 Azure 檔案儲存體與 Linux](../../../storage/files/storage-how-to-use-files-linux.md)。 若要設定持續性連線，請參閱 [Microsoft Azure 檔案的持續性連線](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)。

在下列範例中，在儲存體帳戶上建立 Azure 檔案共用。 若要在前端節點上裝載共用，請開啟 [命令提示字元] 並輸入下列命令：

```command
cmdkey /add:allvhdsje.file.core.windows.net /user:allvhdsje /pass:<storageaccountkey>

net use Z: \\allvhdje.file.core.windows.net\rdma /persistent:yes
```

在此範例中，allvhdsje 是您的儲存體帳戶名稱、storageaccountkey 是您的儲存體帳戶金鑰，而 rdma 則是 Azure 檔案共用名稱。 Azure 檔案共用會掛接到前端節點成為 Z:。

若要在 Linux 節點上裝載 Azure 檔案共用，請在前端節點上執行 **clusrun** 命令。 **[Clusrun](https://technet.microsoft.com/library/cc947685.aspx)** 是實用的 HPC Pack 工具，可在多個節點上執行系統管理工作。 (同時請參閱本文中的 [適用於 Linux 節點的 CLusrun](#Clusrun-for-Linux-nodes) )。

開啟 Windows PowerShell 視窗並輸入下列命令：

```powershell
clusrun /nodegroup:LinuxNodes mkdir -p /rdma

clusrun /nodegroup:LinuxNodes mount -t cifs //allvhdsje.file.core.windows.net/rdma /rdma -o vers=2.1`,username=allvhdsje`,password=<storageaccountkey>'`,dir_mode=0777`,file_mode=0777
```

第一個命令會在 LinuxNodes 群組中的所有節點上建立名為 /rdma 的資料夾。 第二個命令會將 Azure 檔案共用 allvhdsjw.file.core.windows.net/rdma 裝載至 /rdma 資料夾，其 dir 和檔案模式位元設為 777。 在第二個命令中，allvhdsje 是您的儲存體帳戶名稱，而 storageaccountkey 是儲存體帳戶金鑰。

> [!NOTE]
> 第二個命令中的 “\`” 符號是 PowerShell 的逸出符號。 “\`,” 表示 “,” (逗號) 是命令的一部分。
> 
> 

### <a name="head-node-share"></a>前端節點共用
或者，在 Linux 節點上裝載前端節點的共用資料夾。 共用提供共用檔案的最簡單方式，但前端節點和所有 Linux 節點必須都部署在相同的虛擬網路中。 步驟如下：

1. 在前端節點上建立資料夾，並將它分享給具有讀取/寫入權限的每個人。 例如，在前端節點上分享 D:\OpenFOAM 成為 \\\CentOS7RDMA-HN\OpenFOAM。 此處的 CentOS7RDMA-HN 是前端節點的主機名稱。
   
    ![檔案共用權限][fileshareperms]
   
    ![檔案共用][filesharing]
1. 開啟 Windows PowerShell 視窗並執行下列命令：
   
    ```powershell
    clusrun /nodegroup:LinuxNodes mkdir -p /openfoam
   
    clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS7RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file_mode=0777
    ```

第一個命令會在 LinuxNodes 群組中的所有節點上建立名為 /openfoam 的資料夾。 第二個命令會將共用資料夾 //CentOS7RDMA-HN/OpenFOAM 裝載至此資料夾，其 dir 和檔案模式位元設為 777。 命令中的使用者名稱和密碼應該是前端節點上叢集使用者的使用者名稱和密碼。 (請參閱 [Add or remove cluster users (新增或移除叢集使用者)](https://technet.microsoft.com/library/ff919330.aspx)。)

> [!NOTE]
> 第二個命令中的 “\`” 符號是 PowerShell 的逸出符號。 “\`,” 表示 “,” (逗號) 是命令的一部分。
> 
> 

### <a name="nfs-server"></a>NFS 伺服器
NFS 服務可讓您使用 SMB 通訊協定在執行 Windows Server 2012 作業系統的電腦間，共用及移轉檔案，以及在使用 NFS 通訊協定在 Linux 架構的電腦間共用及移轉檔案。 NFS 伺服器與其他所有節點，都必須部署在相同的虛擬網路中。 與 SMB 共用相比，它提供更好的 Linux 節點相容性。 例如，它支援檔案連結。

1. 若要安裝並設定 NFS 伺服器，請依照 [網路檔案系統伺服器首次端對端共用](http://blogs.technet.com/b/filecab/archive/2012/10/08/server-for-network-file-system-first-share-end-to-end.aspx)中的步驟。
   
    例如，使用下列屬性來建立名為 nfs 的 NFS 共用：
   
    ![NFS 授權][nfsauth]
   
    ![NFS 共用權限][nfsshare]
   
    ![NFS NTFS 權限][nfsperm]
   
    ![NFS 管理屬性][nfsmanage]
1. 開啟 Windows PowerShell 視窗並執行下列命令：
   
    ```powershell
    clusrun /nodegroup:LinuxNodes mkdir -p /nfsshare
   
    clusrun /nodegroup:LinuxNodes mount CentOS7RDMA-HN:/nfs /nfsshared
    ```
   
   第一個命令會在 LinuxNodes 群組中的所有節點上建立名為 /nfsshared 的資料夾。 第二個命令會將 NFS 共用 CentOS7RDMA-HN:/nfs 裝載到此資料夾。 此處的 CentOS7RDMA-HN:/nfs 是 NFS 共用的遠端路徑。

## <a name="how-to-submit-jobs"></a>如何提交工作
有數種方式可以將工作提交到 HPC Pack 叢集：

* HPC 叢集管理員或 HPC 工作管理員 GUI
* HPC Web 入口網站
* REST API

透過 HPC Pack GUI 工具和 Web 入口網站將工作提交至 Azure 中的叢集，與提交至 Windows 運算節點相同。 請參閱 [HPC Pack Job Manager (HPC Pack 作業管理員)](https://technet.microsoft.com/library/ff919691.aspx) 與[如何從內部部署用戶端電腦提交作業](../../windows/hpcpack-cluster-submit-jobs.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

若要透過 REST API 提交作業，請參閱 [在 Microsoft HPC Pack 中使用 REST API 建立和提交作業](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx)。 若要從 Linux 用戶端提交作業，另請參閱 [HPC Pack SDK](https://www.microsoft.com/download/details.aspx?id=47756)中的 Python 範例。

## <a name="clusrun-for-linux-nodes"></a>適用於 Linux 節點的 CLusrun
HPC Pack [clusrun](https://technet.microsoft.com/library/cc947685.aspx) 工具可透過命令提示字元或 HPC 叢集管理員用來在 Linux 節點上執行命令。 以下有一些基本範例。

* 顯示叢集中所有節點的目前使用者名稱。
  
    ```command
    clusrun whoami
    ```
* 在 linuxnodes 群組中的所有節點上使用 **yum** 安裝 **gdb** 偵錯工具，然後在 10 分鐘後重新啟動節點。
  
    ```command
    clusrun /nodegroup:linuxnodes yum install gdb –y; shutdown –r 10
    ```
* 建立可在叢集中的每個 Linux 節點上一秒顯示數字 1 到 10 的殼層指令碼、加以執行，並立即顯示節點的輸出。
  
    ```command
    clusrun /interleaved /nodegroup:linuxnodes echo \"for i in {1..10}; do echo \\\"\$i\\\"; sleep 1; done\" ^> script.sh; chmod +x script.sh; ./script.sh
    ```

> [!NOTE]
> 您可能需要在 **clusrun** 命令中使用逸出字元。 如此範例所示，在命令提示字元處使用 ^ 來逸出 ">" 符號。
> 
> 

## <a name="next-steps"></a>後續步驟
* 嘗試擴大叢集中的節點數量，或嘗試在叢集上執行 Linux 工作負載。 如需範例，請參閱 [在 Azure 中的 Linux 計算節點以 Microsoft HPC Pack 執行 NAMD](hpcpack-cluster-namd.md)。
* 請嘗試[具備 RDMA 功能、計算密集型 VM](../../windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 的叢集執行 MPI 工作負載。 如需範例，請參閱 [在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 OpenFOAM](hpcpack-cluster-openfoam.md)。
* 如果您有興趣在內部部署的 HPC Pack 叢集中使用 Linux 節點，請參閱 [TechNet guidance (TechNet 指引)](https://technet.microsoft.com/library/mt595803.aspx)。

<!--Image references-->
[scenario]:media/hpcpack-cluster/scenario.png
[portal]:media/hpcpack-cluster/portal.png
[validate]:media/hpcpack-cluster/validate.png
[resources]:media/hpcpack-cluster/resources.png
[deploy]:media/hpcpack-cluster/deploy.png
[management]:media/hpcpack-cluster/management.png
[heatmap]:media/hpcpack-cluster/heatmap.png
[fileshareperms]:media/hpcpack-cluster/fileshare1.png
[filesharing]:media/hpcpack-cluster/fileshare2.png
[nfsauth]:media/hpcpack-cluster/nfsauth.png
[nfsshare]:media/hpcpack-cluster/nfsshare.png
[nfsperm]:media/hpcpack-cluster/nfsperm.png
[nfsmanage]:media/hpcpack-cluster/nfsmanage.png
