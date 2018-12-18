---
title: 在 Azure 中使用服務對應解決方案 | Microsoft Docs
description: 服務對應是 Azure 中的一個解決方案，可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文會詳細說明如何在環境中部署服務對應並將它用於各種案例。
services: monitoring
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: 3ceb84cc-32d7-4a7a-a916-8858ef70c0bd
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/22/2018
ms.author: daseidma;bwren
ms.openlocfilehash: 812137a8320634364a7d91fd2e61cd3e9d15fc12
ms.sourcegitcommit: 6eb14a2c7ffb1afa4d502f5162f7283d4aceb9e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36751423"
---
# <a name="using-service-map-solution-in-azure"></a>在 Azure 中使用服務對應解決方案
服務對應可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 您可以藉由服務對應，將伺服器視為提供重要服務的互連系統，藉此來檢視伺服器。 不需要進行任何設定，只要安裝了代理程式，服務對應就會顯示橫跨任何 TCP 連線架構的伺服器、處理序和連接埠之間的連線。

本文說明上線和使用服務對應的詳細資訊。 如需設定服務對應和啟用代理程式的相關資訊，請參閱[在 Azure 中設定服務對應解決方案]( monitoring-service-map-configure.md)。

## <a name="sign-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

## <a name="enable-service-map"></a>啟用服務對應
1. 在 Azure 入口網站中，按一下 [+ 建立資源]。
2. 在搜尋列中輸入**服務對應**，然後按 **Enter**。
3. 在市集搜尋結果頁面的清單中，選取 [服務對應]。<br><br> ![從 Azure Marketplace 搜尋結果中選取服務對應解決方案](./media/monitoring-service-map/marketplace-search-results.png)<br>
4. 在 [服務對應] 概觀窗格上，檢閱解決方案詳細資料，然後按一下 [建立] 開始將程序上線到 Log Analytics 工作區。<br><br> ![將服務對應解決方案上線](./media/monitoring-service-map/service-map-onboard.png)。
5. 在 [設定解決方案] 窗格中，選取現有的或建立新的 Log Analytics 工作區。  如需如何建立新工作區的進一步資訊，請參閱[在 Azure 入口網站中建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。 提供必要資訊後，按一下 [建立] 。  

確認資訊並部署解決方案後，您可以在功能表的 [通知] 底下追蹤其進度。 

您可以在 Azure 入口網站中從 Log Analytics 工作區存取服務對應，然後從左窗格選取 [解決方案] 選項。<br><br> ![在工作區中選取 [解決方案] 選項](./media/monitoring-service-map/select-solution-from-workspace.png)。<br> 從解決方案清單中，選取 [ServiceMap(workspaceName)]，然後在 [服務對應解決方案概觀] 頁面中，按一下 [服務對應摘要] 圖格。<br><br> ![[服務對應摘要] 圖格](./media/monitoring-service-map/service-map-summary-tile.png)。

## <a name="use-cases-make-your-it-processes-dependency-aware"></a>使用案例︰讓 IT 處理序可以感知相依性

### <a name="discovery"></a>探索
服務對應會自動建置跨伺服器、處理程序和協力廠商服務的一般相依性參考對應。 它會探索並對應所有 TCP 相依性，其中會識別非預期的連線、您倚賴的遠端協力廠商系統，以及與傳統網路暗區 (例如 Active Directory) 的相依性。 服務對應可探索到受控系統嘗試進行的失敗網路連線，幫助您識別潛在的伺服器錯誤設定、服務中斷和網路問題。

### <a name="incident-management"></a>事件管理
服務對應可藉由顯示系統的連線方式及其如何互相影響，協助您免去推敲問題所在的工作。 除了識別失敗的連線，服務對應還可協助識別設定錯誤的負載平衡器、非預期或過度負載的重要服務，以及與生產系統通訊的開發人員機器等惡意用戶端。 您也可以使用整合變更追蹤的工作流程，以查看後端機器或服務上的變更事件是否能夠說明事件的根本原因。

### <a name="migration-assurance"></a>移轉保證
您可以使用服務對應，有效地規劃、加速和驗證 Azure 移轉，以確保沒有遺漏任何項目，且不會發生意外的中斷。 您可以探索所有需要一起移轉的交互相依系統、評估系統組態和容量，以及識別執行中的系統是否仍為使用者提供服務或是應該解除委任而非移轉。 移動完成之後，您可以檢查用戶端負載和身分識別，以確認測試系統和客戶之間的連線。 如果子網路規劃和防火牆定義有問題，「服務對應」對應中的失敗連線會向您指出需要連線的系統。

### <a name="business-continuity"></a>業務持續性
如果您使用 Azure Site Recovery，且需要協助以定義應用程式環境的復原順序，服務對應可以自動向您顯示系統彼此間的相依情形，以確保復原計畫可靠。 藉由選擇重要伺服器或群組並檢視其用戶端，可以識別在伺服器還原並可用之後要復原的前端系統。 相反地，藉由查看重要伺服器的後端相依性，可以識別在焦點系統還原之前要復原的系統。

### <a name="patch-management"></a>修補程式管理
服務對應藉由向您指出其他哪些小組和伺服器依賴您的服務，增強了系統更新評量的使用性，因此您可以事先通知他們，然後才關閉系統進行修補。 服務對應更藉由向您指出服務在修補並重新啟動之後是否可用並已正確連線，增強了修補程式管理。

## <a name="mapping-overview"></a>對應概觀
服務對應代理程式會收集其安裝所在伺服器上所有 TCP 連線處理程序的相關資訊，以及有關每個處理程序之輸入和輸出連線的詳細資料。

在左窗格的清單中，可以選取具有服務對應代理程式的機器或群組，將它們在指定時間範圍內的相依性視覺化。 機器相依性對應的焦點會集中在特定機器，並顯示屬於該機器的所有直接 TCP 用戶端或伺服器機器。  機器群組對應會顯示多組伺服器及其相依性。

![服務對應概觀](media/monitoring-service-map/service-map-overview.png)

對應內的機器可以展開，以顯示所選時間範圍內具有作用中網路連線的執行中處理序群組與處理序。 展開具有服務對應代理程式的遠端機器以顯示處理序詳細資料時，只會顯示與焦點機器通訊的處理序。 連線到焦點機器的無代理程式前端機器計數，會顯示在所連線的處理序左側。 如果焦點機器連線至無代理程式的後端機器，該後端伺服器會包含在「伺服器連接埠群組」中，連同其他連線至相同連接埠號碼的連線。

根據預設，「服務對應」對應會顯示過去 30 分鐘的相依性資訊。 使用左上角的時間控制項，可查詢過往時間範圍的對應 (最長可達一小時)，以顯示相依性的過往情形 (例如在事件發生期間或變更發生之前)。 付費工作區的服務對應資料會儲存 30 天，免費工作區的服務對應資料則會儲存 7 天。

## <a name="status-badges-and-border-coloring"></a>狀態徽章和框線色彩
在對應中每一部伺服器的底部，可以是一串狀態徽章，傳達伺服器的相關狀態資訊。 徽章表示在其中一個解決方案整合中，有某些伺服器的相關資訊。 按一下徽章，即會在右窗格中直接顯示狀態的詳細資料。 目前可使用的狀態徽章包括警示、服務台、變更、安全性和更新。

根據狀態徽章的嚴重性，機器節點框線會呈現紅色 (重大)、黃色 (警告) 或藍色 (資訊)。 色彩可代表任何狀態徽章最嚴重的狀態。 灰色框線代表沒有狀態指標的節點。

![狀態徽章](media/monitoring-service-map/status-badges.png)

## <a name="process-groups"></a>處理序群組
處理序群組會將與一般產品或服務相關聯的處理序組成處理序群組。  電腦節點展開時，會一併顯示獨立處理序和處理序群組。  如果對處理序群組內之處理序的輸入與輸出連線發生失敗，便會將整個處理序群組的連線顯示為失敗。

## <a name="machine-groups"></a>機器群組
機器群組可顯示以一組伺服器為中心的對應，而不只是單一伺服器的對應。因此，您可以在單一對應中看到多層式應用程式或伺服器叢集的所有成員。

使用者可選取伺服器所屬的群組，並選擇該群組的名稱。  之後，您可以選擇要檢視該群組及其所有處理序與連線，或是只檢視該群組與群組其他成員直接相關的處理序和連線。

![機器群組](media/monitoring-service-map/machine-group.png)

### <a name="creating-a-machine-group"></a>建立機器群組
若要建立群組，請在 [機器] 清單中選取您想要的一或多部機器，然後按一下 [加入群組]。

![建立群組](media/monitoring-service-map/machine-groups-create.png)

在該處選擇 [新建]，並指定群組名稱。

![為群組命名](media/monitoring-service-map/machine-groups-name.png)

>[!NOTE]
>機器群組目前的限制是 10 部伺服器，但預計短期內會提高此限制。

### <a name="viewing-a-group"></a>檢視群組
建立群組之後，可以選擇 [群組] 索引標籤來檢視所建立的群組。

![[群組] 索引標籤](media/monitoring-service-map/machine-groups-tab.png)

然後，選取該群組名稱以檢視該機器群組的對應。
![機器群組](media/monitoring-service-map/machine-group.png)屬於該群組的機器在對應中會以白色框線框住。

展開該群組，將會列出組成機器群組的機器。

![機器群組的機器](media/monitoring-service-map/machine-groups-machines.png)

### <a name="filter-by-processes"></a>依處理序篩選
對應檢視可切換為顯示群組中的所有處理序和連線，或是僅和機器群組直接相關的處理序和連線。  預設檢視是顯示所有處理序。  您可以按一下對應上方的篩選圖示來變更檢視。

![篩選群組](media/monitoring-service-map/machine-groups-filter.png)

選取 [所有處理序] 時，對應將包含群組中每部機器的所有處理序和連線。

![機器群組的所有處理序](media/monitoring-service-map/machine-groups-all.png)

如果您變更檢視為只顯示 [已與群組連線的處理序]，對應的範圍會縮小至只和群組中其他機器直接相關的處理序和連線，以形成精簡的檢視。

![機器群組的已篩選處理序](media/monitoring-service-map/machine-groups-filtered.png)
 
### <a name="adding-machines-to-a-group"></a>將機器加入群組
若要將機器加入現有的群組，請核取您所需機器旁的方塊，然後按一下 [加入群組]。  然後，選擇您想在其中加入機器的群組。
 
### <a name="removing-machines-from-a-group"></a>從群組移除多部機器
在 [群組] 清單中，展開群組名稱以列出機器群組中的機器。  然後，按一下您想移除之機器旁的省略符號功能表，然後選擇 [移除]。

![從群組移除機器](media/monitoring-service-map/machine-groups-remove.png)

### <a name="removing-or-renaming-a-group"></a>移除或重新命名群組
按一下 [群組] 清單中群組名稱旁的省略符號功能表。

![機器群組功能表](media/monitoring-service-map/machine-groups-menu.png)


## <a name="role-icons"></a>角色圖示
某些處理序在機器上扮演特殊角色︰Web 伺服器、應用程式伺服器及資料庫等。 服務對應會為程序和機器方塊加上角色圖示註解，以協助您一下就識別出程序或伺服器所扮演的角色。

| 角色圖示 | 說明 |
|:--|:--|
| ![Web 伺服器](media/monitoring-service-map/role-web-server.png) | Web 伺服器 |
| ![應用程式伺服器](media/monitoring-service-map/role-application-server.png) | 應用程式伺服器 |
| ![資料庫伺服器](media/monitoring-service-map/role-database.png) | 資料庫伺服器 |
| ![LDAP 伺服器](media/monitoring-service-map/role-ldap.png) | LDAP 伺服器 |
| ![SMB 伺服器](media/monitoring-service-map/role-smb.png) | SMB 伺服器 |

![角色圖示](media/monitoring-service-map/role-icons.png)


## <a name="failed-connections"></a>失敗的連線
「服務對應」對應內會顯示處理序和機器的失敗連線，並以紅色虛線指示用戶端系統無法連線到處理序或連接埠。 已部署服務對應代理程式的系統如果就是嘗試進行失敗連線的系統，則會報告失敗的連線。 服務對應會觀察無法建立連線的 TCP 通訊端，藉以衡量此處理序。 連線失敗的原因可能是防火牆、用戶端或伺服器設定不正確，或遠端服務無法使用。

![失敗的連線](media/monitoring-service-map/failed-connections.png)

了解失敗的連線有助於疑難排解、移轉驗證、安全性分析和整體架構理解。 失敗的連線有時無害，但它們通常直指問題所在，例如容錯移轉環境突然變成無法連線，或兩個應用程式層在雲端移轉之後無法通訊。

## <a name="client-groups"></a>用戶端群組
用戶端群組是對應上的方塊，代表沒有相依性代理程式的用戶端電腦。 單一用戶端群組代表個別處理序或機器的用戶端。

![用戶端群組](media/monitoring-service-map/client-groups.png)

若要查看用戶端群組中伺服器的 IP 位址，選取群組。 群組的內容會列在 [用戶端群組屬性] 窗格中。

![用戶端群組屬性](media/monitoring-service-map/client-group-properties.png)

## <a name="server-port-groups"></a>伺服器連接埠群組
伺服器連接埠群組是方塊，代表沒有相依性代理程式的伺服器上的伺服器連接埠。 此方塊包含伺服器連接埠，以及與該連接埠連線的伺服器數目。 展開方塊可查看個別伺服器和連線。 如果在方塊中只有一部伺服器，則會列出其名稱或 IP 位址。

![伺服器連接埠群組](media/monitoring-service-map/server-port-groups.png)

## <a name="context-menu"></a>內容功能表
按一下任何伺服器右上角的省略符號 (...)，會顯示該伺服器的內容功能表。

![失敗的連線](media/monitoring-service-map/context-menu.png)

### <a name="load-server-map"></a>載入伺服器對應
按一下 [載入伺服器對應] 會導向新的對應，並以所選取的伺服器做為新的焦點機器。

### <a name="show-self-links"></a>顯示自我連結
按一下 [顯示自我連結] 將會重繪包括任何自我連結的伺服器節點。自我連結即是以伺服器內處理序做為開始和結束的 TCP 連線。 如果顯示了自我連結，功能表命令會變更為 [隱藏自我連結]，就可以將它們關閉。

## <a name="computer-summary"></a>電腦摘要
[機器摘要] 窗格包含伺服器作業系統的概觀、相依性計數，以及其他解決方案的資料。 這些資料包括效能計量、服務台票證、變更追蹤、安全性和更新。

![[機器摘要] 窗格](media/monitoring-service-map/machine-summary.png)

## <a name="computer-and-process-properties"></a>電腦和處理序屬性
在瀏覽「服務對應」對應時，可以選取機器和處理序來取得其他有關其屬性的內容。 機器會提供有關 DNS 名稱、IPv4 位址、CPU 和記憶體容量、VM 類型、作業系統和版本、上次重新開機時間，以及其 OMS 和服務對應代理程式識別碼的資訊。

![[機器屬性] 窗格](media/monitoring-service-map/machine-properties.png)

處理序詳細資料是收集自作業系統中繼資料，其內容與執行中的處理序有關，包括處理序名稱、處理序描述、使用者名稱和網域 (在 Windows 上)、公司名稱、產品名稱、產品版本、工作目錄、命令列，以及處理序開始時間。

![[處理序屬性] 窗格](media/monitoring-service-map/process-properties.png)

[處理序摘要] 窗格會提供其他有關該處理序連線的資訊，包括其繫結連接埠、輸入及輸出連線，以及失敗的連線。

![[處理序摘要] 窗格](media/monitoring-service-map/process-summary.png)

## <a name="alerts-integration"></a>警示整合
服務對應會與 Azure 警示整合，以顯示所選時間範圍內針對所選伺服器觸發的警示。 如果有最新警示，伺服器會顯示圖示，且 [機器警示] 窗格會列出警示。

![[機器警示] 窗格](media/monitoring-service-map/machine-alerts.png)

若要讓服務對應顯示相關警示，請建立針對特定電腦觸發的警示規則。 若要建立適當的警示︰
- 包含依電腦群組的子句，例如，**by Computer interval 1 minute** (依 1 分鐘的電腦間隔)。
- 選擇依據計量量值來發出警示。

## <a name="log-events-integration"></a>記錄事件整合
服務對應會與記錄搜尋整合，以顯示所選時間範圍期間，適用於所選伺服器的所有可用記錄事件計數。 您可以按一下事件計數清單中的任一列，以移至記錄搜尋並查看個別的記錄事件。

![[機器記錄事件] 窗格](media/monitoring-service-map/log-events.png)

## <a name="service-desk-integration"></a>服務台整合
當「服務對應」和「IT 服務管理連接器」這兩個解決方案皆已在 Log Analytics 工作區中啟用並設定，便會自動進行整合。 服務對應中的整合會標示為「服務台」。 如需詳細資訊，請參閱[使用 IT 服務管理連接器將 ITSM 工作項目集中管理](https://docs.microsoft.com/azure/log-analytics/log-analytics-itsmc-overview)。

[機器服務台] 窗格會列出所選時間範圍內所選伺服器的所有 IT 服務管理事件。 如果有最新項目，伺服器會顯示圖示，且 [機器服務台] 窗格會列出這些項目。

![[機器服務台] 窗格](media/monitoring-service-map/service-desk.png)

若要在連線的 ITSM 解決方案中開啟項目，請按一下 [檢視工作項目]。

若要檢視記錄搜尋中項目的詳細資料，請按一下 [在記錄搜尋中顯示]。


## <a name="change-tracking-integration"></a>變更追蹤整合
當「服務對應」和「變更追蹤」這兩個解決方案皆已在 Log Analytics 工作區中啟用並設定，便會自動進行整合。

[機器變更追蹤] 窗格會列出所有變更，從最新排到最舊，並有連結可供向下鑽研記錄搜尋，以取得其他詳細資料。

![[機器變更追蹤] 窗格](media/monitoring-service-map/change-tracking.png)

下圖是選取 [在 Log Analytics 中顯示] 之後，可能會看到的 ConfigurationChange 事件詳細檢視。

![ConfigurationChange 事件](media/monitoring-service-map/configuration-change-event-01.png)


## <a name="performance-integration"></a>效能整合
[機器效能] 窗格會顯示所選伺服器的標準效能計量。 這些計量包含 CPU 使用率、記憶體使用率、傳送和接收的網路位元組，以及按照傳送和接收的網路位元組排序的前幾個處理序清單。

![[機器效能] 窗格](media/monitoring-service-map/machine-performance.png)

若要查看效能資料，您可能需要[啟用適當的 Log Analytics 效能計數器](https://docs.microsoft.com/azure/log-analytics/log-analytics-data-sources-performance-counters)。  您將要啟用的計數器：

Windows:
- Processor(*)\\% Processor Time
- Memory\\% Committed Bytes In Use
- Network Adapter(*)\\Bytes Sent/sec
- Network Adapter(*)\\Bytes Received/sec

Linux：
- Processor(*)\\% Processor Time
- Memory(*)\\% Used Memory
- Network Adapter(*)\\Bytes Sent/sec
- Network Adapter(*)\\Bytes Received/sec

若要取得網路效能資料，您也必須在工作區中啟用 Wire Data 2.0 解決方案。
 
## <a name="security-integration"></a>安全性整合
當「服務對應」和「安全性與稽核」這兩個解決方案皆已在 Log Analytics 工作區中啟用並設定，便會自動進行整合。

[機器安全性] 面板會顯示安全性與稽核解決方案中針對所選伺服器的資料。 此窗格會列出所選時間範圍內伺服器任何未處理之安全性問題的摘要。 按一下任一安全性問題會向下鑽研到記錄搜尋，以顯示關於安全性問題的詳細資料。

![[機器安全性] 窗格](media/monitoring-service-map/machine-security.png)

## <a name="updates-integration"></a>更新整合
當「服務對應」和「更新管理」這兩個解決方案皆已在 Log Analytics 工作區中啟用並設定，便會自動進行整合。

[機器更新] 面板會顯示更新管理解決方案中針對所選伺服器的資料。 此窗格會列出所選時間範圍內伺服器所缺少之任何更新的摘要。

![[機器變更追蹤] 窗格](media/monitoring-service-map/machine-updates.png)

## <a name="log-analytics-records"></a>Log Analytics 記錄
服務對應的電腦和處理序清查資料可供在 Log Analytics 中進行[搜尋](../log-analytics/log-analytics-log-searches.md)。 您可以將此資料套用至各種案例，包括移轉規劃、容量分析、探索和隨選效能疑難排解。

除了當處理序或電腦啟動時或是新增到服務對應時所產生的記錄外，每小時還會為每個唯一的電腦和處理序產生一筆記錄。 這些記錄具有下表中的屬性。 ServiceMapComputer_CL 事件中的欄位和值對應到 ServiceMap Azure Resource Manager API 中的機器資源欄位。 ServiceMapProcess_CL 事件中的欄位和值對應到 ServiceMap Azure Resource Manager API 中的處理序資源欄位。 ResourceName_s 欄位會符合對應 Resource Manager 資源中的名稱欄位。 

>[!NOTE]
>在服務對應功能增加之際，這些欄位可能會隨之變更。

有可用來識別唯一處理程序和電腦的內部產生屬性︰

- 電腦 - 使用 ResourceId 或 ResourceName_s 來唯一識別 Log Analytics 工作區中的電腦。
- 處理序：使用 ResourceId 來唯一識別 Log Analytics 工作區中的處理序。 ResourceName_s 在執行處理序的機器 (MachineResourceName_s) 的環境中是唯一的 

因為在指定時間範圍內可以有多筆指定處理序和電腦的記錄，針對相同電腦或處理序的查詢可能會傳回多筆記錄。 若只要包含最新的記錄，請在查詢中加入 "| dedup ResourceId"。

### <a name="servicemapcomputercl-records"></a>ServiceMapComputer_CL 記錄
類型為 *ServiceMapComputer_CL* 的記錄會有伺服器 (具有服務對應代理程式) 的清查資料。 這些記錄具有下表中的屬性：

| 屬性 | 說明 |
|:--|:--|
| 類型 | *ServiceMapComputer_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | 工作區中機器的唯一識別碼 |
| ResourceName_s | 工作區中機器的唯一識別碼 |
| ComputerName_s | 電腦 FQDN |
| Ipv4Addresses_s | 伺服器的 IPv4 位址清單 |
| Ipv6Addresses_s | 伺服器的 IPv6 位址清單 |
| DnsNames_s | DNS 名稱的陣列 |
| OperatingSystemFamily_s | Windows 或 Linux |
| OperatingSystemFullName_s | 作業系統的完整名稱  |
| Bitness_s | 機器的運算位元數 (32 位元或 64 位元)  |
| PhysicalMemory_d | 實體記憶體 (MB) |
| Cpus_d | CPU 數目 |
| CpuSpeed_d | CPU 速度 (MHz)|
| VirtualizationState_s | *unknown**physical**virtual* *hypervisor* |
| VirtualMachineType_s | *hyperv*、*vmware* 等等 |
| VirtualMachineNativeMachineId_g | VM 識別碼 (由其 Hypervisor 指派) |
| VirtualMachineName_s | VM 的名稱 |
| BootTime_t | 開機時間 |



### <a name="servicemapprocesscl-type-records"></a>ServiceMapProcess_CL 類型記錄
類型為 *ServiceMapProcess_CL* 的記錄會有伺服器 (具有服務對應代理程式) 上 TCP 連線處理程序的清查資料。 這些記錄具有下表中的屬性：

| 屬性 | 說明 |
|:--|:--|
| 類型 | *ServiceMapProcess_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | 工作區中處理序的唯一識別碼 |
| ResourceName_s | 在執行處理序的機器上，處理序的唯一識別碼|
| MachineResourceName_s | 機器的資源名稱 |
| ExecutableName_s | 處理序可執行檔的名稱 |
| StartTime_t | 處理序集區的開始時間 |
| FirstPid_d | 處理序集區中的第一個 PID |
| Description_s | 處理序的描述 |
| CompanyName_s | 公司的名稱 |
| InternalName_s | 內部名稱 |
| ProductName_s | 產品的名稱 |
| ProductVersion_s | 產品版本 |
| FileVersion_s | 檔案版本 |
| CommandLine_s | 命令列 |
| ExecutablePath _s | 可執行檔的路徑 |
| WorkingDirectory_s | 工作目錄 |
| UserName | 執行處理序的帳戶 |
| UserDomain | 執行處理序的網域 |


## <a name="sample-log-searches"></a>記錄搜尋範例

### <a name="list-all-known-machines"></a>列出所有已知的機器
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>列出所有受控電腦的實體記憶體容量。
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project PhysicalMemory_d, ComputerName_s

### <a name="list-computer-name-dns-ip-and-os"></a>列出電腦名稱、DNS、IP 和 OS。
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project ComputerName_s, OperatingSystemFullName_s, DnsNames_s, Ipv4Addresses_s

### <a name="find-all-processes-with-sql-in-the-command-line"></a>在命令列中尋找具有「sql」的所有處理程序
ServiceMapProcess_CL | where CommandLine_s contains_cs "sql" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>以資源名稱尋找機器 (最新的記錄)
search in (ServiceMapComputer_CL) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>以 IP 位址尋找機器 (最新的記錄)
search in (ServiceMapComputer_CL) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-all-known-processes-on-a-specified-machine"></a>列出指定機器上的所有已知處理序
ServiceMapProcess_CL | where MachineResourceName_s == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by ResourceId

### <a name="list-all-computers-running-sql"></a>列出所有執行 SQL 的電腦
ServiceMapComputer_CL | where ResourceName_s in ((search in (ServiceMapProcess_CL) "\*sql\*" | distinct MachineResourceName_s)) | distinct ComputerName_s

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>列出資料中心內所有唯一 curl 產品版本
ServiceMapProcess_CL | where ExecutableName_s == "curl" | distinct ProductVersion_s

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>為所有執行 CentOS 的電腦建立電腦群組
ServiceMapComputer_CL | where OperatingSystemFullName_s contains_cs "CentOS" | distinct ComputerName_s


## <a name="rest-api"></a>REST API
服務對應中所有的伺服器、處理序及相依性資料，都可透過[服務對應 REST API](https://docs.microsoft.com/rest/api/servicemap/) 取得。


## <a name="diagnostic-and-usage-data"></a>診斷和使用量資料
當您使用服務對應服務時，Microsoft 會自動收集使用量和效能資料。 Microsoft 使用這項資料來提供和改進服務對應服務的品質、安全性和完整性。 資料中包含軟體設定的相關資訊，像是作業系統與版本、IP 位址、DNS 名稱和工作站名稱，以便能夠正確且有效率地進行疑難排解。 Microsoft 不會收集姓名、地址或其他連絡資訊。

如需有關資料收集與使用方式的詳細資訊，請參閱 [Microsoft 線上服務隱私權聲明](https://go.microsoft.com/fwlink/?LinkId=512132)。


## <a name="next-steps"></a>後續步驟
深入了解 Log Analytics 中的[記錄搜尋](../log-analytics/log-analytics-log-searches.md)，以擷取服務對應所收集的資料。


## <a name="troubleshooting"></a>疑難排解
請參閱[設定服務對應文件的疑難排解小節]( monitoring-service-map-configure.md#troubleshooting)。


## <a name="feedback"></a>意見反應
您對「服務對應」或這份文件有任何意見反應要提供給我們嗎？  請瀏覽我們的[使用者意見頁面](https://feedback.azure.com/forums/267889-log-analytics/category/184492-service-map) \(英文\)，您可以在此頁面提出功能建議或對現有的建議進行投票。
