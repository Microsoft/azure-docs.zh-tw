---
title: Azure 自動化 Windows 混合式 Runbook 背景工作
description: 本文提供有關安裝「Azure 自動化混合式 Runbook 背景工作角色」的資訊，您可以使用它在本機資料中心或雲端環境中的 Windows 型電腦上執行 Runbook。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/17/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: c94fd0bb68a03c64935c20fef1fab7b68c9c2c9b
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032983"
---
# <a name="deploy-a-windows-hybrid-runbook-worker"></a>部署 Windows 混合式 Runbook 背景工作角色

您可以使用 Azure 自動化的混合式 Runbook 背景工作角色功能，直接在裝載角色的電腦上以及針對環境中的資源執行 Runbook，從而管理這些本機資源。 Runbook 會儲存並在 Azure 自動化中管理，接著傳遞至一或多個指定的電腦。 本文說明如何在 Windows 機器上安裝「混合式 Runbook 背景工作角色」。

## <a name="installing-the-windows-hybrid-runbook-worker"></a>安裝 Windows 混合式 Runbook 背景工作角色

若要安裝及設定 Windows 混合式 Runbook 背景工作角色，您可以使用兩種方法。 建議的方法是使用自動化 Runbook，將設定 Windows 電腦的程序完全自動化。 第二種方法採取逐步程序來手動安裝和設定角色。

> [!NOTE]
> 若要使用 Desired State Configuration (DSC) 來管理支援混合式 Runbook 背景工作角色的伺服器設定，您必須將它們新增為 DSC 節點。

「Windows 混合式 Runbook 背景工作角色」的最低需求如下：

* Windows Server 2012 或更新版本。
* Windows PowerShell 5.1 或更新版本 ([下載 WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616))。
* .NET Framework 4.6.2 或更新版本。
* 雙核心。
* 4 GB 的 RAM。
* 連接埠 443 (輸出)。

若要了解「混合式 Runbook 背景工作角色」的其他網路需求，請參閱[設定網路](automation-hybrid-runbook-worker.md#network-planning)。

如需有關讓伺服器上線以透過 DSC 進行管理的詳細資訊，請參閱[讓機器上線以透過 Azure 自動化 DSC 進行管理](automation-dsc-onboarding.md)。
如果您啟用[更新管理解決方案](../operations-management-suite/oms-solution-update-management.md)，則任何連線到 Azure Log Analytics 工作區的 Windows 電腦都會自動設定為「混合式 Runbook 背景工作角色」，以支援此解決方案所包含的 Runbook。 不過，它不會向已在您「自動化」帳戶中定義的任何「混合式背景工作角色」群組註冊。 

您可以將電腦新增至您自動化帳戶中的混合式 Runbook 背景工作角色群組來支援自動化 Runbook，只要解決方案和混合式 Runbook 背景工作角色群組成員資格兩者使用相同的帳戶即可。 此功能已新增至 Hybrid Runbook Worker 7.2.12024.0 版。

在您成功部署 Runbook 背景工作角色後，請檢閱[在混合式 Runbook 背景工作角色上執行 Runbook](automation-hrw-run-runbooks.md)，以了解如何設定 Runbook 以將內部部署資料中心或其他雲端環境中的程序自動化。

### <a name="automated-deployment"></a>自動化部署

執行下列步驟，以將「Windows 混合式背景工作角色」的安裝和設定自動化：

1. 直接從執行「混合式 Runbook 背景工作角色」的電腦，或從您環境中的另一部電腦，從[PowerShell 資源庫](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker)下載 New-OnPremiseHybridWorker.ps1 指令碼。 將指令碼複製到背景工作角色。

   New-OnPremiseHybridWorker.ps1 指令碼在執行期間需要下列參數：

   * *AutomationAccountName* (必要)：您的「自動化」帳戶名稱。
   * *AAResourceGroupName* (必要)：與您的自動化帳戶相關聯的資源群組名稱。
   * *OMSResourceGroupName* (選擇性)：Log Analytics 工作區的資源群組名稱。 若未指定此資源群組，將會使用 *AAResourceGroupName*。
   * *HybridGroupName* (必要)：您針對支援此案例的 Runbook，指定作為目標的「混合式 Runbook 背景工作角色」群組名稱。
   * *SubscriptionID* (必要)：您「自動化」帳戶所在的 Azure 訂用帳戶 ID。
   * *WorkspaceName* (選擇性)：Log Analytics 工作區名稱。 如果您沒有 Log Analytics 工作區，此指令碼就會建立並設定一個 Log Analytics 工作區。

     > [!NOTE]
     > 目前，支援與 Log Analytics 整合的「自動化」區域只有**澳大利亞東南部**、**美國東部 2**、**東南亞**和**西歐**。 如果您的「自動化」帳戶不在上述其中一個區域，指令碼會建立 Log Analytics 工作區，但會警告您，指出無法將它們連結在一起。

2. 在您的電腦上，從 [開始] 畫面以系統管理員模式開啟 **Windows PowerShell**。
3. 從 PowerShell 命令列殼層，瀏覽至包含您所下載指令碼的資料夾。 變更參數 *-AutomationAccountName*、*-AAResourceGroupName*、*-OMSResourceGroupName*、*-HybridGroupName*、*-SubscriptionId* 及 *-WorkspaceName* 的值。 然後執行指令碼。

     > [!NOTE]
     > 執行指令碼之後，系統會提示您向 Azure 進行驗證。 您「必須」以「訂用帳戶管理員」角色成員和訂用帳戶共同管理員的帳戶登入。

   ```powershell-interactive
   .\New-OnPremiseHybridWorker.ps1 -AutomationAccountName <NameofAutomationAccount> -AAResourceGroupName <NameofResourceGroup>`
   -OMSResourceGroupName <NameofOResourceGroup> -HybridGroupName <NameofHRWGroup> `
   -SubscriptionId <AzureSubscriptionId> -WorkspaceName <NameOfLogAnalyticsWorkspace>
   ```

4. 系統會提示您同意安裝 NuGet，也會提示使用您的 Azure 認證進行驗證。

5. 指令碼執行完成之後，[混合式背景工作角色群組] 頁面會顯示新的群組和成員數目。 如果這是現有群組，成員數目就會遞增。 您可以從 [Hybrid Worker 群組] 頁面上的清單中選取群組，然後選取 [Hybrid Worker] 圖格。 在 [Hybrid Worker] 頁面上，您會看到列出群組的每個成員。

### <a name="manual-deployment"></a>手動部署

請對您的「自動化」環境執行一次前兩個步驟，再對每一台背景工作角色電腦重複其餘步驟。

#### <a name="1-create-a-log-analytics-workspace"></a>1.建立 Log Analytics 工作區

如果您還沒有 Log Analytics 工作區，請使用[管理您的工作區](../log-analytics/log-analytics-manage-access.md)中的指示來建立。 如果您已經有工作區，可以使用現有的工作區。

#### <a name="2-add-the-automation-solution-to-the-log-analytics-workspace"></a>2.將自動化解決方案新增至 Log Analytics 工作區

方案會將功能加入 Log Analytics。 「自動化」解決方案會增加「Azure 自動化」的功能，包括對「混合式 Runbook 背景工作角色」的支援。 將解決方案新增至工作區時，它會自動將背景工作角色元件推送給您在下一步將安裝的代理程式電腦。

若要將**自動化**解決方案新增至 Log Analytics 工作區，請依照[使用方案庫新增解決方案](../log-analytics/log-analytics-add-solutions.md)中的指示進行操作。

#### <a name="3-install-the-microsoft-monitoring-agent"></a>3.安裝 Microsoft Monitoring Agent

Microsoft Monitoring Agent 可將電腦連線至 Log Analytics。 將代理程式安裝在內部部署電腦上並連線到您的工作區時，它會自動下載「混合式 Runbook 背景工作角色」所需的元件。

若要將代理程式安裝在內部部署電腦上，請依照[將 Windows 電腦連線到 Log Analytics](../log-analytics/log-analytics-windows-agent.md) 中的指示進行操作。 您可以對多部電腦重複此程序，將多個背景工作角色加入至您的環境。

當代理程式成功連線到 Log Analytics 時，它會列在 Log Analytics [設定] 頁面的 [連接的來源] 索引標籤上。 當 C:\Program Files\Microsoft Monitoring Agent\Agent 中出現 **AzureAutomationFiles** 資料夾時，就可確認代理程式已正確下載自動化解決方案。 若要確認「混合式 Runbook 背景工作角色」版本，您可以瀏覽至 C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\，並記下 \\version 子資料夾。

#### <a name="4-install-the-runbook-environment-and-connect-to-azure-automation"></a>4.安裝 Runbook 環境並連接到 Azure 自動化

將代理程式新增至 Log Analytics 時，自動化解決方案會往下推送包含 **Add-HybridRunbookWorker** Cmdlet 的 **HybridRegistration** PowerShell 模組。 您可以使用這個 Cmdlet 在電腦上安裝 Runbook 環境並向 Azure 自動化進行註冊。

以系統管理員模式開啟 PowerShell 工作階段，然後執行下列命令來匯入模組：

```powershell-interactive
cd "C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\<version>\HybridRegistration"
Import-Module HybridRegistration.psd1
```

接著，使用下列語法來執行 **Add-HybridRunbookWorker** Cmdlet：

```powershell-interactive
Add-HybridRunbookWorker –GroupName <String> -EndPoint <Url> -Token <String>
```

您可以從 Azure 入口網站的 [管理金鑰] 頁面取得這個 Cmdlet 所需的資訊。 從您「自動化」帳戶的 [設定] 頁面中選取 [金鑰]，以開啟此頁面。

![[管理金鑰] 頁面](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

* **GroupName** 是「混合式 Runbook 背景工作角色」群組的名稱。 如果「自動化」帳戶中已經有這個群組，系統就會將目前的電腦新增至其中。 如果此群組不存在，則會新增此群組。
* **EndPoint** 是 [管理金鑰] 頁面上的 [URL] 項目。
* **Token** 是 [管理金鑰] 頁面上的 [主要存取金鑰] 項目。

若要收到有關安裝的詳細資訊，請搭配 **Add-HybridRunbookWorker** 使用 **-Verbose** 參數。

#### <a name="5-install-powershell-modules"></a>5.安裝 PowerShell 模組

Runbook 可以使用「Azure 自動化」環境內所安裝模組中定義的任何活動和 Cmdlet。 這些模組不會自動部署到內部部署電腦，因此您必須手動安裝它們。 但 Azure 模組例外，此模組是預設安裝的模組，可讓您存取「Azure 自動化」之所有 Azure 服務和活動的 Cmdlet。

由於「混合式 Runbook 背景工作角色」功能的主要目的是要管理本機資源，因此您很可能必須安裝支援這些資源的模組。 如需有關安裝 Windows PowerShell 模組的詳細資訊，請參閱[安裝模組](http://msdn.microsoft.com/library/dd878350.aspx)。 

安裝的模組必須位於 **PSModulePath** 環境變數所參考的位置中，如此混合式背景工作角色才能自動匯入它們。 如需詳細資訊，請參閱[修改 PSModulePath 安裝路徑](https://msdn.microsoft.com/library/dd878326%28v=vs.85%29.aspx)。

## <a name="troubleshoot"></a>疑難排解

若要了解如何針對「混合式 Runbook 背景工作角色」進行疑難排解，請參閱[針對 Windows 混合式 Runbook 背景工作角色進行疑難排解](troubleshoot/hybrid-runbook-worker.md#windows)

如需如何針對更新管理的問題進行疑難排解的其他步驟，請參閱[更新管理：疑難排解](troubleshoot/update-management.md)。

## <a name="next-steps"></a>後續步驟

* 若要了解如何設定您的 Runbook，將您在內部部署資料中心或其他雲端環境中的程序自動化，請參閱[在混合式 Runbook 背景工作角色上執行 Runbook](automation-hrw-run-runbooks.md)。
* 如需移除混合式 Runbook 背景工作角色的指示，請參閱[移除 Azure 自動化混合式 Runbook 背景工作角色](automation-hybrid-runbook-worker.md#remove-a-hybrid-runbook-worker)。
