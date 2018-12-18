---
title: Azure Service Fabric 修補程式協調流程應用程式 | Microsoft Docs
description: 在 Service Fabric 叢集上將作業系統修補自動化的應用程式。
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
ms.openlocfilehash: f43715b9c419aab1f5b95e140eac72642ef74198
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746892"
---
# <a name="patch-the-windows-operating-system-in-your-service-fabric-cluster"></a>修補 Service Fabric 叢集中的 Windows 作業系統

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
    - 協調整個叢集上的 Windows Update 作業。
    - 儲存已完成之 Windows Update 作業的結果。
- **節點代理程式服務**︰是無狀態服務，在所有 Service Fabric 叢集節點上執行。 此服務負責：
    - 啟動節點代理程式 NTService。
    - 監視節點代理程式 NTService。
- **節點代理程式 NTService**：在較高層級權限 (系統) 執行的 Windows NT 服務。 相比之下，節點代理程式服務和協調器服務則是在較低層級權限 (網路服務) 執行。 此服務會負責執行下列所有叢集節點上的 Windows Update 作業：
    - 將節點上的自動 Windows Update 停用。
    - 根據使用者提供的原則下載並安裝 Windows Update。
    - 在 Windows Update 安裝後重新啟動機器。
    - 將 Windows Update 的結果上傳至協調器服務。
    - 萬一耗盡所有重試之後作業還是失敗，就報告健康情況報告。

> [!NOTE]
> 修補程式協調流程應用程式是使用 Service Fabric 的修復管理器系統服務，將節點停用或啟用以及執行健康情況檢查。 修補程式協調流程應用程式所建立的修復工作會追蹤每個節點的 Windows Update 進度。

## <a name="prerequisites"></a>必要條件

### <a name="enable-the-repair-manager-service-if-its-not-running-already"></a>啟用修復管理器服務 (如果尚未執行中)

叢集必須啟用修復管理器系統服務，修補程式協調流程應用程式才能執行。

#### <a name="azure-clusters"></a>Azure 叢集

銀級耐久性層中的 Azure 叢集依預設會啟用修復管理器。 金級耐久性層中的 Azure 叢集可能會或也可能不會啟用修復管理器，取決於這些叢集的建立時間。 銅級耐久性層中的 Azure 叢集依預設不會啟用修復管理器服務。 如果已啟用服務，可以在 Service Fabric Explorer 的系統服務區段中看到它正在執行。

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

您可以使用[獨立 Windows 叢集的組態設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-manifest)在新的和現有的 Service Fabric 叢集上啟用修復管理器。

啟用修復管理器服務：

1. 首先，檢查[一般叢集設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-manifest#general-cluster-configurations)中的 `apiversion` 是否已設定為 `04-2017` 或更高版本：

    ```json
    {
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        "apiVersion": "04-2017",
        ...
    }
    ```

2. 立即啟用修復管理器服務，方法是在 `fabricSettings` 區段之後新增下列 `addonFeatures` 區段，如下所示：

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. 以這些變更更新您的叢集資訊清單，使用更新後的叢集資訊清單[建立新叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-for-windows-server)或[升級叢集設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-upgrade-windows-server#Upgrade-the-cluster-configuration)。 叢集開始以更新的資訊清單執行後，您現在可以在 Service Fabric Explorer 的系統服務區段下，看到修復管理器系統服務 (亦稱為 `fabric:/System/RepairManagerService`) 在叢集中執行。

### <a name="disable-automatic-windows-update-on-all-nodes"></a>停用所有節點上的自動 Windows Update

自動 Windows Update 可能會導致可用性遺失，因為在相同時間重新啟動多個叢集節點。 依預設，修補程式協調流程應用程式會在每個叢集節點上嘗試將自動 Windows Update 停用。 不過，如果設定是由系統管理員或群組原則所管理，建議將 Windows Update 原則明確地設定為「下載前先通知」。

## <a name="download-the-app-package"></a>下載安裝套件

應用程式和安裝指令碼可以從[封存連結](https://go.microsoft.com/fwlink/?linkid=869566)下載。

Sfpkg 格式的應用程式可以從 [sfpkg 連結](https://aka.ms/POA/POA_v1.2.2.sfpkg)下載。 這對於 [Azure Resource Manager 型應用程式部署](service-fabric-application-arm-resource.md)非常有用。

## <a name="configure-the-app"></a>設定應用程式

您可以設定修補程式協調流程應用程式的行為以符合您的需求。 在應用程式建立或更新期間傳入應用程式參數，將預設值加以覆寫。 在 `Start-ServiceFabricApplicationUpgrade` 或 `New-ServiceFabricApplication` Cmdlet 中指定 `ApplicationParameter`，即可提供應用程式參數。

|**參數**        |**類型**                          | **詳細資料**|
|:-|-|-|
|MaxResultsToCache    |long                              | Windows Update 結果的最大數目，應加以快取。 <br>預設值為 3000，是基於以下假設： <br> - 節點數目 20。 <br> - 每個月在節點上發生的更新數目為 5。 <br> - 每個作業的結果數目可為 10。 <br> - 過去 3 個月的結果皆加以儲存。 |
|TaskApprovalPolicy   |例舉 <br> { NodeWise, UpgradeDomainWise }                          |TaskApprovalPolicy 會指出協調器服務在 Service Fabric 叢集節點中用來安裝 Windows 更新的原則。<br>                         允許的值包括： <br>                                                           <b>NodeWise</b>。 一次只會在一個節點上安裝 Windows Update。 <br>                                                           <b>UpgradeDomainWise</b>。 一次只會在一個升級網域上安裝 Windows Update。 (最多，屬於升級網域的所有節點都可以進行 Windows Update。)<br> 請參閱[常見問題集](#frequently-asked-questions)一節，以了解如何決定最適合您叢集的原則。
|LogsDiskQuotaInMB   |long  <br> (預設值︰1024)               |修補程式協調流程應用程式記錄的大小上限 (以 MB 為單位)，可在節點上本機保留。
| WUQuery               | 字串<br>(預設值："IsInstalled=0")                | 用以取得 Windows 更新的查詢。 如需詳細資訊，請參閱 [WuQuery](https://msdn.microsoft.com/library/windows/desktop/aa386526(v=vs.85).aspx)。
| InstallWindowsOSOnlyUpdates | BOOLEAN <br> (預設值︰True)                 | 這個旗標可讓您安裝 Windows 作業系統更新。            |
| WUOperationTimeOutInMinutes | int <br>(預設值︰90)                   | 指定任何 Windows Update 作業的逾時 (搜尋或下載或安裝)。 如果作業未在指定的逾時內完成，它就會中止。       |
| WURescheduleCount     | int <br> (預設值︰5)                  | 如果作業持續失敗，服務會將 Windows Update 重新排程的次數上限。          |
| WURescheduleTimeInMinutes | int <br>(預設值︰30) | 如果作業持續失敗，服務會將 Windows Update 重新排程的時間間隔。 |
| WUFrequency           | 以逗號分隔的字串 (預設值︰"Weekly, Wednesday, 7:00:00")     | 安裝 Windows Update 的頻率。 格式與可能的值如下： <br>-   Monthly, DD, HH:MM:SS，例如 Monthly, 5,12:22:32。 <br> -   Weekly, DAY, HH:MM:SS，例如 Weekly, Tuesday, 12:22:32。  <br> -   Daily, HH:MM:SS，例如 Daily, 12:22:32。  <br> -  None 表示不應該進行 Windows Update。  <br><br> 請注意，時間會採用 UTC 格式。|
| AcceptWindowsUpdateEula | BOOLEAN <br>(預設值︰true) | 藉由設定這個旗標，應用程式會代表電腦的擁有者接受 Windows Update 的使用者授權合約 (EULA)。              |

> [!TIP]
> 如果需要 Windows Update 立即發生，請將 `WUFrequency` 設定為相對於應用程式部署的時間。 例如，假設您的測試叢集有五個節點，計劃於大約下午 5:00 UTC 部署應用程式。 如果您假設應用程式的升級或部署最多會花費 30 分鐘的時間，則將 WUFrequency 設定為 "Daily, 17:30:00"。

## <a name="deploy-the-app"></a>部署應用程式

1. 完成準備叢集的所有必要條件步驟。
2. 部署修補程式協調流程應用程式，和任何其他 Service Fabric 應用程式一樣。 您可以使用 PowerShell 部署此應用程式。 請遵循[使用 PowerShell 部署與移除應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications)中的步驟。
3. 若要在部署時設定應用程式，可將 `ApplicationParamater` 傳遞給 `New-ServiceFabricApplication` Cmdlet。 為了您的方便，我們提供了 Deploy.ps1 指令碼以及我們的應用程式。 若要使用指令碼：

    - 使用 `Connect-ServiceFabricCluster` 連線至 Service Fabric 叢集。
    - 利用適當的 `ApplicationParameter` 值執行 Powershell 指令碼 Deploy.ps1。

> [!NOTE]
> 將指令碼和應用程式資料夾 PatchOrchestrationApplication 保存在同一個目錄中。

## <a name="upgrade-the-app"></a>升級應用程式

若要使用 PowerShell 將現有的修補程式協調流程應用程式進行升級，請遵循[使用 PowerShell 升級 Service Fabric應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-upgrade-tutorial-powershell)中的步驟。

## <a name="remove-the-app"></a>移除應用程式

若要移除應用程式，請遵循[使用 PowerShell 部署與移除應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications)中的步驟。

為了您的方便，我們提供了 Undeploy.ps1 指令碼以及我們的應用程式。 若要使用指令碼：

  - 使用 ```Connect-ServiceFabricCluster``` 連線至 Service Fabric 叢集。

  - 執行 Powershell 指令碼 Undeploy.ps1。

> [!NOTE]
> 將指令碼和應用程式資料夾 PatchOrchestrationApplication 保存在同一個目錄中。

## <a name="view-the-windows-update-results"></a>檢視 Windows Update 結果

修補程式協調流程應用程式會公開 REST API，以向使用者顯示歷程記錄的結果。 JSON 結果的範例：
```json
[
  {
    "NodeName": "_stg1vm_1",
    "WindowsUpdateOperationResults": [
      {
        "OperationResult": 0,
        "NodeName": "_stg1vm_1",
        "OperationTime": "2017-05-21T11:46:52.1953713Z",
        "UpdateDetails": [
          {
            "UpdateId": "7392acaf-6a85-427c-8a8d-058c25beb0d6",
            "Title": "Cumulative Security Update for Internet Explorer 11 for Windows Server 2012 R2 (KB3185319)",
            "Description": "A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.",
            "ResultCode": 0
          }
        ],
        "OperationType": 1,
        "WindowsUpdateQuery": "IsInstalled=0",
        "WindowsUpdateFrequency": "Daily,10:00:00",
        "RebootRequired": false
      }
    ]
  },
  ...
]
```

以下描述 JSON 的欄位。

欄位 | 值 | 詳細資料
-- | -- | --
OperationResult | 0 - 成功<br> 1 - 成功但發生錯誤<br> 2 - 失敗<br> 3 - 已中止<br> 4 - 中止但逾時 | 指出整體作業 (通常牽涉到安裝一或多個更新) 的結果。
ResultCode | 與 OperationResult 相同 | 此欄位指出個別更新安裝作業的結果。
OperationType | 1 - 安裝<br> 0 - 搜尋和下載。| 「安裝」是依預設會顯示在結果中的唯一一個 OperationType。
WindowsUpdateQuery | 預設值為 "IsInstalled=0" |用來搜尋更新的 Windows Update 查詢。 如需詳細資訊，請參閱 [WuQuery](https://msdn.microsoft.com/library/windows/desktop/aa386526(v=vs.85).aspx)。
RebootRequired | true - 需要重新開機<br> false - 不需要重新開機 | 指出完成更新的安裝是否需要重新開機。

如果尚未排程更新，JSON 結果會是空的。

登入叢集，查詢 Windows Update 的結果。 接著，找出主要協調器服務的複本位址，然後點閱瀏覽器的 URL： http://&lt;REPLICA-IP&gt;:&lt;ApplicationPort&gt;/PatchOrchestrationApplication/v1/GetWindowsUpdateResults。

協調器服務的 REST 端點具有動態連接埠。 若要知道確切 URL，請查看 Service Fabric Explorer。 例如，可在 `http://10.0.0.7:20000/PatchOrchestrationApplication/v1/GetWindowsUpdateResults` 找到結果。

![REST 端點的圖片](media/service-fabric-patch-orchestration-application/Rest_Endpoint.png)


如果在叢集上啟用反向 Proxy，您也可以從叢集外部存取 URL。
需要點閱的端點為 http://&lt;SERVERURL&gt;:&lt;REVERSEPROXYPORT&gt;/PatchOrchestrationApplication/CoordinatorService/v1/GetWindowsUpdateResults。

若要啟用叢集的反向 Proxy，請遵循 [Azure Service Fabric 中的反向 Proxy](https://docs.microsoft.com/azure/service-fabric/service-fabric-reverseproxy) 中的步驟。 

> 
> [!WARNING]
> 設定反向 Proxy 之後，叢集中公開 HTTP 端點的所有微服務就能從叢集外部尋址。

## <a name="diagnosticshealth-events"></a>診斷/健康情況事件

### <a name="diagnostic-logs"></a>診斷記錄檔

收集修補程式協調流程應用程式的記錄，是 Service Fabric 執行階段記錄的一部分。

以免您想要透過您選擇的診斷工具/管線擷取記錄。 修補程式協調流程應用程式使用以下的固定提供者識別碼，透過 [eventsource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracing.eventsource?view=netframework-4.5.1) 記錄事件

- e39b723c-590c-4090-abb0-11e3e6616346
- fc0028ff-bfdc-499f-80dc-ed922c52c5e9
- 24afa313-0d3b-4c7c-b485-1047fd964b60
- 05dc046c-60e9-4ef7-965e-91660adffa68

### <a name="health-reports"></a>健康狀態報告

修補程式協調流程應用程式在下列情況下也會發佈協調器服務或節點代理程式服務的健康情況報告：

#### <a name="a-windows-update-operation-failed"></a>Windows Update 作業失敗

如果節點上的 Windows Update 作業失敗，就會產生節點代理程式服務的健康情況報告。 健康情況報告的詳細資料中包含有問題的節點名稱。

在有問題的節點上順利完成修補後，系統會自動清除報告。

#### <a name="the-node-agent-ntservice-is-down"></a>節點代理程式 NTService 關閉

如果節點上的節點代理程式 NTService 關閉，就會產生節點代理程式服務的警告等級健康情況報告。

#### <a name="the-repair-manager-service-is-not-enabled"></a>修復管理器服務未啟用

如果在叢集上找不到修復管理器服務，就會產生協調器服務的警告等級健康情況報告。

## <a name="frequently-asked-questions"></a>常見問題集

問： **當修補程式協調流程應用程式執行時，為什麼我看到叢集處於錯誤狀態？**

A. 在安裝程序期間，修補程式協調流程應用程式會停用或重新啟動節點，而這可能會導致叢集暫時關閉的健康情況。

根據應用程式的原則，可能是一個節點在修補作業期間關閉，「或是」整個升級網域同時關閉。

Windows Update 安裝結束時，在重新啟動前會重新啟用節點。

在下列範例中，因為兩個節點關閉，且違反 MaxPercentageUnhealthNodes 原則，叢集會暫時進入錯誤狀態。 錯誤是暫時性的，直到修補作業進行中為止。

![健康情況不良之叢集的圖片](media/service-fabric-patch-orchestration-application/MaxPercentage_causing_unhealthy_cluster.png)

如果問題持續發生，請參閱〈疑難排解〉一節。

問： **修補程式協調流程應用程式處於警告狀態**

A. 檢查針對應用程式所公佈的健康情況報告，是否為根本原因。 警告通常包含問題的詳細資料。 如果是暫時性的問題，應用程式預期會從這個狀態中自動復原。

問： **如果我的叢集健康情況不良，我可以做什麼？我需要採取緊急作業系統更新嗎？**

A. 當叢集健康情況不良時，修補程式協調流程應用程式就不會安裝更新。 請嘗試使叢集處於良好的健康情況，以便將修補程式協調流程應用程式工作流程解除封鎖。

問： **我是否應該針對叢集將 TaskApprovalPolicy 設為 'NodeWise' 或 'UpgradeDomainWise'？**

A. 'UpgradeDomainWise' 可以藉由平行修補屬於升級網域的所有節點，來加快整體叢集修補。 這表示屬於整個升級網域的節點在修補程序期間都無法使用 (處於[已停用](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabled)狀態)。

相較之下，'NodeWise' 原則一次只會修補一個節點，這也暗示著整體叢集修補需要較長的時間。 不過，在修補程序期間頂多只有一個節點無法使用 (處於[已停用](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabled)狀態)。

如果您的叢集可以容忍修補週期期間在 N-1 個升級網域上執行 (其中 N 是叢集上升級網域的總數)，則您可以將原則設為 'UpgradeDomainWise'，否則請將其設為 'NodeWise'。

問： **修補節點需要耗費多久時間？**

A. 修補一個節點可能需要數分鐘 (例如：[Windows Defender 定義更新](https://www.microsoft.com/wdsi/definitions)) 到數小時 (例如：[Windows 累積更新](https://www.catalog.update.microsoft.com/Search.aspx?q=windows%20server%20cumulative%20update)) 的時間。 修補節點所需的時間大多取決於下列因素： 
 - 更新的大小
 - 修補期間所需要套用的更新數目
 - 安裝更新、重新啟動節點 (如有必要) 然後完成重新啟動後安裝步驟所耗費的時間。
 - 虛擬機器/電腦和網路狀況的效能。

問： **修補整個叢集需要多久時間？**

A. 修補整個叢集所需的時間取決於下列因素：

- 修補一個節點所需的時間。
- 協調器服務的原則。 - 預設原則 `NodeWise` 一次只會修補一個節點，這樣比 `UpgradeDomainWise` 慢。 例如：如果節點需要約 1 個小時來修補，若要修補 20 個節點 (節點類型相同) 的叢集，其中具有 5 個升級網域，每個網域包含 4 個節點。
    - 如果原則是 `NodeWise`，則需要約 20 個小時來修補整個叢集
    - 如果原則是 `UpgradeDomainWise`，則需要約 5 個小時
- 叢集負載 - 每個修補作業都必須將客戶工作負載重新放置到叢集中的其他可用節點。 正在進行修補的節點屆時會處於[停用](https://docs.microsoft.com/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabling)狀態。 如果正在執行的叢集接近尖峰負載，則停用程序需要更長的時間。 因此，在如此充滿壓力的條件下，整體修補程序可能會變慢。
- 修補期間叢集健康情況失敗 - [叢集健康情況](https://docs.microsoft.com/azure/service-fabric/service-fabric-health-introduction)如果[降低](https://docs.microsoft.com/dotnet/api/system.fabric.health.healthstate?view=azure-dotnet#System_Fabric_Health_HealthState_Error)，就會中斷修補程序。 這樣會增加修補整個叢集所需的整體時間。

問： **為什麼我在 Windows Update 結果中看到某些更新已透過 REST API 取得，但未出現在電腦的 Windows Update 歷程記錄下？**

A. 某些產品更新只會出現在其各自的更新/修補歷程記錄中。 例如，Windows Defender 更新可能不會顯示在 Windows Server 2016 上的 Windows Update 歷程記錄中。

問： **「修補協調流程」應用程式可用來更新我的開發叢集 (單一節點叢集) 嗎？**

A. 否，修補協調流程應用程式無法用來修補單一節點的叢集。 此限制的設計：因為 [Service Fabric 系統服務](https://docs.microsoft.com/azure/service-fabric/service-fabric-technical-overview#system-services)或任何客戶應用程式將會面臨停機時間，所以修復管理員決不會核准以任何修復作業進行修補。

## <a name="disclaimers"></a>免責聲明

- 修補程式協調流程應用程式會代表使用者接受 Windows Update 的使用者授權合約 (EULA)。 可在應用程式的設定中選擇性地將此設定關閉。

- 修補程式協調流程應用程式會收集遙測來追蹤使用情形和效能。 應用程式的遙測會遵循 Service Fabric 執行階段遙測設定 (預設為開啟) 的設定。

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

修補程式協調流程應用程式會根據重新排定的原則，嘗試安裝 Windows Update。 服務會根據應用程式原則，嘗試復原節點並略過更新。

在這種情況下，系統會針對節點代理程式服務產生警告等級的健康情況報告。 Windows Update 的結果也包含可能的失敗原因。

### <a name="the-health-of-the-cluster-goes-to-error-while-the-update-installs"></a>開始安裝更新時，叢集的健康情況出現錯誤

錯誤的 Windows Update 會損及特定節點或升級網域上應用程式或叢集的健康情況。 修補程式協調流程應用程式會中止所有後續的 Windows Update作業，直到叢集恢復良好健康情況。

系統管理員必須介入，判斷應用程式或叢集為何因為 Windows Update 變成健康情況不良。

## <a name="release-notes"></a>版本資訊

### <a name="version-110"></a>1.1.0 版
- 公開版本

### <a name="version-111"></a>1.1.1 版
- 修正 SetupEntryPoint of NodeAgentService 中的錯誤，該錯誤導致無法安裝 NodeAgentNTService。

### <a name="version-120"></a>版本 1.2.0

- 關於系統重新啟動工作流程的錯誤修正。
- 由於準備修復工作期間健康情況檢查未如預期般發生，而在 RM 工作建立時進行的錯誤修正。
- 將 Windows 服務 POANodeSvc 的啟動模式從自動變更為延遲自動。

### <a name="version-121"></a>1.2.1 版

- 縮小叢集範圍工作流程中的錯誤修正。 針對屬於不存在節點的 POA 修復工作，導入了記憶體回收邏輯。

### <a name="version-122-latest"></a>1.2.2 版 (最新)

- 其他錯誤修正。
- 現在已簽署二進位檔。
- sfpkg 下載連結現在會指向特定版本。
