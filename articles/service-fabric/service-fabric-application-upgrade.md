---
title: Service Fabric 應用程式升級 | Microsoft Docs
description: 本文章提供升級 Service Fabric 應用程式的簡介，其中包括選擇升級模式和執行健康狀態檢查。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: 803c9c63-373a-4d6a-8ef2-ea97e16e88dd
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: fca05a1b21e1cefd4146f754d7dedda0d7ff2ac0
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207973"
---
# <a name="service-fabric-application-upgrade"></a>Service Fabric 應用程式升級
Azure Service Fabric 應用程式是服務集合。 在升級期間，Service Fabric 會比較新的 [應用程式資訊清單](service-fabric-application-and-service-manifests.md) 與舊版本，並決定應用程式中哪些服務需要更新。 Service Fabric 會比較服務資訊清單中的版本號碼和上一版中的版本號碼。 如果服務未變更，則該服務不會升級。

## <a name="rolling-upgrades-overview"></a>輪流升級概觀
在輪流應用程式升級中，升級是階段執行。 在每個階段中，升級會套用至叢集中的節點子集，稱為更新網域。 如此一來，應用程式在整個升級過程中仍然可供使用。 升級期間，叢集可能包含舊和新版本的混合。

因此，兩個版本必須是向前及向後相容。 如果它們不相容，應用程式系統管理員負責執行多階段升級，以維護可用性。 在多階段升級中，第一個步驟是升級到與先前版本相容的應用程式中繼版本。 第二個步驟是升級與更新前版本中斷相容性，但與中繼版本相容的最終版本。

當您設定叢集時，會在叢集資訊清單中指定更新網域。 更新網域不會以特定順序收到更新。 更新網域是應用程式的部署邏輯單元。 更新網域可以讓服務在升級期間維持高可用性。

如果升級套用到叢集中的所有節點 (應用程式只有一個更新網域就是這種情形)，則不可能進行輪流升級。 不建議使用此方法，因為升級時服務會中斷且無法使用。 此外，當叢集僅以一個更新網域進行設定時，Azure 不提供任何保證。

升級完成時，所有服務和複本 (執行個體) 都會保留在相同的版本中，也就是說，如果升級成功，就會更新到新版本中；如果升級失敗而且被回復，就會回復到舊版本。

## <a name="health-checks-during-upgrades"></a>升級期間的健康狀態檢查
對於升級，需要設定健康狀態原則 (或可能會使用預設值)。 當所有更新網域在指定的逾時內升級，且所有更新網域都視為健康時，則稱為成功升級。  健康的更新網域意指更新網域會通過健康狀態原則中指定的所有健康狀態檢查。 例如，健康狀態原則可能會要求應用程式執行個體中的所有服務都必須 *健康狀態良好*，如 Service Fabric 定義的健康狀況。

Service Fabric 在升級期間進行的健康狀態原則以及檢查不限於服務和應用程式。 也就是說，不會完成任何服務專屬的測試。  例如，您的服務可能需要輸送量，但 Service Fabric 並沒有檢查輸送量的資訊。 如需了解會執行的檢查，請參閱 [健康狀態文章](service-fabric-health-introduction.md) 。 在升級期間進行的檢查包括是否已正確地複製應用程式封裝、是否已啟動執行個體等等。

應用程式健康狀態是應用程式的子系實體彙總。 簡單地說，Service Fabric 會透過應用程式報告的健康狀態來評估應用程式的健康狀態。 它也會以此種方式評估應用程式所有服務的健康狀態。 Service Fabric 會進一步藉由彙總其子項的健康狀態 (例如服務複本) 來評估應用程式服務的健康狀態。 一旦符合應用程式健康狀態原則，便可以繼續升級。 如果違反健康狀態原則，則應用程式升級將會失敗。

## <a name="upgrade-modes"></a>升級模式
我們建議的應用程式升級模式為監視模式，這是常用的模式。 監視模式會在一個更新網域上執行升級，如果所有健康狀態檢查都通過 (每個指定的原則)，即會自動移至下一個更新網域。  如果健康狀態檢查失敗及/或達到逾時，更新網域的升級就會回復，或者手動變更為未受監視的手動模式。 您可以設定升級在升級失敗時選擇這兩種其中一種模式。 

不受監控手動模式在每次於更新網域上升級之後都需要手動介入，以開始進行下一個更新網域上的升級。 系統不會執行任何 Service Fabric 健康狀態檢查。 系統管理員在開始下一個更新網域中的升級之前，會執行健康狀態或狀態檢查。

## <a name="upgrade-default-services"></a>升級預設服務
某些定義於[應用程式資訊清單](service-fabric-application-and-service-manifests.md)中的預設服務參數，也可在應用程式升級的過程中一併升級。 只有支援透過 [Update-ServiceFabricService](https://docs.microsoft.com/powershell/module/servicefabric/update-servicefabricservice?view=azureservicefabricps) 進行變更的服務參數，才能在升級的過程中變更。 在應用程式升級期間變更預設服務的行為如下：

1. 會建立不存在於叢集中的新應用程式資訊清單中的預設服務。
2. 會更新同時存在於舊有與新的應用程式資訊清單中的預設服務。 新應用程式資訊清單中的預設服務參數會覆寫現有服務的參數。 在預設服務更新失敗時，應用程式升級會自動回復。
3. 不存在於新應用程式資訊清單中的預設服務若存在於叢集中，將會被刪除。 **請注意，刪除預設服務將導致所有服務的狀態都被刪除，而且無法復原。**

當應用程式升級回復時，預設服務參數會還原為升級啟動之前的舊值，但已刪除的服務無法以其舊狀態重新建立。

> [!TIP]
> [EnableDefaultServicesUpgrade](service-fabric-cluster-fabric-settings.md) 叢集組態設定值必須是 *true*，以啟用規則 2) 和 3) 上述項目 (預設服務更新或刪除)。 從 Service Fabric 5.5 版開始即支援此功能。

## <a name="upgrading-multiple-applications-with-https-endpoints"></a>使用 HTTPS 端點來升級多個應用程式
使用 HTTPS**S** 時，請務必小心，不要將**相同的連接埠**用於相同應用程式的不同執行個體。 原因是 Service Fabric 將無法升級其中一個應用程式執行個體的憑證。 舉例來說，如果應用程式 1 或應用程式 2 都想要將其憑證 1 升級至憑證 2。 當進行升級時，Service Fabric 可能已清除憑證 1 在 http.sys 的註冊，儘管另一個應用程式仍然在使用它。 為了防止這種情況，Service Fabric 會偵測到該連接埠上已經有另一個應用程式以該憑證註冊 (因為 http.sys)，而讓作業失敗。

因此，Service Fabric 不支援在不同的應用程式執行個體中，使用「相同的連接埠」來升級兩個不同的服務。 換句話說，您無法在相同連接埠的不同服務上使用相同的憑證。 如果您需要在相同的連接埠上使用共用憑證，就必須使用放置條件約束，以確保將這些服務放在不同的機器上。 或者，可能的話，考慮針對每個應用程式執行個體中的每個服務，使用 Service Fabric 動態連接埠。 

如果您看到使用 https 進行升級失敗，系統將會顯示錯誤警告「Windows HTTP 伺服器 API 針對共用連接埠的應用程式不支援多個憑證」。

## <a name="application-upgrade-flowchart"></a>應用程式升級流程圖
本段落下面的流程圖可以協助您了解 Service Fabric 應用程式的升級程序。 特別是，此流程會說明逾時 (包括 HealthCheckStableDuration、HealthCheckRetryTimeout 和 UpgradeHealthCheckInterval) 如何協助控制一個更新網域中的升級被視為成功或失敗的時機。

![Service Fabric 應用程式的升級程序][image]

## <a name="next-steps"></a>後續步驟
[使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md) 將引導您完成使用 Visual Studio 進行應用程式升級的步驟。

[使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md) 將引導您完成使用 PowerShell 進行應用程式升級的步驟。

使用 [升級參數](service-fabric-application-upgrade-parameters.md)來控制您應用程式的升級方式。

了解如何使用 [資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。

[image]: media/service-fabric-application-upgrade/service-fabric-application-upgrade-flowchart.png
