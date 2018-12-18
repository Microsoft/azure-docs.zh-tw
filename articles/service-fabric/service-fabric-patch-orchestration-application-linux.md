---
title: 適用於 Linux 的 Azure Service Fabric 修補程式協調流程應用程式 | Microsoft Docs
description: 在 Linux Service Fabric 叢集上將作業系統修補自動化的應用程式。
services: service-fabric
documentationcenter: .net
author: novino
manager: timlt
editor: ''
ms.assetid: de7dacf5-4038-434a-a265-5d0de80a9b1d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 5/22/2018
ms.author: nachandr
ms.openlocfilehash: 0aadb5964b5fe08b02397588dd9b2695fb4db4ce
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746712"
---
# <a name="patch-the-linux-operating-system-in-your-service-fabric-cluster"></a>修補 Service Fabric 叢集中的 Linux 作業系統

> [!div class="op_single_selector"]
> * [Windows](service-fabric-patch-orchestration-application.md)
> * [Linux](service-fabric-patch-orchestration-application-linux.md)
>
>

修補程式協調流程應用程式是 Azure Service Fabric 應用程式，可在 Service Fabric 叢集上將作業系統修補自動化，而不需要停機。

修補程式協調流程應用程式提供下列功能：

- **自動化的作業系統更新安裝**。 會自動下載並安裝作業系統更新。 會視需要重新啟動叢集節點，而不需要叢集停機。

- **叢集感知的修補和健康情況整合**。 套用更新時，修補程式協調流程應用程式會監視叢集節點的健康情況。 叢集節點一次只能升級一個節點，或一次升級一個網域。 如果因修補程序而造成叢集的健康情況下降，修補就會停止，防止問題繼續惡化。

## <a name="internal-details-of-the-app"></a>應用程式的內部詳細資料

修補程式協調流程應用程式包含下列子元件︰

- **協調器服務**︰是具狀態服務，負責：
    - 協調整個叢集上的 OS 更新作業。
    - 儲存已完成之 OS 更新作業的結果。
- **節點代理程式服務**︰是無狀態服務，在所有 Service Fabric 叢集節點上執行。 此服務負責：
    - 啟動 Linux 上的節點代理程式精靈。
    - 監視精靈服務。
- **節點代理程式精靈**：此 Linux 精靈服務以較高層級的權限 (根) 執行。 相比之下，節點代理程式服務和協調器服務則以較低層級的權限執行。 此服務會負責執行下列所有叢集節點上的更新作業：
    - 將節點上的自動 OS 更新停用。
    - 根據使用者提供的原則下載並安裝 OS 更新。
    - 在 OS 更新安裝後視需要重新啟動機器。
    - 將 OS 更新的結果上傳至協調員服務。
    - 萬一耗盡所有重試之後作業還是失敗，就報告健康情況報告。

> [!NOTE]
> 修補程式協調流程應用程式是使用 Service Fabric 的修復管理器系統服務，將節點停用或啟用以及執行健康情況檢查。 修補程式協調流程應用程式所建立的修復工作會追蹤每個節點的更新進度。

## <a name="prerequisites"></a>必要條件

### <a name="ensure-that-your-azure-vms-are-running-ubuntu-1604"></a>確定您的 Azure VM 執行 Ubuntu 16.04
在撰寫本文時，Ubuntu 16.04 (`Xenial Xerus`) 是唯一支援的版本。

### <a name="ensure-that-the-service-fabric-linux-cluster-is-version-62x-and-above"></a>確定 Service Fabric Linux 叢集的版本是 6.2.x 和更新版本

修補程式協調流程應用程式 Linux 會使用只能在 Service Fabric 執行階段 6.2.x 版和更新版本中使用的特定執行階段功能。

### <a name="enable-the-repair-manager-service-if-its-not-running-already"></a>啟用修復管理器服務 (如果尚未執行中)

叢集必須啟用修復管理器系統服務，修補程式協調流程應用程式才能執行。

#### <a name="azure-clusters"></a>Azure 叢集

銀級和金級耐久性層中的 Azure Linux 叢集依預設會啟用修復管理員服務。 銅級耐久性層中的 Azure 叢集依預設不會啟用修復管理器服務。 如果已啟用服務，可以在 Service Fabric Explorer 的系統服務區段中看到它正在執行。

##### <a name="azure-portal"></a>Azure 入口網站
您可以在設定叢集時從 Azure 入口網站啟用修復管理員。 設定叢集時，選取 [附加元件功能] 底下的 [包含修復管理員] 選項。
![從 Azure 入口網站啟用修復管理員的映像](media/service-fabric-patch-orchestration-application/EnableRepairManager.png)

##### <a name="azure-resource-manager-deployment-model"></a>Azure Resource Manager 部署模型
或者，您可以使用 [Azure Resource Manager 部署模型](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)，在新的和現有的 Service Fabric 叢集上啟用修復管理員服務。 取得您想要部署之叢集的範本。 您可以使用範例範本，或建立自訂的 Azure Resource Manager 部署模型範本。 

若要使用 [Azure Resource Manager 部署模型範本](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)啟用修復管理員服務：

1. 首先，確認 `Microsoft.ServiceFabric/clusters` 資源的 `apiversion` 已設為 `2017-07-01-preview`。 如果不是，您就需要將 `apiVersion` 更新為 `2017-07-01-preview` 值或更高：

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. 立即啟用修復管理器服務，方法是在 `fabricSettings` 區段之後新增下列 `addonFeatures` 區段：

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. 一旦您使用這些變更將叢集範本進行更新之後，將它們加以套用，使升級完成。 您現在可以看到修復管理器系統服務在您的叢集中執行。 它在 Service Fabric Explorer 的系統服務區段中叫作 `fabric:/System/RepairManagerService`。 

### <a name="standalone-on-premises-clusters"></a>獨立的內部部署叢集

在撰寫本文時，尚不支援獨立 Service Fabric Linux 叢集。

### <a name="disable-automatic-os-update-on-all-nodes"></a>停用所有節點上的自動 OS 更新

自動 OS 更新可能會導致執行中的應用程式失去可用性或改變行為。 依預設，修補程式協調流程應用程式會在每個叢集節點上嘗試將自動 OS 更新停用，以防止這種情況發生。
修補程式協調流程應用程式會停用 Ubuntu 的[自動升級](https://help.ubuntu.com/community/AutomaticSecurityUpdates)。

## <a name="download-the-app-package"></a>下載安裝套件

應用程式和安裝指令碼可以從[封存連結](https://go.microsoft.com/fwlink/?linkid=867984)下載。

Sfpkg 格式的應用程式可以從 [sfpkg 連結](https://aka.ms/POA/POA_v2.0.2.sfpkg)下載。 這對於 [Azure Resource Manager 型應用程式部署](service-fabric-application-arm-resource.md)非常有用。

## <a name="configure-the-app"></a>設定應用程式

您可以設定修補程式協調流程應用程式的行為以符合您的需求。 在應用程式建立或更新期間傳入應用程式參數，將預設值加以覆寫。 在 `Start-ServiceFabricApplicationUpgrade` 或 `New-ServiceFabricApplication` Cmdlet 中指定 `ApplicationParameter`，即可提供應用程式參數。

|**參數**        |**類型**                          | **詳細資料**|
|:-|-|-|
|MaxResultsToCache    |long                              | 更新結果的最大數目，應加以快取。 <br>預設值為 3000，是基於以下假設： <br> - 節點數目 20。 <br> - 每個月在節點上發生的更新數目為 5。 <br> - 每個作業的結果數目可為 10。 <br> - 過去 3 個月的結果皆加以儲存。 |
|TaskApprovalPolicy   |例舉 <br> { NodeWise, UpgradeDomainWise }                          |TaskApprovalPolicy 會指出協調員服務在 Service Fabric 叢集節點中用來安裝更新的原則。<br>                         允許的值包括： <br>                                                           <b>NodeWise</b>。 更新會在各個節點上逐一安裝。 <br>                                                           <b>UpgradeDomainWise</b>。 更新會在各個升級網域上逐一安裝。 (最多，屬於升級網域的所有節點都可以進行更新。)
| UpdateOperationTimeOutInMinutes | int <br>(預設值：180)                   | 指定任何更新作業的逾時 (下載或安裝)。 如果作業未在指定的逾時內完成，它就會中止。       |
| RescheduleCount      | int <br> (預設值︰5)                  | 服務在作業持續失敗時將 OS 更新重新排程的次數上限。          |
| RescheduleTimeInMinutes  | int <br>(預設值︰30) | 服務在作業持續失敗時將 OS 更新重新排程的間隔。 |
| UpdateFrequency           | 以逗號分隔的字串 (預設值︰"Weekly, Wednesday, 7:00:00")     | 在叢集上安裝 OS 更新的頻率。 格式與可能的值如下： <br>-   Monthly, DD, HH:MM:SS，例如 Monthly, 5, 12:22:32。 <br> -   Weekly, DAY, HH:MM:SS，例如 Weekly, Tuesday, 12:22:32。  <br> -   Daily, HH:MM:SS，例如 Daily, 12:22:32。  <br> -  None 表示不應進行更新。  <br><br> 所有時間都採用 UTC 格式。|
| UpdateClassification | 以逗號分隔的字串 (預設值︰“securityupdates”) | 應安裝在叢集節點上的更新類型。 可接受的值為 securityupdates 和 all。 <br> -  securityupdates - 只會安裝安全性更新 <br> -  all - 會從 apt 安裝所有可用的更新。|
| ApprovedPatches | 以逗號分隔的字串 (預設值︰"") | 這是應安裝在叢集節點上的已核准更新清單。 以逗號分隔的清單包含已核准的套件，和可自行選擇的目標版本。<br> 例如："apt-utils = 1.2.10ubuntu1, python3-jwt, apt-transport-https < 1.2.194, libsystemd0 >= 229-4ubuntu16" <br> 以上清單會安裝 <br> - 具有 1.2.10ubuntu1 版的 apt-utils (如果存在於 apt-cache 中)。 如果該版本無法使用，則不會執行任何作業。 <br> - python3-jwt 升級為最新的可用版本。 如果套件不存在，則不會執行任何作業。 <br> - apt-transport-https 升級為低於 1.2.194 的最高版本。 如果此版本不存在，則不會執行任何作業。 <br> - libsystemd0 升級為高於或等於 229-4ubuntu16 的最高版本。 如果這類版本不存在，則不會執行任何作業。|
| RejectedPatches | 以逗號分隔的字串 (預設值︰"") | 這是不應安裝在叢集節點上的更新清單 <br> 例如："bash, sudo" <br> 上述清單會篩選掉 bash 和 sudo，而不接收其任何更新。 |


> [!TIP]
> 如果需要 OS 更新立即發生，請將 `UpdateFrequency` 設為相對於應用程式部署的時間。 例如，假設您的測試叢集有五個節點，計劃於大約下午 5:00 UTC 部署應用程式。 如果您假設應用程式的升級或部署最多會花費 30 分鐘的時間，則將 UpdateFrequency 設為 "Daily, 17:30:00"。

## <a name="deploy-the-app"></a>部署應用程式

1. 完成所有先決條件步驟，以進行叢集準備工作。
2. 部署修補程式協調流程應用程式，和任何其他 Service Fabric 應用程式一樣。 您可以使用 PowerShell 或 Azure Service Fabric CLI 部署此應用程式。 請遵循[使用 PowerShell 部署與移除應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications)或[使用 Azure Service Fabric CLI 部署應用程式](https://docs.microsoft.com/azure/service-fabric/scripts/cli-deploy-application)中的步驟
3. 若要在部署時設定應用程式，請將 `ApplicationParamater` 傳至 `New-ServiceFabricApplication` Cmdlet 或提供的指令碼。 為方便您作業，powershell (Deploy.ps1) 和 bash (Deploy.sh) 指令碼會隨應用程式提供。 若要使用指令碼：

    - 連線至 Service Fabric 叢集。
    - 執行部署指令碼。 選擇性地將應用程式參數傳至指令碼。 例如：.\Deploy.ps1 -ApplicationParameter @{ UpdateFrequency = "Daily, 11:00:00"} OR ./Deploy.sh "{\"UpdateFrequency\":\"Daily, 11:00:00\"}" 

> [!NOTE]
> 將指令碼和應用程式資料夾 PatchOrchestrationApplication 保存在同一個目錄中。

## <a name="upgrade-the-app"></a>升級應用程式

若要升級現有的修補程式協調流程應用程式，請依照[使用 PowerShell 進行 Service Fabric 應用程式升級](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade-tutorial-powershell)或[使用 Azure Service Fabric CLI 進行 Service Fabric 應用程式升級](https://docs.microsoft.com/azure/service-fabric/service-fabric-sfctl-application#sfctl-application-upgrade)中的步驟。

## <a name="remove-the-app"></a>移除應用程式

若要移除應用程式，請遵循[使用 PowerShell 部署與移除應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications)或[使用 Azure Service Fabric CLI 移除應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-sfctl-application#sfctl-application-delete)中的步驟

為方便您作業，powershell (Undeploy.ps1) 和 bash (Undeploy.sh) 指令碼會隨應用程式提供。 若要使用指令碼：

  - 連線至 Service Fabric 叢集。
  - 執行指令碼 Undeploy.ps1 或 Undeploy.sh

> [!NOTE]
> 將指令碼和應用程式資料夾 PatchOrchestrationApplication 保存在同一個目錄中。

## <a name="view-the-update-results"></a>檢視更新結果

修補程式協調流程應用程式會公開 REST API，以向使用者顯示歷程記錄的結果。 以下是範例結果︰```testadm@bronze000001:~$ curl -X GET http://10.0.0.5:20002/PatchOrchestrationApplication/v1/GetResults```
```json
[ 
  { 
    "NodeName": "_bronze_0", 
    "UpdateOperationResults": [ 
      { 
        "OperationResult": "succeeded", 
        "NodeName": "_bronze_0", 
        "OperationTime": "2017-11-21T12:39:29.0435917Z", 
        "UpdateDetails": [ 
          { 
            "UpdateId": "linux-cloud-tools-azure:amd64=4.11.0.1015.15", 
            "ResultCode": "succeeded" 
          }, 
          { 
            "UpdateId": "linux-headers-azure:amd64=4.11.0.1015.15", 
            "ResultCode": "succeeded" 
          }, 
          { 
            "UpdateId": "linux-image-azure:amd64=4.11.0.1015.15", 
            "ResultCode": "succeeded" 
          }, 
          { 
            "UpdateId": "linux-tools-azure:amd64=4.11.0.1015.15", 
            "ResultCode": "succeeded" 
          }, 
          { 
            "UpdateId": "python3-apport:amd64=2.20.1-0ubuntu2.13", 
            "ResultCode": "succeeded" 
          }, 
        ], 
        "OperationType": "installation", 
        "UpdateClassification": "securityupdates", 
        "UpdateFrequency": "Daily, 7:00:00", 
        "RebootRequired": true, 
        "ApprovedList": "", 
        "RejectedList": "" 
      } 
    ] 
  } 
] 
```

JSON 的欄位說明如下：

欄位 | 值 | 詳細資料
-- | -- | --
OperationResult | 0 - 成功<br> 1 - 成功但發生錯誤<br> 2 - 失敗<br> 3 - 已中止<br> 4 - 中止但逾時 | 指出整體作業 (通常牽涉到安裝一或多個更新) 的結果。
ResultCode | 與 OperationResult 相同 | 此欄位指出個別更新安裝作業的結果。
OperationType | 1 - 安裝<br> 0 - 搜尋和下載。| 「安裝」是依預設會顯示在結果中的唯一一個 OperationType。
UpdateClassification | 預設值是 "securityupdates" | 在更新作業期間安裝的更新類型
UpdateFrequency | 預設值為 "Weekly, Wednesday, 7:00:00" | 針對此更新設定的更新頻率。
RebootRequired | true - 需要重新開機<br> false - 不需要重新開機 | 指出必須重新開機才能完成更新的安裝。
ApprovedList | 預設值為 "" | 此更新已核准的修補程式清單
RejectedList | 預設值為 "" | 此更新已拒絕的修補程式清單

如果尚未排程更新，JSON 結果會是空的。

登入叢集以查詢更新結果。 接著，找出主要協調員服務的複本位址，然後點閱瀏覽器的 URL： http://&lt;REPLICA-IP&gt;:&lt;ApplicationPort&gt;/PatchOrchestrationApplication/v1/GetResults。

協調器服務的 REST 端點具有動態連接埠。 若要知道確切 URL，請查看 Service Fabric Explorer。 例如，可在 `http://10.0.0.7:20000/PatchOrchestrationApplication/v1/GetResults` 找到結果。

![REST 端點的圖片](media/service-fabric-patch-orchestration-application/Rest_Endpoint.png)

## <a name="diagnosticshealth-events"></a>診斷/健康情況事件

### <a name="diagnostic-logs"></a>診斷記錄檔

收集修補程式協調流程應用程式的記錄，是 Service Fabric 執行階段記錄的一部分。

以免您想要透過您選擇的診斷工具/管線擷取記錄。 修補程式協調流程應用程式使用以下的固定提供者識別碼，透過 [eventsource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracing.eventsource?view=netstandard-2.0) 記錄事件

- e39b723c-590c-4090-abb0-11e3e6616346
- fc0028ff-bfdc-499f-80dc-ed922c52c5e9
- 24afa313-0d3b-4c7c-b485-1047fd964b60
- 05dc046c-60e9-4ef7-965e-91660adffa68

### <a name="health-reports"></a>健康狀態報告

修補程式協調流程應用程式在下列情況下也會發佈協調器服務或節點代理程式服務的健康情況報告：

#### <a name="an-update-operation-failed"></a>更新作業失敗

如果節點上的更新作業失敗，就會產生節點代理程式服務的健康情況報告。 健康情況報告的詳細資料中包含有問題的節點名稱。

在有問題的節點上順利完成修補後，系統會自動清除報告。

#### <a name="the-node-agent-daemon-service-is-down"></a>節點代理程式精靈已關閉

如果節點上的節點代理程式精靈關閉，就會產生節點代理程式服務的警告層級健康情況報告。

#### <a name="the-repair-manager-service-is-not-enabled"></a>修復管理器服務未啟用

如果在叢集上找不到修復管理員服務，就會產生協調器服務的警告層級健康情況報告。

## <a name="frequently-asked-questions"></a>常見問題集

問： **當修補程式協調流程應用程式執行時，為什麼我看到叢集處於錯誤狀態？**

A. 在安裝程序期間，修補程式協調流程應用程式會停用或重新啟動節點。 這項作業可能會導致叢集暫時關閉的健康情況。

根據應用程式的原則，可能是一個節點在修補作業期間關閉，「或是」整個升級網域同時關閉。

安裝結束時，節點會在系統重新啟動後重新啟用。

在下列範例中，因為兩個節點關閉，且違反 MaxPercentageUnhealthyNodes 原則，叢集會暫時進入錯誤狀態。 錯誤是暫時性的，直到修補作業進行中為止。

![健康情況不良之叢集的圖片](media/service-fabric-patch-orchestration-application/MaxPercentage_causing_unhealthy_cluster.png)

如果問題持續發生，請參閱〈疑難排解〉一節。

問： **修補程式協調流程應用程式處於警告狀態**

A. 檢查針對應用程式所公佈的健康情況報告，是否為根本原因。 警告通常包含問題的詳細資料。 如果是暫時性的問題，應用程式預期會從這個狀態中自動復原。

問： **如果我的叢集健康情況不良，我可以做什麼？我需要採取緊急作業系統更新嗎？**

A. 當叢集健康情況不良時，修補程式協調流程應用程式就不會安裝更新。 若要將修補程式協調流程應用程式工作流程解除封鎖，請嘗試使叢集處於良好的健康情況。

問： **為什麼跨叢集修補的執行時間這麼久？**

A. 修補程式協調流程應用程式所花費的時間大部分是取決於下列因素︰

- 協調器服務的原則。 
  - 預設原則 `NodeWise` 的結果是一次只修補一個節點。 特別是在叢集較大的情況下，建議您使用 `UpgradeDomainWise` 原則以更快速修補多個叢集。
- 適用於下載和安裝的更新數目。 
- 下載並安裝更新時所需的平均時間，不應該超過幾個小時。
- VM 和網路頻寬的效能。

問： **修補程式協調流程應用程式如何決定哪些更新是安全性更新。**

A. 修補程式協調流程應用程式會使用散發套件專用邏輯來判斷可用的更新之中有哪些是安全性更新。 例如：在 ubuntu 中，此應用程式會從 archives $RELEASE-security, $RELEASE-updates ($RELEASE = xenial 或 Linux 標準基礎發行版本) 搜尋更新。 

 
問： **如何鎖定特定版本的套件？**

A. 您可以使用使用 ApprovedPatches 設定讓您的套件鎖定特定版本。 


問： **在 Ubuntu 中啟用自動更新會怎樣？**

A. 當您在叢集上安裝修補程式協調流程應用程式時，您的叢集節點就會停用自動升級。 所有定期更新工作流程皆由修補程式協調流程應用程式驅動。
若要讓整個叢集保有一致性的環境，建議您一律透過修補程式協調流程應用程式來安裝更新。 
 
問： **升級後，修補程式協調流程應用程式是否會清理未使用的套件？**

A. 是，清理會在後續安裝步驟中執行。 

問： **修補協調流程應用程式可用來更新我的開發叢集 (一個雙節點的叢集) 嗎？**

A. 否，修補協調流程應用程式無法用來修補單一節點的叢集。 此限制的設計：因為 [Service Fabric 系統服務](https://docs.microsoft.com/azure/service-fabric/service-fabric-technical-overview#system-services)或任何客戶應用程式將會面臨停機時間，所以修復管理員決不會核准以任何修復作業進行修補。

## <a name="troubleshooting"></a>疑難排解

### <a name="a-node-is-not-coming-back-to-up-state"></a>節點不會回到開啟狀態

**節點可能會卡在停用中狀態，因為**：

安全性檢查擱置。 若要補救這種情況，請確定有足夠多健康情況良好的節點。

**節點可能會卡在已停用狀態，因為**：

- 已將節點手動停用。
- 因為 Azure 基礎結構作業正在進行中而導致節點停用。
- 修補程式協調流程應用程式已暫時將節點停用，以修補節點。

**節點可能會卡在關閉狀態，因為**：

- 已手動將節點置於關閉狀態。
- 節點正在重新啟動 (可能是由修補程式協調流程應用程式觸發)。
- 由於 VM 或機器故障或網路連線問題，使節點關閉。

### <a name="updates-were-skipped-on-some-nodes"></a>已略過某些節點上的更新

修補程式協調流程應用程式會根據重新排定的原則，嘗試安裝更新。 服務會根據應用程式原則，嘗試復原節點並略過更新。

在這種情況下，系統會針對節點代理程式服務產生警告等級的健康情況報告。 更新的結果也包含可能的失敗原因。

### <a name="the-health-of-the-cluster-goes-to-error-while-the-update-installs"></a>開始安裝更新時，叢集的健康情況出現錯誤

錯誤的更新會損及特定節點或升級網域上應用程式或叢集的健康情況。 修補程式協調流程應用程式會中止所有後續的更新作業，直到叢集恢復良好健康情況。

系統管理員必須介入，並判斷應用程式或叢集為何因先前安裝的更新而變得健康情況不良。

## <a name="disclaimer"></a>免責聲明

修補程式協調流程應用程式會收集遙測來追蹤使用情形和效能。 應用程式的遙測會遵循 Service Fabric 執行階段遙測設定 (預設為開啟) 的設定。

## <a name="release-notes"></a>版本資訊

### <a name="version-010"></a>0.1.0 版
- 私人預覽版本

### <a name="version-200"></a>版本 2.0.0
- 公開版本

### <a name="version-201"></a>版本 2.0.1
- 已使用最新 Service Fabric SDK 重新編譯應用程式

### <a name="version-202-latest"></a>版本 2.0.2 (最新)
- 已修正會在重新啟動期間遺留健康情況警告的問題。
