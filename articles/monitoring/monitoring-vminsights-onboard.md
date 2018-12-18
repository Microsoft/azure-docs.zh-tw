---
title: 將適用於 VM 的 Azure 監視器上線 | Microsoft Docs
description: 本文說明您如何將適用於 VM 的 Azure 監視器上線並加以設定，讓您可以開始了解分散式應用程式的執行方式，以及已識別出哪些健康情況問題。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: magoedte
ms.openlocfilehash: 2f0568064eed556429675ffb34c84d588ac670d5
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064351"
---
# <a name="how-to-onboard-the-azure-monitor-for-vms"></a>如何將適用於 VM 的 Azure 監視器上線 
本文說明如何設定適用於 VM 的 Azure 監視器來監視 Azure 虛擬機器的作業系統健康情況，以及探索和對應可能裝載於其上的應用程式相依性。  

啟用適用於 VM 的 Azure 監視器可透過下列其中一種方法來完成，本文稍後將會提供使用每種方法的詳細資料。  

* 直接從 VM 選取 [Insights (預覽)] 的單一 Azure 虛擬機器。
* 使用 Azure 原則的多部 Azure VM，以確保所評估的現有及新 VM 均已安裝必要的相依性且已正確設定。  系統會報告不符合規範的 VM，讓您可以根據不符合規範的情況來決定該如何修復。  
* 使用 PowerShell 跨指定訂用帳戶或資源群組的多個 Azure VM 或虛擬機器擴展集。

## <a name="prerequisites"></a>必要條件
開始之前，請確定您具有下列各小節所述的項目。

### <a name="log-analytics"></a>Log Analytics 

目前支援下列區域中的 Log Analytics 工作區：

  - 美國中西部  
  - 美國東部  
  - 西歐  
  - 東南亞<sup>1</sup>  

<sup>1</sup> 此區域目前不支援適用於 VM 的 Azure 監視器的健康情況功能   

如果您沒有工作區，則可透過 [Azure Resource Manager](../log-analytics/log-analytics-template-workspace-configuration.md)、透過 [PowerShell](https://docs.microsoft.com/azure/log-analytics/scripts/log-analytics-powershell-sample-create-workspace?toc=%2fpowershell%2fmodule%2ftoc.json)，或是在 [Azure 入口網站](../log-analytics/log-analytics-quick-create-workspace.md)中建立它。  

若要啟用解決方案，您必須是 Log Analytics 參與者角色的成員。 如需有關如何控制 Log Analytics 工作區存取的詳細資訊，請參閱[管理工作區](../log-analytics/log-analytics-manage-access.md)。

[!INCLUDE [log-analytics-agent-note](../../includes/log-analytics-agent-note.md)]

### <a name="supported-operating-systems"></a>受支援的作業系統

下表列出適用於 VM 的 Azure 監視器支援的 Windows 和 Linux 作業系統。  這一節稍後會提供詳列支援的主要和次要 Linux OS 版本及核心版本的完整清單。

|作業系統版本 |效能 |地圖 |健康情況 |  
|-----------|------------|-----|-------|  
|Windows Server 2016 1803 | X | X | X |
|Windows Server 2016 | X | X | X |  
|Windows Server 2012 R2 | X | X | |  
|Windows Server 2012 | X | X | |  
|Windows Server 2008 R2 | X | X| |  
|RHEL 7、6| X | X| X |  
|Ubuntu 18.04、16.04、14.04 | X | X | X |  
|Cent OS Linux 7、6 | X | X | X |  
|SLES 12 | X | X | X |  
|Oracle Linux 7 | X<sup>1</sup> | | X |  
|Oracle Linux 6 | X | X | X |  
|Debian 9.4、8 | X<sup>1</sup> | | X | 

<sup>1</sup> 適用於 VM 的 Azure 監視器的效能功能只可從 Azure 監視器取得，當您從 Azure VM 的左側窗格直接存取它時則不適用。  

>[!NOTE]
>下列資訊適用於 Linux 作業系統的支援：  
> - 只支援預設版本和 SMP Linux 核心版本。  
> - 所有 Linux 散發套件皆不支援非標準的核心版本 (例如 PAE 和 Xen)。 舉例來說，版本字串為「2.6.16.21-0.8-xen」的系統就不受支援。  
> - 不支援自訂核心，包括重新編譯的標準核心。  
> - 不支援 CentOSPlus 核心。  


#### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| 作業系統版本 | 核心版本 |
|:--|:--|
| 7.0 | 3.10.0-123 |
| 7.1 | 3.10.0-229 |
| 7.2 | 3.10.0-327 |
| 7.3 | 3.10.0-514 |
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |

#### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| 作業系統版本 | 核心版本 |
|:--|:--|
| 6.0 | 2.6.32-71 |
| 6.1 | 2.6.32-131 |
| 6.2 | 2.6.32-220 |
| 6.3 | 2.6.32-279 |
| 6.4 | 2.6.32-358 |
| 6.5 | 2.6.32-431 |
| 6.6 | 2.6.32-504 |
| 6.7 | 2.6.32-573 |
| 6.8 | 2.6.32-642 |
| 6.9 | 2.6.32-696 |

#### <a name="ubuntu-server"></a>Ubuntu Server

| 作業系統版本 | 核心版本 |
|:--|:--|
| Ubuntu 18.04 | 核心 4.15.* |
| Ubuntu 16.04.3 | 核心 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

#### <a name="oracle-enterprise-linux-6-with-unbreakable-enterprise-kernel"></a>搭載 Unbreakable Enterprise Kernel 的 Oracle Enterprise Linux 6
| 作業系統版本 | 核心版本
|:--|:--|
| 6.2 | Oracle 2.6.32-300 (UEK R1) |
| 6.3 | Oracle 2.6.39-200 (UEK R2) |
| 6.4 | Oracle 2.6.39-400 (UEK R2) |
| 6.5 | Oracle 2.6.39-400 (UEK R2 i386) |
| 6.6 | Oracle 2.6.39-400 (UEK R2 i386) |

#### <a name="oracle-enterprise-linux-5-with-unbreakable-enterprise-kernel"></a>搭載 Unbreakable Enterprise Kernel 的 Oracle Enterprise Linux 5

| 作業系統版本 | 核心版本
|:--|:--|
| 5.10 | Oracle 2.6.39-400 (UEK R2) |
| 5.11 | Oracle 2.6.39-400 (UEK R2) |

#### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| 作業系統版本 | 核心版本
|:--|:--|
|12 SP2 | 4.4.* |
|12 SP3 | 4.4.* |

### <a name="hybrid-environment-connected-sources"></a>混合式環境連線的來源
適用於 VM 的 Azure 監視器對應功能會從 Microsoft Dependency Agent 取得它的資料。 Dependency Agent 須憑藉 Log Analytics 代理程式才能連線至 Log Analytics。 這表示，系統必須安裝 Log Analytics 代理程式並使用 Dependency Agent 加以設定。  下表說明對應功能在混合式環境中支援的連線來源。

| 連線的來源 | 支援 | 說明 |
|:--|:--|:--|
| Windows 代理程式 | 是 | 除了[適用於 Windows 的 Log Analytics 代理程式](../log-analytics/log-analytics-concept-hybrid.md)以外，Windows 代理程式還需要 Microsoft Dependency Agent。 如需作業系統版本的完整清單，請參閱[支援的作業系統](#supported-operating-systems)。 |
| Linux 代理程式 | 是 | 除了[適用於 Linux 的 Log Analytics 代理程式](../log-analytics/log-analytics-concept-hybrid.md)以外，Linux 代理程式還需要 Microsoft Dependency Agent。 如需作業系統版本的完整清單，請參閱[支援的作業系統](#supported-operating-systems)。 |
| System Center Operations Manager 管理群組 | 否 | |  

在 Windows 上，System Center Operations Manager 和 Log Analytics 會使用 Microsoft Monitoring Agent (MMA) 來收集和傳送監視資料。 System Center Operations Manager 和 Log Analytics 提供的預設代理程式版本不同。 這些版本可以各自向 System Center Operations Manager 或 Log Analytics 報告，或同時向兩者報告。  

在 Linux 上，適用於 Linux 的 Log Analytics 代理程式會收集監視資料並傳送給 Log Analytics。   

如果您的 Windows 或 Linux 電腦無法直接連線至服務，您必須將 Log Analytics 代理程式設定為使用 OMS 閘道連線至 Log Analytics。 如需如何部署和設定 OMS 閘道的進一步資源，請參閱[在無網際網路存取下使用 OMS 閘道連線電腦](../log-analytics/log-analytics-oms-gateway.md)。  

您可以從下列位置下載 Dependency Agent。

| 檔案 | 作業系統 | 版本 | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.7.1 | 55030ABF553693D8B5112569FB2F97D7C54B66E9990014FC8CC43EFB70DE56C6 |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.7.1 | 43C75EF0D34471A0CBCE5E396FFEEF4329C9B5517266108FA5D6131A353D29FE |

## <a name="diagnostic-and-usage-data"></a>診斷和使用量資料
當您使用 Azure 監視器服務時，Microsoft 會自動收集使用量和效能資料。 Microsoft 會使用此資料來提供並改進服務的品質、安全性和完整性。 為了提供正確且有效率的疑難排解功能，來自對應功能的資料會包含軟體設定的相關資訊，例如作業系統與版本、IP 位址、DNS 名稱和工作站名稱。 Microsoft 不會收集姓名、地址或其他連絡資訊。

如需有關資料收集與使用方式的詳細資訊，請參閱 [Microsoft 線上服務隱私權聲明](https://go.microsoft.com/fwlink/?LinkId=512132)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="performance-counters-enabled"></a>已啟用效能計數器
適用於 VM 的 Azure 監視器會設定 Log Analytics 工作區來收集解決方案所使用的效能計數器。  下表列出解決方案每隔 60 秒所收集的物件和計數器。

### <a name="windows-performance-counters"></a>Windows 效能計數器

|物件名稱 |計數器名稱 |  
|------------|-------------|  
|LogicalDisk |% Free Space |  
|LogicalDisk |Avg.Disk sec/Read |  
|LogicalDisk |Avg.Disk sec/Transfer |  
|LogicalDisk |Avg.Disk sec/Write |  
|LogicalDisk |Disk Bytes/sec |  
|LogicalDisk |Disk Read Bytes/sec  |  
|LogicalDisk |Disk Reads/sec  |  
|LogicalDisk |Disk Transfers/sec |  
|LogicalDisk |Disk Write Bytes/sec |  
|LogicalDisk |Disk Writes/sec |  
|LogicalDisk |Free Megabytes |  
|記憶體 |可用的 MB |  
|網路介面卡 |Bytes Received/sec |  
|網路介面卡 |Bytes Sent/sec |  
|處理器 |% Processor Time |  

### <a name="linux-performance-counters"></a>Linux 效能計數器

|物件名稱 |計數器名稱 |  
|------------|-------------|  
|Logical Disk |% Used Space |  
|Logical Disk |Disk Read Bytes/sec  |  
|Logical Disk |Disk Reads/sec  |  
|Logical Disk |Disk Transfers/sec |  
|Logical Disk |Disk Write Bytes/sec |  
|Logical Disk |Disk Writes/sec |  
|Logical Disk |Free Megabytes |  
|Logical Disk |Logical Disk Bytes/sec |  
|記憶體 |Available MBytes Memory |  
|網路 |Total Bytes Received |  
|網路 |Total Bytes Transmitted |  
|處理器 |% Processor Time |  

## <a name="enable-from-the-azure-portal"></a>從 Azure 入口網站啟用
若要在 Azure 入口網站中啟用 Azure VM 的監視，請執行下列動作：

1. 在 Azure 入口網站中，選取 [虛擬機器]。 
2. 從清單中選取 VM。 
3. 在 VM 頁面的 [監視] 區段中，選取 [Insights (預覽)]。
4. 在 [Insights (預覽)] 頁面上，選取 [立即試用]。

    ![為 VM 啟用適用於 VM 的 Azure 監視器](./media/monitoring-vminsights-onboard/enable-vminsights-vm-portal-01.png)

5. 在 [將 Azure 監視器 Insights 上線] 頁面上，如果您目前在同一個訂用帳戶中有 Log Analytics 工作區，請在下拉式清單中選取它。  此清單會預先選取已在訂用帳戶中部署虛擬機器的預設工作區和位置。 

    >[!NOTE]
    >如果您想要建立新的 Log Analytics 工作區以儲存來自 VM 的監視資料，請遵循在先前所列的其中一個支援區域中[建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)中的指示。   

啟用監視之後，可能需要約 10 分鐘的時間才能檢視虛擬機器的健康情況計量。 

![啟用適用於 VM 的 Azure 監視器來監視部署處理](./media/monitoring-vminsights-onboard/onboard-vminsights-vm-portal-status.png)

## <a name="enable-using-azure-policy"></a>使用 Azure 原則啟用
若要針對多部 Azure VM 啟用此解決方案，以確保所佈建的新 VM 具有一致的合規性且會自動啟用，建議您使用 [Azure 原則](../azure-policy/azure-policy-introduction.md)。  搭配所提供的原則來使用 Azure 原則，可針對新的 VM 提供下列優點：

* 針對已定義範圍中的每部 VM 啟用適用於 VM 的 Azure 監視器
* 部署 Log Analytics 代理程式 
* 部署 Dependency Agent 來探索應用程式相依性並顯示於對應中
* 稽核您的 Azure VM OS 映像是否位於原則定義中預先定義的清單內  
* 稽核您的 Azure VM 是否會記錄到指定工作區以外的工作區
* 報告合規性結果 
* 支援修復不符合規範的 VM

若要為您的租用戶啟用它，此處理序需要：

- 使用此處所列的步驟來設定 Log Analytics 工作區
- 將方案定義匯入至您的租用戶 (位於管理群組或訂用帳戶層級)
- 將原則指派給所需的範圍
- 檢閱合規性結果

### <a name="add-the-policies-and-initiative-to-your-subscription"></a>將原則與方案新增到您的訂用帳戶
若要使用原則，您可以使用提供的 PowerShell 指令碼 ([Add-VMInsightsPolicy.ps1](https://www.powershellgallery.com/packages/Add-VMInsightsPolicy/1.2)，可從 Azure PowerShell 資源庫取得) 來完成此工作。 指令碼會將原則與方案新增到您的訂用帳戶。  執行下列步驟來設定您訂用帳戶中的 Azure 原則。 

1. 將 PowerShell 指令碼下載到您的本機檔案系統。

2. 在資料夾中使用下列 PowerShell 命令來新增原則。 指令碼支援下列選擇性參數： 

    ```powershell
    -UseLocalPolicies [<SwitchParameter>]
      <Optional> Load the policies from a local folder instead of https://raw.githubusercontent.com/dougbrad/OnBoardVMInsights/Policy/Policy/

    -SubscriptionId <String>
      <Optional> SubscriptionId to add the Policies/Initiatives to
    -ManagementGroupId <String>
      <Optional> Management Group Id to add the Policies/Initiatives to

    -Approve [<SwitchParameter>]
      <Optional> Gives the approval to add the Policies/Initiatives without any prompt
    ```  

    >[!NOTE]
    >附註：如果您計劃要將方案/原則指派給多個訂用帳戶，則必須將定義儲存於包含您要將原則指派到其中的管理群組內。 因此，您必須使用 -ManagementGroupID 參數。
    >
   
    不含參數的範例：`.\Add-VMInsightsPolicy.ps1`

### <a name="create-a-policy-assignment"></a>建立原則指派
執行 `Add-VMInsightsPolicy.ps1` PowerShell 指令碼之後，即會新增下列方案和原則：

* **部署適用於 Windows VM 的 Log Analytics 代理程式 - 預覽**
* **部署適用於 Linux VM 的 Log Analytics 代理程式 - 預覽**
* **部署適用於 Windows VM 的 Dependency Agent - 預覽**
* **部署適用於 Linux VM 的 Dependency Agent - 預覽**
* **稽核 Log Analytics 代理程式部署 - 未列出 VM 映像 (OS) - 預覽**
* **稽核 Dependency Agent 部署 - 未列出 VM 映像 (OS) - 預覽**

新增下列方案參數：

- **Log Analytice 工作區** (如果您使用 PowerShell 或 CLI 套用指派，則必須提供工作區的 ResourceID)

    針對發現不符合 **VM 不在 OS 範圍內...** 稽核原則的 VM，部署原則的準則只會包含從已知 Azure VM 映像部署的 VM。 請參閱文件以了解 VM OS 是否受到支援。  如果不受支援，則您必須複製部署原則，並更新/修改它以使映像位於範圍內。

新增下列獨立的選擇性原則：

- **已針對不相符的 Log Analytics 工作區設定 VM - 預覽**

    這可用來識別已經使用 [Log Analytics VM 延伸模組](../virtual-machines/extensions/oms-windows.md)來設定，但用來設定的工作區與預期 (如原則指派所指定) 不同的 VM。 這會採用適用於 WorkspaceID 的參數。

利用此初始版本，您只能從 Azure 入口網站建立原則指派。 若要了解如何完成這些步驟，請參閱[從 Azure 入口網站建立原則指派](../azure-policy/assign-policy-definition.md)。

## <a name="enable-with-powershell"></a>使用 PowerShell 啟用
若要針對多個 VM 或 VM 擴展集啟用適用於 VM 的 Azure 監視器，您可以使用提供的 PowerShell 指令碼 ([Install-VMInsights.ps1](https://www.powershellgallery.com/packages/Install-VMInsights/1.0)，可從 Azure PowerShell 資源庫取得) 來完成此工作。  此指令碼將在您的訂用帳戶中、在 *ResourceGroup* 所指定的範圍內資源群組中，逐一查看每個虛擬機器和 VM 擴展集，或逐一查看由「名稱」所指定的單一 VM 或擴展集。  針對每個 VM 或 VM 擴展集，指令碼會確認是否已經安裝 VM 延伸模組，以及是否會試著重新安裝它。  否則，它會繼續安裝 Log Analytics 和 Dependency Agent VM 延伸模組。   

此指令碼需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

若要取得有關此指令碼的說明，您可以執行 `Get-Help` 來取得引數詳細資訊和範例使用方式的清單。   

```powershell
Get-Help .\Install-VMInsights.ps1 -Detailed

SYNOPSIS
    This script installs VM extensions for Log Analytics and Dependency Agent as needed for VM Insights.


SYNTAX
    .\Install-VMInsights.ps1 [-WorkspaceId] <String> [-WorkspaceKey] <String> [-SubscriptionId] <String> [[-ResourceGroup]
    <String>] [[-Name] <String>] [[-PolicyAssignmentName] <String>] [-ReInstall] [-TriggerVmssManualVMUpdate] [-Approve] [-WorkspaceRegion] <String>
    [-WhatIf] [-Confirm] [<CommonParameters>]


DESCRIPTION
    This script installs or re-configures following on VM's and VM Scale Sets:
    - Log Analytics VM Extension configured to supplied Log Analytics Workspace
    - Dependency Agent VM Extension

    Can be applied to:
    - Subscription
    - Resource Group in a Subscription
    - Specific VM/VM Scale Set
    - Compliance results of a policy for a VM or VM Extension

    Script will show you list of VM's/VM Scale Sets that will apply to and let you confirm to continue.
    Use -Approve switch to run without prompting, if all required parameters are provided.

    If the extensions are already installed will not install again.
    Use -ReInstall switch if you need to for example update the workspace.

    Use -WhatIf if you would like to see what would happen in terms of installs, what workspace configured to, and status of the extension.


PARAMETERS
    -WorkspaceId <String>
        Log Analytics WorkspaceID (GUID) for the data to be sent to

    -WorkspaceKey <String>
        Log Analytics Workspace primary or secondary key

    -SubscriptionId <String>
        SubscriptionId for the VMs/VM Scale Sets
        If using PolicyAssignmentName parameter, subscription that VM's are in

    -ResourceGroup <String>
        <Optional> Resource Group to which the VMs or VM Scale Sets belong to

    -Name <String>
        <Optional> To install to a single VM/VM Scale Set

    -PolicyAssignmentName <String>
        <Optional> Take the input VM's to operate on as the Compliance results from this Assignment
        If specified will only take from this source.

    -ReInstall [<SwitchParameter>]
        <Optional> If VM/VM Scale Set is already configured for a different workspace, set this to change to the new workspace

    -TriggerVmssManualVMUpdate [<SwitchParameter>]
        <Optional> Set this flag to trigger update of VM instances in a scale set whose upgrade policy is set to Manual

    -Approve [<SwitchParameter>]
        <Optional> Gives the approval for the installation to start with no confirmation prompt for the listed VM's/VM Scale Sets

    -WorkspaceRegion <String>
        Region the Log Analytics Workspace is in
        Suported values: "East US","eastus","Southeast Asia","southeastasia","West Central US","westcentralus","West Europe","westeurope"
        For Health supported is: "East US","eastus","West Central US","westcentralus"

    -WhatIf [<SwitchParameter>]
        <Optional> See what would happen in terms of installs.
        If extension is already installed will show what workspace is currently configured, and status of the VM extension

    -Confirm [<SwitchParameter>]
        <Optional> Confirm every action

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (https:/go.microsoft.com/fwlink/?LinkID=113216).

    -------------------------- EXAMPLE 1 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup>

    Install for all VM's in a Resource Group in a subscription

    -------------------------- EXAMPLE 2 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -ResourceGroup <ResourceGroup> -ReInstall

    Specify to ReInstall extensions even if already installed, for example to update to a different workspace

    -------------------------- EXAMPLE 3 --------------------------
    .\Install-VMInsights.ps1 -WorkspaceRegion eastus -WorkspaceId <WorkspaceId>-WorkspaceKey <WorkspaceKey> -SubscriptionId <SubscriptionId>
    -PolicyAssignmentName a4f79f8ce891455198c08736 -ReInstall

    Specify to use a PolicyAssignmentName for source, and to ReInstall (move to a new workspace)
```

下列範例示範如何在資料夾中使用 PowerShell 命令來啟用適用於 VM 的 Azure 監視器，以及了解預期的輸出：

```powershell
$WorkspaceId = "<GUID>"
$WorkspaceKey = "<Key>"
$SubscriptionId = "<GUID>"
.\Install-VMInsights.ps1 -WorkspaceId $WorkspaceId -WorkspaceKey $WorkspaceKey -SubscriptionId $SubscriptionId -WorkspaceRegion eastus

Getting list of VM's or VM ScaleSets matching criteria specified

VM's or VM ScaleSets matching criteria:

db-ws-1 VM running
db-ws2012 VM running

This operation will install the Log Analytics and Dependency Agent extensions on above 2 VM's or VM Scale Sets.
VM's in a non-running state will be skipped.
Extension will not be re-installed if already installed. Use -ReInstall if desired, for example to update workspace

Confirm
Continue?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

db-ws-1 : Deploying DependencyAgentWindows with name DAExtension
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Deploying DependencyAgentWindows with name DAExtension
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Deploying MicrosoftMonitoringAgent with name MMAExtension
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Summary:

Already Onboarded: (0)

Succeeded: (4)
db-ws-1 : Successfully deployed DependencyAgentWindows
db-ws-1 : Successfully deployed MicrosoftMonitoringAgent
db-ws2012 : Successfully deployed DependencyAgentWindows
db-ws2012 : Successfully deployed MicrosoftMonitoringAgent

Connected to different workspace: (0)

Not running - start VM to configure: (0)

Failed: (0)
```

## <a name="enable-for-hybrid-environment"></a>針對混合式環境啟用
本節說明如何將裝載於您資料中心或其他雲端環境的虛擬機器或實體電腦上線，由適用於 VM 的 Azure 監視器進行監視。  

適用於 VM 的 Azure 監視器對應 Dependency Agent 本身不會傳輸任何資料，因此不需變更防火牆或連接埠。 如果您的 IT 安全性原則不允許網路上的電腦連線到網際網路，則對應中的資料一律會由 Log Analytics 代理程式直接傳輸到 Azure 監視器服務，或透過 [OMS 閘道](../log-analytics/log-analytics-oms-gateway.md)傳輸。

檢閱適用於 [Log Analytics Linux 和 Windows 代理程式](../log-analytics/log-analytics-concept-hybrid.md)的需求和部署方法。

摘要的步驟：

1. 安裝適用於 Windows 或 Linux 的 Log Analytics 代理程式
2. 安裝適用於 VM 的 Azure 監視器對應 Dependency Agent
3. 啟用效能計數器收集
4. 將適用於 VM 的 Azure 監視器上線

### <a name="install-the-dependency-agent-on-windows"></a>在 Windows 上安裝 Dependency Agent 
您可以執行 `InstallDependencyAgent-Windows.exe` 以手動方式將 Dependency Agent 安裝在 Windows 電腦上。 如果您在執行此可執行檔時未使用任何選項，則會啟動以互動方式安裝的安裝精靈。  

>[!NOTE]
>必須有系統管理員權限，以便安裝或解除安裝代理程式。

下表將強調說明命令列中適用於代理程式的安裝程式所支援的特定參數。  

| 參數 | 說明 |
|:--|:--|
| /? | 傳回命令列選項的清單。 |
| /S | 執行無訊息安裝，而不會與使用者互動。 |

例如，若要使用 `/?` 參數來執行安裝程式，請輸入 `InstallDependencyAgent-Windows.exe /?`

適用於 Windows Dependency Agent 的檔案預設會安裝於 `C:\Program Files\Microsoft Dependency Agent`。  如果 Dependency Agent 在安裝完成之後無法啟動，請檢查記錄以取得詳細的錯誤資訊。 記錄目錄為 `%Programfiles%\Microsoft Dependency Agent\logs`。 

### <a name="install-the-dependency-agent-on-linux"></a>在 Linux 上安裝 Dependency Agent
從 `InstallDependencyAgent-Linux64.bin` (具有自我解壓縮二進位檔的殼層指令碼) 在 Linux 伺服器上安裝 Dependency Agent。 您可以使用 `sh` 來執行檔案，或對檔案本身新增執行權限。

>[!NOTE]
> 必須有 root 權限，以便安裝或設定代理程式。
> 

| 參數 | 說明 |
|:--|:--|
| -help | 取得命令列選項的清單。 |
| -s | 執行無訊息安裝，不會出現任何使用者提示。 |
| --check | 檢查權限和作業系統，但不會安裝代理程式。 |

例如，若要使用 `-help` 參數來執行安裝程式，請輸入 `InstallDependencyAgent-Linux64.bin -help`。

執行下列命令，以 root 身分安裝 Linux Dependency Agent：`sh InstallDependencyAgent-Linux64.bin`
    
如果 Dependency Agent 無法啟動，請檢查記錄檔以取得詳細的錯誤資訊。 在 Linux 代理程式中，記錄目錄為 `/var/opt/microsoft/dependency-agent/log`。

Dependency Agent 的檔案位於下列目錄：

| 檔案 | 位置 |
|:--|:--|
| 核心檔案 | /opt/microsoft/dependency-agent |
| 記錄檔 | /var/opt/microsoft/dependency-agent/log |
| 組態檔 | /etc/opt/microsoft/dependency-agent/config |
| 服務可執行檔 | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| 二進位儲存體檔案 | /var/opt/microsoft/dependency-agent/storage |

### <a name="enable-performance-counters"></a>啟用效能計數器
如果解決方案所參考的 Log Analytics 工作區未設定為已收集此解決方案所需的效能計數器，則必須啟用它們。 您可以如[此處](../log-analytics/log-analytics-data-sources-performance-counters.md)所述的方式手動完成此動作，或藉由下載並執行可從 [Azure PowerShell 資源庫](https://www.powershellgallery.com/packages/Enable-VMInsightsPerfCounters/1.1)取得的 PowerShell 指令碼來完成此動作。
 
### <a name="onboard-azure-monitor-for-vms"></a>將適用於 VM 的 Azure 監視器上線
此方法包含一個 JSON 範本，其會指定設定來為您的 Log Analytics 工作區啟用解決方案元件。  

若您不熟悉使用範本來部署資源的概念，請參閱：
* [使用 Resource Manager 範本與 Azure PowerShell 來部署資源](../azure-resource-manager/resource-group-template-deploy.md)
* [使用 Resource Manager 範本與 Azure CLI 部署資源](../azure-resource-manager/resource-group-template-deploy-cli.md) 

如果您選擇使用 Azure CLI，必須先在本機安裝並使用 CLI。 您必須執行 Azure CLI 2.0.27 版或更新版本。 若要知道您使用的版本，請執行 `az --version`。 如果您需要安裝或升級 Azure CLI，請參閱[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 

#### <a name="create-and-execute-a-template"></a>建立並執行範本

1. 複製以下 JSON 語法並貼到您的檔案中：

    ```json
    {

    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "WorkspaceName": {
            "type": "string"
        },
        "WorkspaceLocation": {
            "type": "string"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-03-15-preview",
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('WorkspaceName')]",
            "location": "[parameters('WorkspaceLocation')]",
            "resources": [
                {
                    "apiVersion": "2015-11-01-preview",
                    "location": "[parameters('WorkspaceLocation')]",
                    "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                    "type": "Microsoft.OperationsManagement/solutions",
                    "dependsOn": [
                        "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                    ],
                    "properties": {
                        "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                    },

                    "plan": {
                        "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                        "publisher": "Microsoft",
                        "product": "[Concat('OMSGallery/', 'ServiceMap')]",
                        "promotionCode": ""
                    }
                },
                {
                    "apiVersion": "2015-11-01-preview",
                    "location": "[parameters('WorkspaceLocation')]",
                    "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                    "type": "Microsoft.OperationsManagement/solutions",
                    "dependsOn": [
                        "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                    ],
                    "properties": {
                        "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                    },
                    "plan": {
                        "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                        "publisher": "Microsoft",
                        "product": "[Concat('OMSGallery/', 'InfrastructureInsights')]",
                        "promotionCode": ""
                    }
                }
            ]
        }
    ]
    ```

2. 在本機資料夾中將此檔案儲存為 **installsolutionsforvminsights.json**。
3. 編輯 **WorkspaceName**、**ResourceGroupName** 和 **WorkspaceLocation** 的值。  **WorkspaceName** 的值是您 Log Analytics 工作區的完整資源識別碼 (其中包括工作區名稱)，而 **WorkspaceLocation** 的值是定義工作區的區域。
4. 您已經準備好使用下列 PowerShell 命令來部署此範本：

    ```powershell
    New-AzureRmResourceGroupDeployment -Name DeploySolutions -TemplateFile InstallSolutionsForVMInsights.json -ResourceGroupName ResourceGroupName> -WorkspaceName <WorkspaceName> -WorkspaceLocation <WorkspaceLocation - example: eastus>
    ```

    可能需要幾分鐘的時間才能完成設定變更。 完成之後，將會顯示如下訊息並包含結果：

    ```powershell
    provisioningState       : Succeeded
    ```
啟用監視之後，可能需要約 10 分鐘的時間才能檢視混合式電腦的健康狀態和計量。 

## <a name="next-steps"></a>後續步驟

藉由為您的虛擬機器啟用監視，適用於 VM 的 Azure 監視器就能使用此資訊來進行分析。  若要了解如何使用健康情況功能，請參閱[檢視適用於 VM 的 Azure 監視器健康情況](monitoring-vminsights-health.md)，或者，若要檢視探索到的應用程式相依性，請參閱[檢視適用於 VM 的 Azure 監視器對應](monitoring-vminsights-maps.md)。  