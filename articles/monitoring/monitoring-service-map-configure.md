---
title: 在 Azure 中設定服務對應 | Microsoft Docs
description: 服務對應是 Azure 中的一個解決方案，可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文會詳細說明如何在環境中部署服務對應並將它用於各種案例。
services: monitoring
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: d3d66b45-9874-4aad-9c00-124734944b2e
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: daseidma;bwren
ms.openlocfilehash: a68c35ba2f740720e3d7940d6fafa2dcfe183589
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064368"
---
# <a name="configure-service-map-in-azure"></a>在 Azure 中設定服務對應
服務對應可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 您可以使用服務對應，將伺服器視為提供重要服務的互連系統，藉以檢視伺服器。 不需要進行任何設定，只要安裝了代理程式，服務對應就會顯示橫跨任何 TCP 連線架構的伺服器、處理序和連接埠之間的連線。

本文說明設定服務對應與啟用代理程式的詳細資料。 如需使用服務對應的相關資訊，請參閱[在 Azure 中使用服務對應解決方案]( monitoring-service-map.md)。

## <a name="supported-azure-regions"></a>支援的 Azure 區域
服務對應目前已在以下 Azure 區域中推出：
- 美國東部
- 西歐
- 美國中西部
- 東南亞

## <a name="supported-windows-operating-systems"></a>支援的 Windows 作業系統
下列幾節會列出 Windows 上的 Dependency Agent 所支援的作業系統。 

>[!NOTE]
>服務對應僅支援 64 位元平台。
>

### <a name="windows-server"></a>Windows Server
- Windows Server 2016 1803
- Windows Server 2016
- Windows Server 2012 R2
- Windows Server 2012
- Windows Server 2008 R2 SP1

### <a name="windows-desktop"></a>Windows 桌面
- Windows 10 1803
- Windows 10
- Windows 8.1
- Windows 8
- Windows 7

## <a name="supported-linux-operating-systems"></a>支援的 Linux 作業系統
下節會列出 Red Hat Enterprise Linux、CentOS Linux 和 Oracle Linux (搭載 RHEL 核心) 上的 Dependency Agent 所支援的作業系統。  

- 只支援預設版本和 SMP Linux 核心版本。
- 所有 Linux 散發套件皆不支援非標準的核心版本 (例如 PAE 和 Xen)。 舉例來說，版本字串為「2.6.16.21-0.8-xen」的系統就不受支援。
- 不支援自訂核心，包括重新編譯的標準核心。
- 不支援 CentOSPlus 核心。
- Oracle Unbreakable Enterprise Kernel (UEK) 在本文稍後的章節有相關討論。

### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| 作業系統版本 | 核心版本 |
|:--|:--|
| 7.0 | 3.10.0-123 |
| 7.1 | 3.10.0-229 |
| 7.2 | 3.10.0-327 |
| 7.3 | 3.10.0-514 |
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |

### <a name="red-hat-linux-6"></a>Red Hat Linux 6

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

### <a name="ubuntu-server"></a>Ubuntu Server

| 作業系統版本 | 核心版本 |
|:--|:--|
| Ubuntu 18.04 | 核心 4.15.* |
| Ubuntu 16.04.3 | 核心 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

### <a name="oracle-enterprise-linux-6-with-unbreakable-enterprise-kernel"></a>搭載 Unbreakable Enterprise Kernel 的 Oracle Enterprise Linux 6
| 作業系統版本 | 核心版本
|:--|:--|
| 6.2 | Oracle 2.6.32-300 (UEK R1) |
| 6.3 | Oracle 2.6.39-200 (UEK R2) |
| 6.4 | Oracle 2.6.39-400 (UEK R2) |
| 6.5 | Oracle 2.6.39-400 (UEK R2 i386) |
| 6.6 | Oracle 2.6.39-400 (UEK R2 i386) |

### <a name="oracle-enterprise-linux-5-with-unbreakable-enterprise-kernel"></a>搭載 Unbreakable Enterprise Kernel 的 Oracle Enterprise Linux 5

| 作業系統版本 | 核心版本
|:--|:--|
| 5.10 | Oracle 2.6.39-400 (UEK R2) |
| 5.11 | Oracle 2.6.39-400 (UEK R2) |

## <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| 作業系統版本 | 核心版本
|:--|:--|
|12 SP2 | 4.4.* |
|12 SP3 | 4.4.* |

## <a name="dependency-agent-downloads"></a>Dependency Agent 下載

| 檔案 | 作業系統 | 版本 | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.7.1 | 55030ABF553693D8B5112569FB2F97D7C54B66E9990014FC8CC43EFB70DE56C6 |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.7.1 | 43C75EF0D34471A0CBCE5E396FFEEF4329C9B5517266108FA5D6131A353D29FE |

## <a name="connected-sources"></a>連接的來源
服務對應會從 Microsoft Dependency Agent 取得它的資料。 Dependency Agent 須憑藉 Log Analytics 代理程式才能連線至 Log Analytics。 這表示，伺服器必須先安裝 Log Analytics 代理程式，並設定 Dependency Agent。  下表描述服務對應解決方案支援的連線來源。

| 連線的來源 | 支援 | 說明 |
|:--|:--|:--|
| Windows 代理程式 | 是 | 服務對應會分析並收集來自 Windows 電腦的資料。 <br><br>除了[適用於 Windows 的 Log Analytics 代理程式](../log-analytics/log-analytics-concept-hybrid.md)以外，Windows 代理程式還需要 Microsoft Dependency Agent。 如需作業系統版本的完整清單，請參閱[支援的作業系統](#supported-operating-systems)。 |
| Linux 代理程式 | 是 | 服務對應會分析並收集來自 Linux 電腦的資料。 <br><br>除了[適用於 Linux 的 Log Analytics 代理程式](../log-analytics/log-analytics-concept-hybrid.md)以外，Linux 代理程式還需要 Microsoft Dependency Agent。 如需作業系統版本的完整清單，請參閱[支援的作業系統](#supported-operating-systems)。 |
| System Center Operations Manager 管理群組 | 是 | 服務對應會在連線的 [System Center Operations Manager 管理群組](../log-analytics/log-analytics-om-agents.md)中，分析並收集來自 Windows 和 Linux 代理程式的資料。 <br><br>System Center Operations Manager 代理程式電腦必須直接連線到 Log Analytics。 |
| Azure 儲存體帳戶 | 否 | 服務對應會收集來自代理程式電腦的資料，因此不會從 Azure 儲存體收集資料。 |

在 Windows 上，System Center Operations Manager 和 Log Analytics 會使用 Microsoft Monitoring Agent (MMA) 來收集和傳送監視資料。 (視內容而定，此代理程式可稱為 System Center Operations Manager 代理程式、OMS 代理程式、Log Analytics 代理程式、MMA 或直接代理程式)System Center Operations Manager 和 Log Analytics 提供的預設 MMA 版本不同。 這些版本可以各自向 System Center Operations Manager 或 Log Analytics 報告，或同時向兩者報告。  

在 Linux 上，適用於 Linux 的 Log Analytics 代理程式會收集監視資料並傳送給 Log Analytics。 您可以在伺服器上使用服務對應與直接連線至服務的 Log Analytics 代理程式，或是向與 Log Analytics 整合的 Operations Manager 管理群組回報的代理程式。  

在本文中，我們會將所有代理程式 (無論 Linux 或 Windows、無論是連線到 System Center Operations Manager 管理群組或直接連線到 Log Analytics) 統稱為「Log Analytics 代理程式」。 

服務對應代理程式本身不會傳輸任何資料，因此不需要變更防火牆或連接埠。 服務對應中的資料一律會由 Log Analytics 代理程式 (直接或透過 OMS 閘道) 傳輸給 Log Analytics 服務。

![服務對應代理程式](media/monitoring-service-map/agents.png)

如果您是管理群組連線到 Log Analytics 的 System Center Operations Manager 客戶：

- 當您的 System Center Operations Manager 代理程式可透過網際網路連線到 Log Analytics 時，就不需要額外的設定。  
- 如果您的 System Center Operations Manager 代理程式無法透過網際網路存取 Log Analytics 時，就必須設定 OMS 閘道以搭配 System Center Operations Manager 使用。
  
如果您的 Windows 或 Linux 電腦無法直接連線至服務，您必須將 Log Analytics 代理程式設定為使用 OMS 閘道連線至 Log Analytics。 如需如何部署和設定 OMS 閘道的進一步資源，請參閱[在無網際網路存取下使用 OMS 閘道連線電腦](../log-analytics/log-analytics-oms-gateway.md)。  

### <a name="management-packs"></a>管理組件
在 Log Analytics 工作區中啟動服務對應時，會將 300 KB 的管理組件轉送至該工作區中的所有 Windows 伺服器。 如果您是在[連線的管理群組](../log-analytics/log-analytics-om-agents.md)中使用 System Center Operations Manager 代理程式，則會從 System Center Operations Manager 來部署服務對應管理組件。 

管理組件名稱為 Microsoft.IntelligencePacks.ApplicationDependencyMonitor。 它會寫入至 %Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs\。 管理組件所使用的資料來源是 %Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources\<AutoGeneratedID>\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll。

## <a name="data-collection"></a>資料收集
視系統相依性的複雜程度而定，每個代理程式每天預計會傳輸大約 25 MB。 每個代理程式每隔 15 秒就會傳送服務對應相依性資料。  

Dependency Agent 一般會耗用 0.1% 的系統記憶體和 0.1% 的系統 CPU。

## <a name="diagnostic-and-usage-data"></a>診斷和使用量資料
當您使用服務對應服務時，Microsoft 會自動收集使用量和效能資料。 Microsoft 使用這項資料來提供和改進服務對應服務的品質、安全性和完整性。 這項資料包含軟體組態 (如作業系統和版本) 的相關資訊。 它也會包含 IP 位址、DNS 名稱和工作站名稱，以便能夠正確且有效率地進行疑難排解。 我們不會收集姓名、地址或其他連絡資訊。

如需有關資料收集與使用方式的詳細資訊，請參閱 [Microsoft 線上服務隱私權聲明](https://go.microsoft.com/fwlink/?LinkId=512132)。

## <a name="installation"></a>安裝

## <a name="azure-vm-extension"></a>Azure VM 擴充功能
有一個可供 Windows (DependencyAgentWindows) 和 Linux (DependencyAgentLinux) 使用的擴充功能，您可以使用 [Azure VM 擴充功能](https://docs.microsoft.com/azure/virtual-machines/windows/extensions-features)輕鬆地將 Dependency Agent 部署至您的 Azure VM。  透過 Azure VM 擴充功能，您可以使用 PowerShell 指令碼，或直接在 VM 中使用 Azure Resource Manager 範本，將 Dependency Agent 部署至您的 Windows 和 Linux VM。  如果您使用 Azure VM 擴充功能來部署代理程式，您的代理程式就可自動更新為最新版本。

若要使用 PowerShell 來部署 Azure VM 擴充功能，您可以使用下列範例：

```PowerShell
#
# Deploy the Dependency agent to every VM in a Resource Group
#

$version = "9.4"
$ExtPublisher = "Microsoft.Azure.Monitoring.DependencyAgent"
$OsExtensionMap = @{ "Windows" = "DependencyAgentWindows"; "Linux" = "DependencyAgentLinux" }
$rmgroup = "<Your Resource Group Here>"

Get-AzureRmVM -ResourceGroupName $rmgroup |
ForEach-Object {
    ""
    $name = $_.Name
    $os = $_.StorageProfile.OsDisk.OsType
    $location = $_.Location
    $vmRmGroup = $_.ResourceGroupName
    "${name}: ${os} (${location})"
    Date -Format o
    $ext = $OsExtensionMap.($os.ToString())
    $result = Set-AzureRmVMExtension -ResourceGroupName $vmRmGroup -VMName $name -Location $location `
    -Publisher $ExtPublisher -ExtensionType $ext -Name "DependencyAgent" -TypeHandlerVersion $version
    $result.IsSuccessStatusCode
}
```

要確保 Dependency Agent 已安裝在 VM 上，更簡單方法是在您的 Azure Resource Manager 範本中納入該代理程式。  您可以將下列 JSON 程式碼範例新增至範本的「資源」區段。

```JSON
"type": "Microsoft.Compute/virtualMachines/extensions",
"name": "[concat(parameters('vmName'), '/DependencyAgent')]",
"apiVersion": "2017-03-30",
"location": "[resourceGroup().location]",
"dependsOn": [
"[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
],
"properties": {
    "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
    "type": "DependencyAgentWindows",
    "typeHandlerVersion": "9.4",
    "autoUpgradeMinorVersion": true
}

```

### <a name="install-the-dependency-agent-on-microsoft-windows"></a>在 Microsoft Windows 上安裝 Dependency Agent
您可以執行 `InstallDependencyAgent-Windows.exe` 以手動方式將 Dependency Agent 安裝在 Windows 電腦上。 如果您在執行此可執行檔時未使用任何選項，則會啟動以互動方式安裝的安裝精靈。  

>[!NOTE]
>必須有系統管理員權限，以便安裝或解除安裝代理程式。

請使用下列步驟在每一部 Windows 電腦上安裝 Dependency Agent：

1.  依照[使用 Log Analytics 代理程式在混合式環境中收集資料](../log-analytics/log-analytics-concept-hybrid.md)中說明的其中一種方法，安裝適用於 Windows 的 Log Analytics 代理程式。
2.  下載 Windows 代理程式，並以下列命令來執行： 
    
    `InstallDependencyAgent-Windows.exe`

3.  依照安裝精靈的指示安裝代理程式。
4.  如果 Dependency Agent 無法啟動，請檢查記錄檔以取得詳細的錯誤資訊。 在 Windows 代理程式上，記錄檔的目錄是 %Programfiles%\Microsoft Dependency Agent\logs。 

#### <a name="windows-command-line"></a>Windows 命令列
使用下表中的選項以從命令列安裝。 若要查看安裝旗標的清單，請如下所示使用 /? 旗標執行安裝程式 。

    InstallDependencyAgent-Windows.exe /?

| 旗標 | 說明 |
|:--|:--|
| /? | 取得命令列選項的清單。 |
| /S | 執行無訊息安裝，不會出現任何使用者提示。 |

Windows Dependency Agent 的檔案預設位於 C:\Program Files\Microsoft Dependency Agent。

### <a name="install-the-dependency-agent-on-linux"></a>在 Linux 上安裝 Dependency Agent
使用 `InstallDependencyAgent-Linux64.bin` (具有自我解壓縮二進位檔的殼層指令碼) 可在 Linux 電腦上安裝 Dependency Agent。 您可以使用 `sh` 來執行檔案，或對檔案本身新增執行權限。

>[!NOTE]
> 必須有 root 權限，以便安裝或設定代理程式。

請使用下列步驟在每一部 Linux 電腦上安裝 Dependency Agent：

1.  依照[在混合式環境中使用 Log Analytics 代理程式來收集資料](../log-analytics/log-analytics-concept-hybrid.md)中所述的其中一種方法安裝 Log Analytics 代理程式。
2.  執行下列命令，以 root 身分安裝 Linux Dependency Agent：
    
    `sh InstallDependencyAgent-Linux64.bin`

3.  如果 Dependency Agent 無法啟動，請檢查記錄檔以取得詳細的錯誤資訊。 在 Linux 代理程式上，記錄檔的目錄是 /var/opt/microsoft/dependency-agent/log。

若要查看安裝旗標的清單，請如下所示以 -help 旗標執行安裝程式。

    InstallDependencyAgent-Linux64.bin -help

| 旗標 | 說明 |
|:--|:--|
| -help | 取得命令列選項的清單。 |
| -s | 執行無訊息安裝，不會出現任何使用者提示。 |
| --check | 檢查權限和作業系統，但不會安裝代理程式。 |

Dependency Agent 的檔案位於下列目錄：

| 檔案 | 位置 |
|:--|:--|
| 核心檔案 | /opt/microsoft/dependency-agent |
| 記錄檔 | /var/opt/microsoft/dependency-agent/log |
| 組態檔 | /etc/opt/microsoft/dependency-agent/config |
| 服務可執行檔 | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| 二進位儲存體檔案 | /var/opt/microsoft/dependency-agent/storage |

## <a name="installation-script-examples"></a>安裝指令碼範例
若要同時在許多伺服器上輕鬆部署 Dependency Agent ，請使用下列指令碼範例，在 Windows 或 Linux 上下載並安裝 Dependency Agent。

### <a name="powershell-script-for-windows"></a>適用於 Windows 的 PowerShell 指令碼
```PowerShell
Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S
```

### <a name="shell-script-for-linux"></a>適用於 Linux 的殼層指令碼
```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
sudo sh InstallDependencyAgent-Linux64.bin -s
```
## <a name="desired-state-configuration"></a>期望的狀態設定
若要使用預期狀態設定 (DSC) 部署 Dependency Agent，您可以使用 xPSDesiredStateConfiguration 模組和下列範例程式碼：

```
configuration ServiceMap {

Import-DscResource -ModuleName xPSDesiredStateConfiguration

$DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"

Node localhost
{ 
    # Download and install the Dependency agent
    xRemoteFile DAPackage 
    {
        Uri = "https://aka.ms/dependencyagentwindows"
        DestinationPath = $DAPackageLocalPath
    }

    xPackage DA
    {
        Ensure="Present"
        Name = "Dependency Agent"
        Path = $DAPackageLocalPath
        Arguments = '/S'
        ProductId = ""
        InstalledCheckRegKey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"
        InstalledCheckRegValueName = "DisplayName"
        InstalledCheckRegValueData = "Dependency Agent"
        DependsOn = "[xRemoteFile]DAPackage"
    }
  }
}
```

## <a name="remove-the-dependency-agent"></a>移除 Dependency Agent
### <a name="uinstall-agent-on-windows"></a>在 Windows 上解除安裝代理程式
系統管理員可透過控制台解除安裝 Windows Dependency Agent。

系統管理員也可以執行 %Programfiles%\Microsoft Dependency Agent\Uninstall.exe，以解除安裝 Dependency Agent。

### <a name="uninstall-agent-on-linux"></a>在 Linux 上解除安裝代理程式
您可以使用下列命令在 Linux 上解除安裝 Dependency Agent。

RHEL、CentOs 或 Oracle：

```
sudo rpm -e dependency-agent
```

Ubuntu：

```
sudo apt -y purge dependency-agent
```

## <a name="troubleshooting"></a>疑難排解
如果您在安裝或執行服務對應時遇到任何問題，本節內容可以提供協助。 如果您仍然無法解決問題，請連絡 Microsoft 支援服務。

### <a name="dependency-agent-installation-problems"></a>Dependency Agent 安裝問題
#### <a name="installer-prompts-for-a-reboot"></a>安裝程式提示重新開機
Dependency Agent *通常*不需要在安裝或解除安裝之後重新開機。 不過，在某些罕見情況下，Windows Server 需要重新開機，才能繼續進行安裝。 這會在相依性 (通常是 Microsoft Visual C++ 可轉散發套件) 因為鎖定的檔案而需要重新開機時發生。

#### <a name="message-unable-to-install-dependency-agent-visual-studio-runtime-libraries-failed-to-install-code--codenumber-appears"></a>顯示「無法安裝 Dependency Agent︰無法安裝 Visual Studio 執行階段程式庫 (程式碼 = [code_number])」訊息

Microsoft Dependency Agent 建置於 Microsoft Visual Studio 執行階段程式庫之上。 如果程式庫安裝期間發生問題，就會出現訊息。 

執行階段程式庫安裝程式會在 %LOCALAPPDATA%\temp 資料夾中建立記錄。 檔案將會是 dd_vcredist_arch_yyyymmddhhmmss.log，其中 *arch* 是 "x86" 或 "amd64"，*yyyymmddhhmmss* 是建立記錄的日期和時間 (24 小時制)。 記錄會提供導致無法安裝之問題的詳細資料。

如果可以先自行安裝[最新的執行階段程式庫](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)，將會十分有用。

下表列出代碼和建議的解決方式。

| 代碼 | 說明 | 解決方案 |
|:--|:--|:--|
| 0x17 | 程式庫安裝程式會要求尚未安裝的 Windows 更新。 | 查看最新的程式庫安裝程式記錄。<br><br>如果提到 "Windows8.1-KB2999226-x64.msu"，隨後一行是 "Error 0x80240017: Failed to execute MSU package"，則表示您尚未具備安裝 KB2999226 所需的必要條件。 請依照 [Windows 中的通用 C 執行階段](https://support.microsoft.com/kb/2999226) \(機器翻譯\) 中必要條件一節的指示進行。 您可能需要執行 Windows Update 並重新開機多次，才能安裝必要條件。<br><br>再次執行 Microsoft Dependency Agent 安裝程式。 |

### <a name="post-installation-issues"></a>安裝後問題
#### <a name="server-doesnt-appear-in-service-map"></a>伺服器未出現在服務對應中
如果 Dependency Agent 安裝成功，但是在服務對應解決方案中沒有看到您的伺服器︰
* Dependency Agent 是否安裝成功？ 您可以查看是否已安裝服務並且執行，以便驗證。<br><br>
**Windows**︰尋找名稱為「Microsoft Dependency Agent」的服務。<br>
**Linux**：尋找執行中的處理序 "microsoft-dependency-agent"。

* 您是否屬於 [Operations Management Suite/Log Analytics 的免費定價層](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions#offers-and-pricing-tiers)？ 免費方案允許最多五個唯一的服務對應伺服器。 任何後續伺服器則不會顯示在服務對應中，即使先前五個伺服器不會再傳送資料。

* 您的伺服器是否將記錄和效能資料傳送到 Log Analytics？ 移至 [記錄搜尋]，然後為您的電腦執行下列查詢︰ 

        Usage | where Computer == "admdemo-appsvr" | summarize sum(Quantity), any(QuantityUnit) by DataType

您是否在結果中取得各種事件？ 是否為最新的資料？ 如果是，您的 Log Analytics 代理程式會正確運作並與 Log Analytics 通訊。 如果不是，請檢查您伺服器上的代理程式︰[適用於 Windows 的 Log Analytics 代理程式疑難排解](https://support.microsoft.com/help/3126513/how-to-troubleshoot-monitoring-onboarding-issues)或[適用於 Linux 的 Log Analytics 代理程式疑難排解](../log-analytics/log-analytics-agent-linux-support.md)。

#### <a name="server-appears-in-service-map-but-has-no-processes"></a>伺服器顯示在服務對應中，但是沒有任何處理序
如果在服務對應中看到您的伺服器，但是沒有處理序或連線資料，則表示相依性代理程式已安裝且正在執行，但未載入核心驅動程式。 

請檢查 C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log file (Windows) 檔案或 /var/opt/microsoft/dependency-agent/log/service.log 檔案 (Linux)。 檔案的最後幾行應該會指出未載入核心的原因。 例如，若您更新過核心，在 Linux 上可能會不受支援。

## <a name="next-steps"></a>後續步驟
- 部署和設定後，了解如何[使用服務對應]( monitoring-service-map.md)。
