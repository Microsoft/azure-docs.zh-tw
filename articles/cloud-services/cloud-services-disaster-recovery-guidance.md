---
title: 處理會影響 Azure 雲端服務 (傳統) 的 Azure 服務中斷
description: 了解發生影響 Azure 雲端服務的 Azure 服務中斷事件時該怎麼辦。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: cdd6c9da5a1895d4aadd73133734cd4c8204ecf1
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98742160"
---
# <a name="what-to-do-in-the-event-of-an-azure-service-disruption-that-impacts-azure-cloud-services-classic"></a>發生影響 Azure 雲端服務 (傳統) 的 Azure 服務中斷事件時該怎麼辦

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

Microsoft 的同仁一向努力確保提供您需要的服務。 有時候因為不可抗力之影響，造成服務意外中斷。

Microsoft 為其服務提供服務等級協定 (SLA)，作為執行時間和連接承諾。 個別的 Azure 服務 SLA 位於 [Azure 服務等級協定](https://azure.microsoft.com/support/legal/sla/)。

Azure 已經有許多支援高可用性應用程式的內建平台功能。 如需這些服務的詳細資訊，請參閱 [Azure 應用程式的災害復原與高可用性](/azure/architecture/framework/resiliency/backup-and-recovery)。

本文內含當整個地區因重大天災或大規模服務中斷而發生中斷的真實災害復原案例。 這些都是極其罕見的情況，但您還是必須對整個區域發生中斷的可能性有所準備。 如果整個區域的服務中斷，會暫時無法使用本機的備援資料複本。 如果已啟用異地複寫，會在不同的區域儲存額外三份 Azure 儲存體 Blob 和資料表。 如果發生全面性區域中斷或嚴重損壞，且主要區域無法復原時，Azure 會重新對應異地複寫區域的所有 DNS 項目。

> [!NOTE]
> 請注意，您完全無法控制這個程序，而且它只有在整個資料中心服務中斷時才會發生。 因此，您也必須依賴其他的應用程式特定備份策略，以達到最高層級的可用性。 如需詳細資訊，請參閱[建置在 Microsoft Azure 上之應用程式的災害復原和高可用性](/azure/architecture/framework/resiliency/backup-and-recovery)。 如果您希望能夠影響您自己的容錯移轉，就可能想要考慮使用 [讀取存取異地備援儲存體 (RA-GRS)](../storage/common/storage-redundancy.md)，在其他區域中建立資料的唯讀複本。
>
>


## <a name="option-1-use-a-backup-deployment-through-azure-traffic-manager"></a>選項 1︰透過 Azure 流量管理員使用備份部署
最強固的災害復原解決方案涉及維護您的應用程式在不同區域中的多個部署，然後使用 [Azure 流量管理員](../traffic-manager/traffic-manager-overview.md)導向其間的流量。 Azure 流量管理員會提供多種[路由方式](../traffic-manager/traffic-manager-routing-methods.md)，因此您可以選擇使用主要/備份模型來管理您的部署，或分割其間的流量。

![使用 Azure 流量管理員平衡區域間的 Azure 雲端服務](./media/cloud-services-disaster-recovery-guidance/using-azure-traffic-manager.png)

若要對遺失區域做出最快的回應，請務必設定流量管理員的[端點監視](../traffic-manager/traffic-manager-monitoring.md)。

## <a name="option-2-deploy-your-application-to-a-new-region"></a>選項 2︰將應用程式部署至新的區域
如前一個選項所述維護多個作用中的部署，會造成額外的成本。 如果您的復原時間目標 (RTO) 有足夠的彈性，且您具有原始程式碼或已編譯的雲端服務套件，您即可在其他區域中建立應用程式的新執行個體並更新您的 DNS 記錄，以指向新的部署。

如需如何建立和部署雲端服務應用程式的詳細資訊，請參閱[如何建立和部署雲端服務](cloud-services-how-to-create-deploy-portal.md)。

根據您的應用程式資料來源，您可能需要檢查應用程式資料來源的復原程序。

* 針對 Azure 儲存體資料來源，請參閱 [Azure 儲存體冗余](../storage/common/storage-redundancy.md) ，以根據您的應用程式所選擇的冗余模型來查看可用的選項。
* 如需 SQL 資料庫來源，請閱讀 [概觀：雲端商務持續性和 SQL Database 的資料庫災害復原](../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md) ，以根據您針對應用程式所選擇的複寫模型來查看可用的選項。


## <a name="option-3-wait-for-recovery"></a>選項 3︰等待復原
在此情況下，您不需要採取任何動作，但是直到該區域還原，才可使用您的服務。 您可以在 [Azure 服務健康狀態儀表板](https://azure.microsoft.com/status/)上看見目前的服務狀態。

## <a name="next-steps"></a>後續步驟
若要深入了解如何實作災害復原和高可用性策略，請參閱 [Azure 應用程式的災害復原和高可用性](/azure/architecture/framework/resiliency/backup-and-recovery)。

若要開發雲端平台功能的詳細技術知識，請參閱 [Azure 復原技術指導](/azure/architecture/checklist/resiliency-per-service)。