---
title: Azure Stack 1805 更新 | Microsoft Docs
description: 了解 Azure Stack 整合系統 1805 更新的新功能，包括已知問題和更新下載位置。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2018
ms.author: sethm
ms.reviewer: justini
ms.openlocfilehash: 5224829b59e7ad425fb01a242a702e0d4cad8657
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392375"
---
# <a name="azure-stack-1805-update"></a>Azure Stack 1805 更新

「適用於：Azure Stack 整合系統」

本文說明 1805 更新套件中的改良功能與修正、此版本的已知問題，以及更新下載位置。 已知問題分為直接與更新程序相關的問題，以及組建 (安裝後) 的相關問題。

> [!IMPORTANT]        
> 這個更新程式封裝僅適用於 Azure Stack 整合系統。 請勿將此更新套件套用至 Azure Stack 開發套件。

## <a name="build-reference"></a>建置參考    
Azure Stack 1805 更新組建編號為 **1.1805.1.47**。  

> [!TIP]  
> 根據客戶的意見反應，Microsoft Azure Stack 所使用的版本結構描述會進行更新。  從 1805 這個更新開始，新的結構描述將更能代表現行的雲端版本。  
> 
> 版本結構描述目前為 Version.YearYearMonthMonth.MinorVersion.BuildNumber，其中的第二和第三個集合代表版本和版次。 例如，1805.1 代表 1805 製造 (RTM) 版的版次。  


### <a name="new-features"></a>新功能
此更新包含下列適用於 Azure Stack 的改良功能。
<!-- 2297790 - IS, ASDK --> 
- **Azure Stack 現已將 Syslog 用戶端**納入為預覽功能。 此用戶端允許將與 Azure Stack 基礎結構相關的稽核和安全性記錄，轉寄到 Azure Stack 外的 Syslog 伺服器或安全性資訊與事件管理 (SIEM) 軟體。 目前，Syslog 用戶端只支援透過預設連接埠 514 的未驗證 UDP 連線。 每個 Syslog 訊息的承載會以通用事件格式 (CEF) 來設定格式。 

  若要設定 Syslog 用戶端，請使用特殊權限端點中所公開的 **Set-SyslogServer** Cmdlet。 

  在此預覽版中，您可能會看到下列三種警示。 當 Azure Stack 顯示這些警示時，警示會內含「描述」和「補救」指導方針。 
  - 標題：程式碼完整性關閉  
  - 標題：程式碼完整性處於稽核模式 
  - 標題：已建立使用者帳戶

  此功能目前還是預覽版，所以不應在生產環境中仰賴此功能。   




### <a name="fixed-issues"></a>已修正的問題

<!-- # - applicability -->
- 我們已修正下列問題：無法在系統管理員入口網站中，[從下拉式清單開啟新的支援要求](azure-stack-manage-portals.md#quick-access-to-help-and-support)。 此選項目前已可正常運作。 

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**。


<!-- # Additional releases timed with this update    -->



## <a name="before-you-begin"></a>開始之前    

### <a name="prerequisites"></a>必要條件
- 請先安裝 Azure Stack [1804 更新](azure-stack-update-1804.md)，然後才套用 Azure Stack 1805 更新。  
- 安裝最新的可用 [1804 版更新或 Hotfix](azure-stack-update-1804.md#post-update-steps)。   
- 開始安裝更新 1805 之前，請先執行 [Test-AzureStack](azure-stack-diagnostic-test.md) 來驗證 Azure Stack 的狀態，並解決所發現的一切運作問題。 也請檢閱作用中警示，並將所有需要採取動作的警示解決。 

### <a name="known-issues-with-the-update-process"></a>關於更新程序的已知問題   
- 在安裝 1805 更新期間，您可能會看到警示，標題為「錯誤 - FaultType UserAccounts.New 的範本已遺失」。  您可以放心地忽略這些警示。 在 1805 更新完成之後，這些警示會自動關閉。   

- <!-- 2489559 - IS --> 在此更新安裝期間，請勿嘗試建立虛擬機器。 如需管理更新的詳細資訊，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md#plan-for-updates)。


### <a name="post-update-steps"></a>更新後步驟
在安裝 1805 之後，安裝任何適用的 Hotfix。 如需詳細資訊，請檢視下列知識庫文章，以及我們的[服務原則](azure-stack-servicing-policy.md)。  
 - [KB 4344102 - Azure Stack Hotfix 1.1805.7.57](https://support.microsoft.com/help/4344102).


## <a name="known-issues-post-installation"></a>已知問題 (安裝後)
以下是此組建版本的安裝後已知問題。

### <a name="portal"></a>入口網站  

- Azure Stack 技術文件會以最新版本為主。 由於各版本的入口網站會有所變更，您在使用 Azure Stack 入口網站時所看到的項目，可能與您在文件中所看到的內容不同。 

- <!-- 2931230 – IS  ASDK --> 無法刪除以附加方案形式新增至使用者訂用帳戶的方案 (即使您從使用者訂用帳戶中移除此方案)。 此方案會持續保留，直到參考附加方案的訂用帳戶也遭到刪除為止。 

- <!-- TBD - IS ASDK --> 您無法搭配此版本的 Azure Stack 使用 OEM 延伸模組套件來套用驅動程式更新。  此問題目前沒有因應方式。

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [概觀] 時，[基本資訊] 窗格中並未顯示資訊。  [基本資訊] 窗格會顯示帳戶相關資訊，例如其*資源群組*、*位置*和*訂用帳戶識別碼*。  [概觀] 中有其他選項可供存取，例如 [服務] 和 [監視]，以及用來 [在 Explorer 中開啟] 或 [刪除儲存體帳戶] 的選項。 

  若要檢視未能看到的資訊，請使用 [Get-azureRMstorageaccount](https://docs.microsoft.com/powershell/module/azurerm.storage/get-azurermstorageaccount?view=azurermps-6.2.0) PowerShell Cmdlet。 

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [標籤] 時，資訊無法載入因此並未顯示。  

  若要檢視未能看到的資訊，請使用 [Get-AzureRmTag](https://docs.microsoft.com/powershell/module/azurerm.tags/get-azurermtag?view=azurermps-6.2.0) PowerShell Cmdlet。


- <!-- 2332636 - IS --> 當您的 Azure Stack 身分識別系統使用 AD FS，而您又更新為這一版的 Azure Stack 時，預設提供者訂用帳戶的預設擁有者會重設為內建的 **CloudAdmin** 使用者。  
  因應措施：若要在安裝此更新後解決此問題，請使用[觸發自動化以在 Azure Stack 中設定宣告提供者信任](azure-stack-integrate-identity.md#trigger-automation-to-configure-claims-provider-trust-in-azure-stack-1)程序中的步驟 3，來重設預設提供者訂用帳戶的擁有者。   

- <!-- TBD - IS ASDK --> 某些系統管理訂用帳戶類型無法使用。  當您將 Azure Stack 升級至此版本時，主控台中並未顯示兩個 [1804 版所導入的](azure-stack-update-1804.md#new-features)訂用帳戶類型。 這是預期行為。 無法使用的訂用帳戶類型為「計量訂用帳戶」和「取用訂用帳戶」。 自 1804 版起，這些訂用帳戶類型就已經會在新的 Azure Stack 環境中顯示，只是還未就緒以供使用。 請繼續使用「預設提供者」訂用帳戶類型。  

- <!-- 2403291 - IS ASDK --> 您可能還無法使用系統管理員和使用者入口網站底部的水平捲軸。 如果您無法存取水平捲軸，請使用階層連結來瀏覽至入口網站中的上一個刀鋒視窗，方法是從入口網站左上角的階層連結清單中，選取您想要檢視的刀鋒視窗名稱。
  ![階層連結](media/azure-stack-update-1804/breadcrumb.png)

- <!-- TBD - IS --> 可能無法在管理員入口網站中檢視計算或儲存體資源。 之所以發生此問題，是因為在更新安裝期間發生錯誤，且誤將更新報告為成功。 如果發生此問題，請連絡 Microsoft 客戶支援服務以尋求協助。

- <!-- TBD - IS --> 您可能會在入口網站中看到空白的儀表板。 若要復原儀表板，請選取位於入口網站右上角的齒輪圖示，然後選取 [還原預設設定]。

- <!-- TBD - IS ASDK --> 刪除使用者訂用帳戶會產生孤立的資源。 因應措施是，先刪除使用者資源或整個資源群組，然後再刪除使用者訂用帳戶。

- <!-- TBD - IS ASDK --> 您無法使用 Azure Stack 入口網站來檢視對您訂用帳戶的權限。 因應措施是，使用 PowerShell 來確認權限。


### <a name="health-and-monitoring"></a>健康情況和監視
- <!-- 1264761 - IS ASDK --> 您可能會看到「健康情況控制器」元件出現具有下列詳細資料的警示：  
- 
   警示 #1：
   - NAME：基礎結構角色狀況不良
   - SEVERITY：警告
   - COMPONENT：健康情況控制器
   - DESCRIPTION：健康情況控制器活動訊號掃描器無法使用。 這可能會影響健康情況報告和計量。  

  警示 #2：
   - NAME：基礎結構角色狀況不良
   - SEVERITY：警告
   - COMPONENT：健康情況控制器
   - DESCRIPTION：健康情況控制器錯誤掃描器無法使用。 這可能會影響健康情況報告和計量。

  您可以放心地忽略 #1 和 #2 的警示，這兩個警示會在一段時間過後自動關閉。 

  您也可能會看到下列「容量」警示。 針對此警示，描述中識別的可用記憶體百分比可能不同：  

  警示 #3：
   - NAME：低記憶體容量
   - 嚴重性：嚴重
   - 元件：容量
   - 描述：區域已取用超過 80.00% 的可用記憶體。 使用大量記憶體建立虛擬機器可能會失敗。  

  在此版本的 Azure Stack 中，此警示可能會不正確地引發。 如果租用戶虛擬機器仍可成功地繼續部署，您就可以放心地忽略此警示。 
  
  警示 #3 並不會自動關閉。 如果您關閉此警示，Azure Stack 會在 15 分鐘內建立同一個警示。  

- <!-- 2368581 - IS. ASDK --> 身為 Azure Stack 操作員，如果您收到記憶體不足的警示，而且租用戶虛擬機器無法部署並出現「Fabric VM 建立錯誤」，則可能表示 Azure Stack 戳記的可用記憶體不足。 請使用 [Azure Stack 容量規劃工具](https://gallery.technet.microsoft.com/Azure-Stack-Capacity-24ccd822)來充分了解可供工作負載使用的容量。 


### <a name="compute"></a>計算
- <!-- TBD - IS, ASDK --> 在為虛擬機器部署選取虛擬機器大小時，某些 F 系列的 VM 大小不會在您建立 VM 時顯示在大小選取器中。 選取器中不會顯示下列 VM 大小：*F8s_v2*、*F16s_v2*、*F32s_v2* 和 *F64s_v2*。  
  因應措施是使用下列其中一個方法來部署虛擬機器。 在每個方法中，您需要指定想要使用的虛擬機器大小。

  - **Azure Resource Manager 範本：** 當您使用範本時，請將範本中的 *vmSize* 設定為等於您想要使用的虛擬機器大小。 例如，使用以下項目來部署會使用 *F32s_v2* 大小的虛擬機器：  

    ```
        "properties": {
        "hardwareProfile": {
                "vmSize": "Standard_F32s_v2"
        },
    ```  
  - **Azure CLI：** 您可以使用 [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create) 命令，並且指定虛擬機器大小作為參數，類似於 `--size "Standard_F32s_v2"`。

  - **PowerShell：** 使用 PowerShell，您可以使用 [New-AzureRMVMConfig](https://docs.microsoft.com/powershell/module/azurerm.compute/new-azurermvmconfig?view=azurermps-6.0.0) 搭配指定虛擬機器大小的參數，類似於 `-VMSize "Standard_F32s_v2"`。


- <!-- TBD - IS ASDK --> 無法在入口網站中使用虛擬機器擴展集的調整設定。 您可以使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-powershell#change-the-capacity-of-a-scale-set) 作為因應措施。 由於 PowerShell 版本差異，您必須使用 `-Name` 參數，而不是 `-VMScaleSetName`。

- <!-- TBD - IS --> 當您移至 [新增] > [計算] > [可用性設定組] 在入口網站中建立可用性設定組時，您只能以一個容錯網域和一個更新網域建立可用性設定組。 因應措施是在建立新的虛擬機器時，使用 PowerShell、CLI 或從入口網站建立可用性設定組。

- <!-- TBD - IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結至 DS 系列 VM 的資料磁碟數目會不正確。 DS 系列 VM 可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD - IS ASDK --> 當 VM 映像建立失敗時，可能會在 VM 映像計算刀鋒視窗上新增一個您無法刪除的失敗項目。

  因應措施是，建立一個具有虛擬 VHD 的新 VM 映像，您可以透過 Hyper-V 建立此 VHD (New-VHD -Path C:\dummy.vhd -Fixed -SizeBytes 1 GB)。 此程序應該可以修正防止刪除失敗項目的問題。 接著，在建立虛擬映像後的 15 分鐘之後，您便可以順利刪除它。

  您可以接著嘗試下載先前失敗的 VM 映像。

- <!-- TBD - IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。  


### <a name="networking"></a>網路功能
- <!-- TBD - IS ASDK --> 您無法在系統管理員或使用者入口網站中建立使用者定義的路由。 請使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-powershell) 作為因應措施。

- <!-- 1766332 - IS ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 2388980 - IS ASDK --> 在建立 VM 並與公用 IP 位址建立關聯之後，您就無法將 VM 與該 IP 位址取消關聯。 取消關聯看似可以運作，但先前指派的公用 IP 位址會繼續與原始 VM 保持關聯。

  目前，您只能將新的公用 IP 位址用於新建立的 VM。

  即使您將 IP 位址重新指派給新的 VM (通常稱為 *VIP 交換*)，還是會發生這種行為。 之後透過此 IP 位址連線的所有嘗試都會導致連線到原始 VM，而不是新的 VM。

- <!-- 2292271 - IS ASDK --> 如果供應項目和方案與租用戶訂用帳戶相關聯，而您將屬於供應項目和方案一部分的網路資源提高其配額限制，則新的限制不會套用到該訂用帳戶。 不過，新的限制會套用到配額增加後所建立的新訂用帳戶中。

  若要解決這個問題，請使用附加方案，在方案已與訂用帳戶產生關聯時，增加網路配額。 如需詳細資訊，請參閱如何[提供附加方案](azure-stack-subscribe-plan-provision-vm.md#to-make-an-add-on-plan-available)。

- <!-- 2304134 IS ASDK --> 您無法刪除有相關聯 DNS 區域資源或路由表資源的訂用帳戶。 若要成功刪除訂用帳戶，您必須先從租用戶訂用帳戶中刪除 DNS 區域和路由表資源。


- <!-- 1902460 - IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 - IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- TBD - IS ASDK --> Azure Stack 不支援在部署 VM 之後將額外的網路介面新增至 VM 執行個體。 如果 VM 需要多個網路介面，必須在部署階段就定義這些網路介面。

- <!-- 2096388 IS --> 您無法使用系統管理員入口網站來更新網路安全性群組的規則。

    App Service 的因應措施：如果您需要透過遠端桌面連線至「控制器」執行個體，則需使用 PowerShell 來修改網路安全性群組內的安全性規則。  以下是一些範例，說明如何「允許」，然後再將設定還原至「拒絕」：  

    - *允許：*

      ```powershell    
      Connect-AzureRmAccount -EnvironmentName AzureStackAdmin

      $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

      $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

      ##This doesn’t work. Need to set properties again even in case of edit

      #Set-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389" -NetworkSecurityGroup $nsg -Access Allow  

      Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
        -Name $RuleConfig_Inbound_Rdp_3389.Name `
        -Description "Inbound_Rdp_3389" `
        -Access Allow `
        -Protocol $RuleConfig_Inbound_Rdp_3389.Protocol `
        -Direction $RuleConfig_Inbound_Rdp_3389.Direction `
        -Priority $RuleConfig_Inbound_Rdp_3389.Priority `
        -SourceAddressPrefix $RuleConfig_Inbound_Rdp_3389.SourceAddressPrefix `
        -SourcePortRange $RuleConfig_Inbound_Rdp_3389.SourcePortRange `
        -DestinationAddressPrefix $RuleConfig_Inbound_Rdp_3389.DestinationAddressPrefix `
        -DestinationPortRange $RuleConfig_Inbound_Rdp_3389.DestinationPortRange

      # Commit the changes back to NSG
      Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
      ```

    - *拒絕：*

        ```powershell

        Connect-AzureRmAccount -EnvironmentName AzureStackAdmin

        $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

        $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

        ##This doesn’t work. Need to set properties again even in case of edit

        #Set-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389" -NetworkSecurityGroup $nsg -Access Allow  

        Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
          -Name $RuleConfig_Inbound_Rdp_3389.Name `
          -Description "Inbound_Rdp_3389" `
          -Access Deny `
          -Protocol $RuleConfig_Inbound_Rdp_3389.Protocol `
          -Direction $RuleConfig_Inbound_Rdp_3389.Direction `
          -Priority $RuleConfig_Inbound_Rdp_3389.Priority `
          -SourceAddressPrefix $RuleConfig_Inbound_Rdp_3389.SourceAddressPrefix `
          -SourcePortRange $RuleConfig_Inbound_Rdp_3389.SourcePortRange `
          -DestinationAddressPrefix $RuleConfig_Inbound_Rdp_3389.DestinationAddressPrefix `
          -DestinationPortRange $RuleConfig_Inbound_Rdp_3389.DestinationPortRange

        # Commit the changes back to NSG
        Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
        ```


### <a name="sql-and-mysql"></a>SQL 和 MySQL

- <!-- TBD - IS --> 僅支援資源提供者在裝載 SQL 或 MySQL 的伺服器上建立項目。 在不是由資源提供者建立的主機伺服器上建立項目，可能會導致不相符的狀態。  

- <!-- IS, ASDK --> 當您為 SQL 和 MySQL 資源提供者建立 SKU 時，[系列] 或 [層] 名稱不支援特殊字元，包括空格和句點。


> [!NOTE]  
> <!-- TBD - IS --> 在更新至 Azure Stack 1805 之後，您可以繼續使用先前部署的 SQL 和 MySQL 資源提供者。  建議您在新版本可供使用時更新 SQL 和 MySQL。 如同 Azure Stack，請循序套用 SQL 和 MySQL 資源提供者的更新。 例如，如果您使用 1803 版，請先套用 1804 版，再更新至 1805 版。      
>   
> 安裝更新 1805 並不會影響使用者目前對 SQL 或 MySQL 資源提供者的使用。
> 無論您使用的資源提供者版本為何，使用者在其資料庫中的資料都會保持原狀，且仍可供存取。    



### <a name="app-service"></a>App Service 方案
- <!-- 2352906 - IS ASDK --> 使用者必須在於訂用帳戶中建立第一個 Azure 函式之前，先註冊儲存體資源提供者。

- <!-- 2489178 - IS ASDK --> 為了向外延展基礎結構 (背景工作、管理、前端角色)，您必須如「計算」的版本資訊所述，使用 PowerShell。

- <!-- TBD - IS ASDK --> 目前，App Service 只能部署到「預設提供者訂用帳戶」。 在未來的更新中，App Service 會部署至 Azure Stack 1804 所導入的新「計量訂用帳戶」。 當系統支援使用計量時，現有部署就會全部遷移至這個新的訂用帳戶類型。


### <a name="usage"></a>使用量  
- <!-- TBD - IS ASDK --> 使用量公用 IP 位址使用計量資料會針對每一筆記錄顯示相同的 EventDateTime 值，而不是建立記錄時顯示的 TimeDate 時間戳記。 目前，您無法使用這項資料來執行公用 IP 位址使用狀況的正確計量。


<!-- #### Identity -->
<!-- #### Marketplace -->


## <a name="download-the-update"></a>下載更新
您可以從[這裡](https://aka.ms/azurestackupdatedownload)下載 Azure Stack 1805 更新套件。


## <a name="see-also"></a>另請參閱
- 若要使用具有特殊權限的端點 (PEP) 來監視和繼續更新，請參閱[使用具有特殊權限的端點來監視 Azure Stack 中的更新](azure-stack-monitor-update.md)。
- 如需 Azure Stack 中的更新管理概觀，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md)。
- 如需如何使用 Azure Stack 套用更新的相關資訊，請參閱[在 Azure Stack 中套用更新](azure-stack-apply-updates.md)。
