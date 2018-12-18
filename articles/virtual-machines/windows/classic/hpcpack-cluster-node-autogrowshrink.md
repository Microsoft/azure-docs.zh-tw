---
title: 自動調整 HPC Pack 叢集節點 | Microsoft Docs
description: 自動在 Azure 中增加和縮減 HPC Pack 叢集計算節點的數目
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: ''
editor: tysonn
ms.assetid: 38762cd1-f917-464c-ae5d-b02b1eb21e3f
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 12/08/2016
ms.author: danlep
ms.openlocfilehash: 4a2350183bc0cb9360e9315cd8a351be20b66584
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31603839"
---
# <a name="automatically-grow-and-shrink-the-hpc-pack-cluster-resources-in-azure-according-to-the-cluster-workload"></a>在 Azure 中根據叢集工作負載自動擴增和縮減 HPC Pack 叢集資源
如果您在 HPC Pack 叢集部署 Azure「高載」節點，或您在 Azure VM 中建立 HPC Pack 叢集，您可能會想要能夠根據叢集上的工作負載，自動增加或縮減叢集資源 (例如節點或核心)。 以這樣的方式調整叢集資源可讓您更有效率地使用您的 Azure 資源及控制其成本。

本文將說明 HPC Pack 提供來自動調整計算資源的兩種方式︰

* HPC Pack 叢集屬性 **AutoGrowShrink**

* **AzureAutoGrowShrink.ps1** HPC PowerShell 指令碼

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

目前您只能自動擴增和縮減正在執行 Windows Server 作業系統的 HPC Pack 計算節點。


## <a name="set-the-autogrowshrink-cluster-property"></a>設定 AutoGrowShrink 叢集屬性
### <a name="prerequisites"></a>先決條件

* **HPC Pack 2012 R2 Update 2 或更新版叢集** - 叢集前端節點可以部署在內部部署或在 Azure VM 中。 請參閱 [使用 HPC Pack 設定混合式叢集](../../../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md) ，以開始使用內部部署前端節點與 Azure「高載」節點。 請參閱 [HPC Pack IaaS 部署指令碼](hpcpack-cluster-powershell-script.md)，以在 Azure VM 中快速部署 HPC Pack 叢集。

* **針對在 Azure (Resource Manager 部署模型) 中具有前端節點的叢集** - 從 HPC Pack 2016 開始，Azure Active Directory 應用程式中的憑證驗證會用於自動增加或縮減利用 Azure Resource Manager 部署的叢集 VM。 設定憑證，如下所示︰

  1. 叢集部署之後，透過遠端桌面連接到一個前端節點。

  2. 將憑證 (具有私密金鑰的 PFX 格式) 上傳至每個前端節點，並且安裝至 Cert:\LocalMachine\My 和 Cert:\LocalMachine\Root。

  3. 以系統管理員身分啟動 Azure PowerShell，並且在一個前端節點上執行下列命令︰

    ```powershell
        cd $env:CCP_HOME\bin

        Connect-AzureRmAccount
    ```
        
    如果您的帳戶是位於多個 Azure Active Directory 租用戶或 Azure 訂用帳戶中，您可以執行下列命令來選取正確的租用戶和訂用帳戶︰
  
    ```powershell
        Connect-AzureRmAccount -TenantId <TenantId> -SubscriptionId <subscriptionId>
    ```     
       
    執行下列命令來檢視目前選取的租用戶和訂用帳戶︰
    
    ```powershell
        Get-AzureRMContext
    ```

  4. 執行下列指令碼

    ```powershell
        .\ConfigARMAutoGrowShrinkCert.ps1 -DisplayName “YourHpcPackAppName” -HomePage "https://YourHpcPackAppHomePage" -IdentifierUri "https://YourHpcPackAppUri" -CertificateThumbprint "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" -TenantId xxxxxxxx-xxxxx-xxxxx-xxxxx-xxxxxxxxxxxx
    ```

    其中

    **DisplayName** - Azure Active 應用程式顯示名稱。 如果應用程式不存在，它會建立在 Azure Active Directory 中。

    **HomePage** - 應用程式的首頁。 您可以設定虛擬 URL，如上述範例所示。

    **IdentifierUri** - 應用程式的識別碼。 您可以設定虛擬 URL，如上述範例所示。

    **CertificateThumbprint** - 您在步驟 1 中的前端節點安裝的憑證指紋。

    **TenantId** - Azure Active Directory 的租用戶識別碼。 您可以從 Azure Active Directory 入口網站 [屬性] 頁面來取得租用戶識別碼。

    如需 **ConfigARMAutoGrowShrinkCert.ps1** 的詳細資訊，請執行 `Get-Help .\ConfigARMAutoGrowShrinkCert.ps1 -Detailed`。


* **針對 Azure (傳統部署模型) 中前端節點的叢集** - 如果您使用 HPC Pack IaaS 部署指令碼在傳統部署模型中建立叢集，請在叢集組態檔中設定 AutoGrowShrink 選項來啟用 **AutoGrowShrink** 叢集屬性。 如需詳細資訊，請參閱文件所隨附的 [指令碼下載](https://www.microsoft.com/download/details.aspx?id=44949)。

    或者，使用下一節所述的 HPC PowerShell 命令，在部署叢集之後，啟用 **AutoGrowShrink** 叢集屬性。 若要為此做準備，請先完成下列步驟︰

  1. 在前端節點和 Azure 訂用帳戶上，設定 Azure 管理憑證。 對於測試部署，您可以使用 HPC Pack 在前端節點上安裝的預設 Microsoft HPC Azure 自我簽署憑證，然後將該憑證上傳至您的 Azure 訂用帳戶即可。 如需選項及步驟，請參閱 [TechNet Library 指南](https://technet.microsoft.com/library/gg481759.aspx)。

  2. 在前端節點上執行 **regedit**，移至 HKLM\SOFTWARE\Micorsoft\HPC\IaasInfo，並新增字串值。 將 Value 名稱設為 “ThumbPrint”，並將 Value 資料設為步驟 1 中的憑證指紋。

### <a name="hpc-powershell-commands-to-set-the-autogrowshrink-property"></a>用來設定 AutoGrowShrink 屬性的 HPC PowerShell 命令
以下是 HPC PowerShell 命令範例，可設定 **AutoGrowShrink** 及使用其他參數調整其行為。 請參閱本文稍後的 [AutoGrowShrink 參數](#AutoGrowShrink-parameters) 以取得完整的設定清單。

若要執行這些命令，在叢集前端節點上以系統管理員身分啟動 HPC PowerShell。

**啟用 AutoGrowShrink 屬性**

```powershell
Set-HpcClusterProperty –EnableGrowShrink 1
```

**停用 AutoGrowShrink 屬性**

```powershell
Set-HpcClusterProperty –EnableGrowShrink 0
```

**以分鐘為單位變更增加間隔**

```powershell
Set-HpcClusterProperty –GrowInterval <interval>
```

**以分鐘為單位變更縮減間隔**

```powershell
Set-HpcClusterProperty –ShrinkInterval <interval>
```

**檢視 AutoGrowShrink 的目前 設定**

```powershell
Get-HpcClusterProperty –AutoGrowShrink
```

**若要從 AutoGrowShrink 排除節點群組**

```powershell
Set-HpcClusterProperty –ExcludeNodeGroups <group1,group2,group3>
```

>[!NOTE] 
> 從 HPC Pack 2016 開始支援這個參數
>

### <a name="autogrowshrink-parameters"></a>AutoGrowShrink 參數
以下是您可以使用 **Set-HpcClusterProperty** 命令修改的 AutoGrowShrink 參數。

* **EnableGrowShrink** -切換以啟用或停用 **AutoGrowShrink** 屬性。
* **ParamSweepTasksPerCore** - 用來增加一個核心的參數式掃蕩工作數目。 預設為每個工作增加一個核心。

  > [!NOTE]
  > HPC Pack QFE KB3134307 會將 **ParamSweepTasksPerCore** 變更為 **TasksPerResourceUnit**。 它根據作業資源類型，而且可以是節點、通訊端或核心。
  >
  >
* **GrowThreshold** - 用於觸發自動增加的排入佇列工作的臨界值。 預設值為 1，這表示如果有 1 或更多的工作處於排入佇列狀態，節點會自動增加。
* **GrowInterval** -以分鐘為單位的觸發自動增加間隔。 預設間隔為 5 分鐘。
* **ShrinkInterval** -以分鐘為單位的觸發自動縮減間隔。 預設間隔為 5 分鐘。
* **ShrinkIdleTimes** - 指出節點閒置前，持續檢查縮減的數目。 預設值為 3 次。 例如，如果 **ShrinkInterval** 為 5 分鐘，HPC Pack 每隔 5 分鐘會檢查節點是否閒置。 如果節點連續 3 次檢查 (15 分鐘) 處於閒置狀態，HPC Pack 會縮縮該節點。
* **ExtraNodesGrowRatio** - 針對訊息傳遞介面 (MPI) 作業，要增加的節點的額外百分比。 預設值為 1，表示 HPC Pack 針對 MPI 作業增加節點 1%。
* **GrowByMin** - 用來指出自動增加原則是否會根據作業所需的最少資源的參數。 預設值是 false，這表示 HPC Pack 會根據作業所需的資源數上限，為工作增加節點。
* **SoaJobGrowThreshold** - 傳入 SOA 要求以觸發自動增加程序的臨界值。 預設值是 50000。

  > [!NOTE]
  > 從 HPC Pack 2012 R2 Update 3 開始支援這個參數。
  >
  >
* **SoaRequestsPerCore** - 用來增加一個核心的 傳入 SOA 要求數目。 預設值是 20000。

  > [!NOTE]
  > 從 HPC Pack 2012 R2 Update 3 開始支援這個參數。
  >
* **ExcludeNodeGroups** – 指定節點群組中的節點不會自動增加和縮減。
  
  > [!NOTE]
  > 從 HPC Pack 2016 開始支援這個參數。
  >

### <a name="mpi-example"></a>MPI 範例
根據預設，HPC Pack 會針對 MPI 作業額外增加 1% 節點 (**ExtraNodesGrowRatio** 設為 1)。 原因是 MPI 可能需要多個節點，且只有在準備好所有節點時才能執行作業。 當 Azure 啟動節點時，一個節點偶爾可能需要比其他節點更多的時間才能啟動，使得其他節點在等候該節點就緒時會處於閒置狀態。 藉由增加額外的節點，HPC Pack 會減少此資源的等候時間，並可能節省成本。 若要提升 MPI 作業的額外節點百分比 (例如 10%)，請執行 類似以下的命令

    Set-HpcClusterProperty -ExtraNodesGrowRatio 10

### <a name="soa-example"></a>SOA 範例
根據預設，**SoaJobGrowThreshold** 設為 50000，而 **SoaRequestsPerCore** 設為 20000。 如果您送出一個包含 70000 個要求的 SOA 作業，則會有一個排入佇列工作，且傳入要求為 70000。 在此情況下，HPC Pack 會針對排入佇列工作增加 1 個核心，並針對傳入要求，將增加 (70000 - 50000)/20000 = 1 核心，因此總共會為此 SOA 作業增加 2 個核心。

## <a name="run-the-azureautogrowshrinkps1-script"></a>執行 AzureAutoGrowShrink.ps1 指令碼
### <a name="prerequisites"></a>先決條件

* **HPC Pack 2012 R2 更新 1 或更新版本叢集** - **AzureAutoGrowShrink.ps1** 指令碼會安裝在 %CCP_HOME%bin 資料夾中。 叢集前端節點可以部署在內部部署或在 Azure VM 中。 請參閱 [使用 HPC Pack 設定混合式叢集](../../../cloud-services/cloud-services-setup-hybrid-hpcpack-cluster.md) ，以開始使用內部部署前端節點與 Azure「高載」節點。 請參閱 [HPC Pack IaaS 部署指令碼](hpcpack-cluster-powershell-script.md)，以在 Azure VM 中快速部署 HPC Pack 叢集，或使用 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/create-hpc-cluster/)。
* **Azure PowerShell 1.4.0** - 指令碼目前取決於此特定版本的 Azure PowerShell。
* **針對具備 Azure 高載節點的叢集** - 在已安裝 HPC Pack 的用戶端電腦上或在前端節點上執行指令碼。 如果是在用戶端電腦上執行，請確定您設定變數 $env:CCP_SCHEDULER 以指向前端節點。 Azure「高載」節點必須新增至叢集，但它們可能處於「未部署」狀態。
* **針對 Azure VM (Resource Manager 部署模型) 中部署的叢集** - 針對在 Resource Manager 部署模型中部署的 Azure VM 的叢集，指令碼支援兩個 Azure 驗證的方法：登入您的 Azure 帳戶以便每次執行指令碼 (藉由執行 `Connect-AzureRmAccount`，或設定服務主體以使用憑證進行驗證。 HPC Pack 提供指令碼 **ConfigARMAutoGrowShrinkCert.ps** 來使用憑證建立服務主體。 指令碼會建立 Azure Active Directory (Azure AD) 應用程式和服務主體，並將參與者角色指派給服務主體。 若要執行指令碼，以系統管理員身分啟動 Azure PowerShell，並且執行下列命令︰

    ```powershell
    cd $env:CCP_HOME\bin

    Connect-AzureRmAccount

    .\ConfigARMAutoGrowShrinkCert.ps1 -DisplayName “YourHpcPackAppName” -HomePage "https://YourHpcPackAppHomePage" -IdentifierUri "https://YourHpcPackAppUri" -PfxFile "d:\yourcertificate.pfx"
    ```

    如需 **ConfigARMAutoGrowShrinkCert.ps1** 的詳細資訊，請執行 `Get-Help .\ConfigARMAutoGrowShrinkCert.ps1 -Detailed`。

* **針對部署在 Azure VM (傳統部署模型) 中的叢集** - 在前端節點 VM 執行指令碼，因為它相依於所安裝的 **Start-HpcIaaSNode.ps1** 和 **Stop-HpcIaaSNode.ps1** 指令碼。 這些指令碼額外需要 Azure 管理憑證或發佈設定檔 (請參閱 [在 Azure 中的 HPC Pack 叢集管理計算節點](hpcpack-cluster-node-manage.md))。 請確定您所需的所有計算節點 VM 已加入叢集。 它們可能處於「已停止」狀態。



### <a name="syntax"></a>語法
```powershell
AzureAutoGrowShrink.ps1 [-NodeTemplates <String[]>] [-JobTemplates <String[]>] [-NodeType <String>]
    -NumOfActiveQueuedTasksPerNodeToGrow <Single> [-NumOfActiveQueuedTasksToGrowThreshold <Int32>]
    [-NumOfInitialNodesToGrow <Int32>] [-GrowCheckIntervalMins <Int32>] [-ShrinkCheckIntervalMins <Int32>]
    [-ShrinkCheckIdleTimes <Int32>] [-ExtraNodesGrowRatio <Int32>] [-ArgFile <String>] [-LogFilePrefix <String>]
    [<CommonParameters>]

AzureAutoGrowShrink.ps1 [-NodeTemplates <String[]>] [-JobTemplates <String[]>] [-NodeType <String>]
    -NumOfQueuedJobsPerNodeToGrow <Single> [-NumOfQueuedJobsToGrowThreshold <Int32>] [-NumOfInitialNodesToGrow
    <Int32>] [-GrowCheckIntervalMins <Int32>] [-ShrinkCheckIntervalMins <Int32>] [-ShrinkCheckIdleTimes <Int32>]
    [-ExtraNodesGrowRatio <Int32>] [-ArgFile <String>] [-LogFilePrefix <String>] [<CommonParameters>]

AzureAutoGrowShrink.ps1 -UseLastConfigurations [-ArgFile <String>] [-LogFilePrefix <String>] [<CommonParameters>]
```
### <a name="parameters"></a>參數
* **NodeTemplates** - 定義增加和縮減節點範圍的節點範本的名稱。 如果未指定 (預設值是 @())，當 **NodeType** 具有 AzureNodes 的值時，**AzureNodes** 節點群組中的所有節點會在範圍內，而且當 **NodeType** 具有 ComputeNodes 的值時，**ComputeNodes** 節點群組中的所有節點會在範圍內。
* **JobTemplates** - 用以定義節點增加範圍的作業範本名稱。
* **NodeType** - 增加和縮減節點的類型。 支援的值包括：

  * **AzureNodes** - 針對內部部署或 Azure IaaS 叢集中的 Azure PaaS (高載) 節點。
  * **ComputeNodes** - 針對只在 Azure IaaS 叢集的計算節點 VM。

* **NumOfQueuedJobsPerNodeToGrow** - 要增加一個節點所需的已佇列工作數目。
* **NumOfQueuedJobsToGrowThreshold** - 要開始增加處理程序的已佇列工作的臨界值數目。
* **NumOfActiveQueuedTasksPerNodeToGrow** - 要增加一個節點所需的作用中已佇列工作的數目。 如果 **NumOfQueuedJobsPerNodeToGrow** 指定了大於 0 的值，則會忽略這個參數。
* **NumOfActiveQueuedTasksToGrowThreshold** - 要開始增加處理程序的作用中已佇列工作的臨界值數目。
* **NumOfInitialNodesToGrow** - 如果範圍中的所有節點為**未部署**或**已停止 (取消配置)**，要增加的初始最小節點數目。
* **GrowCheckIntervalMins** - 要進行增加的檢查之間的間隔分鐘數。
* **ShrinkCheckIntervalMins** - 要進行縮減的檢查之間的間隔分鐘數。
* **ShrinkCheckIdleTimes** - 連續縮減檢查的數目 (以 **ShrinkCheckIntervalMins** 分隔)，以指出節點為閒置。
* **UseLastConfigurations** - 儲存於引數檔的先前的組態。
* **ArgFile**- 用來儲存並更新組態以執行指令碼的引數檔案的名稱。
* **LogFilePrefix** - 記錄檔的首碼名稱。 您可以指定路徑。 記錄檔預設會寫入目前的工作目錄。

### <a name="example-1"></a>範例 1
下列範例會設定 Azure 高載節點，其中部署了預設 AzureNode 範本以自動增加和縮減。 如果所有節點最初的狀態為 **未部署** 狀態，則至少啟動了 3 個節點。 如果佇列工作數目超過 8 個，指令碼會啟動節點，直到它們的數目超過 **NumOfQueuedJobsPerNodeToGrow**已佇列工作的比率。 如果節點被發現處於閒置達 3 個連續閒置時間，則會將它停止。

```powershell
.\AzureAutoGrowShrink.ps1 -NodeTemplates @('Default AzureNode
 Template') -NodeType AzureNodes -NumOfQueuedJobsPerNodeToGrow 5
 -NumOfQueuedJobsToGrowThreshold 8 -NumOfInitialNodesToGrow 3
 -GrowCheckIntervalMins 1 -ShrinkCheckIntervalMins 1 -ShrinkCheckIdleTimes 3
```

### <a name="example-2"></a>範例 2
下列範例會設定 Azure 計算節點 VM，其中部署了預設 ComputeNode 範本以自動增加和縮減。
預設工作範本所設定的工作會定義叢集上工作負載的範圍。 如果所有節點最初為已停止，則至少啟動了 5 個節點。 如果使用中任務數目超過 15 個，指令碼會啟動節點，直到它們的數目超過 **NumOfActiveQueuedTasksPerNodeToGrow**使用中已佇列任務的比率。 如果節點被發現處於閒置達 10 個連續閒置時間，則會將它停止。

```powershell
.\AzureAutoGrowShrink.ps1 -NodeTemplates 'Default ComputeNode Template' -JobTemplates 'Default' -NodeType ComputeNodes -NumOfActiveQueuedTasksPerNodeToGrow 10 -NumOfActiveQueuedTasksToGrowThreshold 15 -NumOfInitialNodesToGrow 5 -GrowCheckIntervalMins 1 -ShrinkCheckIntervalMins 1 -ShrinkCheckIdleTimes 10 -ArgFile 'IaaSVMComputeNodes_Arg.xml' -LogFilePrefix 'IaaSVMComputeNodes_log'
```
