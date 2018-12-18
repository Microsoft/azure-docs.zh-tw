---
title: 升級 Azure Service Fabric 叢集 | Microsoft Docs
description: 升級執行 Service Fabric 叢集的 Service Fabric 程式碼和/或組態，包括設定叢集更新模式、升級憑證、新增應用程式連接埠、修補作業系統等等。 執行升級時，您可以期待些什麼？
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: ''
ms.assetid: 15190ace-31ed-491f-a54b-b5ff61e718db
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 8/10/2017
ms.author: aljo
ms.openlocfilehash: 3e5ef2cbc9d7bec82cadc0c7663de52636938505
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207694"
---
# <a name="upgrade-an-azure-service-fabric-cluster"></a>升級 Azure Service Fabric 叢集
> [!div class="op_single_selector"]
> * [Azure 叢集](service-fabric-cluster-upgrade.md)
> * [獨立叢集](service-fabric-cluster-upgrade-windows-server.md)
> 
> 

對於任何現代系統，可升級性的設計是產品達到長期成功的關鍵。 Azure Service Fabric 叢集是您所擁有，但部分由 Microsoft 管理的資源。 本文說明自動管理的部分以及您可以自行設定的部分。

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>控制叢集上執行的網狀架構版本
您可以將叢集設定為在 Microsoft 釋出網狀架構升級時自動接收該升級，或者，您也可以選取您想讓叢集執行的受支援網狀架構版本。

作法是在入口網站上設定 “upgrademode” 叢集組態，或是建立時或稍後在即時叢集上使用 Resource Manager 來設定。 

> [!NOTE]
> 請務必保持您的叢集一律執行支援的網狀架構版本。 當我們宣布發行新版本的 Service Fabric 時，從當日起至少 60 天後，舊版就會標示為結束支援。 新的版本會於 [Service Fabric 小組部落格上](https://blogs.msdn.microsoft.com/azureservicefabric/)發佈。 那時就有新的版本可選擇。 
> 
> 

叢集執行的版本在到期前 14 天會產生健全狀況事件，使您的叢集進入警告健全狀況狀態。 在您升級至支援的網狀架構版本之前，叢集會停留在警告狀態。

### <a name="setting-the-upgrade-mode-via-portal"></a>透過入口網站設定升級模式
當您建立叢集時，您可以將叢集設為自動或手動。

![Create_Manualmode][Create_Manualmode]

在即時叢集上，您可以利用管理體驗將叢集設為自動或手動。 

#### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-portal"></a>在設定為手動模式的叢集上，透過入口網站升級至新的版本。
若要升級至新的版本，只需要從下拉式清單中選取可用的版本並儲存即可。 Fabric 升級會自動開始進行。 升級期間會遵守叢集健康狀態原則 (綜合節點健康狀態和所有在叢集中執行之應用程式的健康狀態)。

如果不符合叢集健康狀態原則，則會回復升級。 請往下捲動本文，以深入了解如何設定這些自訂的健康狀態原則。 

在解決導致復原的問題後，您需要依照之前的相同步驟再次起始升級。

![Manage_Automaticmode][Manage_Automaticmode]

### <a name="setting-the-upgrade-mode-via-a-resource-manager-template"></a>透過 Resource Manager 範本設定升級模式
將 “upgradeMode" 組態新增至 Microsoft.ServiceFabric/clusters 資源定義，並將 “clusterCodeVersion" 設為其中一個支援的網狀架構版本，如下所示，然後部署範本。 “upgradeMode" 的有效值為 “Manual” 或 “Automatic”。

![ARMUpgradeMode][ARMUpgradeMode]

#### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-a-resource-manager-template"></a>在設定為手動模式的叢集上，透過 Resource Manager 範本升級至新的版本。
當叢集處於手動模式時，若要升級至新的版本，請將 “clusterCodeVersion" 變更為支援的版本並部署。 部署範本時會自動展開 Fabric 升級。 升級期間會遵守叢集健康狀態原則 (綜合節點健康狀態和所有在叢集中執行之應用程式的健康狀態)。

如果不符合叢集健康狀態原則，則會回復升級。 請往下捲動本文，以深入了解如何設定這些自訂的健康狀態原則。 

在解決導致復原的問題後，您需要依照之前的相同步驟再次起始升級。

### <a name="get-list-of-all-available-version-for-all-environments-for-a-given-subscription"></a>針對給定的訂用帳戶取得所有環境的所有可用版本清單
執行下列命令，應該會得到類似如下的輸出。

“supportExpiryUtc” 會告訴您給定的版本即將到期或已過期。 最新版本並無有效日期 - 它的值為 "9999-12-31T23:59:59.9999999"，這只是表示到期日還沒有設定。

```REST
GET https://<endpoint>/subscriptions/{{subscriptionId}}/providers/Microsoft.ServiceFabric/locations/{{location}}/clusterVersions?api-version=2016-09-01

Example: https://management.azure.com/subscriptions/1857f442-3bce-4b96-ad95-627f76437a67/providers/Microsoft.ServiceFabric/locations/eastus/clusterVersions?api-version=2016-09-01

Output:
{
                  "value": [
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/5.0.1427.9490",
                      "name": "5.0.1427.9490",
                      "type": "Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.0.1427.9490",
                        "supportExpiryUtc": "2016-11-26T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.0.1427.9490",
                      "name": "5.1.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.1.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.4.1427.9490",
                      "name": "4.4.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "4.4.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Linux"
                      }
                    }
                  ]
                }


```

## <a name="fabric-upgrade-behavior-when-the-cluster-upgrade-mode-is-automatic"></a>叢集升級模式為自動時的 Fabric 升級行為
Microsoft 會維護 Azure 叢集中執行的網狀架構程式碼和組態。 視情況需要，我們會對軟體執行受監視的自動升級。 升級的項目可能是程式碼、組態或兩者。 為了確保您的應用程式不受這些升級影響或將影響降到最低，我們會分成下列階段執行升級。

### <a name="phase-1-an-upgrade-is-performed-by-using-all-cluster-health-policies"></a>階段 1：使用所有叢集健康狀態原則執行升級
在這個階段，升級會一次進行一個升級網域，而已在叢集中執行的應用程式會繼續執行，而不會產生任何停機時間。 升級期間會遵守叢集健康狀態原則 (綜合節點健康狀態和所有在叢集中執行之應用程式的健康狀態)。

如果不符合叢集健康狀態原則，則會回復升級。 然後系統會傳送一封電子郵件給訂用帳戶的擁有者。 電子郵件中包含下列資訊：

* 我們必須回復叢集升級的通知。
* 建議的修復動作 (如果有的話)。
* 距離我們執行階段 2 之前的天數 (n)。

如果有任何升級因為基礎結構方面的原因而失敗，我們會試著多執行幾次相同的升級。 在電子郵件寄送日期的 n 天之後，我們會繼續進行階段 2。

如果符合叢集健康狀態原則，則升級會被視為成功並標示為完成。 在此階段執行初次升級或重新執行任何升級期間，可能會發生這種情形。 執行成功不會有任何電子郵件確認。 這是為了避免傳送給您太多電子郵件，如果您收到電子郵件，則應該將其視為例外狀況。 我們預期大部分的叢集升級都會成功，並不會影響您的應用程式可用性。

### <a name="phase-2-an-upgrade-is-performed-by-using-default-health-policies-only"></a>階段 2：僅使用預設健康狀態原則執行升級
此階段的健康狀態原則的設定方式為，升級開始時健康狀態良好的應用程式數目在升級程序期間會維持不變。 和階段 1 一樣，階段 2 的升級會一次進行一個升級網域，而已在叢集中執行的應用程式會繼續執行，而不會產生任何停機時間。 在升級期間，會遵守叢集健康狀態原則 (節點健康狀態和所有在叢集中執行之應用程式的健康狀態的組合)。

如果不符合生效的叢集健康狀態原則，則會回復升級。 然後系統會傳送一封電子郵件給訂用帳戶的擁有者。 電子郵件中包含下列資訊：

* 我們必須回復叢集升級的通知。
* 建議的修復動作 (如果有的話)。
* 距離我們執行階段 3 之前的天數 (n)。

如果有任何升級因為基礎結構方面的原因而失敗，我們會試著多執行幾次相同的升級。 在 n 天到達的前幾天會傳送提醒電子郵件。 在電子郵件寄送日期的 n 天之後，我們會繼續進行階段 3。 您必須認真看待我們在階段 2 寄送的電子郵件並採取補救動作。

如果符合叢集健康狀態原則，則升級會被視為成功並標示為完成。 在此階段執行初次升級或重新執行任何升級期間，可能會發生這種情形。 執行成功不會有任何電子郵件確認。

### <a name="phase-3-an-upgrade-is-performed-by-using-aggressive-health-policies"></a>階段 3：使用積極的叢集健康狀態原則執行升級
此階段的這些健康狀態原則的目的是升級完成，而不是應用程式的健康狀態。 很少叢集升級會最後會變成這個階段。 如果您的叢集進入這個階段，表示您的應用程式很可能會變成狀況不良和 (或) 失去可用性。

類似其他兩個階段，階段 3 升級會一次進行一個升級網域。

如果不符合叢集健康狀態原則，則會回復升級。 如果有任何升級因為基礎結構方面的原因而失敗，我們會試著多執行幾次相同的升級。 之後，便會鎖住叢集，使它不會再收到支援和 (或) 升級。

系統會傳送含有這項資訊的電子郵件給訂用帳戶擁有者，其中也會附上修復動作。 我們預期不會有任何叢集會進入階段 3 失敗的狀態。

如果符合叢集健康狀態原則，則升級會被視為成功並標示為完成。 在此階段執行初次升級或重新執行任何升級期間，可能會發生這種情形。 執行成功不會有任何電子郵件確認。

## <a name="cluster-configurations-that-you-control"></a>您控制的叢集組態
除了能夠設定叢集升級模式，您還可以在即時叢集上變更以下的設定。

### <a name="certificates"></a>憑證
您可以透過入口網站輕鬆地新增或刪除叢集和用戶端的憑證。 請參閱 [這份文件了解詳細指示](service-fabric-cluster-security-update-certs-azure.md)

![顯示 Azure 入口網站中憑證指紋的螢幕擷取畫面。][CertificateUpgrade]

### <a name="application-ports"></a>應用程式連接埠
您可以透過變更與節點類型相關聯的負載平衡器資源屬性來變更應用程式連接埠。 您可以使用入口網站，或是直接使用資源管理員 PowerShell。

若要在某個節點類型中的所有 VM 上開啟新的連接埠，請執行下列作業：

1. 將新的探查新增至適當的負載平衡器。
   
    如果您已使用入口網站部署您的叢集，則負載平衡器會命名為 "LB-name of the Resource group-NodeTypename"，每個節點類型有一個負載平衡器。 因為負載平衡器名稱只有在資源群組中是唯一的，所以最好在特定資源群組下搜尋名稱。
   
    ![顯示在入口網站中對負載平衡器新增探查的螢幕擷取畫面。][AddingProbes]
2. 將新規則加入至負載平衡器。
   
    使用您在上一個步驟中建立的探查，對相同的負載平衡器加入新的規則。
   
    ![在入口網站中對負載平衡器新增規則。][AddingLBRules]

### <a name="placement-properties"></a>放置屬性
對於每個節點類型，您可以加入您要在應用程式中使用的自訂放置屬性。 NodeType 是您可使用而不需明確新增的預設屬性。

> [!NOTE]
> 如需使用放置條件約束、節點屬性及如何定義它們的詳細資訊，請參閱《Service Fabric 叢集 Resource Manager 文件》 [描述您的叢集](service-fabric-cluster-resource-manager-cluster-description.md)中的＜放置條件約束和節點屬性＞一節。
> 
> 

### <a name="capacity-metrics"></a>容量度量
對於每個節點類型，您可以加入您要在應用程式中用來報告負載的自訂容量計量。 如需使用容量度量報告負載的詳細資訊，請參閱《Service Fabric 叢集 Resource Manager 文件》的[描述您的叢集](service-fabric-cluster-resource-manager-cluster-description.md)和[度量和負載](service-fabric-cluster-resource-manager-metrics.md)。

### <a name="fabric-upgrade-settings---health-polices"></a>Fabric 升級設 - 健全狀況原則
您可以為網狀架構升級指定自訂的健全狀況原則。 如果已將您的叢集設定為自動網狀架構升級，這些原則會套用於自動網狀架構升級的階段 1。
如果已將您的叢集設定為手動網狀架構升級，則每當您選取新版本而觸發系統開始在叢集中展開網狀架構升級時，就會套用這些原則。 如果您不覆寫原則，則會使用預設值。

在 [網狀架構升級] 刀鋒視窗下，您可以選取進階升級設定來指定自訂的健康狀態原則，或檢閱目前的設定。 檢閱下列作法圖片。 

![管理自訂的健康狀態原則][HealthPolices]

### <a name="customize-fabric-settings-for-your-cluster"></a>自訂叢集的 Fabric 設定
請參閱 [Service Fabric 叢集網狀架構設定](service-fabric-cluster-fabric-settings.md) ，以了解該自訂什麼及如何自訂。

### <a name="os-patches-on-the-vms-that-make-up-the-cluster"></a>組成叢集的 VM 上的作業系統修補程式
請參閱[修補程式協調流程應用程式](service-fabric-patch-orchestration-application.md)，可在叢集上部署，以協調的方式從 Windows Update 安裝修補程式，讓服務隨時可供使用。 

### <a name="os-upgrades-on-the-vms-that-make-up-the-cluster"></a>組成叢集的 VM 上的作業系統升級
如果您必須升級叢集的虛擬機器上的作業系統映像，則必須一次升級一部 VM。 您要負責這項升級，因為目前沒有將這項升級自動化。

## <a name="next-steps"></a>後續步驟
* 了解如何自訂一些 [Service Fabric 叢集網狀架構設定](service-fabric-cluster-fabric-settings.md)
* 了解如何 [相應放大和相應縮小叢集](service-fabric-cluster-scale-up-down.md)
* 了解 [應用程式升級](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
