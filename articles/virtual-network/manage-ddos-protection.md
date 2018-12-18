---
title: 使用 Azure 入口網站管理 Azure DDoS Protection Standard | Microsoft Docs
description: 了解如何使用 Azure 監視器中的 Azure DDoS Protection Standard 遙測來降低攻擊風險。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/06/2018
ms.author: jdial
ms.openlocfilehash: 59cfcc72abee100b95cf17033083827fbb30f9f5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986688"
---
# <a name="manage-azure-ddos-protection-standard-using-the-azure-portal"></a>使用 Azure 入口網站管理 Azure DDoS Protection Standard

了解如何啟用和停用分散式阻斷服務 (DDoS) 保護，並使用遙測技術透過 Azure DDoS Protection Standard 來降低 DDoS 攻擊。 DDoS Protection Standard 會保護任何與 Azure [公用 IP 位址](virtual-network-public-ip-address.md)相關聯的 Azure 資源，例如虛擬機器、負載平衡器和應用程式閘道。 若要了解有關 DDoS Protection Standard 和其功能的詳細資訊，請參閱 [DDoS Protection Standard 概觀](ddos-protection-overview.md)。

在完成本教學課程中的任何步驟之前，請先使用指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色或為已指派[權限](#permissions)中所列適當動作之[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的帳戶，登入 Azure 入口網站 (網址為 https://portal.azure.com)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-a-ddos-protection-plan"></a>建立 DDoS 保護計劃

DDoS 保護計劃會定義一組跨訂用帳戶且已啟用標準 DDoS 保護的虛擬網路。 您可以為組織設定一個 DDoS 保護計劃，然後將來自多個訂用帳戶的虛擬網路連結至該相同計劃。 「DDoS 保護計劃」本身也會與您在建立計劃時所選取的訂用帳戶產生關聯。 計劃所關聯的訂用帳戶會為該計劃帶來每月的週期性帳單，也會在受保護的公用 IP 位址數目超過 100 時衍生超額費用。 如需有關 DDoS 定價的詳細資訊，請參閱[價格詳細資料](https://azure.microsoft.com/pricing/details/ddos-protection/)。

對大多數組織來說，並沒有必要建立多個計劃。 計劃無法在訂用帳戶之間移動。 如果您想要變更計劃所在的訂用帳戶，就必須[刪除現有的計劃](#work-with-ddos-protection-plans)，再建立一個新計劃。

1. 選取 Azure 入口網站中左上角的 [建立資源]。
2. 搜尋 *DDoS*。 當 **DDoS 保護計劃**出現在搜尋結果中時，請選取它。
3. 選取 [建立] 。
4. 輸入或選取您自己的值，或者輸入或選取下列範例值，然後選取 [建立]：

    |設定        |值                                              |
    |---------      |---------                                          |
    |名稱           | myDdosProtectionPlan                              |
    |訂用帳戶   | 選取您的訂用帳戶。                         |
    |資源群組 | 選取 [新建]，然後輸入 *myResourceGroup* |
    |位置       | 美國東部                                           |

## <a name="enable-ddos-for-a-new-virtual-network"></a>為新虛擬網路啟用 DDoS

1. 選取 Azure 入口網站中左上角的 [建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 輸入或選取您自己的值，或是輸入或選取下列範例值，接受其餘預設值，然後選取 [建立]：

    | 設定         | 值                                                        |
    | ---------       | ---------                                                    |
    | 名稱            | myVirtualNetwork                                             |
    | 訂用帳戶    | 選取您的訂用帳戶。                                    |
    | 資源群組  | 選取 [使用現有的]，然後選取 [myResourceGroup] |
    | 位置        | 美國東部                                                      |
    | DDoS 保護 | 選取 [標準]，然後在 [DDoS 保護] 底下，選取 [myDdosProtectionPlan]。 您所選取的計劃可以與虛擬網路位於相同或不同的訂用帳戶中，但兩個訂用帳戶必須都與同一個 Azure Active Directory 租用戶關聯。|

已為虛擬網路啟用「標準 DDoS」時，您無法將虛擬網路移到另一個資源群組或訂用帳戶。 如果您需要移動已啟用「標準 DDoS」的虛擬網路，請先將「標準 DDoS」停用，移動虛擬網路，然後再啟用「標準 DDoS」。 移動之後，就會重設虛擬網路中所有受保護公用 IP 位址的自動調整原則閾值。

## <a name="enable-ddos-for-an-existing-virtual-network"></a>為現有的虛擬網路啟用 DDoS 保護

1. 如果您沒有現有的 DDoS 保護計劃，請完成[建立 DDoS 保護計劃](#create-a-ddos-protection-plan)中的步驟來建立 DDoS 保護計劃。
2. 選取 Azure 入口網站中左上角的 [建立資源]。
3. 在入口網站頂端的 [搜尋資源、服務及文件] 方塊中，輸入您要為其啟用「標準 DDoS 保護」的虛擬網路名稱。 當虛擬網路的名稱出現在搜尋結果中時，請選取它。
4. 選取 [設定] 底下的 [DDoS 保護]。
5. 選取 [標準]。 在 [DDoS 保護計劃] 底下，選取現有的 DDoS 保護計劃或是您在步驟 1 中建立的計劃，然後選取 [儲存]。 您所選取的計劃可以與虛擬網路位於相同或不同的訂用帳戶中，但兩個訂用帳戶必須都與同一個 Azure Active Directory 租用戶關聯。

## <a name="disable-ddos-for-a-virtual-network"></a>停用虛擬網路的 DDoS

1. 在入口網站頂端的 [搜尋資源、服務及文件] 方塊中，輸入您要停用其「標準 DDoS 保護」的虛擬網路名稱。 當虛擬網路的名稱出現在搜尋結果中時，請選取它。
2. 選取 [設定] 底下的 [DDoS 保護]。
3. 選取 [DDoS 保護計劃] 底下的 [基本]，然後選取 [儲存]。

## <a name="work-with-ddos-protection-plans"></a>使用 DDoS 保護計劃

1. 選取入口網站左上角的 [所有服務]。
2. 在 [篩選] 方塊中，輸入 *DDoS*。 當 **DDoS 保護計劃**出現在結果中時，請選取它。
3. 從清單中選取您想要檢視的保護計劃。
4. 與該計劃關聯的所有虛擬網路都會列出。
5. 如果您想要刪除某個計劃，必須先將與其關聯的所有虛擬網路都取消關聯。 若要將計劃與虛擬網路取消關聯，請參閱[停用虛擬網路的 DDoS](#disable-ddos-for-a-virtual-network)。

## <a name="configure-alerts-for-ddos-protection-metrics"></a>設定 DDoS 保護計量的警示

透過使用「Azure 監視器」警示設定，您可以選取任何可用的 DDoS 保護計量，以在攻擊期間有作用中的風險降低措施時向您發出警示。 當條件符合時，指定的地址會收到警示電子郵件：

1. 選取入口網站左上角的 [所有服務]。
2. 在 [篩選] 方塊中，輸入「監視器」。 當**監視器**出現在搜尋結果中時，請選取它。
3. 選取 [共用服務] 底下的 [計量]。
4. 輸入或選取您自己的值，或是輸入下列範例值，接受其餘預設值，然後選取 [確定]：

    |設定                  |值                                                                                               |
    |---------                |---------                                                                                           |
    |名稱                     | myDdosAlert                                                                                        |
    |訂用帳戶             | 選取包含您想要接收警示之公用 IP 位址的訂用帳戶。        |
    |資源群組           | 選取包含您想要接收警示之公用 IP 位址的資源群組。      |
    |資源                 | 選取包含您想要接收警示之公用 IP 位址的公用 IP 位址。 DDoS 會監視指派給虛擬網路內資源的公用 IP 位址。 如果您在虛擬網路中沒有任何具有公用 IP 位址的資源，就必須先建立一個具有公用 IP 位址的資源。 針對 [Azure 服務的虛擬網路](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network)中所列出透過 Resource Manager (非傳統) 部署的所有資源，您可以監視資源的公用 IP 位址，但「Azure App Service 環境」和「Azure VPN 閘道」除外。 若要繼續進行本教學課程，您可以快速建立一個 [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虛擬機器。                   |
    |計量                   | 是否正遭受 DDoS 攻擊                                                                            |
    |閾值                | 1 - **1** 表示您正遭受攻擊。 **0** 表示您並未遭受攻擊。                         |
    |期間                   | 選取您選擇的任何值。                                                                   |
    |透過電子郵件通知         | 勾選此核取方塊                                                                                  |
    |其他系統管理員 | 如果您不是訂用帳戶的電子郵件擁有者、參與者或讀者，請輸入您的電子郵件地址。 |

    在進行攻擊偵測的幾分鐘內，您會從「Azure 監視器」收到看似下圖的電子郵件：

    ![攻擊警示](./media/manage-ddos-protection/ddos-alert.png)


若要模擬 DDoS 攻擊來驗證警示，請參閱[驗證 DDoS 偵測](#validate-ddos-detection)。

您也可以深入了解如何[設定 Webhook](../monitoring-and-diagnostics/insights-webhooks-alerts.md?toc=%2fazure%2fvirtual-network%2ftoc.json)和[邏輯應用程式](../logic-apps/logic-apps-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以便建立警示。

## <a name="use-ddos-protection-telemetry"></a>使用 DDoS 保護遙測

攻擊的遙測可透過 Azure 監視器即時提供。 遙測只適用於公用 IP 位址在安全防護之下的期間。 在攻擊風險降低之前或之後，您都不會看到遙測。

1. 選取入口網站左上角的 [所有服務]。
2. 在 [篩選] 方塊中，輸入「監視器」。 當**監視器**出現在搜尋結果中時，請選取它。
3. 選取 [共用服務] 底下的 [計量]。
4. 選取包含您想要其遙測資料之公用 IP 位址的 [訂用帳戶] 和 [資源群組]。
5. 針對 [資源類型] 選取 [公用 IP 位址]，然後選取您想要其遙測資料的特定公用 IP 位址。
6. 畫面的左側會出現一系列**可用的計量**。 若選取這些計量，這些度量會在概觀畫面上的 **Azure 監視器計量圖**中繪製成圖形。

計量名稱呈現不同的套件類型和位元組及封包，包含每個計量的標籤名稱基本結構，如下所示：

- **捨棄的標籤名稱** (例如**捨棄的輸入封包 DDoS**)：DDoS 保護系統所捨棄/清除的封包數目。
- **轉送的標籤名稱** (例如**轉送的輸入封包 DDoS**)：DDoS 系統轉送到目的地 VIP 的封包數目 – 未篩選的流量。
- **沒有標籤名稱** (例如**輸入封包 DDoS**)：進入清除系統的封包總數 – 代表所捨棄和轉送的封包總和。

若要模擬 DDoS 攻擊來驗證遙測，請參閱[驗證 DDoS 偵測](#validate-ddos-detection)。

## <a name="view-ddos-mitigation-policies"></a>檢視 DDoS 風險降低原則

「標準 DDoS 保護」會在啟用 DDoS 的虛擬網路中，針對受保護資源的每個公用 IP 位址套用三個自動調整的風險降低措施 (TCP SYN、TCP 及 UDP)。 您可以選取 [用以觸發 DDoS 風險降低措施的輸入 TCP 封包] 和 [用以觸發 DDoS 風險降低措施的輸入 UDP 封包] 計量來檢視原則閾值，如下圖所示：

![檢視風險降低原則](./media/manage-ddos-protection/view-mitigation-policies.png)

原則閾值會透過以 Azure Machine Learning 為基礎的網路流量分析進行自動設定。 只有在超出原則閾值時，才會針對遭受攻擊的 IP 位址進行 DDoS 風險降低措施。

## <a name="configure-ddos-attack-analytics"></a>設定 DDoS 攻擊分析
Azure DDoS 保護標準透過 DDoS 攻擊分析，提供詳細的攻擊見解和視覺效果。 保護其虛擬網路避免遭受 DDoS 攻擊的客戶可以詳細了解攻擊流量，並透過攻擊風險降低報告和風險降低流程記錄來緩解攻擊。 

## <a name="configure-ddos-attack-mitigation-reports"></a>設定 DDoS 攻擊風險降低報告
攻擊風險降低報告使用 Netflow 通訊協定資料進行彙總，以提供有關資源攻擊的詳細資訊。 每當公用 IP 資源遭受攻擊時，將在啟動風險降低措施後，立即產生風險降低報告。 每隔 5 分鐘就會產生一份增量報告，以及針對整個風險降低期間，產生一份風險降低後續報告。 這是為了確保在 DDoS 攻擊持續進行較長時間的情況下，您將能夠每 5 分鐘檢視最新的風險降低報告快照集，並在攻擊風險降低結束後，檢視完整摘要。 

1. 選取入口網站左上角的 [所有服務]。
2. 在 [篩選] 方塊中，輸入「監視器」。 當**監視器**出現在搜尋結果中時，請選取它。
3. 在 [設定] 底下，選取 [診斷設定]。
4. 選取包含您所要記錄公用 IP 位址的 [訂用帳戶] 和 [資源群組]。
5. 針對 [資源類型] 選取 [公用 IP 位址]，然後選取您想要為其記錄計量的特定公用 IP 位址。
6. 選取 [開啟診斷以收集 DDoSMitigationReports 記錄]，然後視需要選取下列眾多選項：

    - **封存至儲存體帳戶**：可將資料寫入至「Azure 儲存體」帳戶。 若要深入了解這個選項，請參閱[封存診斷記錄](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
    - **串流至事件中樞**：可讓記錄接收者使用「Azure 事件中樞」來挑選記錄。 事件中樞可允許與 Splunk 或其他 SIEM 系統進行整合。 若要深入了解這個選項，請參閱[將診斷記錄串流至事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
    - **傳送至 Log Analytics**：將記錄寫入至 Azure OMS Log Analytics 服務。 若要深入了解此選項，請參閱[收集記錄以便在 Log Analytics 中使用](../log-analytics/log-analytics-azure-storage.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

增量及攻擊風險降低後續報告包括下列欄位
- 攻擊媒介
- 流量統計資料
- 丟棄封包的原因
- 涉及的通訊協定
- 前 10 個來源國家/地區或區域
- 前 10 個來源的 ASN

## <a name="configure-ddos-attack-mitigation-flow-logs"></a>設定 DDoS 攻擊風險降低流程記錄
攻擊風險降低流程記錄可讓您在近乎即時的作用中 DDoS 攻擊期間，查看丟棄的流量、轉寄的流量及其他相關資料點。 您可以透過事件中樞，將將此資料的常數串流內嵌到 SIEM 系統中，以進行近乎即時的監視，採取可能的措施，並滿足您的防禦作業需求。 

1. 選取入口網站左上角的 [所有服務]。
2. 在 [篩選] 方塊中，輸入「監視器」。 當**監視器**出現在搜尋結果中時，請選取它。
3. 在 [設定] 底下，選取 [診斷設定]。
4. 選取包含您所要記錄公用 IP 位址的 [訂用帳戶] 和 [資源群組]。
5. 針對 [資源類型] 選取 [公用 IP 位址]，然後選取您想要為其記錄計量的特定公用 IP 位址。
6. 選取 [開啟診斷以收集 DDoSMitigationFlowLogs 記錄]，然後視需要選取下列眾多選項：

    - **封存至儲存體帳戶**：可將資料寫入至「Azure 儲存體」帳戶。 若要深入了解這個選項，請參閱[封存診斷記錄](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
    - **串流至事件中樞**：可讓記錄接收者使用「Azure 事件中樞」來挑選記錄。 事件中樞可允許與 Splunk 或其他 SIEM 系統進行整合。 若要深入了解這個選項，請參閱[將診斷記錄串流至事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
    - **傳送至 Log Analytics**：將記錄寫入至 Azure OMS Log Analytics 服務。 若要深入了解此選項，請參閱[收集記錄以便在 Log Analytics 中使用](../log-analytics/log-analytics-azure-storage.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
7. 若要在 Azure 分析儀表板中檢視流程記錄資料，您可以從 https://github.com/Anupamvi/Azure-DDoS-Protection/raw/master/flowlogsbyip.zip 匯入範例儀表板

流程記錄將會有下列欄位： 
- 來源 IP
- 目的地 IP
- 來源連接埠 
- 目的地連接埠 
- 通訊協定類型 
- 在風險降低期間所採取的動作



## <a name="validate-ddos-detection"></a>驗證 DDoS 偵測

Microsoft 已與 [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) 合作建立一個介面，您可以針對啟用 DDoS 保護的公用 IP 位址產生流量，以進行模擬。 BreakPoint Cloud 模擬可讓您：

- 驗證「Microsoft Azure DDoS 保護」如何保護您的 Azure 資源免於受到 DDoS 攻擊
- 將遭受 DDoS 攻擊時的事件回應程序最佳化
- 記載 DDoS 合規性
- 訓練您的網路安全性小組

## <a name="permissions"></a>權限

若要使用 DDoS 保護計劃，您的帳戶必須指派為[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，或為已指派下表中所列適當動作的[自訂](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)角色：

| 動作                                            | 名稱                                     |
| ---------                                         | -------------                            |
| Microsoft.Network/ddosProtectionPlans/read        | 讀取 DDoS 保護計劃              |
| Microsoft.Network/ddosProtectionPlans/write       | 建立或更新 DDoS 保護計劃  |
| Microsoft.Network/ddosProtectionPlans/delete      | 刪除 DDoS 保護計劃            |
| Microsoft.Network/ddosProtectionPlans/join/action | 加入 DDoS 保護計劃              |

若要啟用虛擬網路的 DDoS 保護，您的帳戶也必須指派[虛擬網路的適當動作](manage-virtual-network.md#permissions)。

## <a name="next-steps"></a>後續步驟

- 為虛擬網路建立及套用 [Azure 原則](policy-samples.md)