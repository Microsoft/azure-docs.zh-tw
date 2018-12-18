---
title: Azure Service Fabric 生產環境整備檢查清單 | Microsoft Docs
description: 遵循最佳做法來使您的 Service Fabric 應用程式和叢集生產環境準備就緒。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 7/10/2018
ms.author: subramar
ms.openlocfilehash: 6ebe83794f23ed3cf637f668d2810b9ba7521084
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44295865"
---
# <a name="production-readiness-checklist"></a>實際執行整備檢查清單

您的應用程式和叢集已經準備好要接受生產環境流量嗎？ 執行並測試您的應用程式和叢集，不一定表示它已經準備好進入生產環境。 完成下列檢查清單，使您的應用程式和叢集保持順暢執行。 我們強烈建議您檢查這些所有項目。 您顯然可以選擇針對特定明細項目 (例如，您自己的診斷架構) 使用替代的解決方案。


## <a name="pre-requisites-for-production"></a>適用於生產環境的必要條件

1. 若叢集的核心超過 20 個或節點超過 10 個，請建立系統服務專用的主要節點類型。 新增[放置條件約束](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md)，以保留系統服務的主要節點類型。 
2. 針對主要節點類型使用 D2v2 或更高的 SKU。 建議挑選至少具有 50 GB 硬碟容量的 SKU。
2. 生產環境叢集必須是[安全的](service-fabric-cluster-security.md)。 如需設定安全叢集的範例，請參閱此[叢集範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/7-VM-Windows-3-NodeTypes-Secure-NSG) \(英文\)。 針對憑證使用通用名稱，並避免使用自我簽署的憑證。
4. 新增[容器和服務的資源條件約束](service-fabric-resource-governance.md)，如此一來，它們就不會耗用超過 75% 的節點資源。 
5. 了解並設定[持久性層級](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster)。 建議針對執行具狀態工作負載的節點類型使用銀級或更高的持久性層級。 主要節點類型的持久性層級應該設定為銀級或更高層級。
6. 了解並挑選節點類型的[可靠性層級](service-fabric-cluster-capacity.md#the-reliability-characteristics-of-the-cluster)。 建議採用銀級或更高層級的可靠性。
7. 載入您的工作負載並進行調整測試，以識別叢集的[容量需求](service-fabric-cluster-capacity.md)。 
8. 您的服務和應用程式會受到監視，並產生及儲存應用程式記錄，而且會發出警示。 例如，請參閱[為 Service Fabric 應用程式新增記錄功能](service-fabric-how-to-diagnostics-log.md)和[使用 Log Analytics 監視容器](service-fabric-diagnostics-oms-containers.md)。
9. 叢集會受到監視並發出警示 (例如，使用 [OMS](service-fabric-diagnostics-event-analysis-oms.md))。 
10. 基礎的虛擬機器擴展集基礎結構會受到監視並發出警示 (例如，使用 [Log Analytics](service-fabric-diagnostics-oms-agent.md))。
11. 叢集一律具有[主要和次要憑證](service-fabric-cluster-security-update-certs-azure.md) (讓您不會遭到鎖定)。
12. 分別維護適用於開發、預備及生產環境的叢集。 
13. [應用程式升級](service-fabric-application-upgrade.md)和[叢集升級](service-fabric-tutorial-upgrade-cluster.md)均會先在開發和預備叢集中進行測試。 
14. 在生產環境叢集中關閉自動升級，並針對開發和預備叢集 (視需要復原) 開啟它。 
15. 為您的服務建立復原點目標 (RPO)，以及設定[災害復原程序](service-fabric-disaster-recovery.md)並加以測試。
16. 規劃以手動方式或以程式設計方式[調整](service-fabric-cluster-scaling.md)您的叢集。
17. 規劃[修補](service-fabric-patch-orchestration-application.md)您的叢集節點。 
18. 建立 CI/CD 管線，以持續測試您的最新變更。 例如，使用 [Azure DevOps](service-fabric-tutorial-deploy-app-with-cicd-vsts.md) 或 [Jenkins](service-fabric-cicd-your-linux-applications-with-jenkins.md)
19. 使用[錯誤分析服務](service-fabric-testability-overview.md)在低於負載的情況下測試您的開發與預備叢集，並引發受控制的[混亂](service-fabric-controlled-chaos.md)。 
20. 規劃[調整](service-fabric-concepts-scalability.md)您的應用程式。 


如果您使用的是 Service Fabric Reliable Services 或 Reliable Actors 程式設計模型，就必須核對下列項目：
21. 在本機開發期間升級應用程式，以檢查您的服務程式碼是否會遵循 `RunAsync` 方法中的取消權杖，並關閉自訂通訊接聽程式。
22. 使用可靠的集合時避免[常見陷阱](service-fabric-work-with-reliable-collections.md)。
23. 在執行負載測試時監視 .NET CLR 記憶體效能計數器，並檢查是否有高比率的記憶體回收或失控的堆積成長。
24. 維護 [Reliable Services 和 Reliable Actors](service-fabric-reliable-services-backup-restore.md) 的離線備份並測試還原程序。 


## <a name="optional-best-practices"></a>選用的最佳做法

儘管上述清單為移至生產環境的必要條件，但您也應該將下列項目納入考量：
25. 插入 [Service Fabric 健康情況模型](service-fabric-health-introduction.md)，以擴充內建的健康情況評估和報告功能。
26. 部署自訂的看門狗，以監視您的應用程式和報表[負載](service-fabric-cluster-resource-manager-metrics.md)來進行[資源平衡](service-fabric-cluster-resource-manager-balancing.md)。 


## <a name="next-steps"></a>後續步驟
* [部署 Service Fabric Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [部署 Service Fabric Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* 深入了解 Service Fabric [應用程式生命週期](service-fabric-application-lifecycle.md)。
