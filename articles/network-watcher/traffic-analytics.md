---
title: Azure 流量分析 | Microsoft Docs
description: 了解如何透過流量分析來分析 Azure 網路安全性群組流量記錄。
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/15/2018
ms.author: yagup;jdial
ms.openlocfilehash: ad26772650cf052926a2534d343f64765f47b78f
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "36333389"
---
# <a name="traffic-analytics"></a>流量分析

流量分析是一個雲端式解決方案，可顯示雲端網路中的使用者和應用程式活動。 流量分析會分析網路監看員網路安全性群組 (NSG) 流量記錄，讓您深入了解 Azure 雲端中的流量。 透過流量分析，您可以：

- 視覺化您的 Azure 訂用帳戶之間的網路活動，並識別作用點。
- 透過開啟的連接埠、嘗試存取網際網路的應用程式，和連線至 Rogue 網路的虛擬機器 (VM) 等資訊，識別網路的安全性威脅並保護您的網路。
- 了解各個 Azure 區域間和網際網路的流量模式，使網路部署達到最理想的效能和容量。
- 找出導致網路連線失敗的網路設定錯誤。

## <a name="why-traffic-analytics"></a>為何要使用流量分析？

要保有穩固的安全性、合規性和效能，監視、管理及了解您本身的網路，是很重要的。 了解您本身的環境，是保護和最佳化環境的第一要務。 您通常必須了解網路目前的狀態、哪些人正在連線、他們從哪裡連接、對網際網路開啟了哪些連接埠、預期的網路行為、異常的網路行為，以及流量突然上升等。

雲端網路與內部部署企業網路不同，您在雲端網路中會使用 Netflow 或支援通訊協定功能的對等路由器和交換器，來收集進入或離開網路介面的 IP 網路流量。 藉由分析流量資料，您將可建立網路流量和規模的分析。

Azure 虛擬網路具有 NSG 流量記錄，可為您了解透過與個別網路介面、VM 或子網路相關聯的網路安全性群組傳輸的輸入和輸出 IP 流量。 藉由分析原始 NSG 流量記錄，並插入安全性、拓撲和地理位置的智慧功能，流量分析可讓您深入了解環境中的流量。 流量分析提供如下的資訊：最常通訊的主機、最常通訊的應用程式通訊協定、最常交談的主機配對、允許/封鎖的流量、輸入/輸出流量、開啟的網際網路連接埠、最常用的封鎖規則、每個 Azure 資料中心的流量分布、虛擬網路、子網路或 Rogue 網路。

## <a name="key-components"></a>重要元件

- **網路安全性群組 (NSG)** 包含安全性規則清單，可允許或拒絕連線至 Azure 虛擬網路之資源的網路流量。 NSG 可以與子網路、個別 VM (傳統) 或已連結至 VM (Resource Manager) 的個別網路介面 (NIC) 建立關聯。 如需詳細資訊，請參閱[網路安全性群組概觀](../virtual-network/security-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。
- **網路安全性群組 (NSG) 流量記錄**：可讓您檢視透過網路安全性群組傳輸之輸入和輸出 IP 流量的相關資訊。 NSG 流量記錄是以 json 格式撰寫，會顯示每一規則的輸出和輸入流量、套用流量的 NIC、關於流量的 5 個 Tuple 資訊 (來源/目的地 IP 位址、來源/目的地連接埠、通訊協定)，以及流量是被允許或拒絕。 如須 NSG 流量記錄的詳細資訊，請參閱 [NSG 流量記錄](network-watcher-nsg-flow-logging-overview.md)。
- **Log Analytics**：一項 Azure 服務，會收集監視資料並將其儲存在中央存放庫中。 此資料可能包含事件、效能資料，或透過 Azure API 提供的自訂資料。 所收集的資料即可用於警示、分析和匯出。 網路效能監視器和流量分析等監視應用程式皆以 Log Analytics 作為建置基礎。 如需詳細資訊，請參閱 [Log Analytics](../log-analytics/log-analytics-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。
- **Log Analytics 工作區**：一個 Log Analytics 執行個體，會儲存 Azure 帳戶的相關資料。 如需 Log Analytics 工作區的詳細資訊，請參閱[建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。
- **網路監看員**：一項區域性服務，可讓您監視與診斷 Azure 中的網路案例層級情況。 您可以透過網路監看員將 NSG 流量記錄開啟和關閉。 如需詳細資訊，請參閱[網路監看員](network-watcher-monitoring-overview.md)。

## <a name="how-traffic-analytics-works"></a>流量分析的運作方式

流量分析會檢查原始 NSG 流量記錄，並藉由彙總相同來源 IP 位址、目的地 IP 位址、目的地連接埠和通訊協定間的共同流量來擷取縮減記錄。 例如，假設主機 1 (IP 位址：10.10.10.10) 在 1 小時的期間內使用連接埠 (例如 80) 和通訊協定 (例如 HTTP) 與主機 2 (IP 位址：10.10.20.10) 通訊了 100 次。 縮減記錄會以一個項目來說明主機 1 和主機 2 在 1 小時的期間內使用連接埠 *80* 和通訊協定 *HTTP* 通訊了 100 次，而不會有 100 個項目。 縮減記錄會在增列地理位置、安全性與拓撲資訊後，儲存在 Log Analytics 工作區中。 下圖顯示資料流程：

![NSG 流量記錄處理的資料流程](./media/traffic-analytics/data-flow-for-nsg-flow-log-processing.png)

## <a name="supported-regions"></a>支援區域

您可以在下列任何一個區域中使用 NSG 的流量分析：美國中西部、美國東部、美國東部 2、美國中北部、美國中南部、美國中部、美國西部、美國西部 2、西歐、北歐、英國西部、英國南部、澳大利亞東部、澳大利亞東南部和東南亞。 記錄分析工作區必須存在於美國中西部、美國東部、西歐、英國南部、澳大利亞東南部或東南亞地區。

## <a name="prerequisites"></a>先決條件

### <a name="user-access-requirements"></a>使用者存取需求

您的帳戶必須是下列 Azure [內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)的成員之一：

|部署模型   | 角色                   |
|---------          |---------               |
|Resource Manager   | 擁有者                  |
|                   | 參與者            |
|                   | 讀取者                 |
|                   | 網路參與者    |
|傳統            | 帳戶管理員  |
|                   | 服務管理員  |
|                   | 共同管理員       |

如果您的帳戶未指派給其中一個內建角色，則必須指派給一個[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)，且該角色已獲得下列訂用帳戶層級的動作：

- "Microsoft.Network/applicationGateways/read"
- "Microsoft.Network/connections/read"
- "Microsoft.Network/loadBalancers/read"
- "Microsoft.Network/localNetworkGateways/read"
- "Microsoft.Network/networkInterfaces/read"
- "Microsoft.Network/networkSecurityGroups/read"
- "Microsoft.Network/publicIPAddresses/read"
- "Microsoft.Network/routeTables/read"
- "Microsoft.Network/virtualNetworkGateways/read"
- "Microsoft.Network/virtualNetworks/read"

如需了解如何檢查使用者存取權限，請參閱[流量分析常見問題集](traffic-analytics-faq.md)。

### <a name="enable-network-watcher"></a>啟用網路監看員

若要分析流量，您必須具有現有的網路監看員，或在每個有 NSG 需要分析流量的區域中[啟用網路監看員](network-watcher-create.md)。 您可以為任何[支援區域](#supported-regions)中裝載的 NSG 啟用流量分析。

### <a name="re-register-the-network-resource-provider"></a>重新註冊網路資源提供者

您必須先重新註冊網路資源提供者，才能使用流量分析。 請按一下以下程式碼方塊中的**試試看**，以開啟 Azure Cloud Shell。 Cloud Shell 會自動將您登入 Azure 訂用帳戶。 Cloud Shell 開啟後，請輸入下列命令以重新註冊網路資源提供者：

```azurepowershell-interactive
Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.Network"
```

### <a name="select-a-network-security-group"></a>選取網路安全性群組

啟用 NSG 流量記錄之前，您必須具有要記錄流量的網路安全性群組。 如果您沒有網路安全性群組，請參閱[建立網路安全性群組](../virtual-network/manage-network-security-group.md#create-a-network-security-group)，以建立此群組。

在 Azure 入口網站左半部，依序選取 [監視器]、[網路監看員] 和 [NSG 流量記錄]。 選取要啟用 NSG 流量記錄的網路安全性群組，如下圖所示：

![選取需要啟用 NSG 流量記錄的 NSG](./media/traffic-analytics/selection-of-nsgs-that-require-enablement-of-nsg-flow-logging.png)

如果您嘗試為[支援區域](#supported-regions)以外的任何區域中裝載的 NSG 啟用流量分析，將會出現「找不到」錯誤。

## <a name="enable-flow-log-settings"></a>啟用流量記錄設定

在啟用流量記錄設定之前，您必須先完成下列工作：

註冊 Azure Insights 提供者 (如果您的訂用帳戶尚未註冊)：

```azurepowershell-interactive
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Insights
```

如果您還沒有 Azure 儲存體帳戶可儲存 NSG 流量記錄，您必須建立儲存體帳戶。 您可以使用下列命令建立儲存體帳戶。 執行命令之前，請將 `<replace-with-your-unique-storage-account-name>` 取代為在所有 Azure 位置間具有唯一性、長度介於 3-24 個字元，且僅使用數字和小寫字母的名稱。 如有必要，您也可以變更資源群組名稱。

```azurepowershell-interactive
New-AzureRmStorageAccount `
  -Location westcentralus `
  -Name <replace-with-your-unique-storage-account-name> `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

選取下列選項，如下圖所示：

1. 針對 [狀態] 選取 [開啟]。
2. 選取用來儲存流量記錄的現有儲存體帳戶。 如果您想要永久儲存資料，請將值設為 *0*。 您的儲存體帳戶會產生 Azure 儲存體費用。
3. 將 [保留] 設為您要儲存資料的天數。
4. 針對 [流量分析狀態]，選取 [開啟]。
5. 選取現有的 Log Analytics (OMS) 工作區，或選取 [建立新的工作區] 以建立新工作區。 流量分析會使用 Log Analytics 工作區來儲存已彙總並編製索引的資料，而這些資料後續將用來產生分析。 如果您選取現有的工作區時，該工作區必須存在於其中一個[支援區域](#traffic-analytics-supported-regions)中，並且已升級至新的查詢語言。 如果您不想要升級現有的工作區，或是沒有支援區域中的工作區，請建立新的工作區。 如需關於查詢語言的詳細資訊，請參閱 [Azure Log Analytics 升級為新的記錄搜尋](../log-analytics/log-analytics-log-search-upgrade.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)。

    裝載流量分析解決方案和 NSG 的 Log Analytics 工作區不需要位於相同區域中。 例如，您可以用西歐區域的工作區來處理流量分析，而 NSG 則位於美國東部和美國西部。 在相同的工作區中可設定多個 NSG。
6. 選取 [ **儲存**]。

    ![選取儲存體帳戶、Log Analytics 工作區和啟用流量分析](./media/traffic-analytics/selection-of-storage-account-log-analytics-workspace-and-traffic-analytics-enablement.png)

針對要啟用流量分析的任何其他 NSG 重複前述步驟。 流量記錄中的資料會傳送至工作區，因此請確定您所在國家/地區的當地法律和規範允許在工作區所在的區域中進行資料儲存。

您也可以使用 AzureRm PowerShell 模組版本 6.2.1 或更新版本中的 [Set-AzureRmNetworkWatcherConfigFlowLog](/powershell/module/azurerm.network/set-azurermnetworkwatcherconfigflowlog) PowerShell Cmdlet 來設定流量分析。 執行 `Get-Module -ListAvailable AzureRM` 來了解您安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

## <a name="view-traffic-analytics"></a>檢視流量分析

在入口網站左側選取 [所有服務]，然後在 [篩選] 方塊中輸入 [監視器]。 當 [監視器] 出現在搜尋結果中時，請加以選取。 若要開始瀏覽流量分析及其功能，請選取 [網路監看員]，然後選取 [流量分析]。

![存取流量分析儀表板](./media/traffic-analytics/accessing-the-traffic-analytics-dashboard.png)

此儀表板第一次可能需要 30 分鐘才會出現，因為流量分析必須先彙總足夠的資料以衍生有意義的深入資訊，才能夠產生任何報告。

## <a name="usage-scenarios"></a>使用案例

您在流量分析完整設定後可能想要取得的深入資訊如下：

### <a name="find-traffic-hotspots"></a>尋找流量作用點

**尋找**

- 哪些主機、子網路和虛擬網路正在傳送或接收大部分的流量、周遊最大惡意流量以及封鎖重要流程？
    - 參閱主機、子網路和虛擬網路的比較圖表。 了解哪些主機、子網路和虛擬網路正在傳送或接收大部分的流量，有助於您識別哪些主機正在處理最多的流量，以及流量是否分攤合理。
    - 您可以評估該流量對主機而言是否適當。 該流量是否屬於正常行為，或是值得進一步調查？
- 有多少輸入/輸出流量？
    -   是否預期主機接收的輸入流量應高於輸出流量，或者相反？
- 封鎖的流量統計資料。
    - 主機封鎖大量良性流量的原因為何？ 這種行為需要進一步調查與可能的組態最佳化
- 允許/封鎖的惡意流量統計資料
    - 為什麼主機收到惡意流量，且為什麼允許惡意來源？ 這種行為需要進一步調查與可能的組態最佳化。

    選取 [主機] 底下的 [檢視全部]，如下圖所示：

    ![詳盡顯示流量最大之主機的儀表板](media/traffic-analytics/dashboard-showcasing-host-with-most-traffic-details.png)

- 下圖顯示前五大通訊主機的時間趨勢，以及主機的流量相關詳細資料 (允許 – 輸入/輸出、拒絕 - 輸入/輸出流量)：

    ![前五大通訊主機的趨勢](media/traffic-analytics/top-five-most-talking-host-trend.png)

**尋找**

- 最常交談的主機配對為何？
    - 預期行為 (如前端或後端通訊) 或異常行為 (如後端-網際網路流量)。
- 允許/封鎖流量的統計資料
    - 主機允許或封鎖大量流量的原因為何？
- 最常交談的主機配對之間最常用的應用程式通訊協定：
    - 此網路上是否允許這些應用程式？
    - 應用程式已正確設定？ 它們是否使用適當的通訊協定進行通訊？ 選取 [經常交談] 底下的 [檢視全部]，如下圖所示：

        ![顯示最常交談的儀表板](./media/traffic-analytics/dashboard-showcasing-most-frequent-conversation.png)

- 下圖顯示前五大交談的時間趨勢，以及交談配對的流量相關詳細資料，例如允許和拒絕的輸入與輸出流量：

    ![前五大交談的詳細資料與趨勢](./media/traffic-analytics/top-five-chatty-conversation-details-and-trend.png)

**尋找**

- 您的環境中最常使用哪個應用程式通訊協定？哪些交談主機配對最常使用該應用程式通訊協定？
    - 此網路上是否允許這些應用程式？
    - 應用程式已正確設定？ 它們是否使用適當的通訊協定進行通訊？ 預期的行為是通用連接埠，例如 80 和 443。 就標準通訊而言，如果顯示了任何不尋常的連接埠，則可能需要為其變更設定。 選取 [應用程式連接埠] 底下的 [檢視全部]，如下圖所示：

        ![顯示最高排名應用程式通訊協定的儀表板](./media/traffic-analytics/dashboard-showcasing-top-application-protocols.png)

- 下圖顯示前五大 L7 通訊協定的時間趨勢，以及 L7 通訊協定的流量相關詳細資料 (例如，允許和拒絕的流量)：

    ![前五大第 7 層通訊協定的詳細資料和趨勢](./media/traffic-analytics/top-five-layer-seven-protocols-details-and-trend.png)

    ![記錄搜尋中的應用程式通訊協定的流量詳細資料](./media/traffic-analytics/flow-details-for-application-protocol-in-log-search.png)

**尋找**

- 您的環境中 VPN 閘道的容量使用率趨勢。
    - 每個 VPN SKU 都允許特定數量的頻寬。 VPN 閘道是否使用率過低？
    - 您的閘道是否達到容量？ 您應升級至下一個較高的 SKU 嗎？
- 哪些是最常交談的主機？經由哪個 VPN 閘道？透過哪個連接埠？
    - 此模式是否正常？ 選取 [VPN 閘道] 底下的 [檢視全部]，如下圖所示：

        ![顯示最高排名使用中 VPN 連線的儀表板](./media/traffic-analytics/dashboard-showcasing-top-active-vpn-connections.png)

- 下圖顯示 Azure VPN 閘道在容量使用率方面的時間趨勢，以及流量相關詳細資料 (例如允許的流量和連接埠)：

    ![VPN 閘道使用率趨勢和流量詳細資料](./media/traffic-analytics/vpn-gateway-utilization-trend-and-flow-details.png)

### <a name="visualize-traffic-distribution-by-geography"></a>依地理位置呈現流量分布

**尋找**

- 每個資料中心的流量分布，例如，資料中心流量最常見的來源、最常與資料中心交談的 Rogue 網路，以及最常用的交談應用程式通訊協定。
    - 如果您發現資料中心的負載較高，您可以規劃有效率的流量分配。
    - 如果有 Rogue 網路在資料中心內交談，請更正 NSG 規則以封鎖這些網路。

    選取 [您的環境] 底下的 [檢視對應]，如下圖所示：

    ![顯示流量分布的儀表板](./media/traffic-analytics/dashboard-showcasing-traffic-distribution.png)

- 地理地圖會顯示頂端功能區以供您選取參數，例如資料中心 (已部署/無部署/使用中/非使用中/已啟用流量分析/未啟用流量分析) 以及對使用中部署貢獻良性/惡意流量的國家/地區：

    ![顯示使用中部署的地理地圖檢視](./media/traffic-analytics/geo-map-view-showcasing-active-deployment.png)

- 地理地圖會顯示與資料中心通訊的國家/地區及大陸對該資料中心的流量分布，良性流量以藍色線段表示，惡意流量以紅色線段表示：

    ![顯示國家/地區及大陸之流量分布的地理地圖檢視](./media/traffic-analytics/geo-map-view-showcasing-traffic-distribution-to-countries-and-continents.png)

    ![記錄搜尋中的流量分布的流量詳細資料](./media/traffic-analytics/flow-details-for-traffic-distribution-in-log-search.png)

### <a name="visualize-traffic-distribution-by-virtual-networks"></a>依虛擬網路呈現流量分布

**尋找**

- 每個虛擬網路的流量分布、拓撲、虛擬網路流量最常見的來源、最常與虛擬網路交談的 Rogue 網路，以及最常用的交談應用程式通訊協定。
    - 了解哪些虛擬網路正在互相交談。 如果交談不符預期，可加以更正。
    - 如果有 Rogue 網路與虛擬網路交談，您可以更正 NSG 規則以封鎖 Rogue 網路。
 
    選取 [您的環境] 底下的 [檢視 Vnet]，如下圖所示：

    ![顯示虛擬網路分布的儀表板](./media/traffic-analytics/dashboard-showcasing-virtual-network-distribution.png)

- 「虛擬網路拓撲」會顯示頂端功能區以供您選取參數，例如虛擬網路 (內部虛擬網路連線/使用中/非使用中)、外部連線、使用中流量，以及虛擬網路的惡意流量。
- 「虛擬網路拓撲」會以若干層面來顯示虛擬網路的流量分布，例如流量 (允許/封鎖/輸入/輸出/良性/惡意)、應用程式通訊協定，以及網路安全性群組：

    ![顯示流量分布和流量詳細資料的虛擬網路拓撲](./media/traffic-analytics/virtual-network-topology-showcasing-traffic-distribution-and-flow-details.png)

    ![記錄搜尋中的虛擬網路流量分布的流量詳細資料](./media/traffic-analytics/flow-details-for-virtual-network-traffic-distribution-in-log-search.png)

**尋找**

- 每個子網路的流量分布、拓撲、子網路流量最常見的來源、最常與子網路交談的 Rogue 網路，以及最常用的交談應用程式通訊協定。
    - 了解哪些子網路正在互相交談。 如果您發現非預期的交談，您可以更正設定。
    - 如果有 Rouge 網路與子網路交談，您可以設定 NSG 規則來封鎖 Rogue 網路，將其予以更正。
- 「子網路拓撲」會顯示頂端功能區以供您選取參數，例如使用中/非使用中子網路、外部連線、使用中流量，以及子網路的惡意流量。
- 「子網路拓撲」會以若干層面來顯示虛擬網路的流量分布，例如流量 (允許/封鎖/輸入/輸出/良性/惡意)、應用程式通訊協定以及 NSG：

    ![顯示虛擬網路子網路在流量方面之流量分布的子網路拓撲](./media/traffic-analytics/subnet-topology-showcasing-traffic-distribution-to-a-virtual-subnet-with-regards-to-flows.png)

**尋找**

每個應用程式閘道和負載平衡器的網路流量分佈、拓撲、最常見的流量來源、最常與應用程式閘道和負載平衡器交談的 Rogue 網路，以及最常用的交談應用程式通訊協定。 
    
 - 了解哪一個子網路與哪一個應用程式閘道或負載平衡器正在交談中。 如果您注意到非預期的交談，便可以更正設定。
 - 如果有 Rogue 網路與子網路交談，您可以設定 NSG 規則來封鎖 Rogue 網路，將其予以更正。 

    ![subnet-topology-showcasing-traffic-distribution-to-a-application-gateway-subnet-with-regards-to-flows](./media/traffic-analytics/subnet-topology-showcasing-traffic-distribution-to-a-application-gateway-subnet-with-regards-to-flows.png)

### <a name="view-ports-and-virtual-machines-receiving-traffic-from-the-internet"></a>檢視接收來自網際網路之流量的連接埠和虛擬機器

**尋找**

- 哪些開啟的連接埠透過網際網路進行交談？
    - 如果您發現有非預期的連接埠已開啟，您可以更正設定：

    ![顯示對網際網路接收和傳送流量之連接埠的儀表板](./media/traffic-analytics/dashboard-showcasing-ports-receiving-and-sending-traffic-to-the-internet.png)

    ![Azure 目的地連接埠和主機的詳細資料](./media/traffic-analytics/details-of-azure-destination-ports-and-hosts.png)

**尋找**

您的環境中是否有惡意流量？ 它來自於何處？ 它將傳送到何處？

![記錄搜尋中的惡意流量詳細資料](./media/traffic-analytics/malicious-traffic-flows-detail-in-log-search.png)


### <a name="visualize-the-trends-in-nsgnsg-rules-hits"></a>視覺化 NSG/NSG 規則命中的趨勢

**尋找**

- 在含流量分佈的比較圖表中，哪些是最常被叫用的 NSG/NSG 規則？
- 每個 NSG/NSG 規則有哪些最常用的來源與目的地交談對組？

    ![顯示 NSG 命中統計資料的儀表板](./media/traffic-analytics/dashboard-showcasing-nsg-hits-statistics.png)

- 下圖顯示 NSG 規則命中的時間趨勢，以及網路安全性群組的來源-目的地流量詳細資料：

    - 快速偵測哪些 NSG 與 NSG 規則正在周遊惡意流量，以及哪些惡意 IP 位址最常存取您的雲端環境
    - 識別哪些 NSG/NSG 規則允許/封鎖重要的網路流量
    - 選取最常用於 NSG 或 NSG 規則細微檢查的篩選

    ![顯示 NSG 規則命中的時間趨勢以及最高排名 NSG 規則](./media/traffic-analytics/showcasing-time-trending-for-nsg-rule-hits-and-top-nsg-rules.png)

    ![記錄搜尋中的 NSG 規則詳細統計資料](./media/traffic-analytics/top-nsg-rules-statistics-details-in-log-search.png)

## <a name="frequently-asked-questions"></a>常見問題集

若要取得常見問題的解答，請參閱[流量分析常見問題集](traffic-analytics-faq.md)。
