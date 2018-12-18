---
title: 應用程式升級：升級參數 | Microsoft Docs
description: 描述升級 Service Fabric 應用程式的相關參數，包括執行健全狀況檢查和自動復原升級的原則。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: a4170ac6-192e-44a8-b93d-7e39c92a347e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 9/17/2018
ms.author: subramar
ms.openlocfilehash: 3f321775ba112471760e627e6b43ed17ff8c5b6b
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182870"
---
# <a name="application-upgrade-parameters"></a>應用程式升級參數
本文說明在 Azure Service Fabric 應用程式升級期間套用的各種參數。 應用程式升級參數會控制升級時套用的逾時與健康情況檢查，並且指定升級失敗時必須套用的原則。

應用程式參數是使用 PowerShell 或 Visual Studio 來套用到升級的。 下列「必要參數」和「選擇性參數」資料表會說明適用於 PowerShell 和/或 Visual Studio 的必要和選擇性參數。

您可以從三個使用者可選取的升級模式中選一個來起始應用程式升級。 每一種模式都有自己的一組應用程式參數：
- 受監視的升級 (Monitored)
- 未受監視的自動升級 (Unmonitored Auto)
- 未受監視的手動升級 (Unmonitored Manual)

使用 PowerShell 升級的 Service Fabric 應用程式會使用 [Start-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricapplicationupgrade) 命令。 若要選擇升級模式，可將 **Monitored**、**UnmonitoredAuto** 或 **UnmonitoredManual** 參數傳遞給 [Start-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricapplicationupgrade)。

Visual Studio Service Fabric 應用程式升級參數會透過 [Visual Studio 升級設定] 對話方塊來設定。 若要選取 Visual Studio 升級模式，可使用 [升級模式] 下拉式清單來選取 **Monitored**、**UnmonitoredAuto** 或 **UnmonitoredManual**。 如需詳細資訊，請參閱[在 Visual Studio 中設定 Service Fabric 應用程式的升級](service-fabric-visualstudio-configure-upgrade.md)。

## <a name="required-parameters"></a>必要參數
(PS=PowerShell, VS=Visual Studio)

| 參數 | 套用至 | 說明 |
| --- | --- | --- |
ApplicationName |PS| 正在升級的應用程式名稱。 範例：fabric:/VisualObjects、fabric:/ClusterMonitor。 |
ApplicationTypeVersion|PS|升級目標的應用程式類型的版本。 |
FailureAction |PS, VS|允許的值為 **Rollback**、**Manual** 和 **Invalid**。 「受監視」的升級遇到監視原則或健康原則違規時，所要執行的補償動作。 <br>**Rollback** 會指定升級自動復原為升級前的版本。 <br>**Manual** 表示升級將切換為 UnmonitoredManual 升級模式。 <br>**Invalid** 表示失敗動作無效。|
Monitored |PS|表示升級模式受到監視。 當 Cmdlet 完成一個升級網域的升級後，如果升級網域和叢集的健康情況符合您定義的健康情況原則，則 Service Fabric 會升級下一個升級網域。 如果升級網域或叢集不符合健康原則，升級將會失敗，而且 Service Fabric 會根據指定的原則復原該升級網域的升級，或還原為手動模式。 這是在生產環境中升級應用程式的建議模式。 |
UpgradeMode | VS | 允許的值為 **Monitored** (預設值)、**UnmonitoredAuto**或 **UnmonitoredManual**。 請在本文中參閱每個模式的 PowerShell 參數，以取得詳細資訊。 |
UnmonitoredAuto | PS | 表示升級模式為未受監視的自動升級。 當 Service Fabric 升級一個升級網域後，不管應用程式健康狀態為何，Service Fabric 都會升級下一個升級網域。 此模式中不建議用於生產環境中，而且僅適用於應用程式的開發。 |
UnmonitoredManual | PS | 表示升級模式為未受監視的手動升級。 當 Service Fabric 升級一個升級網域後，它會等候您使用 *Resume-servicefabricapplicationupgrade* Cmdlet 來升級下一個升級網域。 |

## <a name="optional-parameters"></a>選擇性參數

健康情況評估參數是選擇性的。 如果啟動升級時未指定健康狀態評估準則，則 Service Fabric 會使用應用程式執行個體的 ApplicationManifest.xml 中指定的應用程式健康狀態原則。

使用資料表底部的水平捲軸來檢視完整描述欄位。

(PS=PowerShell, VS=Visual Studio)

| 參數 | 套用至 | 說明 |
| --- | --- | --- |
| ApplicationParameter |PS, VS| 指定覆寫應用程式參數。<br>PowerShell 應用程式參數會指定為雜湊表的名稱/值組。 例如，@{ "VotingData_MinReplicaSetSize" = "3"; "VotingData_PartitionCount" = "1" }.<br>您可以在 [應用程式參數檔案] 欄位中的 [發行 Service Fabric 應用程式] 對話方塊中，指定 Visual Studio 應用程式參數。
| 確認 |PS| 允許的值為 **True** 和 **False**。 在執行 Cmdlet 前提示您確認。 |
| ConsiderWarningAsError |PS, VS |允許的值為 **True** 和 **False**。 預設值為 [False]。 升級期間評估應用程式健康狀態時，將應用程式的警告健康狀態事件視為錯誤。 根據預設，Service Fabric 不會將警告健康狀態事件評估為失敗 (錯誤)，因此，即使有警告事件，升級還是可以繼續執行。 |
| DefaultServiceTypeHealthPolicy | PS, VS |以下列格式指定預設服務類型的健康原則，以用於受監視的升級：MaxPercentUnhealthyPartitionsPerService、MaxPercentUnhealthyReplicasPerPartition、MaxPercentUnhealthyServices。 例如，5、10、15 表示下列值：MaxPercentUnhealthyPartitionsPerService = 5、MaxPercentUnhealthyReplicasPerPartition = 10、MaxPercentUnhealthyServices = 15。 |
| Force | PS, VS | 允許的值為 **True** 和 **False**。 表示即使版本號碼未變更，升級程序還是會略過警告訊息並強制升級。 這適用於本機測試，但不建議用於生產環境，因為其需要移除現有的部署，這會導致停機和潛在的資料遺失問題。 |
| ForceRestart |PS, VS |如果您更新組態或資料套件而不更新服務程式碼，只有當 ForceRestart 屬性設為 **True** 時，服務才會重新啟動。 更新完成時，Service Fabric 會通知服務，新的組態封裝或資料封裝已可使用。 服務會負責套用變更。 必要時，服務可以自行重新啟動。 |
| HealthCheckRetryTimeoutSec |PS, VS |Service Fabric 在將升級宣告為失敗之前，繼續執行健康狀態評估的持續時間 (以秒為單位)。 預設值為 600 秒。 此期間會在達到 HealthCheckWaitDurationSec 之後開始計算。 在此 HealthCheckRetryTimeout 之內，Service Fabric 可能會執行多個應用程式健康情況的健康情況檢查。 預設值為 10 分鐘，您應該針對您的應用程式適當地加以自訂。 |
| HealthCheckStableDurationSec |PS, VS |移至下一個升級網域或完成升級之前，要確認應用程式是否穩定的持續時間 (以秒為單位)。 這個等候持續時間是用來在執行健康狀態檢查後，防止未偵測到的健康狀態變更。 預設值為 120 秒鐘，您應該針對您的應用程式適當地加以自訂。 |
| HealthCheckWaitDurationSec |PS, VS | 在升級網域上完成升級之後，Service Fabric 評估應用程式健康狀態的等待時間 (以秒為單位)。 這段期間也可視為應用程式在被視為健康之前應該執行的時間。 如果健康狀態檢查都通過，則升級程序會繼續進行下一個升級網域。  如果健康情況檢查失敗，Service Fabric 會等候 [UpgradeHealthCheckInterval](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-fabric-settings#clustermanager)，然後再重試一次健康情況檢查，直到達到 HealthCheckRetryTimeoutSec 為止。 預設值和建議值為 0 秒。 |
| MaxPercentUnhealthyDeployedApplications|PS, VS |預設值和建議值為 0。 指定應用程式被視為不健康和更新失敗之前，應用程式不健康之已部署的應用程式數目上限 (請參閱 [健康狀態章節](service-fabric-health-introduction.md))。 這個參數定義節點上的應用程式健康狀態，協助在升級期間偵測問題。 應用程式複本通常會負載平衡至其他節點，讓應用程式健康狀態良好，以便繼續升級。 藉由指定嚴謹的 MaxPercentUnhealthyDeployedApplications 健康情況，Service Fabric 可以快速偵測應用程式封裝的問題，並協助產生快速檢錯升級。 |
| MaxPercentUnhealthyServices |PS, VS |DefaultServiceTypeHealthPolicy 和 ServiceTypeHealthPolicyMap 的參數。 預設值和建議值為 0。 指定應用程式被視為不健康和升級失敗之前，不健康之應用程式執行個體中服務的數目上限。 |
| MaxPercentUnhealthyPartitionsPerService|PS, VS |預設值和建議值為 0。 指定服務被視為不健康之前，不健康之服務中分割的數目上限。 |
| MaxPercentUnhealthyReplicasPerPartition|PS, VS |預設值和建議值為 0。 指定分割被視為不健康之前，不健康之分割中複本的數目上限。 |
| ServiceTypeHealthPolicyMap | PS, VS | 表示此健康原則用來對某個服務類型的服務評估建康情況。 使用下列格式取用雜湊表的輸入：@ {"ServiceTypeName" : "MaxPercentUnhealthyPartitionsPerService,MaxPercentUnhealthyReplicasPerPartition,MaxPercentUnhealthyServices"} For example: @{ "ServiceTypeName01" = "5,10,5"; "ServiceTypeName02" = "5,5,5" } |
| TimeoutSec | PS, VS | 指定作業的逾時期限 (以秒為單位)。 |
| UpgradeDomainTimeoutSec |PS, VS |升級單一升級網域的時間上限 (以秒為單位)。 如果達到此逾時，升級會停止，然後根據 FailureAction 的設定繼續。 預設值為永不 (無限)，您應該針對您的應用程式適當地加以自訂。 |
| UpgradeReplicaSetCheckTimeoutSec |PS, VS |**無狀態服務**- 在單一升級網域內，Service Fabric 會嘗試確保服務的額外執行個體可用。 如果目標執行個體計數超過一個，Service Fabric 會等到有一個以上的執行個體可用，最多到逾時值上限。 此逾時會使用 UpgradeReplicaSetCheckTimeoutSec 屬性來指定。 如果逾時已到期，Service Fabric 會繼續進行升級，不論服務執行個體數目。 如果目標執行個體計數是一個，Service Fabric 不會等待，它會立即繼續進行升級。<br><br>**可設定狀態的服務**- 在單一升級網域內，Service Fabric 會嘗試確保複本集有仲裁。 Service Fabric 會等待有仲裁可用，最多到逾時值上限 (由 UpgradeReplicaSetCheckTimeoutSec 屬性指定)。 如果逾時已到期，Service Fabric 會繼續進行升級，不論是否有仲裁。 此設定在向前回復時是設定為永不 (無限)，向後回復時則為 1200 秒。 |
| UpgradeTimeoutSec |PS, VS |逾時 (以秒為單位) 適用於整個升級。 如果達到此逾時，升級會停止，並且會觸發 FailureAction。 預設值為永不 (無限)，您應該針對您的應用程式適當地加以自訂。 |
| WhatIf | PS | 允許的值為 **True** 和 **False**。 顯示執行 Cmdlet 後會發生的情況。 Cmdlet 並不會執行。 |

可以針對應用程式執行個體的各個服務類型指定 MaxPercentUnhealthyServices、MaxPercentUnhealthyPartitionsPerService 和 MaxPercentUnhealthyReplicasPerPartition 準則。 為每個服務設定這些參數可讓應用程式包含有不同評估原則的不同服務類型。 例如，特定應用程式執行個體的無狀態閘道器服務類型，可以有與可設定狀態的引擎服務類型不同的 MaxPercentUnhealthyPartitionsPerService。

## <a name="next-steps"></a>後續步驟
[使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md) 將引導您完成使用 Visual Studio 進行應用程式升級的步驟。

[使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md) 將引導您完成使用 PowerShell 進行應用程式升級的步驟。

[在 Linux 上使用 Service Fabric CLI 升級應用程式](service-fabric-application-lifecycle-sfctl.md#upgrade-application)會引導您完成使用 Service Fabric CLI 升級應用程式的步驟。

[使用 Service Fabric Eclipse 外掛程式升級應用程式](service-fabric-get-started-eclipse.md#upgrade-your-service-fabric-java-application)

了解如何使用 [資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。
