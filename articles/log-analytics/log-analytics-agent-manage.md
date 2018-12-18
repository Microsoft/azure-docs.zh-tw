---
title: 管理 Azure Log Analytics 代理程式 | Microsoft Docs
description: 本文描述 Microsoft Monitoring Agent (MMA) 部署在機器上的生命週期內，您通常會執行的不同管理工作。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/30/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 908418dffaffc25be320bd0008edf03493aa4e55
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128787"
---
# <a name="managing-and-maintaining-the-log-analytics-agent-for-windows-and-linux"></a>管理和維護適用於 Windows 和 Linux 的 Log Analytics 代理程式

初始部署 Log Analytics 的 Windows 或 Linux 代理程式後，您可能需要重新設定代理程式，或是在代理程式達到生命週期的淘汰階段時從電腦中移除它。  您可以手動或自動的方式輕鬆地管理這些例行維護工作，後者可以降低操作錯誤和費用。

## <a name="adding-or-removing-a-workspace"></a>新增或移除工作區 

### <a name="windows-agent"></a>Windows 代理程式

#### <a name="update-settings-from-control-panel"></a>從控制台更新設定

1. 以具有系統管理權限的帳戶登入電腦。
2. 開啟 [ **控制台**]。
3. 選取 [Microsoft Monitoring Agent]，然後按一下 [Azure Log Analytics (OMS)] 索引標籤。
4. 若要移除工作區，請選取工作區，然後按一下 [移除]。 針對您希望代理程式停止向其報告的任何其他工作區，重複此步驟。
5. 若要新增工作區，請按一下 [新增]，然後在 [新增 Log Analytics 工作區] 對話方塊中，貼上工作區識別碼和工作區索引鍵 (主索引鍵)。 如果電腦應該向 Azure Government 雲端中的 Log Analytics 工作區報告，請從 Azure 雲端下拉式清單中選取 [Azure US Government]。 
6. 按一下 [確定]  以儲存變更。

#### <a name="remove-a-workspace-using-powershell"></a>使用 PowerShell 移除工作區 

```PowerShell
$workspaceId = "<Your workspace Id>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.RemoveCloudWorkspace($workspaceId)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-commercial-using-powershell"></a>使用 PowerShell 在 Azure Commercial 中新增工作區 

```PowerShell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>”
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-for-us-government-using-powershell"></a>使用 PowerShell 在 Azure US Government 中新增工作區 

```PowerShell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>”
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey, 1)
$mma.ReloadConfiguration()
```

>[!NOTE]
>如果您先前使用過命令列或指令碼來安裝或設定代理程式，`EnableAzureOperationalInsights` 會換成 `AddCloudWorkspace` 和 `RemoveCloudWorkspace`。
>

### <a name="linux-agent"></a>Linux 代理程式
如果您決定要向不同工作區註冊 Linux 代理程式，或想要從其設定中移除工作區，下列步驟示範如何重新設定該代理程式。  

1.  若要確認已向工作區註冊該代理程式，請執行下列命令。

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l` 

    它應該會傳回類似下列範例的狀態： 

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

    請務必確認狀態也會顯示代理程式正在執行，否則，重新設定代理程式的後續步驟將無法順利完成。  

2. 如果已向工作區註冊，執行下列命令來移除已註冊的工作區。  否則，如果尚未註冊，請繼續進行下一個步驟。

    `/opt/microsoft/omsagent/bin/omsadmin.sh -X`  
    
3. 若要向不同工作區註冊，執行命令 `/opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <shared key> [-d <top level domain>]` 
4. 若要確認您的變更已生效，請執行下列命令。

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l` 

    它應該會傳回類似下列範例的狀態： 

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

代理程式服務不需重新啟動，就能使變更生效。

## <a name="update-proxy-settings"></a>更新 Proxy 設定 
若要在部署之後將代理程式設定為透過 Proxy 伺服器或 [OMS 閘道](log-analytics-oms-gateway.md)來與服務通訊，可使用下列其中一種方法來完成這項工作。

### <a name="windows-agent"></a>Windows 代理程式

#### <a name="update-settings-using-control-panel"></a>使用控制台更新設定

1. 以具有系統管理權限的帳戶登入電腦。
2. 開啟 [ **控制台**]。
3. 選取 [Microsoft Monitoring Agent]，然後按一下 [Proxy 設定] 索引標籤。
4. 按一下 [使用 Proxy 伺服器]，並提供 Proxy 伺服器或閘道的 URL 和連接埠號碼。 如果您的 Proxy 伺服器或 OMS 閘道要求驗證，請輸入要驗證的使用者名稱和密碼，然後按一下 [確定]。 

#### <a name="update-settings-using-powershell"></a>使用 PowerShell 更新設定 

複製下列 PowerShell 程式碼範例，以您環境的特定資訊加以更新，並使用 PS1 副檔名進行儲存。  在直接連線到 Log Analytics 服務的每一部電腦上執行指令碼。

```PowerShell
param($ProxyDomainName="https://proxy.contoso.com:30443", $cred=(Get-Credential))

# First we get the Health Service configuration object.  We need to determine if we
#have the right update rollup with the API we need.  If not, no need to run the rest of the script.
$healthServiceSettings = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'

$proxyMethod = $healthServiceSettings | Get-Member -Name 'SetProxyInfo'

if (!$proxyMethod)
{
     Write-Output 'Health Service proxy API not present, will not update settings.'
     return
}

Write-Output "Clearing proxy settings."
$healthServiceSettings.SetProxyInfo('', '', '')

$ProxyUserName = $cred.username

Write-Output "Setting proxy to $ProxyDomainName with proxy username $ProxyUserName."
$healthServiceSettings.SetProxyInfo($ProxyDomainName, $ProxyUserName, $cred.GetNetworkCredential().password)
```  

### <a name="linux-agent"></a>Linux 代理程式
如果您的 Linux 電腦需要透過 Proxy 伺服器或 OMS 閘道來與 Log Analytics 通訊，請執行下列步驟。  Proxy 組態值的語法如下：`[protocol://][user:password@]proxyhost[:port]`。  *proxyhost* 屬性接受 Proxy 伺服器的完整網域名稱或 IP 位址。

1. 藉由執行下列命令來編輯檔案 `/etc/opt/microsoft/omsagent/proxy.conf`，並將值變更為您的特定設定。

    ```
    proxyconf="https://proxyuser:proxypassword@proxyserver01:30443"
    sudo echo $proxyconf >>/etc/opt/microsoft/omsagent/proxy.conf
    sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf 
    ```

2. 執行下列命令來重新啟動代理程式： 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 

## <a name="uninstall-agent"></a>解除安裝代理程式
使用命令列或安裝精靈，透過下列其中一個程序將 Windows 或 Linux 代理程式解除安裝。

### <a name="windows-agent"></a>Windows 代理程式

#### <a name="uninstall-from-control-panel"></a>從控制台解除安裝
1. 以具有系統管理權限的帳戶登入電腦。  
2. 在 [控制台] 中按一下 [程式和功能]。
3. 在 [程式和功能] 中，依序按一下 [Microsoft Monitoring Agent]、[解除安裝] 和 [是]。

>[!NOTE]
>按兩下 **MMASetup-<platform>.exe** 也可以執行代理程式安裝精靈，您可從 Azure 入口網站中的工作區下載此檔案。

#### <a name="uninstall-from-the-command-line"></a>從命令列解除安裝
為代理程式下載的檔案是使用 IExpress 建立的獨立安裝套件。  套件中包含代理程式的安裝程式和支援檔案，這些檔案需要先解壓縮，才能使用命令列正確解除安裝，如下列範例所示。 

1. 以具有系統管理權限的帳戶登入電腦。  
2. 若要解壓縮代理程式安裝檔案，請從提升權限的命令提示字元執行 `extract MMASetup-<platform>.exe`，它會提示檔案解壓縮的路徑。  或者，您可以傳遞 `extract MMASetup-<platform>.exe /c:<Path> /t:<Path>` 引數來指定路徑。  如需 IExpress 支援的命令列參數詳細資訊，請參閱 [IExpress 的命令列參數](https://support.microsoft.com/help/197147/command-line-switches-for-iexpress-software-update-packages)，然後更新範例以符合您的需求。
3. 在出現提示時輸入 `%WinDir%\System32\msiexec.exe /x <Path>:\MOMAgent.msi /qb`。  

### <a name="linux-agent"></a>Linux 代理程式
若要移除代理程式，請在 Linux 電腦上執行下列命令。  *--purge* 引數可將代理程式及其組態完全移除。

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

## <a name="configure-agent-to-report-to-an-operations-manager-management-group"></a>將代理程式設定為向 Operations Manager 管理群組報告

### <a name="windows-agent"></a>Windows 代理程式
執行下列步驟，將適用於 Windows 的 OMS 代理程式設定為向 System Center Operations Manager 管理群組報告。 

1. 以具有系統管理權限的帳戶登入電腦。
2. 開啟 [ **控制台**]。 
3. 按一下 [Microsoft Monitoring Agent]，然後按一下 [Operations Manager] 索引標籤。
4. 如果 Operations Manager 伺服器已與 Active Directory 整合，請按一下 [自動更新來自 AD DS 的管理群組指派] 。
5. 按一下 [新增] 以開啟 [新增管理群組] 對話方塊。
6. 在 [管理群組名稱] 欄位中，輸入您的管理群組名稱。
7. 在 [主要管理伺服器] 欄位中，輸入主要管理伺服器的電腦名稱。
8. 在 [管理伺服器連接埠]  欄位中，輸入 TCP 連接埠號碼。
9. 在 [代理程式動作帳戶] 頁面下，選擇本機系統帳戶或本機網域帳戶。
10. 按一下 [確定] 關閉 [新增管理群組] 對話方塊，然後按一下 [確定] 關閉 [Microsoft 監視代理程式內容] 對話方塊。

### <a name="linux-agent"></a>Linux 代理程式
執行下列步驟設定 OMS Agent for Linux 向 System Center Operations Manager 管理群組報告。 

1. 編輯 `/etc/opt/omi/conf/omiserver.conf`
2. 確認以 `httpsport=` 開頭的這一行會定義連接埠 1270。 例如：`httpsport=1270`
3. 重新啟動 OMI 伺服器：`sudo /opt/omi/bin/service_control restart`

## <a name="next-steps"></a>後續步驟

如果您安裝或管理代理程式時遇到的問題，請參閱[針對 Linux 代理程式進行疑難排解](log-analytics-agent-linux-support.md)。  