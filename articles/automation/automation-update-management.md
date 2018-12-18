---
title: Azure 中的更新管理解決方案
description: 本文旨在協助您了解如何利用 Azure 更新管理解決方案來管理 Windows 和 Linux 電腦的更新。
services: automation
ms.service: automation
ms.component: update-management
author: georgewallace
ms.author: gwallace
ms.date: 08/29/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 62a7bb9bf63e8ebf97f9aeb5b08bf08ef06da43b
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43782785"
---
# <a name="update-management-solution-in-azure"></a>Azure 中的更新管理解決方案

「Azure 自動化」中的「更新管理」解決方案可讓您管理 Azure、內部部署環境或其他雲端提供者中部署之 Windows 和 Linux 電腦的作業系統更新。 您可以快速評估所有代理程式電腦上可用更新的狀態，並管理為伺服器安裝必要更新的程序。

您可以直接從您的「Azure 自動化」帳戶啟用虛擬機器的「更新管理」。 若要了解如何從您的自動化帳戶啟用虛擬機器的「更新管理」，請參閱[管理多部虛擬機器的更新](manage-update-multi.md)。 您也可以從 Azure 入口網站的虛擬機器窗格，啟用單一虛擬機器的「更新管理」。 此案例適用於 [Linux](../virtual-machines/linux/tutorial-monitoring.md#enable-update-management) 與 [Windows](../virtual-machines/windows/tutorial-monitoring.md#enable-update-management) 虛擬機器。

## <a name="solution-overview"></a>解決方案概觀

「更新管理」所管理的電腦會使用下列設定來執行評估和更新部署：

* 適用於 Windows 或 Linux 的 Microsoft Monitoring Agent (MMA)
* 適用於 Linux 的 PowerShell 預期狀態組態 (DSC)
* 自動化 Hybrid Runbook Worker
* 適用於 Windows 電腦的 Microsoft Update 或 Windows Server Update Services (WSUS)

下列圖表顯示行為和資料流程的概念性檢視，說明解決方案如何評估安全性更新並將其套用至工作區中所有連線的 Windows Server 和 Linux 電腦：

![更新管理程序流程](media/automation-update-management/update-mgmt-updateworkflow.png)

您可以使用更新管理在相同租用戶中的多個訂用帳戶中以原生方式上架機器。 若要管理不同租用戶上的機器，您必須將其上架為[非 Azure 機器](automation-onboard-solutions-from-automation-account.md#onboard-a-non-azure-machine)。

在電腦執行更新合規性掃描之後，代理程式會將大量資訊轉送至 Azure Log Analytics。 在 Windows 電腦上，合規性掃描預設會每 12 小時執行一次。

除了掃描排程，如果在更新安裝之前與更新安裝之後重新啟動 MMA，則會在 15 分鐘內起始更新合規性掃描。

針對 Linux 電腦，合規性掃描預設每 3 小時執行一次。 若 MMA 代理程式重新啟動，則會在 15 分鐘內起始合規性掃描。

解決方案會根據您設定要同步處理的來源，回報電腦的最新狀態。 如果 Windows 電腦設定為向 WSUS 回報，則視 WSUS 上次與 Microsoft Update 同步處理的時間而定，結果可能與 Microsoft Update 所顯示的結果不同。 針對設定為向本機存放庫 (而非公用存放庫) 回報的 Linux 電腦，同樣也是如此。

> [!NOTE]
> 為了能正確地向服務回報，「更新管理」需要啟用特定 URL 和連接埠。 若要深入了解這些需求，請參閱[混合式背景工作角色的網路規劃](automation-hybrid-runbook-worker.md#network-planning)。

您可以藉由建立排定的部署，在需要更新的電腦上部署和安裝軟體更新。 歸類為「選擇性」的更新不會包含在 Windows 電腦的部署範圍內。 部署範圍中僅包含必要更新。 

已排定的部署會透過明確指定電腦，或透過選取以一組特定電腦之記錄搜尋為基礎的[電腦群組](../log-analytics/log-analytics-computer-groups.md)，來定義哪些目標電腦會收到適用的更新。 您也可以指定核准排程，並指定允許安裝更新的一段時間。

在 Azure 自動化中，會由 Runbook 安裝更新。 您無法檢視這些 Runbook，而且這些 Runbook 也不需要任何設定。 建立更新部署之後，更新部署會建立排程，以便在指定的時間內，針對包含的電腦啟動主要更新 Runbook。 這個主要 Runbook 會在每個代理程式上啟動子 Runbook，以安裝必要更新。

到了在更新部署中指定的日期和時間時，目標電腦會以平行方式執行部署。 系統會先執行掃描，以確認更新仍然需要並加以安裝。 針對 WSUS 用戶端電腦，若更新並未在 WSUS 中核准，則更新部署會失敗。

## <a name="clients"></a>用戶端

### <a name="supported-client-types"></a>支援的用戶端類型

下表列出支援的作業系統：

|作業系統  |注意  |
|---------|---------|
|Windows Server 2008、Windows Server 2008 R2 RTM    | 僅支援更新評估。         |
|Windows Server 2008 R2 SP1 和更新版本     |必須要有 .NET Framework 4.5 或更新版本。 ([下載 .NET Framework](/dotnet/framework/install/guide-for-developers))<br/> 必須要有 Windows PowerShell 4.0 或更新的版本。 ([下載 WMF 4.0](https://www.microsoft.com/download/details.aspx?id=40855))<br/> 建議使用 Windows PowerShell 5.1 以增加可靠性。  ([下載 WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616))        |
|CentOS 6 (x86/x64) 和 7 (x64)      | Linux 代理程式必須能夠存取更新存放庫。 分類型修補需要 'yum' 來傳回 CentOS 未內建的安全性資料。         |
|Red Hat Enterprise 6 (x86/x64) 和 7 (x64)     | Linux 代理程式必須能夠存取更新存放庫。        |
|SUSE Linux Enterprise Server 11 (x86/x64) 和 12 (x64)     | Linux 代理程式必須能夠存取更新存放庫。        |
|Ubuntu 14.04 LTS 和 16.04 LTS (x86/x64)      |Linux 代理程式必須能夠存取更新存放庫。         |

### <a name="unsupported-client-types"></a>不支援的用戶端類型

下表列出不支援的作業系統：

|作業系統  |注意  |
|---------|---------|
|Windows 用戶端     | 不支援用戶端作業系統 (例如 Windows 7 和 Windows 10)。        |
|Windows Server 2016 Nano Server     | 不支援。       |

### <a name="client-requirements"></a>用戶端需求

#### <a name="windows"></a>Windows

Windows 代理程式必須設定為可與 WSUS 伺服器通訊，或必須能夠存取 Microsoft Update。 您可以搭配 System Center Configuration Manager 使用「更新管理」。 若要深入了解整合案例，請參閱[整合 System Center Configuration Manager 與更新管理](oms-solution-updatemgmt-sccmintegration.md#configuration)。 需要 [Windows 代理程式](../log-analytics/log-analytics-agent-windows.md)。 此代理程式會在您讓 Azure 虛擬機器上線時自動安裝。

#### <a name="linux"></a>Linux

針對 Linux，機器必須能夠存取更新存放庫。 更新存放庫可以是私人或公用。 必須使用 TLS 1.1 或 TLS 1.2，才能與更新管理互動。 此解決方案不支援適用於 Linux 且設定為向多個 Log Analytics 工作區回報的 Operations Management Suite (OMS) 代理程式。

如需關於如何安裝適用於 Linux 的 OMS 代理程式及下載最新版本的資訊，請參閱[適用於 Linux 的 Operations Management Suite 代理程式](https://github.com/microsoft/oms-agent-for-linux) \(英文\)。 如需有關如何安裝適用於 Windows 的 OMS 代理程式，請參閱[適用於 Windows 的 Operations Management Suite 代理程式](../log-analytics/log-analytics-windows-agent.md)。

## <a name="permissions"></a>權限

若要建立及管理更新部署，您必須具有特定權限。 若要了解這些權限，請參閱[角色型存取 - 更新管理](automation-role-based-access-control.md#update-management)。

## <a name="solution-components"></a>方案元件

解決方案包含下列資源。 資源會新增到您的自動化帳戶。 它們是直接連線的代理程式或是位於連線到 Operations Manager 的管理群組。

### <a name="hybrid-worker-groups"></a>混合式背景工作群組

啟用此解決方案之後，任何直接連線到 Log Analytics 工作區的 Windows 電腦都會自動設定為「混合式 Runbook 背景工作角色」，以支援此解決方案中所包含的 Runbook。

此解決方案管理的每部 Windows 電腦都會列在自動化帳戶的 [混合式背景工作角色群組] 窗格中，作為自動化帳戶的**系統混合式背景工作角色群組**。 解決方案會使用命名慣例 *Hostname FQDN_GUID*。 您不能讓這些群組以您帳戶中的 Runbook 為目標。 若您嘗試這樣做，它們會失敗。 這些群組只支援管理解決方案。

您可以將 Windows 電腦新增到自動化帳戶中的「混合式 Runbook 背景工作角色」群組來支援自動化 Runbook，只要解決方案和「混合式 Runbook 背景工作角色」群組成員資格兩者所用的帳戶相同即可。 此功能已新增至混合式 Runbook 背景工作角色 7.2.12024.0 版。

### <a name="management-packs"></a>管理組件

如果 System Center Operations Manager 管理群組已連線到 Log Analytics 工作區，則下列管理組件會安裝在 Operations Manager 中。 新增此解決方案之後，這些管理組件也會安裝在直接連線的 Windows 電腦上。 您不需要設定或管理這些管理組件。

* Microsoft System Center Advisor 更新評估智慧套件 (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* 更新部署 MP

如需有關方案管理組件如何更新的詳細資訊，請參閱[將 Operations Manager 連線到 Log Analytics](../log-analytics/log-analytics-om-agents.md)。

> [!NOTE]
> 就使用 Operations Manager 代理程式的系統而言，若要能夠完全受控於「更新管理」，則必須將代理程式更新為 Microsoft Monitoring Agent。 若要深入了解如何更新代理程式，請參閱[如何升級 Operations Manager 代理程式](https://docs.microsoft.com/system-center/scom/deploy-upgrade-agents)。

### <a name="confirm-that-non-azure-machines-are-onboarded"></a>確認非 Azure 機器已上線

若要確認直接連線的機器正與 Log Analytics 通訊，您可以在幾分鐘之後執行下列其中一個記錄搜尋。

#### <a name="linux"></a>Linux

```
Heartbeat
| where OSType == "Linux" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

#### <a name="windows"></a>Windows

```
Heartbeat
| where OSType == "Windows" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

在 Windows 電腦上，您可以檢閱下列資訊來確認與 Log Analytics 的代理程式連線能力︰

1. 在 [控制台] 中，開啟 [Microsoft Monitoring Agent]。 在 [Azure Log Analytics] 索引標籤中，代理程式會顯示以下訊息：**Microsoft Monitoring Agent 已成功連線至 Log Analytics**。
2. 開啟 Windows 事件記錄檔。 移至 [應用程式及服務記錄\Operations Manager] 並從來源 [服務連接器] 搜尋事件識別碼 3000 和事件識別碼 5002。 這些事件表示電腦已向 Log Analytics 工作區註冊，並且正在接收設定。

如果代理程式無法與 Log Analytics 通訊，而且代理程式已設定為透過防火牆或 Proxy 伺服器來與網際網路通訊，請確認防火牆或 Proxy 伺服器已正確設定。 若要了解如何確認防火牆或 proxy 伺服器已正確設定，請參閱 [Windows 代理程式的網路設定](../log-analytics/log-analytics-agent-windows.md)或 [Linux 代理程式的網路設定](../log-analytics/log-analytics-agent-linux.md)。

> [!NOTE]
> 如果您的 Linux 系統是設定為與 Proxy 或 OMS 閘道通訊，而且您要將此解決方案上線，請更新 *proxy.conf* 權限，並使用下列命令將檔案讀取權限授與 omiuser 群組：
>
> `sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf`
> `sudo chmod 644 /etc/opt/microsoft/omsagent/proxy.conf`

在執行評估之後，新增的 Linux 代理程式狀態會顯示為「已更新」。 此程序可能需要多達 6 小時的時間。

若要確認 Operations Manager 管理群組正在與 Log Analytics 通訊，請參閱[驗證 Operations Manager 與 Log Analytics 的整合](../log-analytics/log-analytics-om-agents.md#validate-operations-manager-integration-with-log-analytics)。

## <a name="data-collection"></a>資料收集

### <a name="supported-agents"></a>支援的代理程式

下表描述此方案支援的連線來源：

| 連線的來源 | 支援 | 說明 |
| --- | --- | --- |
| Windows 代理程式 |是 |解決方案會從 Windows 代理程式收集系統更新的相關資訊，然後起始必要更新的安裝。 |
| Linux 代理程式 |是 |解決方案會從 Linux 代理程式收集系統更新的相關資訊，然後在支援的發行版本上起始必要更新的安裝。 |
| Operations Manager 管理群組 |是 |方案會從所連線之管理群組中的代理程式收集系統更新的相關資訊。<br/>Operations Manager 代理程式不需要直接連線到 Log Analytics。 資料會從管理群組轉送至 Log Analytics 工作區。 |

### <a name="collection-frequency"></a>收集頻率

針對每部受控 Windows 電腦，每天會掃描兩次。 系統會每隔 15 分鐘呼叫一次 Windows API 來查詢上次更新時間，以判斷狀態是否已變更。 如果狀態已變更，則會起始合規性掃描。 

針對每部受控 Linux 電腦，每 3 小時會掃描一次。

儀表板可能需要 30 分鐘到 6 小時，才能顯示來自受控電腦的已更新資料。

## <a name="viewing-update-assessments"></a>檢視更新評估

在您的自動化帳戶中，選取 [更新管理] 來檢視機器的狀態。

此檢視會提供您的機器、缺少的更新、更新部署以及已排定之更新部署的相關資訊。 在 [合規性欄] 中，您可以看到機器上次的評估時間。 在 [更新代理程式整備程度] 欄中，您可以查看更新代理的健康情況。 如果發生問題，請選取連結來移至疑難排解文件，協助您了解應該執行哪些步驟來更正問題。

若要執行記錄搜尋來傳回與機器、更新或部署相關的資訊，請選取清單中的項目。 [記錄搜尋] 窗格隨即開啟，並顯示所選項目的查詢：

![更新管理的預設檢視](media/automation-update-management/update-management-view.png)

## <a name="install-updates"></a>安裝更新

工作區中的所有 Linux 和 Windows 電腦皆進行過更新評估之後，您可以建立「更新部署」來安裝必要的更新。 更新部署是為一或多部電腦排定的必要更新安裝作業。 您應該指定部署的日期和時間，以及應該包含在部署範圍中的電腦或電腦群組。 若要深入瞭解電腦群組，請參閱 [Log Analytics 中的電腦群組](../log-analytics/log-analytics-computer-groups.md)。

 當您將電腦群組納入更新部署時，只會在建立排程時評估一次群組成員資格。 系統不會反映群組的後續變更。 若要因應這個問題，請刪除已排定的更新部署並重新建立。

> [!NOTE]
> 依預設，從 Azure Marketplace 部署的 Windows 虛擬機器會設定為從 Windows Update 服務接收自動更新。 當您新增這個解決方案或將 Windows 虛擬機器新增至您的工作區時，此行為並不會變更。 如果您未使用這個解決方案來主動管理更新，則會套用預設行為 (自動套用更新)。

若要避免在 Ubuntu 維護時間範圍以外套用更新，請重新設定自動安裝升級套件以停用自動更新。 如需有關如何設定套件的資訊，請參閱 [Ubuntu Server 指南中的自動更新主題](https://help.ubuntu.com/lts/serverguide/automatic-updates.html) \(英文\)。

需註冊從隨選 Red Hat Enterprise Linux (RHEL) 映像 (可在 Azure Marketplace 中找到) 建立的虛擬機器，以存取部署在 Azure 中的 [Red Hat Update Infrastructure (RHUI)](../virtual-machines/virtual-machines-linux-update-infrastructure-redhat.md)。 針對任何其他 Linux 發行版本，則必須從發行版本線上檔案存放庫，依照其支援的方法來更新。

## <a name="view-missing-updates"></a>檢視缺少的更新

選取 [缺少的更新] 以檢視機器缺少的更新清單。 會列出每個更新，而且您可以選取更新。 畫面上會顯示需要更新的機器數目、作業系統以及一個能提供詳細資訊的連結。 [記錄搜尋] 窗格會顯示與更新有關的詳細資訊。

## <a name="view-update-deployments"></a>檢視更新部署

選取 [更新部署] 索引標籤以檢視現有更新部署的清單。 選取表格中的任一個更新部署以開啟該更新部署的 [更新部署執行] 窗格。

![更新部署結果的概觀](./media/automation-update-management/update-deployment-run.png)

## <a name="create-or-edit-an-update-deployment"></a>建立或編輯更新部署

若要建立新的更新部署，請選取 [排程更新部署]。 [新增更新部署] 窗格隨即開啟。 為下表描述的屬性輸入相關的值，然後按一下 [建立]：

| 屬性 | 說明 |
| --- | --- |
| 名稱 |用以識別更新部署的唯一名稱。 |
|作業系統| Linux 或 Windows|
| 要更新的機器 |選取已儲存的搜尋、已匯入的群組，或從下拉式清單中選擇 [機器]，然後選取個別的機器。 如果您選擇 [機器]，機器的整備程度會顯示於 [更新代理程式整備程度] 欄中。</br> 若要深入了解在 Log Analytics 中建立電腦群組的不同方法，請參閱 [Log Analytics 中的電腦群組](../log-analytics/log-analytics-computer-groups.md) |
|更新分類|選取您需要的所有更新分類|
|要排除的更新|輸入要排除的更新。 針對 Windows，輸入不含 'KB' 前置詞的 KB。 針對 Linux，輸入套件名稱或使用萬用字元。  |
|排程設定|選取開始時間，並選取 [一次] 或 [週期性] 以定期執行|
| 維護時間範圍 |為更新設定的分鐘數。 此值不可小於 30 分鐘，且不可超過 6 小時 |
| 重新開機控制| 決定應該如何處理重新開機。 可用選項包括：</br>在必要時重新開機 (預設值)</br>一律重新開機</br>永不重新開機</br>僅重新開機 - 將不會安裝更新|

## <a name="update-classifications"></a>更新分類

下表列出「更新管理」中的更新分類清單，以及每個分類的定義。

### <a name="windows"></a>Windows

|分類  |說明  |
|---------|---------|
|重大更新     | 特定問題的更新，負責處理與安全性無關的重大錯誤。        |
|安全性更新     | 特定產品的安全性相關更新。        |
|更新彙總套件     | 一組累計的 Hotfix，封裝在一起以便於部署。        |
|Feature Pack     | 在產品版本之外散發的新產品功能。        |
|Service Pack     | 一組套用到應用程式的累計 Hotfix。        |
|定義更新     | 病毒或其他定義檔案的更新。        |
|工具     | 有助於完成一或多個工作的公用程式或功能。        |
|更新     | 目前安裝之應用程式或檔案的更新。        |

### <a name="linux"></a>Linux

|分類  |說明  |
|---------|---------|
|重大更新和安全性更新     | 特定問題或特定產品的安全性相關問題的更新，         |
|其他更新     | 在性質上不重要或非安全性更新的所有其他更新。        |

針對 Linux，「更新管理」可以區分雲端中的重大更新和安全性更新，同時由於雲端中的資料擴充而顯示評估資料。 針對修補，「更新管理」仰賴機器上可用的分類資料。 與其他發行版本不同，CentOS 預設沒有此資訊。 如果您以某種方式將 CentOS 機器設定成傳回以下命令的安全性資料，則「更新管理」將能夠根據分類來進行修補。

```bash
sudo yum -q --security check-update
```

目前沒有支援的方法可以在 CentOS 上啟用原生分類資料可用性。 目前，只為那些已自行啟用此功能的客戶，提供盡力而為的支援。

## <a name="ports"></a>連接埠

以下為「更新管理」特別需求的位址。 與這些位址的通訊皆經由連接埠 443 進行。

|Azure 公用  |Azure Government  |
|---------|---------|
|*.ods.opinsights.azure.com     |*.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|*.blob.core.windows.net|*.blob.core.usgovcloudapi.net|

如需混合式 Runbook 背景工作角色所需連接埠的詳細資訊，請參閱[混合式背景工作角色連接埠](automation-hybrid-runbook-worker.md#hybrid-worker-role)。

建議使用定義例外狀況時所列出的位址。 針對 IP 位址，您可以下載 [Microsoft Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。 此檔案會每週更新，並反映目前已部署的範圍及任何即將進行的 IP 範圍變更。

## <a name="search-logs"></a>搜尋記錄

除了 Azure 入口網站中提供的詳細資料以外，您也可以對記錄執行搜尋。 在解決方案頁面上，選取 [Log Analytics]。 [記錄搜尋] 窗格隨即開啟。

請參閱 [Log Analytics 搜尋 API 文件](
https://dev.loganalytics.io/)，了解如何自訂查詢或從不同的用戶端使用它們。

### <a name="sample-queries"></a>範例查詢

以下各節為這個解決方案收集的更新記錄，提供範例記錄查詢：

#### <a name="single-azure-vm-assessment-queries-windows"></a>單一 Azure VM 評估查詢 (Windows)

將 VMUUID 值取代為您目前查詢之虛擬機器的 VM GUID。 您可以在 Log Analytics 中執行以下查詢來尋找應該使用的 VMUUID：`Update | where Computer == "<machine name>" | summarize by Computer, VMUUID`

##### <a name="missing-updates-summary"></a>缺少的更新摘要

```
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"b08d5afa-1471-4b52-bd95-a44fea6e4ca8"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security"
```

##### <a name="missing-updates-list"></a>缺少的更新清單

```
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"8bf1ccc6-b6d3-4a0b-a643-23f346dfdf82"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

#### <a name="single-azure-vm-assessment-queries-linux"></a>單一 Azure VM 評估查詢 (Linux)

針對某些 Linux 發行版本，[位元組順序](https://en.wikipedia.org/wiki/Endianness)與 Azure Resource Manager 的 VMUUID 值以及 Log Analytics 中儲存的內容不相符。 以下查詢會檢查這上述兩者的位元組順序是否相符。 將 VMUUID 值取代為 GUID 的位元組由大到小和位元組由小到大格式，以正確地傳回結果。 您可以在 Log Analytics 中執行以下查詢來尋找應該使用的 VMUUID：`Update | where Computer == "<machine name>"
| summarize by Computer, VMUUID`

##### <a name="missing-updates-summary"></a>缺少的更新摘要

```
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

##### <a name="missing-updates-list"></a>缺少的更新清單

```
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl

```

#### <a name="multi-vm-assessment-queries"></a>多部 VM 評估查詢

##### <a name="computers-summary"></a>電腦摘要

```
Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Approved, Optional, Classification) by SourceComputerId, UpdateID
    | distinct SourceComputerId, Classification, UpdateState, Approved, Optional
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed" and (Optional==false or Classification has "Critical" or Classification has "Security") and Approved!=false, iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity
| union (Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by SourceComputerId, Product, ProductArch
    | distinct SourceComputerId, Classification, UpdateState
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed", iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity)
| summarize assessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity>-1), notAssessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==-1), computersNeedCriticalUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==4), computersNeedSecurityUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==2), computersNeeedOtherUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==1), upToDateComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==0)
| summarize assessedComputersCount=sum(assessedComputersCount), computersNeedCriticalUpdatesCount=sum(computersNeedCriticalUpdatesCount),  computersNeedSecurityUpdatesCount=sum(computersNeedSecurityUpdatesCount), computersNeeedOtherUpdatesCount=sum(computersNeeedOtherUpdatesCount), upToDateComputersCount=sum(upToDateComputersCount), notAssessedComputersCount=sum(notAssessedComputersCount)
| extend allComputersCount=assessedComputersCount+notAssessedComputersCount


```

##### <a name="missing-updates-summary"></a>缺少的更新摘要

```
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| union (Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification )
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security"
```

##### <a name="computers-list"></a>電腦清單

```
Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Product, Computer, ComputerEnvironment) by SourceComputerId, Product, ProductArch
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed"), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed"), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed"), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2)
| union(Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, Optional, Approved, Computer, ComputerEnvironment) by Computer, SourceComputerId, UpdateID
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed" and Approved!=false), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed" and Approved!=false), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed" and Optional==false and Approved!=false), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2) )
| order by ComplianceOrder asc, missingCriticalUpdatesCount desc, missingSecurityUpdatesCount desc, missingOtherUpdatesCount desc, displayName asc
| project-away ComplianceOrder
```

##### <a name="missing-updates-list"></a>缺少的更新清單

```
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| union(Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2)
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

## <a name="integrate-with-system-center-configuration-manager"></a>與 System Center Configuration Manager 進行整合

投資了 System Center Configuration Manager 來管理電腦、伺服器和行動裝置的客戶也需仰賴 Configuration Manager 的強度和成熟度，以協助他們管理軟體更新。 Configuration Manager 是其軟體更新管理 (SUM) 週期的一部分。

若要了解如何將管理解決方案與 Sytem Center Configuration Manager 整合，請參閱[將 System Center Configuration Manager 與更新管理整合](oms-solution-updatemgmt-sccmintegration.md)。

## <a name="patch-linux-machines"></a>修補 Linux 機器

以下各節說明使用 Linux 修補的潛在問題。

### <a name="unexpected-os-level-upgrades"></a>未預期的 OS 層級升級

在某些 Linux 版本上 (例如 Red Hat Enterprise Linux)，安裝特定套件可能會造成 OS 層級升級。 這可能會導致「更新管理」在 OS 版本號碼變更時執行。 因為「更新管理」會使用與系統管理員在本機 Linux 電腦上相同的方式來更新套件，因此這個行為是刻意的。

若要避免透過「更新管理」執行來更新 OS 版本，請使用**排除**功能。

在 Red Hat Enterprise Linux 中，要排除的套件名稱將是：redhat-release-server.x86_64。

![要為 Linux 排除的套件](./media/automation-update-management/linuxpatches.png)

### <a name="critical--security-patches-arent-applied"></a>未套用重大/安全性修補程式

當您將更新部署至 Linux 機器時，可以選取更新分類。 這樣會篩選出要對符合指定準則的項目套用的更新。 此篩選會在更新部署時套用至本機電腦上。

由於「更新管理」會在雲端中執行更新擴充，因此有些更新可能會在「更新管理」中標示為有安全性影響，即使本機電腦並沒有該資訊。 因此，如果您將重大更新套用至 Linux 機器，某些未標示為在該機器上有安全性影響的更新，可能不會被套用。

不過，「更新管理」仍可能將該機器回報為不符合規範的機器，因為它對於該相關更新還有其他資訊。

CentOS 預設不支援依更新分類來部署更新。 針對 SUSE，只選取 [其他更新] 作為分類時，如果首先要的是與 zypper (套件管理員) 有關的安全性更新或其相依性時，有可能也會安裝某些安全性更新。 這是 zypper 的限制。 在某些情況下，您可能需要重新執行更新部署，以驗證更新記錄檔。

## <a name="troubleshoot"></a>疑難排解

若要了解如何針對「更新管理」問題進行疑難排解，請參閱[更新管理疑難排解](troubleshoot/update-management.md)

## <a name="next-steps"></a>後續步驟

繼續進行本教學課程，以了解如何管理 Windows 虛擬機器的更新。

> [!div class="nextstepaction"]
> [管理 Azure Windows VM 的更新和修補程式](automation-tutorial-update-management.md)

* 使用 [Log Analytics](../log-analytics/log-analytics-log-searches.md) 中的記錄搜尋，檢視詳細的更新資料。
* 在偵測到電腦缺少重大更新或電腦已停用自動更新時[建立警示](../log-analytics/log-analytics-alerts.md)。
