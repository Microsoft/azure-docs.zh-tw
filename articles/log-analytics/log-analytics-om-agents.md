---
title: 將 Operations Manager 連接到 Log Analytics | Microsoft Docs
description: 若要維護 System Center Operations Manager 中的現有投資，並使用 Log Analytics 的延伸功能，您可以整合 Operations Manager 與工作區。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 245ef71e-15a2-4be8-81a1-60101ee2f6e6
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/02/2018
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: f02da6f21d0c11724b4192d5af0d5cc75d2c770c
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49321558"
---
# <a name="connect-operations-manager-to-log-analytics"></a>將 Operations Manager 連接到 Log Analytics
若要維護 System Center Operations Manager 中的現有投資，並使用 Log Analytics 的延伸功能，您可以整合 Operations Manager 與 Log Analytics 工作區。 這可讓您利用 Log Analytics 的機會，同時繼續使用 Operations Manager：

* 使用 Operations Manager 監視 IT 服務的健全狀況
* 維護與支援事件和問題管理之 ITSM 解決方案的整合
* 管理代理程式的生命週期，這些代理程式部署到內部部署以及您使用 Operations Manager 所監視的公用雲端 IaaS 虛擬機器

與 System Center Operations Manager 整合後，從 Operations Manager 收集、儲存和分析資料時可發揮 Log Analytics 速度和效率，提高服務作業策略的價值。 Log Analytics 有助於找出問題的關聯性，並努力查明原因和杜絕再次發生，以支援現有的問題管理程序。 檢查效能、事件和警示資料的搜尋引擎的彈性 (具有豐富的儀表板和報告功能可透過有意義的方式公開此資料) 示範 Log Analytics 對值得讚揚的 Operations Manager 的強度。

向 Operations Manager 管理群組回報的代理程式，會根據已在工作區中啟用的 Log Analytics 資料來源和解決方案以從您的伺服器收集資料。 根據已啟用的解決方案，其資料會從 Operations Manager 管理伺服器直接傳送給服務，或者因為代理程式受控系統上收集的資料量，而將這些解決方案中的資料從代理程式直接傳送給 Log Analytics。 管理伺服器會直接將資料轉送到服務，永遠不會寫入作業或資料倉儲資料庫。 如果管理伺服器中斷與 Log Analytics 的連線，則會在重新建立與 Log Analytics 的通訊之前，將資料快取在本機。 如果管理伺服器因發生規劃的維護或未規劃的中斷而離線，管理群組中的另一部管理伺服器會繼續與 Log Analytics 連線。  

下圖顯示管理伺服器與 System Center Operations Manager 管理群組中的代理程式和 Log Analytics 之間的連接，包括方向和連接埠。   

![oms-operations-manager-integration-diagram](./media/log-analytics-om-agents/oms-operations-manager-connection.png)

如果 IT 安全性原則不允許您網路上的電腦連線到網際網路，則可以將管理伺服器設定為連線到 OMS 閘道，以根據已啟用的解決方案來接收組態資訊和傳送收集到的資料。 如需如何設定 Operations Manager 管理群組以透過 OMS 閘道與 Log Analytics 服務進行通訊的其他資訊和步驟，請參閱[使用 OMS 閘道將電腦連線到 OMS](log-analytics-oms-gateway.md)。  

## <a name="prerequisites"></a>必要條件 
開始之前，請檢閱下列需求。

* Log Analytics 只支援 System Center Operations Manager 1807、Operations Manager 1801、Operations Manager 2016、Operations Manager 2012 SP1 UR6 或更新版本，以及 Operations Manager 2012 R2 UR2 或更新版本。 Operations Manager 2012 SP1 UR7 和 Operations Manager 2012 R2 UR3 中已加入 Proxy 支援。
* 所有 Operations Manager 代理程式必須符合最低支援需求。 請確定代理程式已安裝最低更新版本，否則 Windows 代理程式通訊可能會失敗，而且會在 Operations Manager 事件記錄中產生錯誤。
* Log Analytics 工作區。 如需詳細資訊，請檢閱[將電腦從您的環境連線到 Log Analytics](log-analytics-concept-hybrid.md)。
* 您必須使用屬於 [Log Analytics 參與者角色](log-analytics-manage-access.md#manage-accounts-and-users)向 Azure 驗證。  

>[!NOTE]
>最近對 Azure API 所做的變更將會防止客戶第一次就成功地在其管理群組與 Log Analytics 之間設定整合。 已將其管理群組與該服務整合的客戶則不會受影響，除非您需要重新設定您的現有連線。  
>我們已針對每個版本的 Operations Manager 發行新的管理組件：  
>* 針對 System Center Operations Manager 1801，請從[這裡](https://www.microsoft.com/download/details.aspx?id=57173)下載管理組件  
>* 針對 System Center 2016 - Operations Manager，請從[這裡](https://www.microsoft.com/download/details.aspx?id=57172)下載管理組件  
>* 針對 System Center Operations Manager 2012 R2，請從[這裡](https://www.microsoft.com/en-us/download/details.aspx?id=57171)下載管理組件  

### <a name="network"></a>網路
下列資訊列出 Operations Manager 代理程式、管理伺服器及 Operations 主控台與 Log Analytics 通訊所需的 Proxy 和防火牆組態資訊。 每個元件的流量會從您的網路輸出至 Log Analytics 服務。   

|資源 | 連接埠號碼| 略過 HTTPS 檢查|  
|---------|------|-----------------------|  
|**代理程式**|||  
|\*.ods.opinsights.azure.com| 443 |是|  
|\*.oms.opinsights.azure.com| 443|是|  
|\*.blob.core.windows.net| 443|是|  
|\*.azure-automation.net| 443|是|  
|**管理伺服器**|||  
|\*.service.opinsights.azure.com| 443||  
|\*.blob.core.windows.net| 443| 是|  
|\*.ods.opinsights.azure.com| 443| 是|  
|*.azure-automation.net | 443| 是|  
|**Operations Manager 主控台至 OMS**|||  
|service.systemcenteradvisor.com| 443||  
|\*.service.opinsights.azure.com| 443||  
|\*.live.com| 80 和 443||  
|\*.microsoft.com| 80 和 443||  
|\*.microsoftonline.com| 80 和 443||  
|\*.mms.microsoft.com| 80 和 443||  
|login.windows.net| 80 和 443||  
|portal.loganalytics.io| 80 和 443||
|api.loganalytics.io| 80 和 443||
|docs.loganalytics.io| 80 和 443||  

### <a name="tls-12-protocol"></a>TLS 1.2 通訊協定
為了確保資料傳送至 Log Analytics 時的安全性，我們強烈建議您將代理程式和管理群組設定為至少使用傳輸層安全性 (TLS) 1.2。 我們已發現較舊版本的 TLS/安全通訊端層 (SSL) 較易受到攻擊，而且在其目前的運作中仍允許回溯相容性，因此並**不建議使用**這些版本。 如需其它資訊，請檢閱[使用 TLS 1.2 安全地傳送](log-analytics-data-security.md#sending-data-securely-using-tls-12)。 

## <a name="connecting-operations-manager-to-log-analytics"></a>將 Operations Manager 連接到 Log Analytics
執行下列一系列的步驟，設定 Operations Manager 管理群組來連接到其中一個 Log Analytics 工作區。

在 Operations Manager 管理群組第一次向 Log Analytics 工作區進行登錄期間，在 Operations 主控台中為管理群組指定 Proxy 設定的選項無法使用。  必須先向服務成功登錄該管理群組，之後此選項才可供使用。  若要因應此問題，您需要在執行 Operations 主控台的系統上使用 Netsh 更新系統 Proxy 設定，以在管理群組中設定整合，以及所有管理伺服器。  

1. 開啟提升權限的命令提示字元。
   a. 移至 [開始] 並輸入 **cmd**。
   b. 用滑鼠右鍵按一下 [命令提示字元]，然後選取 [以系統管理員身分執行]**。
1. 輸入下列命令並按 **Enter** 鍵：

    `netsh winhttp set proxy <proxy>:<port>`

完成下列步驟與 Log Analytics 的整合之後，您可以透過執行 `netsh winhttp reset proxy` 來移除設定，然後使用 Operations 主控台中的 [設定 Proxy 伺服器] 選項來指定 Proxy 或 OMS 閘道伺服器。 

1. 在 Operations Manager 主控台中，選取 [管理]  工作區。
1. 展開 Operations Management Suite 節點，然後按一下 [連接] 。
1. 按一下 [註冊到 Operations Management Suite]  連結。
1. 在 [Operations Management Suite 登入精靈：驗證] 頁面上，輸入與 OMS 訂用帳戶相關聯之系統管理員帳戶的電子郵件地址或電話號碼和密碼，然後按一下 [登入]。
1. 成功通過驗證之後，在 [Operations Management Suite 登入精靈: 選取工作區] 頁面上，系統會提示您選取 Azure 租用戶、訂用帳戶與 Log Analytics 工作區。 如果您有多個工作區，請從下拉式清單中選取您想要向 Operations Manager 管理群組註冊的工作區，然後按 [下一步] 。
   
   > [!NOTE]
   > Operations Manager 一次只支援一個 Log Analytics 工作區。 會從 Log Analytics 移除連接以及使用前一個工作區向 Log Analytics 註冊的電腦。
   > 
   > 
1. 在 [Operations Management Suite 登入精靈：摘要] 頁面上，確認您的設定，如果正確無誤，請按一下 [建立]。
1. 在 [Operations Management Suite 登入精靈：完成] 頁面上，按一下 [關閉]。

### <a name="add-agent-managed-computers"></a>加入代理程式的受控電腦
設定與 Log Analytics 工作區的整合之後，只會建立與 Log Analytics 的連線，並不會從向管理群組回報的代理程式收集任何資料。 除非您設定哪些特定代理程式的受控電腦會收集 Log Analytics 的資料，否則不會發生這種情況。 您可以個別選取電腦物件，也可以選取包含 Windows 電腦物件的群組。 您無法選取包含另一個類別之執行個體 (例如邏輯磁碟或 SQL 資料庫) 的群組。

1. 開啟 Operations Manager 主控台，然後選取 [ **管理** ] 工作區。
1. 展開 Operations Management Suite 節點，然後按一下 [連接] 。
1. 按一下窗格右側之 [執行] 標題下方的 [加入電腦/群組]  連結。
1. 在 [電腦搜尋] 對話方塊中，您可以搜尋 Operations Manager 監視的電腦或群組。 選取要上架到 Log Analytics 的電腦或群組，按一下 [新增]，然後按一下 [確定]。

在 Operations 主控台的 [管理]  工作區中，您可以檢視電腦和群組，這些電腦和群組設定成收集來自 Operations Management Suite 下方之 [受控電腦] 節點的資料。 您可以視需要在這裡新增或移除電腦和群組。

### <a name="configure-proxy-settings-in-the-operations-console"></a>在 Operations 主控台中設定 Proxy 設定
如果內部 Proxy 伺服器位在管理群組與 Log Analytics 服務之間，請執行下列步驟。 這些設定是從管理群組進行集中管理，並且散發至範圍中所含的代理程式所管理系統，來收集 Log Analytics 的資料。  特定解決方案略過管理伺服器並將資料直接傳送給服務時，這十分有幫助。

1. 開啟 Operations Manager 主控台，然後選取 [ **管理** ] 工作區。
1. 展開 Operations Management Suite，然後按一下 [連接] 。
1. 在 [OMS 連線] 檢視中，按一下 [設定 Proxy 伺服器] 。
1. 在 [Operations Management Suite 精靈：Proxy 伺服器] 頁面上，選取 [使用 Proxy 伺服器來存取 Operations Management Suite]，然後輸入具有連接埠號碼的 URL (例如， http://corpproxy:80 )，然後按一下 [完成]。

如果 Proxy 伺服器需要驗證，請執行下列步驟來設定認證和設定，這些需要傳播到管理群組中向 OMS 回報的受控電腦。

1. 開啟 Operations Manager 主控台，然後選取 [ **管理** ] 工作區。
1. 在 [RunAs 組態] 底下，選取 [設定檔]。
1. 開啟 [ **System Center Advisor 執行身份設定檔 Proxy** ] 設定檔。
1. 在 [執行身分設定檔精靈] 中，按一下 [加入] 使用執行身分帳戶。 您可以建立[執行身分帳戶](https://technet.microsoft.com/library/hh321655.aspx)，或使用現有的帳戶。 此帳戶必須有足夠的權限，才能通過 Proxy 伺服器。
1. 若要設定帳戶來管理，請選擇 [選取的類別、群組或物件]，按一下 [選取…] 然後按一下 [群組…] 開啟 [群組搜尋] 方塊。
1. 搜尋，然後選取 [Microsoft System Center Advisor 監控伺服器群組] 。 選取群組之後，按一下 [確定] 關閉 [群組搜尋] 方塊。
1. 按一下 [確定] 以關閉 [新增執行身分帳戶] 方塊。
1. 按一下 [儲存]  完成精靈並儲存變更。

建立連線並設定哪些代理程式將收集資料並回報給 Log Analytics 之後，管理群組中就會套用下列組態，但未必依照下列順序︰

* 建立執行身分帳戶 **Microsoft.SystemCenter.Advisor.RunAsAccount.Certificate** 。 它是與執行身分設定檔 **Microsoft System Center Advisor Run As Profile Blob** 相關聯，而且將目標設為兩個類別：[收集伺服器] 和 [Operations Manager 管理群組]。
* 會建立兩個連接器。  第一個稱為 **Microsoft.SystemCenter.Advisor.DataConnector**，並自動以一個訂用帳戶設定，這個訂用帳戶會將管理群組中所有類別之執行個體產生的所有警示轉送給 Log Analytics。 第二個連接器是 **Advisor 連接器**，負責與 OMS Web 服務進行通訊以及共用資料。
* 您在管理群組中已選擇來收集資料的代理程式和群組，將會新增至 [Microsoft System Center Advisor 監控伺服器群組]。

## <a name="management-pack-updates"></a>管理組件更新
組態完成之後，Operations Manager 管理群組就會建立與 Log Analytics 服務的連接。 針對您已啟用且與 Operations Manager 整合的解決方案，管理伺服器會與 Web 服務同步處理，並以管理組件的形式接收更新的組態資訊。 Operations Manager 會檢查這些管理組件的更新，如果有更新，就會自動下載並匯入。 有兩個規則特別可控制這個行為︰

* **Microsoft.SystemCenter.Advisor.MPUpdate** - 更新基底 Log Analytics 管理組件。 預設會每 12 小時執行一次。
* **Microsoft.SystemCenter.Advisor.Core.GetIntelligencePacksRule** - 更新工作區中所啟用的解決方案管理組件。 預設會每五 (5) 分鐘執行一次。

您可以透過停用來覆寫這兩個規則以防止自動下載，或者修改管理伺服器與 OMS 同步處理之頻率的頻率，來決定新的管理組件是否可用而且應該予以下載。 遵循[如何覆寫規則或監視器](https://technet.microsoft.com/library/hh212869.aspx)步驟，使用值 (秒) 修改 [頻率] 參數來變更同步處理排程，或修改 [已啟用] 參數來停用規則。 將目標設為覆寫 [Operations Manager 管理群組] 類別的所有物件。

若要繼續遵循現有的變更控制程序來控制生產管理群組中的管理組件發行版本，則可以停用規則，並在允許更新時於特定期間啟用它們。 如果您的環境中有開發或 QA 管理群組，而且該管理群組連接到網際網路，則可以設定該管理群組與 Log Analytics 工作區，以支援此案例。 這可讓您先檢閱和評估 Log Analytics 管理組件的反覆版本，再發行到生產管理群組。

## <a name="switch-an-operations-manager-group-to-a-new-log-analytics-workspace"></a>將 Operations Manager 群組切換到新的 Log Analytics 工作區
1. 在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。
1. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]，然後建立工作區。  
1. 使用身為 Operations Manager 系統管理員角色成員的帳戶開啟 Operations Manager 主控台，然後選取 [管理]  工作區。
1. 展開 Operations Management Suite，選取 [連接] 。
1. 選取窗格中間的 [重新設定 Operation Management Suite]  連結。
1. 遵循 [Operations Management Suite 登入精靈]  進行，輸入與新的 Log Analytics 工作區相關聯之系統管理員帳戶的電子郵件地址或電話號碼和密碼。
   
   > [!NOTE]
   > [Operations Management Suite 登入精靈：選取工作區] 頁面會顯示使用中的現有工作區。
   > 
   > 

## <a name="validate-operations-manager-integration-with-log-analytics"></a>驗證 Operations Manager 與 Log Analytics 的整合
您有幾種不同的方式可以確認 Log Analytics 與 Operations Manager 的整合成功。

### <a name="to-confirm-integration-from-the-azure-portal"></a>從 Azure 入口網站確認整合
1. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。
1. 在 Log Analytics 工作區清單中，選取適用的工作區。  
1. 選取 [進階設定]，選取 [連接的來源]然後選取 [系統中心]。 
1. 在 [System Center Operations Manager] 區段下方的表格中，您應該會看到管理群組名稱，還會列出上次收到資料時的代理程式數目和狀態。
   
   ![oms-settings-connectedsources](./media/log-analytics-om-agents/oms-settings-connectedsources.png)

### <a name="to-confirm-integration-from-the-operations-console"></a>從 Operations 主控台確認整合
1. 開啟 Operations Manager 主控台，然後選取 [ **管理** ] 工作區。
1. 選取 [管理組件]，並在 [尋找:] 文字方塊中輸入 **Advisor** 或 **Intelligence**。
1. 根據您已啟用的解決方案，您會看到搜尋結果中列出對應的管理組件。  例如，如果您已啟用警示管理解決方案，則 [Microsoft System Center Advisor 警示管理] 管理組件會在清單中。
1. 從 [監視] 檢視中，瀏覽至 [Operations Management Suite\健全狀況狀態] 檢視。  在 [管理伺服器狀態] 窗格下選取管理伺服器，然後在 [詳細資料檢視] 窗格中，確認 [驗證服務 URI] 屬性的值符合 Log Analytics 工作區識別碼。
   
   ![oms-opsmgr-mg-authsvcuri-property-ms](./media/log-analytics-om-agents/oms-opsmgr-mg-authsvcuri-property-ms.png)

## <a name="remove-integration-with-log-analytics"></a>移除與 Log Analytics 的整合
當您不再需要整合 Operations Manager 管理群組和 Log Analytics 工作區時，需要執行幾個步驟，才能適當移除管理群組中的連接和組態。 下列程序可讓您刪除管理群組的參考來更新 Log Analytics 工作區、刪除 Log Analytics 連接器，然後刪除支援與該服務整合的管理組件。  

已啟用且與 Operations Manager 整合之解決方案的管理組件，以及支援與 Log Analytics 服務整合所需的管理組件，都無法輕易地從管理群組中刪除。 這是因為有些 Log Analytics 管理組件與其他相關管理組件相依。 若要刪除與其他管理組件相依的管理組件，請從 TechNet 指令碼中心下載[移除具有相依性的管理組件](https://gallery.technet.microsoft.com/scriptcenter/Script-to-remove-a-84f6873e) (英文) 指令碼。  

1. 使用身為 Operations Manager 系統管理員角色成員的帳戶開啟 Operations Manager 命令殼層。
   
    > [!WARNING]
    > 繼續之前，請確認您的任何自訂管理組件名稱中沒有 Advisor 或 IntelligencePack 這個字，否則下列步驟會從管理群組中刪除它們。
    > 

1. 從命令殼層提示字元中，輸入 `Get-SCOMManagementPack -name "*Advisor*" | Remove-SCOMManagementPack -ErrorAction SilentlyContinue`
1. 接著輸入 `Get-SCOMManagementPack -name “*IntelligencePack*” | Remove-SCOMManagementPack -ErrorAction SilentlyContinue`
1. 若要移除與其他 System Center Advisor 管理組件具有相依性的任何其餘管理組件，請使用您稍早從 TechNet 指令碼中心下載的 *RecursiveRemove.ps1* 指令碼。  
 
    > [!NOTE]
    > 使用 PowerShell 移除 Advisor 管理組件的步驟將不會自動刪除 Microsoft System Center Advisor 或 Microsoft System Center Advisor 內部管理組件。  不要嘗試刪除它們。  
    >  

1. 使用身為 Operations Manager 系統管理員角色成員的帳戶開啟 Operations Manager Operations 主控台。
1. 在 [管理] 下，選取 [管理組件] 節點，然後在 [尋找:] 方塊中輸入 **Advisor**，並確認下列管理組件仍匯入到管理群組中︰
   
   * Microsoft System Center Advisor
   * Microsoft System Center Advisor Internal

1. 在 OMS 入口網站中，按一下 [設定] 圖格。
1. 選取 [連接的來源] 。
1. 在 [System Center Operations Manager] 區段下方的表格中，您應該會看到想要從工作區移除的管理群組名稱。 在 [最後一筆資料] 資料行之下，按一下 [移除]。  
   
    > [!NOTE]
    > 在偵測到已連線的管理群組有 14 天沒有任何活動之後，[移除] 連結才能使用。  
    > 

1. 將出現視窗，要求您確認想要繼續移除。  按一下 [是]  以繼續。 

若要刪除兩個連接器 - Microsoft.SystemCenter.Advisor.DataConnector 和 Advisor 連接器，請將以下 PowerShell 指令碼儲存至您的電腦，並使用下列範例來執行：

```
    .\OM2012_DeleteConnectors.ps1 “Advisor Connector” <ManagementServerName>
    .\OM2012_DeleteConnectors.ps1 “Microsoft.SystemCenter.Advisor.DataConnector” <ManagementServerName>
```

> [!NOTE]
> 您執行此指令碼的電腦 (如果不是管理伺服器) 應該已安裝 Operations Manager 命令殼層，視您的管理群組版本而定。
> 
> 

```
    param(
    [String] $connectorName,
    [String] $msName="localhost"
    )
    $mg = new-object Microsoft.EnterpriseManagement.ManagementGroup $msName
    $admin = $mg.GetConnectorFrameworkAdministration()
    ##########################################################################################
    # Configures a connector with the specified name.
    ##########################################################################################
    function New-Connector([String] $name)
    {
         $connectorForTest = $null;
         foreach($connector in $admin.GetMonitoringConnectors())
    {
    if($connectorName.Name -eq ${name})
    {
         $connectorForTest = Get-SCOMConnector -id $connector.id
    }
    }
    if ($connectorForTest -eq $null)
    {
         $testConnector = New-Object Microsoft.EnterpriseManagement.ConnectorFramework.ConnectorInfo
         $testConnector.Name = $name
         $testConnector.Description = "${name} Description"
         $testConnector.DiscoveryDataIsManaged = $false
         $connectorForTest = $admin.Setup($testConnector)
         $connectorForTest.Initialize();
    }
    return $connectorForTest
    }
    ##########################################################################################
    # Removes a connector with the specified name.
    ##########################################################################################
    function Remove-Connector([String] $name)
    {
        $testConnector = $null
        foreach($connector in $admin.GetMonitoringConnectors())
       {
        if($connector.Name -eq ${name})
       {
         $testConnector = Get-SCOMConnector -id $connector.id
       }
      }
     if ($testConnector -ne $null)
     {
        if($testConnector.Initialized)
     {
     foreach($alert in $testConnector.GetMonitoringAlerts())
     {
       $alert.ConnectorId = $null;
       $alert.Update("Delete Connector");
     }
     $testConnector.Uninitialize()
     }
     $connectorIdForTest = $admin.Cleanup($testConnector)
     }
    }
    ##########################################################################################
    # Delete a connector's Subscription
    ##########################################################################################
    function Delete-Subscription([String] $name)
    {
      foreach($testconnector in $admin.GetMonitoringConnectors())
      {
      if($testconnector.Name -eq $name)
      {
        $connector = Get-SCOMConnector -id $testconnector.id
      }
    }
    $subs = $admin.GetConnectorSubscriptions()
    foreach($sub in $subs)
    {
      if($sub.MonitoringConnectorId -eq $connector.id)
      {
        $admin.DeleteConnectorSubscription($admin.GetConnectorSubscription($sub.Id))
      }
     }
    }
    #New-Connector $connectorName
    write-host "Delete-Subscription"
    Delete-Subscription $connectorName
    write-host "Remove-Connector"
    Remove-Connector $connectorName
```

未來，如果您打算將管理群組重新連接至 Log Analytics 工作區，您必須重新匯入 `Microsoft.SystemCenter.Advisor.Resources.\<Language>\.mpb` 管理組件檔案。 視部署在您環境中的 System Center Operations Manager 版本而定，您可以在下列位置中找到此檔案：

* 在 System Center 2016 - Operations Manager 或更新版本的 `\ManagementPacks` 資料夾下方的來源媒體上。
* 從套用到您管理群組的最新更新彙總。 若為 Operations Manager 2012，來源資料夾為 ` %ProgramFiles%\Microsoft System Center 2012\Operations Manager\Server\Management Packs for Update Rollups`，若為 2012 R2，則位於 `System Center 2012 R2\Operations Manager\Server\Management Packs for Update Rollups` 中。

## <a name="next-steps"></a>後續步驟
若要新增功能和收集資料，請參閱[從方案庫新增 Log Analytics 方案](log-analytics-add-solutions.md)。


