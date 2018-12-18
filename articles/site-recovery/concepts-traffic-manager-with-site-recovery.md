---
title: Azure 流量管理員搭配 Azure Site Recovery | Microsoft Docs
description: 說明如何搭配使用 Azure 流量管理員與 Azure Site Recovery 以進行災害復原和移轉
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: 8f303ec3349b0003ef764e437c6f06cf498dcc57
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353249"
---
# <a name="azure-traffic-manager-with-azure-site-recovery"></a>Azure 流量管理員搭配 Azure Site Recovery

Azure 流量管理員可讓您控制流量分散到應用程式端點的方式。 端點是裝載於 Azure 內部或外部的任何網際網路對向服務。

流量管理員會使用網域名稱系統 (DNS)，根據流量路由方法和端點的健康情況，將用戶端要求導向到最適當的端點。 流量管理員提供[流量路由方法](../traffic-manager/traffic-manager-routing-methods.md)和[端點監視選項](../traffic-manager/traffic-manager-monitoring.md)的範圍，以符合不同的應用程式需求和自動容錯移轉模型。 用戶端會直接連線至選取的端點。 流量管理員不是 Proxy 或閘道，它看不到在用戶端與服務之間傳遞的流量。

本文說明如何結合 Azure 流量監視器的智慧型路由與 Azure Site Recovery 強大的災害復原和移轉功能。

## <a name="on-premises-to-azure-failover"></a>從內部部署容錯移轉至 Azure

針對第一個案例，我們假設**公司 A** 的所有應用程式基礎結構都是在內部部署環境中執行。 基於商務持續性和合規性因素，**公司 A** 決定使用 Azure Site Recovery 來保護其應用程式。

**公司 A** 以公用端點執行應用程式，而希望獲得在災害事件發生時順暢地將流量重新導向至 Azure 的能力。 Azure 流量管理員中的[優先順序](../traffic-manager/traffic-manager-configure-priority-routing-method.md)流量路由方法，可讓公司 A 輕鬆實作此容錯移轉模式。

設定方式如下︰
- **公司 A** 建立[流量管理員設定檔](../traffic-manager/traffic-manager-create-profile.md)。
- **公司 A** 利用**優先順序**路由方法建立兩個端點 – **主要**用於內部部署，**容錯移轉**用於 Azure。 **主要**會被指派優先順序 1，**容錯移轉**會被指派優先順序 2。
- 由於**主要**端點裝載於 Azure 以外，因此這個端點會建立為[外部](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints)端點。
- 在使用 Azure Site Recovery 的情況下，Azure 網站並沒有任何在容錯移轉前執行的虛擬機器或應用程式。 因此，**容錯移轉**端點也會建立為**外部**端點。
- 根據預設，使用者流量會導向至內部部署應用程式，因為該端點具有最高的相關優先順序。 如果**主要**端點狀況良好，則沒有流量會導向至 Azure。

![容錯移轉之前從內部部署到 Azure 的流程](./media/concepts-traffic-manager-with-site-recovery/on-premises-failover-before.png)

在災害事件發生時，公司 A 可觸發對 Azure 的[容錯移轉](site-recovery-failover.md)，並在 Azure 上復原其應用程式。 當 Azure 流量管理員偵測到**主要**端點已處於不良狀況時，它會自動在 DNS 回應中使用**容錯移轉**端點，而使用者會連線至在 Azure 上復原的應用程式。

![容錯移轉之後從內部部署到 Azure 的流程](./media/concepts-traffic-manager-with-site-recovery/on-premises-failover-after.png)

根據商業需求，**公司 A** 可以選擇較高或較低的[探查頻率](../traffic-manager/traffic-manager-monitoring.md)來執行災害事件發生時在內部部署與 Azure 之間的切換，並確保使用者可以有最短的停機時間。

在災害獲得抑制後，**公司 A** 即可使用 Azure Site Recovery 從 Azure 容錯回復至其內部部署環境 ([VMware](vmware-azure-failback.md) 或 [Hyper-V](hyper-v-azure-failback.md))。 此時，當流量管理員偵測到**主要**端點恢復良好的狀況時，將會自動在其 DNS 回應中使用**主要**端點。

## <a name="on-premises-to-azure-migration"></a>從內部部署移轉至 Azure

除了災害復原以外，Azure Site Recovery 也可讓您[移轉至 Azure](migrate-overview.md)。 客戶可使用 Azure Site Recovery 強大的測試容錯移轉功能來評估 Azure 上的應用程式效能，而不會影響其內部部署環境。 此外，當客戶準備好要移轉時，他們可以選擇同時移轉整個工作負載，或選擇在移轉後逐漸調整。

Azure 流量管理員的[加權](../traffic-manager/traffic-manager-configure-weighted-routing-method.md)路由方法可用來將部分傳入的流量導向至 Azure，並將大部分的流量導向至內部部署環境。 這種方法有助於評估調整效能，因為您可以在越來越多的工作負載移轉至 Azure 時，持續增加指派給 Azure 的加權。

例如，**公司 B** 選擇分階段進行移轉，在移轉其部分的應用程式環境時保留其餘內部部署。 在初始階段期間，大部分的環境仍處於內部部署時，內部部署環境會被指派較大的加權。 流量管理員會根據指派給可用端點的加權傳回端點。

![從內部部署移轉至 Azure](./media/concepts-traffic-manager-with-site-recovery/on-premises-migration.png)

在移轉期間，兩個端點皆處於作用中，而大部分的流量會導向至內部部署環境。 當移轉繼續進行時，您可以將較大的加權指派給 Azure 上的端點，而在移轉完成後最終將可停用內部部署端點。

## <a name="azure-to-azure-failover"></a>從 Azure 容錯移轉至 Azure

在此範例中，我們假設**公司 C** 的所有應用程式基礎結構皆執行 Azure。 基於商務持續性和合規性因素，**公司 C** 決定使用 Azure Site Recovery 來保護其應用程式。

**公司 C** 以公用端點執行應用程式，而希望獲得在災害事件發生時順暢地將流量重新導向至不同 Azure 區域的能力。 [優先順序](../traffic-manager/traffic-manager-configure-priority-routing-method.md)流量路由方法可讓**公司 C** 輕鬆實作此容錯移轉模式。

設定方式如下︰
- **公司 C** 建立[流量管理員設定檔](../traffic-manager/traffic-manager-create-profile.md)。
- **公司 C** 利用**優先順序**路由方法建立兩個端點 – **主要**用於來源區域 (Azure 東亞)，**容錯移轉**用於復原區域 (Azure 東南亞)。 **主要**會被指派優先順序 1，**容錯移轉**會被指派優先順序 2。
- 由於**主要**端點裝載於 Azure 中，因此這個端點可作為 [Azure](../traffic-manager/traffic-manager-endpoint-types.md#azure-endpoints) 端點。
- 在使用 Azure Site Recovery 的情況下，復原 Azure 網站並沒有任何在容錯移轉前執行的虛擬機器或應用程式。 因此，**容錯移轉**端點可建立為[外部](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints)端點。
- 根據預設，使用者流量會導向至來源區域 (東亞) 應用程式，因為該端點具有最高的相關優先順序。 如果**主要**端點狀況良好，則沒有流量會導向至復原區域。

![容錯移轉之前從 Azure 到 Azure 的流程](./media/concepts-traffic-manager-with-site-recovery/azure-failover-before.png)

在災害事件發生時，**公司 C** 可觸發[容錯移轉](azure-to-azure-tutorial-failover-failback.md)，並在復原 Azure 區域復原其應用程式。 當 Azure 流量管理員偵測到主要端點已處於不良狀況時，它會自動在 DNS 回應中使用**容錯移轉**端點，而使用者會連線至在復原 Azure 區域 (東南亞) 復原的應用程式。

![容錯移轉之後從 Azure 到 Azure 的流程](./media/concepts-traffic-manager-with-site-recovery/azure-failover-after.png)

根據商業需求，**公司 C** 可以選擇較高或較低的[探查頻率](../traffic-manager/traffic-manager-monitoring.md)來執行來源與復原區域之間的切換，並確保使用者可以有最短的停機時間。

在災害獲得抑制後，**公司 C** 即可使用 Azure Site Recovery 從復原 Azure 區域容錯回復至來源 Azure 區域。 此時，當流量管理員偵測到**主要**端點恢復良好的狀況時，將會自動在其 DNS 回應中使用**主要**端點。

## <a name="protecting-multi-region-enterprise-applications"></a>保護多區域企業應用程式

跨國企業常會針對不同區域的需求修改其應用程式，藉以提升客戶體驗。 當地語系化和延遲降低可能會導致不同區域的應用程式基礎結構出現歧異。 企業也受限於特定區域的區域資料法規，而選擇將其一部分的應用程式基礎結構侷限在區域界限內。  

我們來看看以下範例：**公司 D** 將其應用程式端點進行分割，分別為德國和其餘國家提供服務。 **公司 D** 使用 Azure 流量管理員的[地理](../traffic-manager/traffic-manager-configure-geographic-routing-method.md)路由方法進行此設定。 任何來自德國的流量都會導向至**端點 1**，而來自德國以外的任何流量則導向至**端點 2**。

此設定的問題在於，如果**端點 1** 因故停止運作，流量將無法重新導向至**端點 2**。 無論端點的狀況是否良好，來自德國的流量都會繼續導向至**端點 1**，而使德國的使用者無法存取**公司 D** 的應用程式。 同樣地，如果**端點 2** 離線，流量也無法重新導向至**端點 1**。

![之前的多區域應用程式](./media/concepts-traffic-manager-with-site-recovery/geographic-application-before.png)

為了避免發生此問題並確保應用程式的復原能力，**公司 D** 使用了[巢狀流量管理員設定檔](../traffic-manager/traffic-manager-nested-profiles.md)與 Azure Site Recovery。 在巢狀設定檔設定中，流量不會導向至個別的端點，而是導向其他流量管理員設定檔。 以下是此設定的運作方式：
- **公司 D** 不使用具有個別端點的地理路由，而是使用流量管理員設定檔的地理路由。
- 每個子流量管理員設定檔都會使用具有主要和復原端點的**優先順序**路由，因而將**優先順序**路由內嵌在**地理**路由內。
- 為了啟用應用程式復原功能，每個分配的工作負載在災害事件發生時都會使用 Azure Site Recovery 容錯移轉至既定的復原區域。
- 當父流量管理員收到 DNS 查詢時，該查詢會導向至相關的子流量管理員，以透過可用的端點回應查詢。

![之後的多區域應用程式](./media/concepts-traffic-manager-with-site-recovery/geographic-application-after.png)

例如，如果德國中部的端點失效，應用程式將可快速復原至德國東北部。 新端點會以最短的使用者停機時間，處理來自德國的流量。 同樣地，「西歐」端點的中斷也可藉由將應用程式工作負載復原至「北歐」來處理，只要將負責處理 DNS 的 Azure 流量管理員重新導向至可用的端點即可。

前述設定可視需要擴充而納入數量不限的區域和端點組合。 流量管理員最多允許 10 個層級的巢狀設定檔，但不容許巢狀組態內出現迴圈。

## <a name="recovery-time-objective-rto-considerations"></a>復原時間目標 (RTO) 考量

在大部分的組織中，DNS 記錄的新增或修改都由個別的團隊或組織外部人員來處理。 這使得變更 DNS 記錄成為非常具挑戰性的工作。 由其他團隊或管理 DNS 基礎結構的組織更新 DNS 記錄所需的時間，會隨組織而不同，且可能會影響應用程式的 RTO。

藉由使用流量管理員，您可以優先執行 DNS 更新所需的工作。 在進行實際的容錯移轉時，無須執行任何手動或指令碼動作。 這種方法有助於快速切換 (並因此降低 RTO)，同時避免在災害事件發生時產生耗時的 DNS 變更錯誤。 透過流量管理員，即使原本必須個別管理的容錯回復步驟都可自動執行。

透過基本或快速間隔的健康情況檢查設定正確的[探查間隔](../traffic-manager/traffic-manager-monitoring.md)，可大幅降低容錯移轉期間的 RTO，並縮短使用者的停機時間。

此外，您還可以最佳化流量管理員設定檔的 DNS 存留時間 (TTL) 值。 TTL 是用戶端會快取 DNS 項目的值。 就記錄而言，在 TTL 期間內並不需要查詢 DNS 兩次。 每個 DNS 記錄都有其相關聯的 TTL。 降低此值可對流量管理員進行更多次 DNS 查詢，但也可能因更快速地探索中斷而降低 RTO。

如果用戶端與授權 DNS 伺服器之間的 DNS 解析程式數目增加，用戶端所經歷的 TTL 也不會增加。 DNS 解析程式會「倒數」TTL，並僅傳遞 TTL 值來反映快取記錄後所經過的時間。 這可確保無論鏈結中的 DNS 解析程式數目為何，用戶端在 TTL 之後都會更新 DNS 記錄。

## <a name="next-steps"></a>後續步驟
- 深入了解流量管理員的[路由方法](../traffic-manager/traffic-manager-routing-methods.md)。
- 深入了解[巢狀流量管理員設定檔](../traffic-manager/traffic-manager-nested-profiles.md)。
- 深入了解[端點監視](../traffic-manager/traffic-manager-monitoring.md)。
- 深入了解自動執行應用程式容錯移轉的[復原計劃](site-recovery-create-recovery-plans.md)。
