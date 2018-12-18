---
title: 使用 OMS 閘道連線電腦 | Microsoft Docs
description: 將您裝置與受 Operations Manager 監視的電腦與 OMS 閘道連線，以在它們沒有網際網路存取時傳送資料給 Azure 自動化和 Log Analytics 服務。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ae9a1623-d2ba-41d3-bd97-36e65d3ca119
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/02/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 74da7e96ed52b441bc63d5fb5a032db9c6d57774
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494271"
---
# <a name="connect-computers-without-internet-access-using-the-oms-gateway"></a>在無網際網路存取下使用 OMS 閘道連線電腦
本文件說明如何在直接連線或 Operations Manager 監視的電腦沒有網際網路存取時，設定使用 OMS 閘道與 Azure 自動化和 Log Analytics 的通訊。  OMS 閘道是使用 HTTP CONNECT 命令支援 HTTP 通道的 HTTP 正向 Proxy，可以代表這些電腦收集資料並傳送給 Azure 自動化和 Log Analytics 服務。  

OMS 閘道支援︰

* Azure 自動化混合式 Runbook 背景工作  
* 直接連線到 Log Analytics 工作區、具有 Microsoft Monitoring Agent 的 Windows 電腦
* 直接連線到 Log Analytics 工作區、具有適用於 Linux 的 OMS 代理程式的 Linux 電腦  
* System Center Operations Manager 2012 SP1 (含 UR7)、Operations Manager 2012 R2 (含 UR3)、Operations Manager 2016 和與 Log Analytics 整合的 Operations Manager 1801 版管理群組。  

如果您的 IT 安全性原則不允許網路上的電腦連線到網際網路，例如銷售點 (POS) 裝置或支援 IT 服務的伺服器，但您需要將它們連線至 Azure 自動化或 Log Analytics 來加以管理和監視，則可以將它們設定為直接與 OMS 閘道通訊，以代表它們接收組態和轉送資料。  如果您為這些電腦設定了 OMS 代理程式以直接連線到 Log Analytics 工作區，所有電腦會改為與 OMS 閘道通訊。  閘道會將資料直接從代理程式傳輸到服務，而不會分析傳輸中的任何資料。

當 Operations Manager 管理群組與 Log Analytics 整合時，您可以將管理伺服器設定為連線至 OMS 閘道，以根據您已啟用的解決方案來接收組態資訊和傳送收集到的資料。  Operations Manager 代理程式會將一些資料 (例如 Operations Manager 警示、組態評估、執行個體空間，以及容量資料) 傳送到管理伺服器。 其他大量資料 (例如 IIS 記錄、效能及安全性事件) 會直接傳送到 OMS 閘道。  如果您將一或多個 Operations Manager 閘道伺服器部署在 DMZ 或其他隔離的網路以監視未受信任的系統，它就無法與 OMS 閘道通訊。  Operations Manager 閘道伺服器只能向管理伺服器報告。  當您將 Operations Manager 管理群組設定為與 OMS 閘道通訊時，Proxy 組態資訊會自動發佈到每個受代理程式管理、並已設定為會收集資料以供 Log Analytics 使用的電腦，即使設定是空的也會如此。    

若要為透過閘道與 Log Analytics 通訊的直接連線群組或 Operations Management 群組提供高可用性，您可以使用網路負載平衡來將流量重新導向，並將流量分散到多個閘道伺服器。  如果某個閘道伺服器故障，系統就會將流量重新導向到其他可用節點。  

執行 OMS 閘道的電腦上需要有 OMS 代理程式，才能識別其需要與之通訊的服務端點，並監視 OMS 閘道以分析其效能或事件資料。

每個代理程式必須具備對其閘道的網路連線，代理程式才能自動將資料傳輸到閘道，或從閘道傳輸資料。 不建議您將閘道安裝在網域控制站上。

下圖顯示使用閘道伺服器從直接代理程式流向 Azure 自動化和 Log Analytics 的資料流程。  代理程式的 Proxy 組態必須符合 OMS 閘道所設定的相同連接埠，以便與服務通訊。  

![代理程式直接與服務通訊的圖表](./media/log-analytics-oms-gateway/oms-omsgateway-agentdirectconnect.png)

下圖顯示從 Operations Manager 管理群組流向 Log Analytics 的資料流程。   

![Operations Manager 與 Log Analytics 通訊的圖表](./media/log-analytics-oms-gateway/log-analytics-agent-opsmgrconnect.png)

## <a name="prerequisites"></a>必要條件

指定電腦來執行 OMS 閘道時，這台電腦必須具備下列條件︰

* Windows 10、Windows 8.1、Windows 7
* Windows Server 2016、Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 R2 及 Windows Server 2008
* .Net Framework 4.5
* 至少為 4 核心處理器和 8 GB 記憶體 
* OMS Agent for Windows 

### <a name="language-availability"></a>提供的語言

OMS 閘道有下列語言版本︰

- 中文 (簡體)
- 中文 (繁體)
- 捷克文
- 荷蘭文
- English
- 法文
- 德文
- 匈牙利文
- 義大利文
- 日文
- 韓文
- 波蘭文
- 葡萄牙文 (巴西)
- 葡萄牙文 (葡萄牙)
- 俄文
- 西班牙文 (國際)

### <a name="supported-encryption-protocols"></a>支援的加密通訊協定
OMS 閘道僅支援傳輸層安全性 (TLS) 1.0、1.1 及 1.2。  不支援安全通訊端層 (SSL)。  為了確保資料傳送至 Log Analytics 時的安全性，我們強烈建議您將閘道設定為至少使用傳輸層安全性 (TLS) 1.2。 我們已發現較舊版本的 TLS/安全通訊端層 (SSL) 較易受到攻擊，而且在其目前的運作中仍允許回溯相容性，因此並**不建議使用**這些版本。  如需其它資訊，請檢閱[使用 TLS 1.2 安全地傳送](log-analytics-data-security.md#sending-data-securely-using-tls-12)。 

### <a name="supported-number-of-agent-connections"></a>支援的代理程式連線數目
下表強調支援與閘道伺服器通訊的代理程式數目。  此支援是根據代理程式每 6 秒上傳 ~200 KB 資料計算。 測試得出每個代理程式的資料量是每天大約 2.7 GB。

|閘道器 |支援的約略代理程式數目|  
|--------|----------------------------------|  
|- CPU：Intel XEON CPU E5-2660 v3 \@ 2.6GHz 2 核心<br> - 記憶體：4 GB<br> - 網路頻寬：1 Gbps| 600|  
|- CPU：Intel XEON CPU E5-2660 v3 \@ 2.6GHz 4 核心<br> - 記憶體：8 GB<br> - 網路頻寬：1 Gbps| 1000|  

## <a name="download-the-oms-gateway"></a>下載 OMS 閘道

有兩種方法可以取得最新版的 OMS 閘道安裝程式檔案。

1. 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=54443)下載。

1. 從 Azure 入口網站下載。  登入 Azure 入口網站之後：  

   1. 瀏覽服務清單，然後選取 [Log Analytics]。  
   1. 選取工作區。
   1. 在 [工作區] 刀鋒視窗的 [一般] 下方，按一下 [快速入門]。
   1. 在 [選擇用來連線到工作區的資料來源] 下方，按一下 [電腦]。
   1. 在 [直接代理程式] 刀鋒視窗中，按一下 [下載 OMS 閘道]。<br><br> ![下載 OMS 閘道](./media/log-analytics-oms-gateway/download-gateway.png)

或 

   1. 在您的工作區刀鋒視窗中，於 [設定] 下按一下 [進階設定]。
   1. 瀏覽至 [連接的來源] > [Windows 伺服器]，然後按一下 [下載 OMS 閘道]。

## <a name="install-the-oms-gateway"></a>安裝 OMS 閘道

若要安裝閘道，請執行下列步驟。  如果您已安裝舊版 (先前稱為「Log Analytics 轉寄站」)，該舊版會升級為此版本。  

1. 從目的地資料夾中，按兩下 **OMS Gateway.msi**。
1. 在 [歡迎] 頁面中按 [下一步]。<br><br> ![閘道設定精靈](./media/log-analytics-oms-gateway/gateway-wizard01.png)<br> 
1. 在 [授權合約] 頁面上，選取 [我接受授權合約中的條款] 以同意 EULA，然後按 [下一步]。
1. 在 [連接埠和 Proxy 位址] 頁面上：
   1. 輸入要用於閘道的 TCP 連接埠號碼。 安裝程式會使用此連接埠號碼在 Windows 防火牆上設定輸入規則。  預設值為 8080。
      連接埠號碼的有效範圍是 1 到 65535。 如果輸入的值不在此範圍內，就會顯示錯誤訊息。
   1. 如果閘道安裝所在的伺服器需要透過 Proxy 進行通訊，您也可以選擇輸入閘道需要連線的 Proxy 位址。 例如： `http://myorgname.corp.contoso.com:80`。  如果空白，閘道將會嘗試直接連線到網際網路。  如果您的 Proxy 伺服器需要驗證，請輸入使用者名稱與密碼。<br><br> ![閘道精靈 Proxy 組態](./media/log-analytics-oms-gateway/gateway-wizard02.png)<br>   
   1. 按 [下一步] 。
1. 如果您沒有啟用 Microsoft Update，系統會顯示 Microsoft Update 頁面，您可以在其中選擇啟用 Microsoft Update。 選擇想要的選項，然後按一下 [下一步]。 否則，請繼續下一個步驟。
1. 在 [目的地資料夾] 頁面上，保留預設資料夾 C:\Program Files\OMS Gateway 或輸入您想要安裝閘道的位置，然後按 [下一步]。
1. 在 [準備安裝] 頁面上，按一下 [安裝]。 系統可能會顯示 [使用者帳戶控制] 來要求安裝權限。 如果有顯示，請按一下 [是]。
1. 安裝完成後，請按一下 [完成]。 您可以透過開啟 services.msc 嵌入式管理單元來確認服務正在執行，並確認服務清單中已顯示 [OMS 閘道]，且其狀態為 [執行中]。<br><br> ![服務 – OMS 閘道](./media/log-analytics-oms-gateway/gateway-service.png)  

## <a name="configure-network-load-balancing"></a>設定網路負載平衡 
您可以設定閘道以獲得使用網路負載平衡 (NLB) 的高可用性，此平衡可能是使用 Microsoft 網路負載平衡 (NLB) 或硬體式負載平衡器來達成。  負載平衡器可藉由跨其節點將來自 OMS 代理程式或 Operations Manager 管理伺服器的要求連線進行重新導向，來管理流量。 如果閘道伺服器故障，流量就會被重新導向到其他節點。

若要了解如何設計和部署 Windows Server 2016 網路負載平衡叢集，請參閱[網路負載平衡](https://technet.microsoft.com/windows-server-docs/networking/technologies/network-load-balancing)。  下列步驟說明如何設定 Microsoft 網路負載平衡叢集。  

1. 使用系統管理帳戶登入屬於 NLB 叢集成員的 Windows 伺服器。  
1. 在 [伺服器管理員] 中開啟網路負載平衡管理員，然後依序按一下 [工具] 和 [網路負載平衡管理員]。
1. 若要連線已安裝 Microsoft Monitoring Agent 的 OMS 閘道伺服器，請用滑鼠右鍵按一下叢集的 IP 位址，然後按一下 [新增主機到叢集]。<br><br> ![網路負載平衡管理員 – 新增主機到叢集](./media/log-analytics-oms-gateway/nlb02.png)<br> 
1. 輸入您要連線之閘道伺服器的 IP 位址。<br><br> ![網路負載平衡管理員 – 新增主機到叢集：連線](./media/log-analytics-oms-gateway/nlb03.png) 
    
## <a name="configure-oms-agent-and-operations-manager-management-group"></a>設定 OMS 代理程式和 Operations Manager 管理群組
下一節包含的步驟會說明如何為直接連線的 OMS 代理程式、Operations Manager 管理群組或 Azure 自動化混合式 Runbook 背景工作設定 OMS 閘道以與 Azure 自動化或 Log Analytics 通訊。  

### <a name="configure-standalone-oms-agent"></a>設定獨立的 OMS 代理程式
若要了解如何在直接連線到 Log Analytics 的 Windows 電腦上安裝 OMS 代理程式的相關需求和步驟，請參閱[將 Windows 電腦連線到 Log Analytics](log-analytics-windows-agents.md)，若是 Linux 電腦，則請參閱[將 Linux 電腦連線到 Log Analytics](log-analytics-quick-collect-linux-computer.md)。 您不需在設定代理程式時指定 Proxy 伺服器，而是以 OMS 閘道伺服器的 IP 位址及其連接埠號碼取代該值。  如果您已在網路負載平衡器後方部署了多個閘道伺服器，OMS 代理程式 Proxy 組態會是 NLB 的虛擬 IP 位址。  

如需自動化混合式 Runbook 背景工作角色 的相關資訊，請參閱[部署混合式 Runbook 背景工作角色](../automation/automation-hybrid-runbook-worker.md)。

### <a name="configure-operations-manager---all-agents-use-the-same-proxy-server"></a>設定 Operations Manager - 所有代理程式使用相同的 Proxy 伺服器
您可以設定 Operations Manager 來新增閘道伺服器。  即使設定是空的，Operations Manager Proxy 組態仍會自動套用到所有向 Operations Manager 報告的代理程式。  

若要使用閘道來支援 Operations Manager，您必須具備下列條件：

* 在閘道伺服器上安裝 Microsoft Monitoring Agent (代理程式版本 - **8.0.10900.0** 或更新版本)，並針對您想要與之通訊的 Log Analytics 工作區進行設定。
* 閘道必須有網際網路連線，或連線到有網際網路連線的 Proxy 伺服器。

> [!NOTE]
> 如果未指定閘道的值，系統會將空白值推送到所有代理程式。
> 

如果這是 Operations Manager 管理群組第一次向 Log Analytics 工作區進行登錄，在 Operations 主控台中為管理群組指定 Proxy 設定的選項無法使用。  必須先向服務成功登錄該管理群組，之後此選項才可供使用。  您需要在執行 Operations 主控台的系統上使用 Netsh 更新系統 Proxy 設定，以在管理群組中設定整合，以及所有管理伺服器。  

1. 開啟提升權限的命令提示字元。
   a. 移至 [開始] 並輸入 **cmd**。
   b. 用滑鼠右鍵按一下 [命令提示字元]，然後選取 [以系統管理員身分執行]**。
1. 輸入下列命令並按 **Enter** 鍵：

    `netsh winhttp set proxy <proxy>:<port>`

完成與 Log Analytics 的整合之後，您可以透過執行 `netsh winhttp reset proxy` 來移除變更，然後使用 Operations 主控台中的 [設定 Proxy 伺服器] 選項來指定 OMS 閘道伺服器。 

1. 開啟 Operations Manager 主控台並在 [Operations Management Suite] 底下，按一下 [連線]，然後按一下 [設定 Proxy 伺服器]。<br><br> ![Operations Manager – 設定 Proxy 伺服器](./media/log-analytics-oms-gateway/scom01.png)<br> 
1. 選取 [使用 Proxy 伺服器來存取 Operations Management Suite]，然後輸入 OMS 閘道伺服器的 IP 位址或 NLB 的虛擬 IP 位址。 確定您已在開頭使用 `http://` 前置詞。<br><br> ![Operations Manager – Proxy 伺服器位址](./media/log-analytics-oms-gateway/scom02.png)<br> 
1. 按一下 [完成] 。 Operations Manager 管理群組現在已設定為透過閘道伺服器來與 Log Analytics 服務進行通訊。

### <a name="configure-operations-manager---specific-agents-use-proxy-server"></a>設定 Operations Manager - 特定代理程式使用 Proxy 伺服器
對於大型或複雜的環境，您可能只想要讓特定伺服器 (或群組) 使用 OMS 閘道伺服器。  對於這些伺服器，您無法直接更新 Operations Manager 代理程式，因為這個值會遭到管理群組的全域值所覆寫。  相反地，您需要覆寫用來推送這些值的規則。  

> [!NOTE] 
> 這個相同的設定技巧可用來允許您的環境使用多個 OMS 閘道伺服器。  例如，您可能需要以每個區域為基礎來指定特定的 OMS 閘道伺服器。
>  

1. 開啟 Operations Manager 主控台，然後選取 [撰寫] 工作區。  
1. 在 [撰寫] 工作區中選取 [規則]，然後按一下 Operations Manager 工具列上的 [範圍] 按鈕。 如果此按鈕無法使用，請進行檢查以確定您在 [監視] 窗格中選取的是物件，而非資料夾。 [範圍管理組件物件] 對話方塊會顯示一般會作為目標之類別、群組或物件的清單。 
1. 在 [尋找] 欄位中輸入**健康狀態服務**，然後從清單中加以選取。  按一下 [確定]。  
1. 搜尋規則 [Advisor Proxy 設定規則]，在 Operations 主控台工具列中按一下 [覆寫]，然後指向 [覆寫規則\針對下列類別的特定物件︰健康狀態服務]，並從清單中選取特定物件。  (選擇性) 您可以建立自訂群組，在其中包含您想要套用此覆寫之伺服器的健康狀態服務物件，然後對該群組套用覆寫。
1. 在 [覆寫屬性] 對話方塊中，按一下以勾選 [WebProxyAddress] 參數旁的 [覆寫] 資料行。  在 [覆寫值] 欄位中，輸入 OMS 閘道伺服器的 URL，並確保其開頭有 `http://` 前置詞。  

    >[!NOTE]
    > 您不需要啟用此規則，因為系統已自動使用覆寫來管理它，該覆寫包含在 Microsoft System Center Advisor 安全參考覆寫管理組件中，而此組件是以 Microsoft System Center Advisor 監控伺服器群組為目標。
    >   

1. 從 [選取目的地管理組件] 清單選取管理組件，或按一下 [新增] 來建立新的未密封管理組件。 
1. 當您變更完成時，按一下 [確定]。 

### <a name="configure-for-automation-hybrid-workers"></a>設定自動 Hybrid Worker
如果您的環境中有自動化混合式 Runbook 背景工作，下列步驟可提供手動、暫時性解決方式來設定閘道以支援它們。

在下列步驟中，您需要知道自動化帳戶所在的 Azure 區域。 找出位置：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
1. 選取 Azure 自動化服務。
1. 選取適當的 Azure 自動化帳戶。
1. 在 [位置] 底下檢視其區域。<br><br> ![Azure 入口網站 – 自動化帳戶位置](./media/log-analytics-oms-gateway/location.png)  

使用下列表格來識別每個位置的 URL：

**作業執行階段資料服務 URL**

| **位置** | **URL** |
| --- | --- |
| 美國中北部 |ncus-jobruntimedata-prod-su1.azure-automation.net |
| 西歐 |we-jobruntimedata-prod-su1.azure-automation.net |
| 美國中南部 |scus-jobruntimedata-prod-su1.azure-automation.net |
| 美國東部 2 |eus2-jobruntimedata-prod-su1.azure-automation.net |
| 加拿大中部 |cc-jobruntimedata-prod-su1.azure-automation.net |
| 北歐 |ne-jobruntimedata-prod-su1.azure-automation.net |
| 東南亞 |sea-jobruntimedata-prod-su1.azure-automation.net |
| 印度中部 |cid-jobruntimedata-prod-su1.azure-automation.net |
| 日本 |jpe-jobruntimedata-prod-su1.azure-automation.net |
| 澳大利亞 |ase-jobruntimedata-prod-su1.azure-automation.net |

**代理程式服務 URL**

| **位置** | **URL** |
| --- | --- |
| 美國中北部 |ncus-agentservice-prod-1.azure-automation.net |
| 西歐 |we-agentservice-prod-1.azure-automation.net |
| 美國中南部 |scus-agentservice-prod-1.azure-automation.net |
| 美國東部 2 |eus2-agentservice-prod-1.azure-automation.net |
| 加拿大中部 |cc-agentservice-prod-1.azure-automation.net |
| 北歐 |ne-agentservice-prod-1.azure-automation.net |
| 東南亞 |sea-agentservice-prod-1.azure-automation.net |
| 印度中部 |cid-agentservice-prod-1.azure-automation.net |
| 日本 |jpe-agentservice-prod-1.azure-automation.net |
| 澳大利亞 |ase-agentservice-prod-1.azure-automation.net |

如果您的電腦是使用更新管理方案自動註冊為混合式 Runbook 背景工作以進行修補，請遵循下列步驟：

1. 將作業執行階段資料服務 URL 新增到 OMS 閘道上的「允許的主機」清單。 例如：`Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. 使用以下 PowerShell Cmdlet 重新啟動 OMS 閘道服務：`Restart-Service OMSGatewayService`

如果您的電腦已使用混合式 Runbook 背景工作註冊 Cmdlet 加入 Azure 自動化，請遵循下列步驟：

1. 將代理程式服務註冊 URL 新增到 OMS 閘道上的「允許的主機」清單。 例如：`Add-OMSGatewayAllowedHost ncus-agentservice-prod-1.azure-automation.net`
1. 將作業執行階段資料服務 URL 新增到 OMS 閘道上的「允許的主機」清單。 例如：`Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. 重新啟動 OMS 閘道服務。
    `Restart-Service OMSGatewayService`

## <a name="useful-powershell-cmdlets"></a>PowerShell Cmdlet
Cmdlet 可以協助您完成更新 OMS 閘道的組態設定時需執行的作業。 在您使用它們之前，請務必先：

1. 安裝 OMS 閘道 (MSI)。
1. 開啟 PowerShell 主控台視窗。
1. 若要匯入模組，請輸入此命令：`Import-Module OMSGateway`
1. 如果上一個步驟沒有發生錯誤，就表示模組已經成功匯入，且可以使用 Cmdlet。 輸入 `Get-Module OMSGateway`
1. 在您使用 Cmdlet 進行變更之後，請確定您已重新啟動閘道服務。

如果您在步驟 3 發生錯誤，表示模組並未匯入。 當 PowerShell 找不到模組時，就可能會發生錯誤。 您可以在閘道的安裝路徑中找到它：*C:\Program Files\Microsoft OMS Gateway\PowerShell\OmsGateway*。

| **Cmdlet** | **參數** | **說明** | **範例** |
| --- | --- | --- | --- |  
| `Get-OMSGatewayConfig` |Key |取得服務的組態 |`Get-OMSGatewayConfig` |  
| `Set-OMSGatewayConfig` |索引鍵 (必要) <br> 值 |變更服務的組態 |`Set-OMSGatewayConfig -Name ListenPort -Value 8080` |  
| `Get-OMSGatewayRelayProxy` | |取得轉送 (上游) Proxy 的位址 |`Get-OMSGatewayRelayProxy` |  
| `Set-OMSGatewayRelayProxy` |位址<br> 使用者名稱<br> 密碼 |設定轉送 (上游) Proxy 的位址 (與認證) |1.設定回覆 Proxy 和認證：<br> `Set-OMSGatewayRelayProxy`<br>`-Address http://www.myproxy.com:8080`<br>`-Username user1 -Password 123` <br><br> 2.設定不需要驗證的回覆 Proxy：`Set-OMSGatewayRelayProxy`<br> `-Address http://www.myproxy.com:8080` <br><br> 3.清除轉送 Proxy 設定︰<br> `Set-OMSGatewayRelayProxy` <br> `-Address ""` |  
| `Get-OMSGatewayAllowedHost` | |取得目前允許的主機 (僅限本機設定的允許的主機，不包括自動下載的允許的主機) |`Get-OMSGatewayAllowedHost` | 
| `Add-OMSGatewayAllowedHost` |主機 (必要) |將主機加到允許清單 |`Add-OMSGatewayAllowedHost -Host www.test.com` |  
| `Remove-OMSGatewayAllowedHost` |主機 (必要) |將主機從允許清單移除 |`Remove-OMSGatewayAllowedHost`<br> `-Host www.test.com` |  
| `Add-OMSGatewayAllowedClientCertificate` |主體 (必要) |將用戶端憑證主體加到允許清單 |`Add-OMSGatewayAllowed`<br>`ClientCertificate` <br> `-Subject mycert` |  
| `Remove-OMSGatewayAllowedClientCertificate` |主體 (必要) |將用戶端憑證主體從允許清單移除 |`Remove-OMSGatewayAllowed` <br> `ClientCertificate` <br> `-Subject mycert` |  
| `Get-OMSGatewayAllowedClientCertificate` | |取得目前允許的用戶端憑證主體 (僅限本機設定的允許的主體，不包括自動下載的允許的主體) |`Get-`<br>`OMSGatewayAllowed`<br>`ClientCertificate` |  

## <a name="troubleshooting"></a>疑難排解
若要收集閘道所記錄的事件，您還需要安裝 OMS 代理程式。<br><br> ![事件檢視器 – OMS 閘道記錄](./media/log-analytics-oms-gateway/event-viewer.png)

**OMS 閘道事件識別碼與描述**

以下表格顯示 OMS 閘道記錄事件的事件識別碼與描述。

| **識別碼** | **說明** |
| --- | --- |
| 400 |沒有特定識別碼的任何應用程式錯誤 |
| 401 |錯誤組態。 例如：listenPort = "text"，而不是整數 |
| 402 |剖析 TLS 交握訊息時發生例外狀況 |
| 403 |網路錯誤。 例如：無法連線到目標伺服器 |
| 100 |一般資訊 |
| 101 |服務已經啟動 |
| 102 |服務已經停止 |
| 103 |已從用戶端接收到 HTTP CONNECT 命令 |
| 104 |不是 HTTP CONNECT 命令 |
| 105 |目的地伺服器不在允許的清單中，或目的地連接埠不是安全的連接埠 (443) <br> <br> 請確定您閘道伺服器上的 MMA 代理程式和與閘道通訊的代理程式，是連線到相同的 Log Analytics 工作區。 |
| 105 |ERROR TcpConnection – 無效的用戶端憑證: CN=Gateway <br><br> 請確定： <br>    <br> &#149; 您是使用版本號碼為 1.0.395.0 或更新版本的閘道。 <br> &#149; 您閘道伺服器上的 MMA 代理程式和與閘道通訊的代理程式，是連線到相同的 Log Analytics 工作區。 |
| 106 |OMS 閘道只支援 TLS 1.0、TLS 1.1 和 1.2。  不支援 SSL。 針對任何不支援的 TLS/SSL 通訊協定版本，OMS 閘道會產生事件識別碼 106。|
| 107 |已經驗證 TLS 工作階段 |

**要收集的效能計數器**

以下表格顯示可供 OMS 閘道使用的效能計數器。 您可以使用效能監視器新增計數器。

| **名稱** | **描述** |
| --- | --- |
| OMS 閘道/使用中用戶端連線 |使用中用戶端網路 (TCP) 連線數目 |
| OMS 閘道/錯誤計數 |錯誤數目 |
| OMS 閘道/已連線用戶端 |已連線用戶端數目 |
| OMS 閘道/拒絕計數 |因為任何 TLS 驗證錯誤而被拒絕的次數 |

![OMS 閘道效能計數器](./media/log-analytics-oms-gateway/counters.png)

## <a name="get-assistance"></a>取得協助
當您已經登入 Azure 入口網站時，您可以建立要求，來要求在 OMS 閘道或任何其他 Azure 服務或服務功能上提供協助。
若要要求協助，請按一下入口網站右上角的問號符號，然後按一下 [新增支援要求]。 然後，完成新的支援要求表單。

![新增支援要求](./media/log-analytics-oms-gateway/support.png)

## <a name="next-steps"></a>後續步驟
[新增資料來源](log-analytics-data-sources.md)會從 [連接的來源] 收集資料，並將它儲存在 Log Analytics 工作區中。
