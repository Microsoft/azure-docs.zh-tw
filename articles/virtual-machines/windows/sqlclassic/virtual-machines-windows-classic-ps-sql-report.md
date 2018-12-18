---
title: 使用 PowerShell 建立具有原生模式報表伺服器的 VM | Microsoft Docs
description: '本主題說明並逐步引導您在 Azure 虛擬機器中，部署並設定 SQL Server Reporting Services 原生模式報表伺服器。 '
services: virtual-machines-windows
documentationcenter: na
author: markingmyname
manager: erikre
editor: monicar
tags: azure-service-management
ms.assetid: 553af55b-d02e-4e32-904c-682bfa20fa0f
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/11/2017
ms.author: maghan
ms.openlocfilehash: c688de79c985eda9b0898d0f7d989de58f7dbc71
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038014"
---
# <a name="use-powershell-to-create-an-azure-vm-with-a-native-mode-report-server"></a>使用 PowerShell 建立具有原生模式報表伺服器的 Azure VM
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。

本主題說明並逐步引導您在 Azure 虛擬機器中，部署並設定 SQL Server Reporting Services 原生模式報表伺服器。 本文件的步驟採用一連串手動步驟的組合建立虛擬機器，並使用 Windows PowerShell 指令碼設定 VM 上的 Reporting Services。 組態指令碼包含針對 HTTP 或 HTTPS 開啟防火牆連接埠。

> [!NOTE]
> 如果報表伺服器不需要 **HTTPS**，請**略過步驟 2**。
> 
> 在步驟 1 建立 VM 之後，請移至＜使用指令碼設定報表伺服器和 HTTP＞一節。 執行指令碼之後，就可以使用報表伺服器。

## <a name="prerequisites-and-assumptions"></a>必要條件與假設
* **Azure 訂用帳戶**：確認您的 Azure 訂用帳戶中可用的核心數量。 若要建立建議的 VM 大小 **A3**，將需要 **4** 個可用的核心。 若您採用 **A2** 的 VM 大小，則需要 **2** 個可用的核心。
  
  * 若要確認訂用帳戶的核心限制，請在 Azure 入口網站中按一下左窗格中的 [設定]，然後按一下上層功能表中的 [使用方式]。
  * 若要增加核心配額，請連絡 [Azure 支援](https://azure.microsoft.com/support/options/)。 如需 VM 大小的資訊，請參閱 [Azure 的虛擬機器大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* **Windows PowerShell 指令碼**：本主題假設您對 Windows PowerShell 已有基本的技術知識。 如需有關如何使用 Windows PowerShell 的詳細資訊，請參閱下列文章：
  
  * [在 Windows Server 上啟動 Windows PowerShell (英文)](https://docs.microsoft.com/powershell/scripting/setup/starting-windows-powershell)
  * [開始使用 Windows PowerShell (英文)](https://technet.microsoft.com/library/hh857337.aspx)

## <a name="step-1-provision-an-azure-virtual-machine"></a>步驟 1：佈建 Azure 虛擬機器
1. 瀏覽至 Azure 入口網站。
2. 按一下左窗格中的 [虛擬機器]  。
   
    ![Microsoft Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-report/IC660124.gif)
3. 按一下 [新增] 。
   
    ![新增按鈕](./media/virtual-machines-windows-classic-ps-sql-report/IC692019.gif)
4. 按一下 [從資源庫] 。
   
    ![從資源庫新增 VM](./media/virtual-machines-windows-classic-ps-sql-report/IC692020.gif)
5. 按一下 [SQL Server 2014 RTM Standard – Windows Server 2012 R2]  ，然後按一下箭頭以繼續。
   
    ![下一步](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)
   
    如果您需要 Reporting Services 資料驅動的訂用帳戶功能，請選擇 [SQL Server 2014 RTM Enterprise – Windows Server 2012 R2] 。 如需有關 SQL Server 版本及功能支援的詳細資訊，請參閱 [SQL Server 2012 版本支援的功能](https://msdn.microsoft.com/library/cc645993.aspx#Reporting)。
6. 在 [虛擬機器組態]  頁面中，編輯下列欄位：
   
   * 如果有多個 [版本發行日期] ，請選取最新版本。
   * [虛擬機器名稱]：下一個組態頁面中也使用此機器名稱，作為預設的雲端服務 DNS 名稱。 DNS 名稱在 Azure 服務中必須是獨一無二的。 請考慮以能夠描述 VM 用途的電腦名稱命名 VM。 例如 ssrsnativecloud。
   * [層]：標準
   *  是建議 SQL Server 工作負載採用的 VM 大小。 若 VM 只作為報表伺服器使用，那麼除非報表伺服器需處理大量的工作負載，否則 A2 的 VM 大小就已經足夠。 如需 VM 的價格資訊，請參閱 [虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/)。
   * [新的使用者名稱]：系統會使用您提供的名稱建立 VM 上的系統管理員。
   * [新增密碼] 並**確認**。 此密碼將用於新的系統管理員帳戶，因此建議您使用強式密碼。
   * 按 [下一步] 。 ![next](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)
7. 在下一個頁面中，編輯下列欄位：
   
   * [雲端服務]：選取 [建立新的雲端服務]。
   * [雲端服務 DNS 名稱]：這是與 VM 相關聯的雲端服務的公用 DNS 名稱。 預設名稱就是您為 VM 名稱鍵入的名稱。 如果您在本主題的後續步驟中建立信任的 SSL 憑證，系統會將 DNS 名稱作為該憑證的 [核發給]值。
   * [區域/同質群組/虛擬網路]：選擇最靠近您使用者的區域。
   * [儲存體帳戶]：使用自動產生的儲存體帳戶。
   * [可用性設定組]：無。
   * [端點]：保留 [遠端桌面] 和 [PowerShell] 端點，然後根據您的環境新增 HTTP 或 HTTPS 端點。
     
     * [HTTP]：預設的公用和私人連接埠都是 **80**。 請注意，如果您使用的私人連接埠不是 80，請修改 http 指令碼中的 **$HTTPport = 80** 。
     * [HTTPS]：預設的公用和私人連接埠都是 **443**。 安全性最佳作法就是變更私用連接埠，並將防火牆和報表伺服器設為使用私用連接埠。 如需有關端點的詳細資訊，請參閱 [如何設定與虛擬機器的通訊](../classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。 請注意，如果您使用的連接埠不是 443，請變更 HTTPS 指令碼中的 **$HTTPsport = 443** 參數。
   * 按 [下一步]。 ![下一步](./media/virtual-machines-windows-classic-ps-sql-report/IC692021.gif)
8. 在精靈的最後一頁，保持選取預設值 [安裝 VM 代理程式]  。 本主題的步驟不會使用 VM 代理程式，但若您打算保留此 VM，VM 代理程式和延伸模組可讓您增強 CM。  如需有關 VM 代理程式的詳細資訊，請參閱 [VM 代理程式和延伸模組 – 第 1 部分](https://azure.microsoft.com/blog/2014/04/11/vm-agent-and-extensions-part-1/)。 其中一個已安裝並執行的預設延伸模組是「BGINFO」延伸模組，它會在 VM 桌面上顯示系統資訊，例如內部 IP 和可用磁碟空間。
9. 按一下 [完成]。 ![Ok](./media/virtual-machines-windows-classic-ps-sql-report/IC660122.gif)
10. VM 的 [狀態] 在佈建程序進行期間會顯示為 [啟動中 (佈建中)]，然後在 VM 已佈建完成，可供使用時顯示為 [正在執行]。

## <a name="step-2-create-a-server-certificate"></a>步驟 2：建立伺服器憑證
> [!NOTE]
> 如果報表伺服器不需要 HTTPS，可以**略過步驟 2**，並移至**使用指令碼設定報表伺服器和 HTTP** 一節。 透過 HTTP 指令碼快速設定報表伺服器，報表伺服器即可供使用。

您需要信任的 SSL 憑證，才能在 VM 上使用 HTTPS。 您可以根據您的情況，使用下列其中一種方法：

* 由憑證授權單位 (CA) 核發，且受到 Microsoft 信任的有效 SSL 憑證。 CA 根憑證必須透過 Microsoft 根憑證計劃散發。 如需有關此計劃的詳細資訊，請參閱 [Windows 和 Windows Phone 8 SSL 根憑證計劃 (成員 CA)](http://social.technet.microsoft.com/wiki/contents/articles/14215.windows-and-windows-phone-8-ssl-root-certificate-program-member-cas.aspx)，以及 [Microsoft 根憑證計劃簡介](http://social.technet.microsoft.com/wiki/contents/articles/3281.introduction-to-the-microsoft-root-certificate-program.aspx)。
* 自我簽署憑證。 不建議將自我簽署憑證用於生產環境。

### <a name="to-use-a-certificate-created-by-a-trusted-certificate-authority-ca"></a>使用由信任的憑證授權單位 (CA) 所建立的憑證
1. **針對網站向憑證授權單位要求伺服器憑證**。 
   
    您可以使用 Web 伺服器憑證精靈，產生傳送至憑證授權單位的憑證要求檔案 (Certreq.txt)，或產生給線上憑證授權單位的要求。 例如 Windows Server 2012 中的 Microsoft Certificate Services。 視您伺服器憑證提供的識別保證層級而定，憑證授權單位核准您的要求並將憑證檔案傳送給您的時間，可能需要幾天或幾個月。 
   
    如需有關如何要求伺服器憑證的詳細資訊，請參閱下列文章： 
   
   * 使用 [Certreq](https://technet.microsoft.com/library/cc725793.aspx)、[Certreq](https://technet.microsoft.com/library/cc725793.aspx)。
   * 管理 Windows Server 2012 的安全性工具 (英文)。
     
     [管理 Windows Server 2012 的安全性工具 (英文)](https://technet.microsoft.com/library/jj730960.aspx)
     
     > [!NOTE]
     > 信任的 SSL 憑證的 [核發給] 欄位應該與新 VM 的 [雲端服務 DNS 名稱] 相同。

2. **在 Web 伺服器上安裝伺服器憑證**。 此案例中的 Web 伺服器是作為裝載報表伺服器的 VM，而我們會在後面設定 Reporting Services 的步驟中建立網站。 如需有關如何使用憑證 MMC 嵌入式管理單元，在 Web 伺服器上安裝伺服器憑證的詳細資訊，請參閱 [安裝伺服器憑證](https://technet.microsoft.com/library/cc740068)。
   
    若您要使用本主題中的指令碼設定報表伺服器，需將 [指紋]  憑證的值做為指令碼的參數。 有關如何取得憑證指紋的詳細資料，請參閱下一節。
3. 將伺服器憑證指派給報表伺服器。 當您在下一節設定報表伺服器時，便會完成指派。

### <a name="to-use-the-virtual-machines-self-signed-certificate"></a>使用虛擬機器的自我簽署憑證
佈建 VM 時，會在 VM 上建立自我簽署憑證。 憑證名稱與 VM DNS 名稱相同。 為了避免憑證錯誤，VM 本身以及網站的所有使用者都必須信任憑證。

1. 若要信任本機 VM 的憑證根 CA，請將憑證新增至 [信任的根憑證授權單位] 。 以下是所需步驟的摘要。 如需有關如何信任 CA 的詳細步驟，請參閱 [安裝伺服器憑證](https://technet.microsoft.com/library/cc740068)。
   
   1. 在 Azure 入口網站中選取 VM，然後按一下 [連接]。 根據您的瀏覽器設定，可能會提示您儲存 .rdp 檔案以連接至 VM。
      
       ![連接至 Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 請使用您建立 VM 時所設定的使用者 VM 名稱、使用者名稱和密碼。 
      
       例如，下圖的 VM 名稱是 **ssrsnativecloud**，而使用者名稱是 **testuser**。
      
       ![登入包括 VM 名稱](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)
   2. 執行 mmc.exe。 如需詳細資訊，請參閱 [做法：使用 MMC 嵌入式管理單元檢視憑證](https://msdn.microsoft.com/library/ms788967.aspx)。
   3. 在主控台應用程式的 [檔案] 功能表中，新增 [憑證] 嵌入式管理單元，在系統提示時選取 [電腦帳戶]，然後按 [下一步]。
   4. 選取要管理的 [本機電腦]，然後按一下 [完成]。
   5. 按一下 [確認]，展開 [憑證 - 個人] 節點，然後按一下 [憑證]。 憑證是以 VM 的 DNS 名稱來命名，並以**cloudapp.net** 結尾。 以滑鼠右鍵按一下憑證名稱，並按一下 [複製]。
   6. 展開 [信任的根憑證授權] 節點，接著以滑鼠右鍵按一下 [憑證]，然後按一下 [貼上]。
   7. 若要驗證，請按兩下 [信任的根憑證授權]  下的憑證名稱，確認沒有任何錯誤，且您能看到您的憑證。 若您要使用本主題中的 HTTPS 指令碼設定報表伺服器，需將 [指紋] 憑證的值作為指令碼的參數。 **若要取得憑證指紋值**，請完成下列步驟。 在[使用指令碼設定報表伺服器和 HTTPS](#use-script-to-configure-the-report-server-and-HTTPS)一節中，還有可擷取憑證指紋的 PowerShell 範例。
      
      1. 按兩下憑證名稱，例如 ssrsnativecloud.cloudapp.net。
      2. 按一下 [詳細資料]  索引標籤。
      3. 按一下 [指紋] 。 憑證指紋的值會顯示在 [詳細資料] 欄位中，例如 ‎a6 08 3c df f9 0b f7 e3 7c 25 ed a4 ed 7e ac 91 9c 2c fb 2f。
      4. 複製憑證指紋並儲存其值，即可在稍後或現在編輯指令碼。
      5. (*) 執行指令碼之前，請先移除每一組值之間的空格。 例如，之前記下的憑證指紋會成為 a6083cdff90bf7e37c25eda4ed7eac919c2cfb2f。
      6. 將伺服器憑證指派給報表伺服器。 當您在下一節設定報表伺服器時，便會完成指派。

如果您使用自我簽署 SSL 憑證，則憑證的名稱已經符合 VM 的主機名稱。 因此，機器的 DNS 已在全域註冊，且可從任何用戶端存取。

## <a name="step-3-configure-the-report-server"></a>步驟 3：設定報表伺服器
本節將引導您把 VM 設為 Reporting Services 原生模式報表伺服器。 您可以使用下列方法之一來設定報表伺服器：

* 使用指令碼設定報表伺服器
* 使用組態管理員設定報表伺服器。

更詳細的步驟請參閱 [連接到虛擬機器並啟動 Reporting Services 組態管理員](virtual-machines-windows-classic-ps-sql-bi.md#connect-to-the-virtual-machine-and-start-the-reporting-services-configuration-manager)一節。

**驗證注意事項：** 建議的驗證方法是 Windows 驗證，它也是預設的 Reporting Services 驗證。 只有在 VM 上設定的使用者，才能夠存取 Reporting Services 及 Reporting Services 角色指派。

### <a name="use-script-to-configure-the-report-server-and-http"></a>使用指令碼設定報表伺服器和 HTTP
若要使用 Windows PowerShell 指令碼設定報表伺服器，請完成下列步驟。 設定包括 HTTP 而非 HTTPS：

1. 在 Azure 入口網站中選取 VM，然後按一下 [連接]。 根據您的瀏覽器設定，可能會提示您儲存 .rdp 檔案以連接至 VM。
   
    ![連接至 Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 請使用您建立 VM 時所設定的使用者 VM 名稱、使用者名稱和密碼。 
   
    例如，下圖的 VM 名稱是 **ssrsnativecloud**，而使用者名稱是 **testuser**。
   
    ![登入包括 VM 名稱](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)
2. 在 VM 上，以系統管理權限開啟 **Windows PowerShell ISE** 。 Windows Server 2012 預設會安裝 PowerShell ISE。 建議您使用 ISE 代替標準 Windows PowerShell 視窗，以便將指令碼貼到 ISE、修改指令碼，並接著執行指令碼。
3. 在 Windows PowerShell ISE 中，按一下 [檢視] 功能表，然後按一下 [顯示指令碼窗格]。
4. 複製下列指令碼，然後將指令碼貼到 [Windows PowerShell ISE 指令碼] 窗格。
   
        ## This script configures a Native mode report server without HTTPS
        $ErrorActionPreference = "Stop"
   
        $server = $env:COMPUTERNAME
        $HTTPport = 80 # change the value if you used a different port for the private HTTP endpoint when the VM was created.
   
        ## Set PowerShell execution policy to be able to run scripts
        Set-ExecutionPolicy RemoteSigned -Force
   
        ## Utility method for verifying an operation's result
        function CheckResult
        {
            param($wmi_result, $actionname)
            if ($wmi_result.HRESULT -ne 0) {
                write-error "$actionname failed. Error from WMI: $($wmi_result.Error)"
            }
        }
   
        $starttime=Get-Date
        write-host -foregroundcolor DarkGray $starttime StartTime
   
        ## ReportServer Database name - this can be changed if needed
        $dbName='ReportServer'
   
        ## Register for MSReportServer_ConfigurationSetting
        ## Change the version portion of the path to "v11" to use the script for SQL Server 2012
        $RSObject = Get-WmiObject -class "MSReportServer_ConfigurationSetting" -namespace "root\Microsoft\SqlServer\ReportServer\RS_MSSQLSERVER\v12\Admin"
   
        ## Report Server Configuration Steps
   
        ## Setting the web service URL ##
        write-host -foregroundcolor green "Setting the web service URL"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## SetVirtualDirectory for ReportServer site
            write-host 'Calling SetVirtualDirectory'
            $r = $RSObject.SetVirtualDirectory('ReportServerWebService','ReportServer',1033)
            CheckResult $r "SetVirtualDirectory for ReportServer"
   
        ## ReserveURL for ReportServerWebService - port $HTTPport (for local usage)
            write-host "Calling ReserveURL port $HTTPport"
            $r = $RSObject.ReserveURL('ReportServerWebService',"http://+:$HTTPport",1033)
            CheckResult $r "ReserveURL for ReportServer port $HTTPport" 
   
        ## Setting the Database ##
        write-host -foregroundcolor green "Setting the Database"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## GenerateDatabaseScript - for creating the database
            write-host "Calling GenerateDatabaseCreationScript for database $dbName"
            $r = $RSObject.GenerateDatabaseCreationScript($dbName,1033,$false)
            CheckResult $r "GenerateDatabaseCreationScript"
            $script = $r.Script
   
        ## Execute sql script to create the database
            write-host 'Executing Database Creation Script'
            $savedcvd = Get-Location
            Import-Module SQLPS              ## this automatically changes to sqlserver provider
            Invoke-SqlCmd -Query $script
            Set-Location $savedcvd
   
        ## GenerateGrantRightsScript 
            $DBUser = "NT Service\ReportServer"
            write-host "Calling GenerateDatabaseRightsScript with user $DBUser"
            $r = $RSObject.GenerateDatabaseRightsScript($DBUser,$dbName,$false,$true)
            CheckResult $r "GenerateDatabaseRightsScript"
            $script = $r.Script
   
        ## Execute grant rights script
            write-host 'Executing Database Rights Script'
            $savedcvd = Get-Location
            cd sqlserver:\
            Invoke-SqlCmd -Query $script
            Set-Location $savedcvd
   
        ## SetDBConnection - uses Windows Service (type 2), username is ignored
            write-host "Calling SetDatabaseConnection server $server, DB $dbName"
            $r = $RSObject.SetDatabaseConnection($server,$dbName,2,'','')
            CheckResult $r "SetDatabaseConnection"  
   
        ## Setting the Report Manager URL ##
   
        write-host -foregroundcolor green "Setting the Report Manager URL"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## SetVirtualDirectory for Reports (Report Manager) site
            write-host 'Calling SetVirtualDirectory'
            $r = $RSObject.SetVirtualDirectory('ReportManager','Reports',1033)
            CheckResult $r "SetVirtualDirectory"
   
        ## ReserveURL for ReportManager  - port $HTTPport
            write-host "Calling ReserveURL for ReportManager, port $HTTPport"
            $r = $RSObject.ReserveURL('ReportManager',"http://+:$HTTPport",1033)
            CheckResult $r "ReserveURL for ReportManager port $HTTPport"
   
        write-host -foregroundcolor green "Open Firewall port for $HTTPport"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## Open Firewall port for $HTTPport
            New-NetFirewallRule -DisplayName “Report Server (TCP on port $HTTPport)” -Direction Inbound –Protocol TCP –LocalPort $HTTPport
            write-host "Added rule Report Server (TCP on port $HTTPport) in Windows Firewall"
   
        write-host 'Operations completed, Report Server is ready'
        write-host -foregroundcolor DarkGray $starttime StartTime
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
5. 如果您用來建立 VM 的 HTTP 連接埠不是 80，請修改參數 $HTTPport = 80。
6. 已針對 Reporting Services 設定指令碼。 如果您要執行 Reporting Services 的指令碼，請在 Get-WmiObject 陳述式上，將命名空間的路徑版本部分修改為 "v11"。
7. 執行指令碼。

**驗證**：若要確認報表伺服器基本功能正常運作，請參閱本主題後半部的 [驗證組態](#verify-the-configuration) 一節。

### <a name="use-script-to-configure-the-report-server-and-https"></a>使用指令碼設定報表伺服器和 HTTPS
若要使用 Windows PowerShell 設定報表伺服器，請完成下列步驟。 設定包括 HTTP 而非 HTTPS。

1. 在 Azure 入口網站中選取 VM，然後按一下 [連接]。 根據您的瀏覽器設定，可能會提示您儲存 .rdp 檔案以連接至 VM。
   
    ![連接至 Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif) 請使用您建立 VM 時所設定的使用者 VM 名稱、使用者名稱和密碼。 
   
    例如，下圖的 VM 名稱是 **ssrsnativecloud**，而使用者名稱是 **testuser**。
   
    ![登入包括 VM 名稱](./media/virtual-machines-windows-classic-ps-sql-report/IC764111.png)
2. 在 VM 上，以系統管理權限開啟 **Windows PowerShell ISE** 。 Windows Server 2012 預設會安裝 PowerShell ISE。 建議您使用 ISE 代替標準 Windows PowerShell 視窗，以便將指令碼貼到 ISE、修改指令碼，並接著執行指令碼。
3. 若要允許執行指令碼，請執行下列 Windows PowerShell 命令：
   
        Set-ExecutionPolicy RemoteSigned
   
    然後您就能夠執行下列命令以驗證原則：
   
        Get-ExecutionPolicy
4. 在 [Windows PowerShell ISE] 中，按一下 [檢視]功能表，然後按一下 [顯示指令碼窗格]。
5. 複製下列指令碼，並將指令碼貼到 [Windows PowerShell ISE 指令碼] 窗格。
   
        ## This script configures the report server, including HTTPS
        $ErrorActionPreference = "Stop"
        $httpsport=443 # modify if you used a different port number when the HTTPS endpoint was created.
   
        # You can run the following command to get (.cloudapp.net certificates) so you can copy the thumbprint / certificate hash
        #dir cert:\LocalMachine -rec | Select-Object * | where {$_.issuer -like "*cloudapp*" -and $_.pspath -like "*root*"} | select dnsnamelist, thumbprint, issuer
        #
        # The certifacte hash is a REQUIRED parameter
        $certificatehash="" 
        # the certificate hash should not contain spaces
   
        if ($certificatehash.Length -lt 1) 
        {
            write-error "certificatehash is a required parameter"
        } 
        # Certificates should be all lower case
        $certificatehash=$certificatehash.ToLower()
        $server = $env:COMPUTERNAME
        # If the certificate is not a wildcard certificate, comment out the following line, and enable the full $DNSNAme reference.
        $DNSName="+"
        #$DNSName="$server.cloudapp.net"
        $DNSNameAndPort = $DNSName + ":$httpsport"
   
        ## Utility method for verifying an operation's result
        function CheckResult
        {
            param($wmi_result, $actionname)
            if ($wmi_result.HRESULT -ne 0) {
                write-error "$actionname failed. Error from WMI: $($wmi_result.Error)"
            }
        }
   
        $starttime=Get-Date
        write-host -foregroundcolor DarkGray $starttime StartTime
   
        ## ReportServer Database name - this can be changed if needed
        $dbName='ReportServer'
   
        write-host "The script will use $DNSNameAndPort as the DNS name and port" 
   
        ## Register for MSReportServer_ConfigurationSetting
        ## Change the version portion of the path to "v11" to use the script for SQL Server 2012
        $RSObject = Get-WmiObject -class "MSReportServer_ConfigurationSetting" -namespace "root\Microsoft\SqlServer\ReportServer\RS_MSSQLSERVER\v12\Admin"
   
        ## Reporting Services Report Server Configuration Steps
   
        ## 1. Setting the web service URL ##
        write-host -foregroundcolor green "Setting the web service URL"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## SetVirtualDirectory for ReportServer site
            write-host 'Calling SetVirtualDirectory'
            $r = $RSObject.SetVirtualDirectory('ReportServerWebService','ReportServer',1033)
            CheckResult $r "SetVirtualDirectory for ReportServer"
   
        ## ReserveURL for ReportServerWebService - port 80 (for local usage)
            write-host 'Calling ReserveURL port 80'
            $r = $RSObject.ReserveURL('ReportServerWebService','http://+:80',1033)
            CheckResult $r "ReserveURL for ReportServer port 80" 
   
        ## ReserveURL for ReportServerWebService - port $httpsport
            write-host "Calling ReserveURL port $httpsport, for URL: https://$DNSNameAndPort"
            $r = $RSObject.ReserveURL('ReportServerWebService',"https://$DNSNameAndPort",1033)
            CheckResult $r "ReserveURL for ReportServer port $httpsport" 
   
        ## CreateSSLCertificateBinding for ReportServerWebService port $httpsport
            write-host "Calling CreateSSLCertificateBinding port $httpsport, with certificate hash: $certificatehash"
            $r = $RSObject.CreateSSLCertificateBinding('ReportServerWebService',$certificatehash,'0.0.0.0',$httpsport,1033)
            CheckResult $r "CreateSSLCertificateBinding for ReportServer port $httpsport" 
   
        ## 2. Setting the Database ##
        write-host -foregroundcolor green "Setting the Database"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## GenerateDatabaseScript - for creating the database
            write-host "Calling GenerateDatabaseCreationScript for database $dbName"
            $r = $RSObject.GenerateDatabaseCreationScript($dbName,1033,$false)
            CheckResult $r "GenerateDatabaseCreationScript"
            $script = $r.Script
   
        ## Execute sql script to create the database
            write-host 'Executing Database Creation Script'
            $savedcvd = Get-Location
            Import-Module SQLPS                    ## this automatically changes to sqlserver provider
            Invoke-SqlCmd -Query $script
            Set-Location $savedcvd
   
        ## GenerateGrantRightsScript 
            $DBUser = "NT Service\ReportServer"
            write-host "Calling GenerateDatabaseRightsScript with user $DBUser"
            $r = $RSObject.GenerateDatabaseRightsScript($DBUser,$dbName,$false,$true)
            CheckResult $r "GenerateDatabaseRightsScript"
            $script = $r.Script
   
        ## Execute grant rights script
            write-host 'Executing Database Rights Script'
            $savedcvd = Get-Location
            cd sqlserver:\
            Invoke-SqlCmd -Query $script
            Set-Location $savedcvd
   
        ## SetDBConnection - uses Windows Service (type 2), username is ignored
            write-host "Calling SetDatabaseConnection server $server, DB $dbName"
            $r = $RSObject.SetDatabaseConnection($server,$dbName,2,'','')
            CheckResult $r "SetDatabaseConnection"  
   
        ## 3. Setting the Report Manager URL ##
   
        write-host -foregroundcolor green "Setting the Report Manager URL"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## SetVirtualDirectory for Reports (Report Manager) site
            write-host 'Calling SetVirtualDirectory'
            $r = $RSObject.SetVirtualDirectory('ReportManager','Reports',1033)
            CheckResult $r "SetVirtualDirectory"
   
        ## ReserveURL for ReportManager  - port 80
            write-host 'Calling ReserveURL for ReportManager, port 80'
            $r = $RSObject.ReserveURL('ReportManager','http://+:80',1033)
            CheckResult $r "ReserveURL for ReportManager port 80"
   
        ## ReserveURL for ReportManager - port $httpsport
            write-host "Calling ReserveURL port $httpsport, for URL: https://$DNSNameAndPort"
            $r = $RSObject.ReserveURL('ReportManager',"https://$DNSNameAndPort",1033)
            CheckResult $r "ReserveURL for ReportManager port $httpsport" 
   
        ## CreateSSLCertificateBinding for ReportManager port $httpsport
            write-host "Calling CreateSSLCertificateBinding port $httpsport with certificate hash: $certificatehash"
            $r = $RSObject.CreateSSLCertificateBinding('ReportManager',$certificatehash,'0.0.0.0',$httpsport,1033)
            CheckResult $r "CreateSSLCertificateBinding for ReportManager port $httpsport" 
   
        write-host -foregroundcolor green "Open Firewall port for $httpsport"
        write-host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
   
        ## Open Firewall port for $httpsport
            New-NetFirewallRule -DisplayName “Report Server (TCP on port $httpsport)” -Direction Inbound –Protocol TCP –LocalPort $httpsport
            write-host "Added rule Report Server (TCP on port $httpsport) in Windows Firewall"
   
        write-host 'Operations completed, Report Server is ready'
        write-host -foregroundcolor DarkGray $starttime StartTime
        $time=Get-Date
        write-host -foregroundcolor DarkGray $time
6. 修改指令碼中的 **$certificatehash** 參數：
   
   * 這是 **必要** 參數。 如果您未在先前步驟儲存憑證值，可使用下列方法之一，從憑證指紋複製憑證雜湊值：
     
       在 VM 上開啟 Windows PowerShell ISE 並執行下列命令：
     
           dir cert:\LocalMachine -rec | Select-Object * | where {$_.issuer -like "*cloudapp*" -and $_.pspath -like "*root*"} | select dnsnamelist, thumbprint, issuer
     
       輸出類似如下範例。 如果指令碼傳回空白行，可能表示 VM 沒有設定憑證；請參閱 [使用虛擬機器的自我簽署憑證](#to-use-the-virtual-machines-self-signed-certificate)一節。
     
     或
   * 在 VM 上執行 mmc.exe，然後新增 [憑證]  嵌入式管理單元。
   * 在 [信任的根憑證授權] 節點下，按兩下您的憑證名稱。 如果您使用 VM 的自我簽署憑證，則憑證是以 VM 的 DNS 名稱來命名，並以 **cloudapp.net** 結尾。
   * 按一下 [詳細資料]  索引標籤。
   * 按一下 [指紋] 。 憑證指紋的值會顯示在 [詳細資料] 欄位中，例如 af 11 60 b6 4b 28 8d 89 0a 82 12 ff 6b a9 c3 66 4f 31 90 48
   * **執行指令碼之前**，請先移除每一組值之間的空格。 例如 af1160b64b288d890a8212ff6ba9c3664f319048
7. 修改 **$httpsport** 參數： 
   
   * 如果您將連接埠 443 用於 HTTPS 端點，則不需要在指令碼中更新此參數。 否則請使用您在 VM 上設定 HTTPS 私用端點時所設定的連接埠值。
8. 修改 **$DNSName** 參數： 
   
   * 指令碼設定為萬用字元憑證 $DNSName="+"。 如果您不想設定萬用字元憑證繫結，請將 $DNSName="+" 標記為註解，並啟用以下這一行完整 $DNSNAme 參考：##$DNSName="$server.cloudapp.net"。
     
       如果您不想將虛擬機器的 DNS 名稱用於 Reporting Services，請變更 $DNSName 值。 如果您使用此參數，則憑證也必須使用這個名稱，且您必須在 DNS 伺服器全域註冊該名稱。
9. 已針對 Reporting Services 設定指令碼。 如果您要執行 Reporting Services 的指令碼，請在 Get-WmiObject 陳述式上，將命名空間的路徑版本部分修改為 "v11"。
10. 執行指令碼。

**驗證**：若要確認報表伺服器基本功能正常運作，請參閱本主題後半部的 [驗證組態](#verify-the-connection) 一節。 若要確認憑證繫結，請以系統管理權限開啟命令提示字元，然後執行下列命令：

    netsh http show sslcert

結果將包含下列項目：

    IP:port                      : 0.0.0.0:443

    Certificate Hash             : f98adf786994c1e4a153f53fe20f94210267d0e7

### <a name="use-configuration-manager-to-configure-the-report-server"></a>使用組態管理員設定報表伺服器
如果您不想執行 PowerShell 指令碼設定報表伺服器，請遵循本節的步驟，使用 Reporting Services 原生模式組態管理員設定報表伺服器。

1. 在 Azure 入口網站中選取 VM，然後按一下 [連接]。 請使用您建立 VM 時所設定的使用者名稱和密碼。
   
    ![連接至 Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-report/IC650112.gif)
2. 執行 Windows Update 並安裝 VM 的更新。 如果需要重新啟動 VM，請重新啟動 VM，並從 Azure 入口網站重新連接到 VM。
3. 在 VM 的 [開始] 功能表中，鍵入 **Reporting Services**，並開啟 [Reporting Services 組態管理員]。
4. 保留 [伺服器名稱] 和 [報表伺服器執行個體] 的預設值。 按一下 [ **連接**]。
5. 按一下左窗格中的 [Web 服務 URL] 。
6. 根據預設，RS 會設定為 HTTP 連接埠 80，而 IP 為全部指派。 若要新增 HTTPS：
   
   1. 在 [SSL 憑證] 中：選取您要使用的憑證，例如 [VM 名稱].cloudapp.net。 如果未列出憑證，請參閱**步驟 2：建立伺服器憑證**一節，以取得有關如何在 VM 上安裝和信任憑證的資訊。
   2. 在 [SSL 連接埠：] 下選擇 443。 如果您在 VM 中，以不同的私用連接埠設定 HTTPS 私用端點，請在此處使用該值。
   3. 按一下 [套用]  ，並等候作業完成。
7. 在左窗格中，按一下 [資料庫] 。
   
   1. 按一下 [變更資料庫] 。
   2. 按一下 [建立新的報告伺服器資料庫]，然後按 [下一步]。
   3. 將預設的 [伺服器名稱] 保留為 VM 名稱，並將預設的 [驗證類型] 保留為 [目前使用者 – 整合式安全性]。 按 [下一步] 。
   4. 將預設的 [資料庫名稱] 保留為 **ReportServer**，然後按 [下一步]。
   5. 將預設的 [驗證類型] 保留為 [服務認證]，然後按 [下一步]。
   6. 按一下左窗格中的 [虛擬機器]  on the  。
   7. 組態設定完成後，按一下 [完成] 。
8. 在左窗格中，按一下 [報告管理員 URL] 。 將預設的 [虛擬目錄] 保留為 [報告]，然後按一下 [套用]。
9. 按一下 [結束]  ，關閉 Reporting Services 組態管理員。

## <a name="step-4-open-windows-firewall-port"></a>步驟 4：開啟 Windows 防火牆連接埠
> [!NOTE]
> 如果您使用其中一個指令碼來設定報表伺服器，則可略過本節。 指令碼已包含可開啟防火牆連接埠的步驟。 預設值為連接埠 80 用於 HTTP，而連接埠 443 用於 HTTPS。
> 
> 

若要遠端連接報表管理員或虛擬機器上的報表伺服器，則 VM 必須有 TCP 端點。 必須在 VM 的防火牆中開啟相同的連接埠。 端點是在佈建 VM 時建立。

本節提供有關如何開啟防火牆連接埠的基本資訊。 如需詳細資訊，請參閱 [為報告伺服器存取設定防火牆](https://technet.microsoft.com/library/bb934283.aspx)

> [!NOTE]
> 如果您使用指令碼來設定報表伺服器，則可略過本節。 指令碼已包含可開啟防火牆連接埠的步驟。
> 
> 

如果您為 HTTPS 設定的私用連接埠不是 443，請據以修改下列指令碼。 若要開啟 Windows 防火牆上的連接埠 **443** ，請完成下列步驟：

1. 以系統管理權限開啟 Windows PowerShell 視窗。
2. 如果您在 VM 上設定 HTTPS 端點時使用 443 以外的連接埠，請在下列命令中更新連接埠，然後執行命令：
   
        New-NetFirewallRule -DisplayName “Report Server (TCP on port 443)” -Direction Inbound –Protocol TCP –LocalPort 443
3. 命令完成時，命令提示字元會顯示 **Ok** 。

若要確認連接埠已開啟，請開啟 Windows PowerShell 視窗並執行下列命令：

    get-netfirewallrule | where {$_.displayname -like "*report*"} | select displayname,enabled,action

## <a name="verify-the-configuration"></a>驗證組態
若要確認報表伺服器基本功能現已正常運作，請以系統管理權限開啟瀏覽器，並瀏覽至下列報表伺服器和報表管理員 URL：

* 在 VM 上，瀏覽至報表伺服器 URL：
  
        http://localhost/reportserver
* 在 VM 上，瀏覽至報表管理員 URL：
  
        http://localhost/Reports
* 從本機電腦瀏覽至 VM 上的 **遠端** 報告管理員。 視需要更新下列範例中的 DNS 名稱。 如果系統提示輸入密碼，請使用您佈建 VM 時所建立的系統管理員認證。 使用者名稱的格式是 [網域]\[使用者名稱]，其中的網域是 VM 電腦名稱，例如 ssrsnativecloud\testuser。 如果您不是使用 HTTP**S**，請移除 URL 中的 **s**。 如需有關如何在 VM 上建立其他使用者的資訊，請參閱下一節。
  
        https://ssrsnativecloud.cloudapp.net/Reports
* 從本機電腦瀏覽至遠端報表伺服器 URL。 視需要更新下列範例中的 DNS 名稱。 如果您不是使用 HTTPS，請移除 URL 中的 s。
  
        https://ssrsnativecloud.cloudapp.net/ReportServer

## <a name="create-users-and-assign-roles"></a>建立使用者和指派角色
設定和確認報表伺服器之後的一件常見管理工作，就是建立一或多個使用者，並將使用者指派給 Reporting Services 角色。 如需詳細資訊，請參閱下列：

* [建立本機使用者帳戶](https://technet.microsoft.com/library/cc770642.aspx)
* [將報告伺服器存取權授與使用者 (報告管理員)](https://msdn.microsoft.com/library/ms156034.aspx))
* [建立和管理角色指派](https://msdn.microsoft.com/library/ms155843.aspx)

## <a name="to-create-and-publish-reports-to-the-azure-virtual-machine"></a>建立報表並發佈至 Azure 虛擬機器
下表摘要列出一些選項，可將現有報表從內部部署電腦發佈至 Microsoft Azure 虛擬機器上託管的報表伺服器：

* **RS.exe 指令碼**：使用 RS.exe 指令碼，將報告項目從現有報告伺服器複製到 Microsoft Azure 虛擬機器。 如需詳細資訊，請參閱 [在報告伺服器之間移轉內容的範例 Reporting Services rs.exe 指令碼](https://msdn.microsoft.com/library/dn531017.aspx)的「原生模式到原生模式 – Microsoft Azure 虛擬機器」一節。
* **報告產生器**：虛擬機器包含 Click Once 版本的 Microsoft SQL Server 報告產生器。 若要在虛擬機器上首次啟動報表產生器：
  
  1. 以管理權限啟動瀏覽器。
  2. 瀏覽至虛擬機器上的報表管理員，然後按一下功能區中的 [報表產生器] 。
     
     如需詳細資訊，請參閱 [安裝、解除安裝和支援報告產生器](https://technet.microsoft.com/library/dd207038.aspx)。
* **SQL Server Data Tools：VM**：如果您使用 SQL Server 2012 建立 VM，則會在虛擬機器上安裝 SQL Server Data Tools，且可在虛擬機器上用來建立**報告伺服器專案**與報告。 SQL Server Data Tools 可將報表發佈至虛擬機器上的報表伺服器。
  
    如果您使用 SQL Server 2014 建立 VM，則可安裝 SQL Server Data Tools - BI for Visual Studio。 如需詳細資訊，請參閱下列：
  
  * [Microsoft SQL Server Data Tools - Business Intelligence for Visual Studio 2013](https://www.microsoft.com/download/details.aspx?id=42313)
  * [Microsoft SQL Server Data Tools - Business Intelligence for Visual Studio 2012](https://www.microsoft.com/download/details.aspx?id=36843)
  * [SQL Server Data Tools and SQL Server Business Intelligence (SSDT-BI)](https://docs.microsoft.com/sql/ssdt/previous-releases-of-sql-server-data-tools-ssdt-and-ssdt-bi)
* **SQL Server Data Tools：遠端**：在本機電腦上，以 SQL Server Data Tools 建立含有 Reporting Services 報告的 Reporting Services 專案。 設定專案以連接至 Web 服務 URL。
  
    ![SSRS 專案的 SSDT 專案屬性](./media/virtual-machines-windows-classic-ps-sql-report/IC650114.gif)
* **使用指令碼**：使用指令碼複製報告伺服器內容。 如需詳細資訊，請參閱 [在報告伺服器之間移轉內容的 Reporting Services rs.exe 指令碼範例](https://msdn.microsoft.com/library/dn531017.aspx)。

## <a name="minimize-cost-if-you-are-not-using-the-vm"></a>在不使用 VM 時將成本降至最低
> [!NOTE]
> 若要在不使用 Azure 虛擬機器時將費用降至最低，請從 Azure 入口網站關閉 VM。 如果您使用 VM 內的 Windows 電源選項關閉 VM，則仍需針對 VM 支付相同金額。 若要降低費用，您必須在 Azure 入口網站關閉 VM。 如果您不再需要 VM，請記得刪除 VM 和相關聯的 .vhd 檔案，以節省儲存空間費用。如需詳細資訊，請參閱[虛擬機器價格詳細資料](https://azure.microsoft.com/pricing/details/virtual-machines/)的＜常見問題集＞一節。

## <a name="more-information"></a>相關資訊
### <a name="resources"></a>資源
* 如需與 SQL Server Business Intelligence 及 SharePoint 2013 單一伺服器部署相關的類似內容，請參閱 [使用 Windows PowerShell 建立 Azure VM 搭配 SQL Server BI 及 SharePoint 2013](https://blogs.technet.microsoft.com/ptsblog/2013/10/24/use-powershell-to-create-a-windows-azure-vm-with-sql-server-bi-and-sharepoint-2013/)。
* 如需與 SQL Server Business Intelligence 及 SharePoint 2013 多伺服器部署相關的類似內容，請參閱 [在 Azure 虛擬機器部署 SQL Server Business Intelligence](https://msdn.microsoft.com/library/dn321998.aspx)。
* 如需與 Azure 虛擬機器的 SQL Server Business Intelligence 部署相關的一般資訊，請參閱 [Azure 虛擬機器的 SQL Server Business Intelligence](virtual-machines-windows-classic-ps-sql-bi.md)。
* 如需有關 Azure 計算費用成本的詳細資訊，請參閱 [Azure 價格計算機](https://azure.microsoft.com/pricing/calculator/?scenario=virtual-machines)中的 [虛擬機器] 索引標籤。

### <a name="community-content"></a>社群內容
* 如需有關如何不使用指令碼來建立 Reporting Services 原生模式報告伺服器的逐步指示，請參閱 [在 Azure 虛擬機器上託管 SQL Reporting Services](http://adititechnologiesblog.blogspot.in/2012/07/hosting-sql-reporting-service-on-azure.html)。

### <a name="links-to-other-resources-for-sql-server-in-azure-vms"></a>Azure VM 中的 SQL Server 的其他資源連結
[Azure 虛擬機器上的 SQL Server 概觀](../sql/virtual-machines-windows-sql-server-iaas-overview.md)

