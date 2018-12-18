---
title: App Service on Azure Stack 更新 1 版本資訊 | Microsoft Docs
description: 了解適用於 App Service on Azure Stack 更新 1 的新功能、已知問題，以及可下載更新的位置。
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2018
ms.author: anwestg
ms.reviewer: sethm
ms.openlocfilehash: ee6e4397345b4cb169e7e22d951d4c4fdff5b7b7
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078710"
---
# <a name="app-service-on-azure-stack-update-1-release-notes"></a>App Service on Azure Stack 更新 1 版本資訊

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

這些版本資訊說明 Azure App Service on Azure Stack 更新 1 的增強功能和修正，以及任何已知問題。 已知問題分為直接與部署、更新程序相關的問題，以及組建 (安裝後) 的相關問題。

> [!IMPORTANT]
> 在部署 Azure App Service 之前，請先將 1802 更新套用到您的 Azure Stack 整合式系統，或部署最新的 Azure Stack 開發套件。
>
>

## <a name="build-reference"></a>建置參考

App Service on Azure Stack 更新 1 組建編號是 **69.0.13698.9**

### <a name="prerequisites"></a>必要條件

> [!IMPORTANT]
> 新的 Azure App Service on Azure Stack 部署現在需要[三主體萬用字元憑證](azure-stack-app-service-before-you-get-started.md#get-certificates)，因為現在已改進在 Azure App Service 中處理 Kudu 之 SSO 的方式。 新主體為 **\*.sso.appservice.\<region\>.\<domainname\>.\<extension\>**
>
>

開始部署之前，請參閱[「在您開始之前」文件](azure-stack-app-service-before-you-get-started.md)。

### <a name="new-features-and-fixes"></a>新功能和修正

Azure App Service on Azure Stack 更新 1 包含下列改良功能和修正：

- **Azure App Service 的高可用性** - Azure Stack 1802 更新可以讓工作負載跨容錯網域部署。 因此 App Service 基礎結構可以容錯，因為它會跨容錯網域進行部署。 根據預設，Azure App Service 的所有新部署都會有這項功能，不過對於在套用 Azure Stack 1802 以前就完成的部署，請參閱 [App Service 容錯網域文件](azure-stack-app-service-fault-domain-update.md)

- **在現有的虛擬網路中部署** - 客戶現在可以在現有的虛擬網路內部署 App Service on Azure Stack。 在現有的虛擬網路中部署可以讓客戶透過私人連接埠連線至 SQL Server 和檔案伺服器，這些項目是 Azure App Service 必須的。 在部署期間，客戶可以選取在現有的虛擬網路中部署，但是在部署之前，[必須建立要供 App Service 使用的子網路](azure-stack-app-service-before-you-get-started.md#virtual-network)。

- **App Service 租用戶、系統管理員、Functions 入口網站和 Kudu 工具**的更新。 與 Azure Stack 入口網站 SDK 版本保持一致。

- **下列應用程式架構和工具的更新**：
    - 已新增 **.Net Core 2.0** 支援
    - 已新增 **Node.JS** 版本：
        - 6.11.2
        - 6.11.5
        - 7.10.1
        - 8.0.0
        - 8.1.4
        - 8.4.0
        - 8.5.0
        - 8.7.0
        - 8.8.1
        - 8.9.0
    - 已新增 **NPM** 版本：
        - 3.10.10
        - 4.2.0
        - 5.0.0
        - 5.0.3
        - 5.3.0
        - 5.4.2
        - 5.5.1
    - 已新增 **PHP** 更新：
        - 5.6.32
        - 7.0.26 (x86 和 x64)
        - 7.1.12 (x86 和 x64)
    - 將 **Git for Windows** 更新為 v 2.14.1
    - 將 **Mercurial** 更新為 v4.5.0

  - 已在 App Service 租用戶入口網站中的自訂網域功能內，新增**僅限 HTTPS** 功能的支援。 

  - 已在 Azure Functions 的自訂儲存體選擇器中新增儲存體連線的驗證 

#### <a name="fixes"></a>修正

- 建立離線部署套件時，如果客戶從 App Service 安裝程式開啟資料夾，不會再收到拒絕存取錯誤訊息

- 解決在 App Service 租用戶入口網站中使用自訂網域功能時發生的問題。

- 防止客戶在安裝期間使用保留的系統管理員名稱

- 使用**已加入網域**檔案伺服器啟用 App Service 部署

- 已改善指令碼中的 Azure Stack 根憑證擷取，現在會在 App Service 安裝程式中驗證根憑證。

- 修正了當訂用帳戶 (在 Microsoft.Web 命名空間中包含資源) 遭到刪除時會將不正確的狀態傳回給 Azure Resource Manager。

### <a name="known-issues-with-the-deployment-process"></a>關於部署程序的已知問題

- 憑證驗證錯誤

有些客戶在於整合式系統上部署時，因為 App Service 安裝程式中的驗證過度嚴格，而導致提供憑證給安裝程式時發生問題。 App Service 安裝程式已經重新發行，客戶應該[下載已更新的安裝程式](https://aka.ms/appsvconmasinstaller)。 如果您在使用已更新的安裝程式來驗證憑證時持續發生問題，請連絡支援人員。

- 從整合式系統擷取 Azure Stack 根憑證時發生問題。

Get-AzureStackRootCert.ps1 發生錯誤，導致客戶在未安裝 Azure Stack 根憑證的電腦上執行該指令碼時，無法擷取該根憑證。 此指令碼現在也已重新發行，可解決此問題，客戶可[下載已更新的協助程式指令碼](https://aka.ms/appsvconmashelpers)。 如果您在使用已更新的指令碼來擷取根憑證時持續發生問題，請連絡支援人員。

### <a name="known-issues-with-the-update-process"></a>關於更新程序的已知問題

- Azure App Service on Azure Stack 更新 1 的更新沒有任何已知問題。

### <a name="known-issues-post-installation"></a>已知問題 (安裝後)

- 位置交換無法運作

網站位置交換功能在此版本中故障。 若要還原此功能，請完成下列步驟：

1. 將 ControllersNSG 網路安全性群組修改成**允許**透過遠端桌面連線至 App Service 控制器執行個體。 將 AppService.local 取代成您部署 App Service 的資源群組名稱。

    ```powershell
      Add-AzureRmAccount -EnvironmentName AzureStackAdmin

      $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

      $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

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

2. 在「Azure Stack 系統管理員」入口網站中，瀏覽至 [虛擬機器] 底下的 [CN0-VM]，然後按一下 [連線] 以開啟與控制器執行個體的遠端桌面工作階段。 使用在部署 App Service 期間指定的認證。
3. **以系統管理員身分啟動 PowerShell**，然後執行下列指令碼

    ```powershell
        Import-Module appservice

        $sm = new-object Microsoft.Web.Hosting.SiteManager

        if($sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus=$true)
        {
          $sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus=$false
        #  'Slot swap mode reverted'
        }
        
        # Confirm new setting is false
        $sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus
        
        # Commit Changes
        $sm.CommitChanges()

        Get-AppServiceServer -ServerType ManagementServer | ForEach-Object Repair-AppServiceServer
        
    ```

4. 關閉遠端桌面工作階段。
5. 將 ControllersNSG 網路安全性群組還原成**拒絕**透過遠端桌面連線至 App Service 控制器執行個體。 將 AppService.local 取代成您部署 App Service 的資源群組名稱。

    ```powershell

        Add-AzureRmAccount -EnvironmentName AzureStackAdmin

        $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

        $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

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
- 將應用程式服務部署在現有的虛擬網路中且只能在私人網路上使用檔案伺服器時，背景工作角色無法連線到檔案伺服器。
 
如果您選擇要部署到現有的虛擬網路並以內部 IP 位址連線到檔案伺服器，便必須新增輸出安全性規則，以啟用背景工作角色子網路與檔案伺服器之間的 SMB 流量。 若要這樣做，請移至管理入口網站中的 WorkersNsg，然後使用下列屬性新增輸出安全性規則：
 * 來源：任何
 * 來源連接埠範圍：*
 * 目的地：IP 位址
 * 目的地 IP 位址範圍：檔案伺服器的 IP 範圍
 * 目的地連接埠範圍：445
 * 通訊協定：TCP
 * 動作：允許
 * 優先順序：700
 * 名稱：Outbound_Allow_SMB445

### <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack"></a>雲端管理員操作 Azure App Service on Azure Stack 時的已知問題

請參閱 [Azure Stack 1802 版本資訊](azure-stack-update-1802.md)中的文件

## <a name="next-steps"></a>後續步驟

- 如需 Azure App Service 的概觀，請參閱 [Azure App Service on Azure Stack 概觀](azure-stack-app-service-overview.md)。
- 如需如何準備部署 App Service on Azure Stack 的詳細資訊，請參閱[開始使用 App Service on Azure Stack 之前](azure-stack-app-service-before-you-get-started.md)。
