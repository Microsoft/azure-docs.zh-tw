---
title: 設定 Windows RDMA 叢集以執行 MPI 應用程式 | Microsoft Docs
description: 了解如何建立具有 H16r、H16mr、A8 或 A9 大小之 VM 的 Windows HPC Pack 叢集，以使用 Azure RDMA 網路來執行 MPI 應用程式。
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,hpc-pack
ms.assetid: 7d9f5bc8-012f-48dd-b290-db81c7592215
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 03/06/2018
ms.author: danlep
ms.openlocfilehash: 7bb49e48f3777304aa6f40cee40e0b7147994201
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345232"
---
# <a name="set-up-a-windows-rdma-cluster-with-hpc-pack-to-run-mpi-applications"></a>使用 HPC Pack 設定 Windows RDMA 叢集以執行 MPI 應用程式
使用 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) \(英文\) 和[支援 RDMA 的 HPC VM 大小](../sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#rdma-capable-instances)在 Azure 中設定 Windows RDMA 叢集，以執行平行訊息傳遞介面 (MPI) 應用程式。 當您在 HPC Pack 叢集中設定了支援 RDMA 的 Windows Server 型節點時，MPI 應用程式會在 Azure 中透過以遠端直接記憶體存取 (RDMA) 技術為基礎的低延遲、高輸送量網路，有效率地進行通訊。

## <a name="hpc-pack-cluster-deployment-options"></a>HPC Pack 叢集部署選項
Microsoft HPC Pack 是免費提供的工具，可在內部部署環境中或 Azure 中建立 HPC 叢集來執行 Windows 或 Linux HPC 應用程式。 HPC Pack 包含 Windows 訊息傳遞介面 (MS-MPI) 之 Microsoft 實作的執行階段環境。 與支援 RDMA 且執行支援之 Windows Server 作業系統的執行個體搭配使用時，HPC Pack 可提供一個有效率的選項來執行存取 Azure RDMA 網路的 Windows MPI 應用程式。 

本文介紹兩個案例，並提供使用 Microsoft HPC Pack 2012 R2 設定 Windows RDMA 叢集之詳細指導方針的連結。 

* 案例 1. 部署計算密集型背景工作角色執行個體 (PaaS)
* 案例 2. 在大量運算 VM 中部署運算節點 (IaaS)

## <a name="scenario-1-deploy-compute-intensive-worker-role-instances-paas"></a>案例 1︰部署計算密集型背景工作角色執行個體 (PaaS)
從現有的 HPC Pack 叢集，在執行於雲端服務 (PaaS) 的 Azure 背景工作角色執行個體 (Azure 節點) 中新增額外的運算資源。 此功能也稱為使用 HPC Pack「將暴增的工作負載移至 Azure」，可支援多種大小的背景工作角色執行個體。 新增 Azure 節點時，指定其中一個支援 RDMA 的大小。

以下是從現有的 (通常是內部部署) 叢集將暴增的工作負載移至支援 RDMA 之 Azure 執行個體的考量與步驟。 使用類似的程序，可將背景工作角色執行個體新增至部署在 Azure VM 中的 HPC Pack 前端節點。

> [!NOTE]
> 如需使用 HPC Pack 將暴增的工作負載移至 Azure 的教學課程，請參閱 [使用 HPC Pack 設定混合式叢集](../../../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md)。 請注意下列步驟中僅適用於支援 RDMA 之 Azure 節點的考量。
> 
> 

![將暴增的工作負載移至 Azure][burst]

### <a name="steps"></a>步驟
1. **部署及設定 HPC Pack 2012 R2 前端節點**
   
    從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=49922)下載 HPC Pack 安裝套件。 如需 Azure 高載部署準備工作的需求和指示，請參閱[使用 Microsoft HPC Pack 將暴增的工作負載移至 Azure 背景工作執行個體](https://technet.microsoft.com/library/gg481749.aspx)。
2. **在 Azure 訂用帳戶中設定管理憑證**
   
    設定憑證以保護前端節點與 Azure 之間的連線。 如需相關選項和程序，請參閱 [為 HPC Pack 設定 Azure 管理憑證的案例](http://technet.microsoft.com/library/gg481759.aspx)。 針對測試部署，HPC Pack 會安裝可快速上傳至您的 Azure 訂用帳戶的預設 Microsoft HPC Azure 管理憑證。
3. **建立新的雲端服務和儲存體帳戶**
   
    使用 Azure 入口網站，建立適用於部署的雲端服務 (傳統) 和儲存體帳戶 (傳統)。 在有提供您想要使用之 H 系列、A8 或 A9 大小的區域中建立這些資源。 請參閱[不同區域的 Azure 產品](https://azure.microsoft.com/regions/services/)。

4. **建立 Azure 節點範本**
   
    使用 [HPC 叢集管理員] 中的 [建立節點範本精靈]。 如需相關步驟，請參閱「使用 Microsoft HPC Pack 部署 Azure 節點的步驟」中的 [建立 Azure 節點範本](http://technet.microsoft.com/library/gg481758.aspx#BKMK_Templ) 。
   
    為了進行初始測試，建議您在範本中設定手動可用性原則。
5. **將節點新增至叢集**
   
    使用 [HPC 叢集管理員] 中的 [新增節點精靈]。 如需詳細資訊，請參閱 [將 Azure 節點新增至 Windows HPC 叢集](http://technet.microsoft.com/library/gg481758.aspx#BKMK_Add)。
   
    指定節點的大小時，請選取其中一個支援 RDMA 的執行個體大小。
   
   > [!NOTE]
   > 在每個使用計算密集型執行個體來進行的「將暴增的工作負載移至 Azure」部署中，除了您指定的 Azure 背景工作角色執行個體以外，HPC Pack 還會自動部署至少兩個支援 RDMA 的執行個體 (例如 A8) 作為 Proxy 節點。 Proxy 節點會使用配置給訂用帳戶的核心，因此會隨著 Azure 背景工作角色執行個體產生費用。
   > 
   > 
6. **啟動 (佈建) 節點並使其上線執行工作**
   
    選取節點，然後使用 [HPC 叢集管理員] 中的 [啟動]  動作。 佈建完成時，請選取節點，然後使用 [HPC 叢集管理員] 中的 [上線]  動作。 節點已備妥而可執行工作。
7. **將工作提交至叢集**
   
   使用 HPC Pack 工作提交工具來執行叢集工作。 請參閱 [Microsoft HPC Pack：工作管理](http://technet.microsoft.com/library/jj899585.aspx)。
8. **停止 (解除佈建) 節點**
   
   工作執行完成後，請讓節點離線，然後使用 [HPC 叢集管理員] 中的 [停止]  動作。

## <a name="scenario-2-deploy-compute-nodes-in-compute-intensive-vms-iaas"></a>案例 2︰在大量計算 VM 中部署計算節點 (IaaS)
在此案例中，您會在 Azure 虛擬網路中的 VM 上部署 HPC Pack 前端節點和叢集計算節點。 HPC Pack 提供一些 [Azure VM 中的部署選項](../../windows/hpcpack-cluster-options.md)，包括自動部署指令碼和 Azure 快速入門範本。 舉例來說，下列考量和步驟會引導您使用 [HPC Pack IaaS 部署指令碼](hpcpack-cluster-powershell-script.md)，在 Azure 中自動部署 HPC Pack 2012 R2 叢集。

![Azure VM 中的叢集][iaas]

### <a name="steps"></a>步驟
1. **在用戶端電腦上執行 HPC Pack IaaS 部署指令碼，以建立叢集前端節點和運算節點 VM**
   
    從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=44949)下載 HPC Pack IaaS 部署指令碼套件。
   
    若要進行用戶端電腦的準備工作，請建立指令碼組態檔並執行指令碼；請參閱 [使用 HPC Pack IaaS 部署指令碼建立 HPC 叢集](hpcpack-cluster-powershell-script.md)。 
   
    如需部署支援 RDMA 計算節點的考量，請參閱[高效能運算 VM 大小](../sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#rdma-capable-instances)並注意以下內容：
   
   * **虛擬網路**：在有提供您想要使用之 H 系列、A8 或 A9 大小的區域中指定新的虛擬網路。 請參閱[不同區域的 Azure 產品](https://azure.microsoft.com/regions/services/)。

   * **Windows Server 作業系統**：若要支援 RDMA 連線能力，請為計算節點 VM 指定可相容的 Windows Server 作業系統 (例如 Windows Server 2012 R2)。
   * **雲端服務**：由於指令碼使用傳統部署模型，因此會使用 Azure 雲端服務 (設定檔中的 `ServiceName` 設定) 來部署叢集 VM。 建議您將前端節點部署在一個雲端服務中，並將計算節點部署在另一個雲端服務中。 
   * **前端節點大小**：在此案例中，請考慮至少使用 A4 大小 (超大) 的前端節點。
   * **HpcVmDrivers 延伸模組**：當您使用 Windows Server 作業系統部署 A8 或 A9 大小的計算節點時，部署指令碼會自動安裝 Azure VM 代理程式和 HpcVmDrivers 延伸模組。 HpcVmDrivers 會在運算節點 VM 上安裝驅動程式，使其可以連接到 RDMA 網路。 在支援 RDMA 的 H 系列 VM 上，您必須手動安裝 HpcVmDrivers 擴充功能。 請參閱[高效能運算 VM 大小](../sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
   * **叢集網路組態**：部署指令碼會自動以「拓撲 5」(所有節點在企業網路上) 設定 HPC Pack 叢集。 VM 中的所有 HPC Pack 叢集部署都需要此拓撲。 後續請勿變更此叢集網路拓撲。
1. **讓運算節點上線執行工作**
   
    選取節點，然後使用 [HPC 叢集管理員] 中的 [上線]  動作。 節點已備妥而可執行工作。
3. **將工作提交至叢集**
   
    連接到前端節點以提交工作，或設定內部部署電腦來執行這項操作。 如需相關資訊，請參閱 [將工作提交至 Azure 中的 HPC 叢集](../../virtual-machines-windows-hpcpack-cluster-submit-jobs.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
4. **讓節點離線並加以停止 (取消配置)**
   
    工作執行完成後，請使用 [HPC 叢集管理員] 讓節點離線。 然後，使用 Azure 管理工具將其關閉。

## <a name="run-mpi-applications-on-the-cluster"></a>在叢集上執行 MPI 應用程式
### <a name="example-run-mpipingpong-on-an-hpc-pack-cluster"></a>範例：在 HPC Pack 叢集上執行 mpipingpong
若要驗證支援 RDMA 之執行個體的 HPC Pack 部署，請在叢集上執行 HPC Pack **mpipingpong** 命令。 **mpipingpong** 會在成對的節點之間重複傳送資料封包，為具有 RDMA 功能的應用程式網路計算延遲和輸送量的量值和統計資料。 此範例說明使用叢集 **mpiexec** 命令執行 MPI 工作的一般模式 (在此案例中為 **mpipingpong**)。

此範例假設您在「將暴增的工作負載移至 Azure」組態中新增了 Azure 節點 ([案例 1](#scenario-1.-deploy-compute-intensive-worker-role-instances-\(PaaS\) in this article) 如果您已在 Azure VM 的叢集上部署 HPC Pack，您就必須修改命令語法以指定不同節點群組並設定其他環境變數，將網路流量導向至 RDMA 網路。

若要在叢集上執行 mpipingpong：

1. 在前端節點上或正確設定的用戶端電腦上，開啟命令提示字元。
2. 若要估計有四個節點的 Azure 高載部署中的成對節點之間的延遲，請輸入下列提交作業的命令，使用小型封包和許多反覆項目執行 mpipingpong：
   
    ```Command
    job submit /nodegroup:azurenodes /numnodes:4 mpiexec -c 1 -affinity mpipingpong -p 1:100000 -op -s nul
    ```
   
    命令會傳回已提交之工作的 ID。
   
    如果您將 HPC Pack 叢集部署在 Azure VM 上，請指定在單一雲端服務中部署的運算節點 VM 所屬的節點群組，並修改 **mpiexec** 命令，如下所示：
   
    ```Command
    job submit /nodegroup:vmcomputenodes /numnodes:4 mpiexec -c 1 -affinity -env MSMPI_DISABLE_SOCK 1 -env MSMPI_PRECONNECT all -env MPICH_NETMASK 172.16.0.0/255.255.0.0 mpipingpong -p 1:100000 -op -s nul
    ```
3. 工作完成後，若要檢視輸出 (在此案例中，是指工作之任務 1 的輸出)，請輸入下列命令
   
    ```Command
    task view <JobID>.1
    ```
   
    其中，&lt;*JobID*&gt; 是已提交之工作的 ID。
   
    輸出包含如下的延遲結果。
   
    ![Ping pong 延遲][pingpong1]
4. 若要估計 Azure 高載成對節點之間的延遲，請輸入下列提交工作的命令，使用大型封包和少量反覆項目執行 **mpipingpong**：
   
    ```Command
    job submit /nodegroup:azurenodes /numnodes:4 mpiexec -c 1 -affinity mpipingpong -p 4000000:1000 -op -s nul
    ```
   
    命令會傳回已提交之工作的 ID。
   
    在部署於 Azure VM 的 HPC Pack 叢集上，修改在步驟 2 中記下的命令。
5. 工作完成後，若要檢視輸出 (在此案例中，是指工作之任務 1 的輸出)，請輸入下列命令：
   
    ```Command
    task view <JobID>.1
    ```
   
   輸出包含如下的輸送量結果。
   
   ![Ping pong 輸送量][pingpong2]

### <a name="mpi-application-considerations"></a>MPI 應用程式考量
以下是在 Azure 中使用 HPC Pack 執行 MPI 應用程式時的考量。 某些考量只適用於 Azure 節點的部署 (在「將暴增的工作負載移至 Azure」組態中新增的背景工作角色執行個體)。

* Azure 會定期重新佈建雲端服務中的背景工作角色執行個體，不另行通知 (例如為了維護系統，或是因為執行個體失敗)。 如果在執行個體執行 MPI 作業時重新佈建執行個體，該執行個體將會遺失其資料，並回復到一開始部署時的狀態，這會導致 MPI 工作失敗。 您為單一 MPI 工作使用的節點愈多，工作執行的時間愈久，且其中一個執行個體就愈有可能在工作執行時重新佈建。 如果您在部署中將單一節點指定為檔案伺服器，也要考量這一點。
* 若要在 Azure 中執行 MPI 作業，並不需要使用支援 RDMA 的執行個體。 您可以使用 HPC Pack 支援的任何執行個體大小。 不過，建議使用支援 RDMA 的執行個體來執行相對上較大規模、對連接節點之網路的延遲和頻寬較為敏感的 MPI 作業。 如果您使用其他大小來執行對延遲和頻寬較敏感的 MPI 作業，建議您執行其單一工作只在幾個節點上執行的小型作業。
* 部署至 Azure 執行個體的應用程式，會受限於應用程式的相關授權條款。 請向廠商確認任何商業應用程式在雲端中的執行授權或其他限制。 並非所有廠商都提供隨用隨付授權。
* Azure 執行個體需要進一步的安裝，才能存取內部部署節點、共用和授權伺服器。 例如，若要讓 Azure 節點能夠存取內部部署授權伺服器，您可以設定站對站 Azure 虛擬網路。
* 若要在 Azure 執行個體上執行 MPI 應用程式，請執行 **hpcfwutil** 命令，對 Windows 防火牆註冊執行個體上的每個 MPI 應用程式。 如此，即可在防火牆動態指派的連接埠上進行 MPI 通訊。
  
  > [!NOTE]
  > 針對「將量擴大到 Azure」部署，您也可以設定防火牆例外狀況命令，使其在所有新增至您的叢集的新 Azure 節點上自動執行。 在您執行 **hpcfwutil** 命令並驗證應用程式可運作之後，請將命令新增至 Azure 節點的啟動指令碼。 如需詳細資訊，請參閱 [Use a Startup Script for Azure Nodes (使用 Azure 節點的啟動指令碼)](https://technet.microsoft.com/library/jj899632.aspx)。
  > 
  > 
* HPC Pack 會使用 CCP_MPI_NETMASK 叢集環境變數，來指定 MPI 通訊可接受的位址範圍。 從 HPC Pack 2012 R2 開始，CCP_MPI_NETMASK 叢集環境變數只對加入網域的叢集運算節點之間的 MPI 通訊 (內部部署或在 Azure VM 中) 具有效用。 在「將暴增的工作負載移至 Azure」組態中新增的節點會忽略此變數。
* MPI 工作無法在部署於不同雲端服務的 Azure 執行個體之間執行 (例如，在使用不同節點範本的「將量擴大到 Azure」部署之間，或部署在多個雲端服務中的 Azure VM 運算節點之間)。 如果您有多個以不同節點範本起始的 Azure 節點部署，MPI 工作就必須一組 Azure 節點上執行。
* 當您將 Azure 節點新增至您的叢集並使其上線時，HPC 工作排程器服務會立即嘗試在節點上啟動工作。 如果您的工作負載只有一部分可在 Azure 上執行，請確實更新或建立工作範本，以定義可在 Azure 上執行的工作類型。 例如，若要確保使用工作範本的已提交工作只會在 Azure 節點上執行，請在工作範本中新增 [節點群組] 屬性，並選取 AzureNodes 做為必要的值。 若要為您的 Azure 節點建立自訂群組，請使用 Add-HpcGroup HPC PowerShell Cmdlet。

## <a name="next-steps"></a>後續步驟
* 除了使用 HPC Pack 之外，還可以在 Azure 中計算節點的受控集區上使用 Azure Batch 服務來進行開發以執行 MPI 應用程式。 請參閱 [在 Azure Batch 中使用多重執行個體工作來執行訊息傳遞介面 (MPI) 應用程式](../../../batch/batch-mpi.md)。

<!--Image references-->
[burst]:media/hpcpack-rdma-cluster/burst.png
[iaas]:media/hpcpack-rdma-cluster/iaas.png
[pingpong1]:media/hpcpack-rdma-cluster/pingpong1.png
[pingpong2]:media/hpcpack-rdma-cluster/pingpong2.png
