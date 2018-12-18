---
title: Microsoft Azure Stack 開發套件版本資訊 | Microsoft Docs
description: Azure Stack 開發套件的增強功能、修正及已知問題。
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/27/2018
ms.author: brenduns
ms.reviewer: misainat
ms.openlocfilehash: 6016d6fd4dcae699efba49667676373a6586e723
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43101640"
---
# <a name="azure-stack-development-kit-release-notes"></a>Azure Stack 開發套件版本資訊  
這些版本資訊提供 Azure Stack 開發套件中的增強功能、修正和已知問題的相關資訊。 如果您不確定所執行的版本，可以使用[入口網站來進行檢查](.\.\azure-stack-updates.md#determine-the-current-version)。

> 請訂閱 [![RSS](./media/asdk-release-notes/feed-icon-14x14.png)](https://docs.microsoft.com/api/search/rss?search=Azure+Stack+Development+Kit+release+notes&locale=en-us#) [摘要](https://docs.microsoft.com/api/search/rss?search=Azure+Stack+Development+Kit+release+notes&locale=en-us#)，以便隨時收到 ASDK 的最新訊息。


## <a name="build-11807076"></a>組建 1.1807.0.76

### <a name="new-features"></a>新功能
此組建包含下列適用於 Azure Stack 的改良功能與修正。  

- <!-- 1658937 | ASDK, IS --> **根據預先定義的排程啟動備份** - 作為設備的 Azure Stack 現在可以自動地定期觸發基礎結構備份，無須人為介入。 若備份超過定義的保留期限，Azure Stack 也會自動清除其外部共用。 了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](.\.\azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。

- <!-- 2496385 | ASDK, IS -->**已將資料轉送時間新增到備份時間總計中。** 了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](.\.\azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。

-   <!-- 1702130 | ASDK, IS -->**備份外部容量現在會顯示正確的外部共用容量。** (以前是以硬式編碼將其設為 10 GB。)了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](.\.\azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。
 
- <!-- 2753130 |  IS, ASDK   -->  **Azure Resource Manager 範本現在支援條件元素** - 您現在可以使用條件在 Azure Resource Manger 範本中部署資源。 您可以將範本設計成根據條件 (例如評估某個參數值是否存在) 來部署資源。 如需有關使用範本作為條件的資訊，請參閱 Azure 文件中的[依條件部署資源](https://docs.microsoft.com/azure/architecture/building-blocks/extending-templates/conditional-deploy)和 [Azure Resource Manager 範本的 Variables 區段](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-templates-variables)。 

   您也可以使用範本[將資源部署至多個訂用帳戶或資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-cross-resource-group-deployment)。  

- <!--2753073 | IS, ASDK -->  **Microsoft.Network API 資源版本支援已更新**，從適用於 Azure Stack 網路資源的 2015-06-15 更新成包含對 API 2017-10-01 版的支援。  此版本未包含對 2017-10-01 與 2015-06-15 之間資源版本的支援，但未來的版本中將會包含。  如需了解功能差異，請參閱 [Azure Stack 網路服務的注意事項](.\.\user\azure-stack-network-differences.md)。

- <!-- 2272116 | IS, ASDK   -->  **Azure Stack 已針對對外的 Azure Stack 基礎結構端點 (亦即針對 portal、adminportal、management 及 adminmanagement) 新增反向 DNS 查閱支援**。 這可允許從 IP 位址解析 Azure Stack 外部端點名稱。

- <!-- 2780899 |  IS, ASDK   --> **Azure Stack 現在支援為現有的 VM 新增額外的網路介面。**  藉由使用入口網站、PowerShell 及 CLI，即可使用此功能。 如需詳細資訊，請參閱 Azure 文件中的[新增或移除網路介面](https://docs.microsoft.com/azure/virtual-network/virtual-network-network-interface-vm)。 

- <!-- 2222444 | IS, ASDK   -->  **已提升網路使用量計量的精確度和彈性**。 網路使用量計量現在更精確，並將暫停的訂用帳戶、中斷期及競爭條件納入考慮。

- <!-- 2297790 | IS, ASDK -->  **改進 Azure Stack Syslog 用戶端 (預覽功能)**. 此用戶端允許將與 Azure Stack 基礎結構相關的稽核和記錄轉送到 Azure Stack 外的 Syslog 伺服器或安全性資訊與事件管理 (SIEM) 軟體。 Syslog 用戶端現在支援搭配純文字或 TLS 1.2 加密的 TCP 通訊協定，後者為預設設定。 您可以為 TLS 連線設定僅限伺服器的驗證或相互驗證。

  若要設定 Syslog 用戶端與 Syslog 伺服器的通訊方式 (例如通訊協定、加密及驗證)，請使用 Set-SyslogServer Cmdlet。 您可以從具特殊權限的端點 (PEP) 使用此 Cmdlet。 

  若要為 Syslog 用戶端 TLS 1.2 相互驗證新增用戶端憑證，請使用 PEP 中的 Set-SyslogClient Cmdlet。

  在此預覽版中，您可看到的稽核和警示數量會多出許多。 

  由於此功能仍是預覽版，因此在生產環境中請勿倚賴此功能。

  如需詳細資訊，請參閱 [Azure Stack Syslog 轉送](.\.\azure-stack-integrate-security.md)。

- <!-- ####### | IS, ASDK -->  **Azure Resource Manager 包含區域名稱。** 在此版本中，從 Azure Resource Manager 擷取的物件現在會包含區域名稱屬性。 如果現有的 PowerShell 指令碼會將物件直接傳遞給另一個 Cmdlet，則此指令碼可能會產生錯誤並失敗。 這是符合 Azure Resource Manager 規範的行為，而會需要發出呼叫的用戶端去除區域屬性。 如需有關 Azure Resource Manager 的詳細資訊，請參閱 [Azure Resource Manager 文件](https://docs.microsoft.com/azure/azure-resource-manager/)。

- <!-- TBD | IS, ASDK -->  **在委派的提供者之間移動訂用帳戶。** 您現在可以在屬於相同目錄租用戶之新的或現有的「委派的提供者」訂用帳戶之間，移動訂用帳戶。 您也可以將屬於「預設提供者訂用帳戶」的訂用帳戶移至相同目錄租用戶中的「委派的提供者訂用帳戶」。 如需詳細資訊，請參閱 [Azure Stack 中的委派供應項目](.\.\azure-stack-delegated-provider.md)。
 
- <!-- 2536808 IS ASDK --> 針對使用從 Azure Marketplace 下載之映像建立的 VM，**縮短 VM 建立時間**。

### <a name="fixed-issues"></a>已修正的問題

- <!-- TBD | ASDK, IS --> 已對更新程序進行各種改進，使其變得更可靠。 此外，也已對基底基礎結構進行修正，改善了節點清空狀況，因而可將更新期間工作負載的可能停機時間縮到最短。

-   <!--2292271 | ASDK, IS --> 修正問題：已修改的配額限制無法套用到現有訂用帳戶。  現在，如果「供應項目」和「方案」與使用者訂用帳戶相關聯，而您將該「供應項目」和「方案」所含網路資源的配額限制提高，則新限制不但會套用到新的訂用帳戶，也會套用到既有的訂用帳戶。

- <!-- 2448955 | IS ASDK --> 您現在可以順利地對部署在 UTC+N 時區的系統查詢活動記錄。    

- <!-- 2319627 |  ASDK, IS --> 備份組態參數 (路徑/使用者名稱/密碼/加密金鑰) 的前置檢查不會再對備份組態進行不正確的設定。 (之前，如果在備份中設定了不正確的設定，當觸發備份時，備份就會發生失敗)。

- <!-- 2215948 |  ASDK, IS --> 現在，當您從外部共用手動刪除備份時，備份清單會進行重新整理。

- <!-- 2360715 |  ASDK, IS -->  當您設定資料中心整合時，您無法再從共用中存取 AD FS 中繼資料檔案。 如需詳細資訊，請參閱[藉由提供同盟中繼資料檔案設定 AD FS 整合](.\.\azure-stack-integrate-identity.md#setting-up-ad-fs-integration-by-providing-federation-metadata-file)。 

- <!-- 2388980 | ASDK, IS --> 修正問題：防止使用者將先前已指派給網路介面或 Load Balancer 的現有公用 IP 位址，指派給新的網路介面或 Load Balancer。  

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [概觀] 時，[基本資訊] 窗格現在會正確地顯示所有預期的資訊。 

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [標籤] 時，資訊現在會正確地顯示。

- <!-- TBD - IS ASDK --> 此版本的 Azure Stack 修正了導致無法從 OEM 延伸模組套件套用驅動程式更新的問題。

-   <!-- 2055809- IS ASDK --> 我們已修正導致您在 VM 建立失敗時無法從 [計算] 刀鋒視窗刪除 VM 的問題。  

- <!--  2643962 IS ASDK -->  「記憶體容量不足」的警示不再不正確地顯示。

- <!--  TBD ASDK --> 裝載具特殊權限端點 (PEP) 的虛擬機器已增加至 4 GB。 在 ASDK 中，此虛擬機器名為 AzS ERCS01。

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**


<!-- ### Changes  -->
<!--   ### Additional releases timed with this update  -->


### <a name="known-issues"></a>已知問題

#### <a name="portal"></a>入口網站  
- <!-- 2931230 – IS  ASDK --> 無法刪除以附加方案形式新增至使用者訂用帳戶的方案 (即使您從使用者訂用帳戶中移除此方案)。 此方案會持續保留，直到參考附加方案的訂用帳戶也遭到刪除為止。 

- <!--2760466 – IS  ASDK --> 當您安裝執行此版本的新 Azure Stack 環境時，可能不會顯示指出「需要啟用」的警示。 必須[啟用](.\.\azure-stack-registration.md)，才能使用市集摘要整合。 

- <!-- TBD - IS ASDK --> 不應該使用[隨 1804 版導入的](.\.\azure-stack-update-1804.md#new-features)兩個系統管理訂用帳戶類型。 這些訂用帳戶類型為「計量訂用帳戶」和「取用訂用帳戶」。 這些訂用帳戶類型為「計量訂用帳戶」和「取用訂用帳戶」。 自 1804 版起，這些訂用帳戶類型就已經會在新的 Azure Stack 環境中顯示，只是還未就緒以供使用。 您應該繼續使用「預設提供者訂用帳戶」類型。

- <!-- 2403291 - IS ASDK --> 您可能還無法使用系統管理員和使用者入口網站底部的水平捲軸。 如果您無法存取水平捲軸，請使用階層連結來瀏覽至入口網站中的上一個刀鋒視窗，方法是從入口網站左上角的階層連結清單中，選取您想要檢視的刀鋒視窗名稱。
  ![階層連結](media/asdk-release-notes/breadcrumb.png)

- <!-- TBD -  IS ASDK --> 刪除使用者訂用帳戶會產生孤立的資源。 因應措施是，先刪除使用者資源或整個資源群組，然後再刪除使用者訂用帳戶。

- <!-- TBD -  IS ASDK --> 您無法使用 Azure Stack 入口網站來檢視對您訂用帳戶的權限。 因應措施是，使用 PowerShell 來確認權限。

- <!--  TBD | ASDK -->  您 Azure Stack 部署的預設時區現在會設定為 UTC。 安裝 Azure Stack 時，您可以選取時區，不過，在安裝期間，它會自動還原成以 UTC 作為預設值。

#### <a name="health-and-monitoring"></a>健康情況和監視
- <!-- 1264761 - IS ASDK --> 您可能會看到「健康情況控制器」元件出現具有下列詳細資料的警示：  

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

  您可以放心地忽略這兩個警示，這兩個警示會在一段時間過後自動關閉。  

- <!-- 2368581 - IS. ASDK --> 身為 Azure Stack 操作員，如果您收到記憶體不足的警示，而且租用戶虛擬機器無法部署並出現「Fabric VM 建立錯誤」，則可能表示 Azure Stack 戳記的可用記憶體不足。 請使用 [Azure Stack 容量規劃工具](https://gallery.technet.microsoft.com/Azure-Stack-Capacity-24ccd822)來充分了解可供工作負載使用的容量。

- <!-- TBD - IS. ASDK --> 在具特殊權限的端點 (PEP) 上執行 Test-AzureStack Cmdlet 時，將會針對 ERCS VM 產生 WARN 訊息。 您可以繼續使用 ASDK。 

#### <a name="compute"></a>計算
- <!-- 2494144 - IS, ASDK --> 在為虛擬機器部署選取虛擬機器大小時，某些 F 系列的 VM 大小不會在您建立 VM 時顯示在大小選取器中。 選取器中不會顯示下列 VM 大小：*F8s_v2*、*F16s_v2*、*F32s_v2* 和 *F64s_v2*。  
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


- <!-- TBD -  IS ASDK --> 無法在入口網站中使用虛擬機器擴展集的調整設定。 您可以使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-powershell#change-the-capacity-of-a-scale-set) 作為因應措施。 由於 PowerShell 版本差異，您必須使用 `-Name` 參數，而不是 `-VMScaleSetName`。

- <!-- TBD -  IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結 D 系列 VM 的資料磁碟數目會不正確。 所有受支援的 D 系列 VM 均可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD -  IS ASDK --> 當 VM 映像建立失敗時，可能會在 VM 映像計算刀鋒視窗上新增一個您無法刪除的失敗項目。

  因應措施是，建立一個具有虛擬 VHD 的新 VM 映像，您可以透過 Hyper-V 建立此 VHD (New-VHD -Path C:\dummy.vhd -Fixed -SizeBytes 1 GB)。 此程序應該可以修正防止刪除失敗項目的問題。 接著，在建立虛擬映像後的 15 分鐘之後，您便可以順利刪除它。

  您可以接著嘗試下載先前失敗的 VM 映像。

- <!-- TBD -  IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 - IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。

- <!-- 2724961- IS ASDK --> 當您在「訂用帳戶」設定中註冊 **Microsoft.Insight** 資源提供者，並建立已啟用「客體 OS 診斷」功能的 Windows VM 時，VM 概觀頁面不會顯示計量資料。 

   若要尋找該 VM 的計量資料 (例如「CPU 百分比」圖表)，請移至 [計量] 刀鋒視窗，並顯示所有支援的 Windows VM 客體計量。

#### <a name="networking"></a>網路功能
- <!-- 1766332 - IS, ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 1902460 -  IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 -  IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- 2702741 -  IS ASDK --> 針對使用「動態」配置方法來部署的公用 IP，不保證在發出 Stop-Deallocate 之後還能予以保留。

- <!-- 2529607 - IS ASDK --> 在 Azure Stack「祕密輪替」期間，會有一段 2 到 5 分鐘的期間無法存取「公用 IP 位址」。

-   <!-- 2664148 - IS ASDK --> 在租用戶使用 S2S VPN 通道來存取其虛擬機器的案例中，如果在已經建立閘道之後，才將內部部署子網路新增至「區域網路閘道」，租用戶就可能會遇到連線嘗試失敗的情況。 


#### <a name="sql-and-mysql"></a>SQL 和 MySQL
- <!-- TBD - ASDK --> 裝載伺服器的資料庫必須專供資源提供者和使用者工作負載使用。 您無法使用任何其他取用者 (包括「應用程式服務」) 正在使用的執行個體。

- <!-- IS, ASDK --> 當您為 SQL 和 MySQL 資源提供者建立 SKU 時，[系列] 名稱不支援特殊字元，包括空格和句點。

#### <a name="app-service"></a>App Service 方案
- <!-- 2352906 - IS ASDK --> 使用者必須在於訂用帳戶中建立第一個 Azure 函式之前，先註冊儲存體資源提供者。

- <!-- TBD - IS ASDK --> 為了向外延展基礎結構 (背景工作、管理、前端角色)，您必須如「計算」的版本資訊所述，使用 PowerShell。  

- <!-- TBD - IS ASDK --> 目前，App Service 只能部署到「預設提供者訂用帳戶」。 在未來的更新中，App Service 會部署至 Azure Stack 1804 所導入的新「計量訂用帳戶」。 當系統支援使用計量時，現有部署就會全部遷移至這個新的訂用帳戶類型。

#### <a name="usage"></a>使用量  
- <!-- TBD -  IS ASDK --> 使用量公用 IP 位址使用計量資料會針對每一筆記錄顯示相同的 EventDateTime 值，而不是建立記錄時顯示的 TimeDate 時間戳記。 目前，您無法使用這項資料來執行公用 IP 位址使用狀況的正確計量。

<!-- #### Identity -->






## <a name="build-11805147"></a>組建 1.1805.1.47

> [!TIP]  
> 根據客戶的意見反應，Microsoft Azure Stack 所使用的版本結構描述會進行更新。 從 1805 這個更新開始，新的結構描述將更能代表現行的雲端版本。  
>
> 版本結構描述目前為 Version.YearYearMonthMonth.MinorVersion.BuildNumber，其中的第二和第三個集合代表版本和版次。 例如，1805.1 代表 1805 製造 (RTM) 版的版次。  


### <a name="new-features"></a>新功能
此組建包含下列適用於 Azure Stack 的改良功能與修正。  

- <!-- 2297790 - IS, ASDK --> **Azure Stack 現已將 Syslog 用戶端**納入為預覽功能。 此用戶端允許將與 Azure Stack 基礎結構相關的稽核和安全性記錄，轉寄到 Azure Stack 外的 Syslog 伺服器或安全性資訊與事件管理 (SIEM) 軟體。 目前，Syslog 用戶端只支援透過預設連接埠 514 的未驗證 UDP 連線。 每個 Syslog 訊息的承載會以通用事件格式 (CEF) 來設定格式。

  若要設定 Syslog 用戶端，請使用特殊權限端點中所公開的 **Set-SyslogServer** Cmdlet。

  在此預覽版中，您可能會看到下列三種警示。 當 Azure Stack 顯示這些警示時，警示會內含「描述」和「補救」指導方針。
  - 標題：程式碼完整性關閉  
  - 標題：程式碼完整性處於稽核模式
  - 標題：已建立使用者帳戶

  此功能目前還是預覽版，所以不應在生產環境中仰賴此功能。   


### <a name="fixed-issues"></a>已修正的問題
- 我們已修正下列問題：無法在系統管理員入口網站中，[從下拉式清單開啟新的支援要求](.\.\azure-stack-manage-portals.md#quick-access-to-help-and-support)。 此選項目前已可正常運作。

- <!--  TBD ASDK --> 裝載具特殊權限端點 (PEP) 的虛擬機器已增加至 4 GB。 在 ASDK 中，此虛擬機器名為 AzS ERCS01。

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**


<!-- ### Changes  -->


<!--   ### Additional releases timed with this update  -->


### <a name="known-issues"></a>已知問題

#### <a name="portal"></a>入口網站
- <!-- 2931230 – IS  ASDK --> 無法刪除以附加方案形式新增至使用者訂用帳戶的方案 (即使您從使用者訂用帳戶中移除此方案)。 此方案會持續保留，直到參考附加方案的訂用帳戶也遭到刪除為止。 

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [概觀] 時，[基本資訊] 窗格中並未顯示資訊。  [基本資訊] 窗格會顯示帳戶相關資訊，例如其*資源群組*、*位置*和*訂用帳戶識別碼*。  [概觀] 中有其他選項可供存取，例如 [服務] 和 [監視]，以及用來 [在 Explorer 中開啟] 或 [刪除儲存體帳戶] 的選項。  

  若要檢視未能看到的資訊，請使用 [Get-azureRMstorageaccount](https://docs.microsoft.com/powershell/module/azurerm.storage/get-azurermstorageaccount?view=azurermps-6.2.0) PowerShell Cmdlet。

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [標籤] 時，資訊無法載入因此並未顯示。  

  若要檢視未能看到的資訊，請使用 [Get-AzureRmTag](https://docs.microsoft.com/powershell/module/azurerm.tags/get-azurermtag?view=azurermps-6.2.0) PowerShell Cmdlet。

- <!-- TBD - IS ASDK --> 請勿使用新系統管理訂用帳戶類型的「計量訂用帳戶」和「取用訂用帳戶」。 這些新的訂用帳戶類型已於 1804 版導入，但還未就緒以供使用。 請繼續使用「預設提供者」訂用帳戶類型。  
- <!-- TBD - IS ASDK --> 您無法搭配此版本的 Azure Stack 使用 OEM 延伸模組套件來套用驅動程式更新。  此問題目前沒有因應方式。
 
- <!-- TBD - IS ASDK --> 無法在系統管理員入口網站內，[從下拉式清單開啟新的支援要求](.\.\azure-stack-manage-portals.md#quick-access-to-help-and-support)。 請改用下列連結：     
    - 針對 Azure Stack 開發套件，請使用 https://aka.ms/azurestackforum。    

- <!-- 2403291 - IS ASDK --> 您可能還無法使用系統管理員和使用者入口網站底部的水平捲軸。 如果您無法存取水平捲軸，請使用階層連結來瀏覽至入口網站中的上一個刀鋒視窗，方法是從入口網站左上角的階層連結清單中，選取您想要檢視的刀鋒視窗名稱。
  ![階層連結](media/asdk-release-notes/breadcrumb.png)

- <!-- TBD -  IS ASDK --> 刪除使用者訂用帳戶會產生孤立的資源。 因應措施是，先刪除使用者資源或整個資源群組，然後再刪除使用者訂用帳戶。

- <!-- TBD -  IS ASDK --> 您無法使用 Azure Stack 入口網站來檢視對您訂用帳戶的權限。 因應措施是，使用 PowerShell 來確認權限。


#### <a name="health-and-monitoring"></a>健康情況和監視
- <!-- 1264761 - IS ASDK --> 您可能會看到「健康情況控制器」元件出現具有下列詳細資料的警示：  

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

- <!-- TBD - IS. ASDK --> 在具特殊權限的端點 (PEP) 上執行 Test-AzureStack Cmdlet 時，將會針對 ERCS VM 產生 WARN 訊息。 您可以繼續使用 ASDK。 


#### <a name="compute"></a>計算
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


- <!-- TBD -  IS ASDK --> 無法在入口網站中使用虛擬機器擴展集的調整設定。 您可以使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-powershell#change-the-capacity-of-a-scale-set) 作為因應措施。 由於 PowerShell 版本差異，您必須使用 `-Name` 參數，而不是 `-VMScaleSetName`。

- <!-- TBD -  IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結 D 系列 VM 的資料磁碟數目會不正確。 所有受支援的 D 系列 VM 均可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD -  IS ASDK --> 當 VM 映像建立失敗時，可能會在 VM 映像計算刀鋒視窗上新增一個您無法刪除的失敗項目。

  因應措施是，建立一個具有虛擬 VHD 的新 VM 映像，您可以透過 Hyper-V 建立此 VHD (New-VHD -Path C:\dummy.vhd -Fixed -SizeBytes 1 GB)。 此程序應該可以修正防止刪除失敗項目的問題。 接著，在建立虛擬映像後的 15 分鐘之後，您便可以順利刪除它。

  您可以接著嘗試下載先前失敗的 VM 映像。

- <!-- TBD -  IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 - IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。

#### <a name="networking"></a>網路功能
- <!-- TBD - IS ASDK --> 您無法在系統管理員或使用者入口網站中建立使用者定義的路由。 請使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-powershell) 作為因應措施。

- <!-- 1766332 - IS, ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 2388980 -  IS ASDK --> 在建立 VM 並與公用 IP 位址建立關聯之後，您就無法將 VM 與該 IP 位址取消關聯。 取消關聯看似可以運作，但先前指派的公用 IP 位址會繼續與原始 VM 保持關聯。

  目前，您只能將新的公用 IP 位址用於新建立的 VM。

  即使您將 IP 位址重新指派給新的 VM (通常稱為 *VIP 交換*)，還是會發生這種行為。 之後透過此 IP 位址連線的所有嘗試都會導致連線到原始 VM，而不是新的 VM。

- <!-- 2292271 - IS ASDK --> 如果供應項目和方案與租用戶訂用帳戶相關聯，而您將屬於供應項目和方案一部分的網路資源提高其配額限制，則新的限制不會套用到該訂用帳戶。 不過，新的限制會套用到配額增加後所建立的新訂用帳戶中。

  若要解決這個問題，請使用附加方案，在方案已與訂用帳戶產生關聯時，增加網路配額。 如需詳細資訊，請參閱如何[提供附加方案](.\.\azure-stack-subscribe-plan-provision-vm.md#to-make-an-add-on-plan-available)。

- <!-- 2304134 IS ASDK --> 您無法刪除有相關聯 DNS 區域資源或路由表資源的訂用帳戶。 若要成功刪除訂用帳戶，您必須先從租用戶訂用帳戶中刪除 DNS 區域和路由表資源。


- <!-- 1902460 -  IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 -  IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- TBD -  IS ASDK --> Azure Stack 不支援在部署 VM 之後將額外的網路介面新增至 VM 執行個體。 如果 VM 需要多個網路介面，必須在部署階段就定義這些網路介面。


#### <a name="sql-and-mysql"></a>SQL 和 MySQL
- <!-- TBD - ASDK --> 裝載伺服器的資料庫必須專供資源提供者和使用者工作負載使用。 您無法使用任何其他取用者 (包括「應用程式服務」) 正在使用的執行個體。

- <!-- IS, ASDK --> 當您為 SQL 和 MySQL 資源提供者建立 SKU 時，[系列] 名稱不支援特殊字元，包括空格和句點。

#### <a name="app-service"></a>App Service 方案
- <!-- 2352906 - IS ASDK --> 使用者必須在於訂用帳戶中建立第一個 Azure 函式之前，先註冊儲存體資源提供者。

- <!-- TBD - IS ASDK --> 為了向外延展基礎結構 (背景工作、管理、前端角色)，您必須如「計算」的版本資訊所述，使用 PowerShell。  

- <!-- TBD - IS ASDK --> 目前，App Service 只能部署到「預設提供者訂用帳戶」。 <!-- In a future update, App Service will deploy into the new *Metering subscription* that was introduced in Azure Stack 1804. When Metering is supported for use, all existing deployments will be migrated to this new subscription type. -->

#### <a name="usage"></a>使用量  
- <!-- TBD -  IS ASDK --> 使用量公用 IP 位址使用計量資料會針對每一筆記錄顯示相同的 EventDateTime 值，而不是建立記錄時顯示的 TimeDate 時間戳記。 目前，您無法使用這項資料來執行公用 IP 位址使用狀況的正確計量。

<!-- #### Identity -->


## <a name="build-201805131"></a>組建 20180513.1

### <a name="new-features"></a>新功能
此組建包含下列適用於 Azure Stack 的改良功能與修正。  

- <!-- 1759172 - IS, ASDK --> **新的系統管理訂用帳戶**。 1804 版的入口網站中有兩個新的可用訂用帳戶類型。 這些新的訂用帳戶類型會新增至預設提供者訂用帳戶，從 1804 版開始，新的 Azure Stack 安裝中就能看見這些類型。 請勿在這一版 Azure Stack 中使用這些新的訂用帳戶類型。 <!-- We will announce the availability to use these subscription types in with a future update. -->

  這些新的訂用帳戶類型會顯示出來，但會是較大變更的一部分，目的是要保護預設提供者訂用帳戶，並讓您更輕鬆地部署共用資源，例如 SQL 主控伺服器。

  目前可用的三種訂用帳戶類型如下：  
  - 預設提供者訂用帳戶：繼續使用此訂用帳戶類型。
  - 計量訂用帳戶：請勿使用此訂用帳戶類型。
  - 取用訂用帳戶：請勿使用此訂用帳戶類型

### <a name="fixed-issues"></a>已修正的問題
- <!-- IS, ASDK --> 您再也不必先重新整理 [更新] 圖格，系統管理員入口網站就會顯示資訊。

- <!-- 2050709 - IS, ASDK --> 您現在可以使用系統管理員入口網站，來編輯 Blob 服務、表格服務和佇列服務的儲存體計量。

- <!-- IS, ASDK --> 當您在 [網路] 下按一下 [連線] 來設定 VPN 連線時，目前可用的選項就只有 [站對站 (IPsec)]。

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**

<!-- ### Changes  -->
### <a name="additional-releases-timed-with-this-update"></a>與此更新同時發佈的其他項目  
以下項目現在可用，但是不需要 Azure Stack 更新 1804。
- **Microsoft Azure Stack System Center Operations Manager 監視組件的更新**。 適用於 Azure Stack 的新版本 (1.0.3.0) Microsoft System Center Operations Manager 監視套件，現在可供[下載](https://www.microsoft.com/download/details.aspx?id=55184)。 使用此版本，您可以在您新增已連線的 Azure Stack 部署時使用服務主體。 此版本也提供更新管理體驗，可讓您直接從 Operations Manager 中採取補救措施。 也有新的儀表板，會顯示資源提供者、縮放單位以及縮放單位節點。

- **新的 Azure Stack 管理員 PowerShell 1.3.0 版**。  Azure Stack PowerShell 1.3.0 現在可供安裝。 這個版本會為所有管理員資源提供者，提供用來管理 Azure Stack 的命令。  有了這個版本，我們就會從 Azure Stack Tools GitHub [存放庫](https://github.com/Azure/AzureStack-Tools)取代部分內容。

   如需安裝詳細資訊，請遵循 Azure Stack 模組 1.3.0 的[指示](.\.\azure-stack-powershell-install.md)或[說明](https://docs.microsoft.com/powershell/azure/azure-stack/overview?view=azurestackps-1.3.0)內容。

- **初始版本的 Azure Stack API Rest 參考**。 [所有 Azure Stack 管理員資源提供者的 API 參考](https://docs.microsoft.com/rest/api/azure-stack/)現已發行。

### <a name="known-issues"></a>已知問題

#### <a name="portal"></a>入口網站
- <!-- TBD - IS ASDK --> 無法在系統管理員入口網站內，[從下拉式清單開啟新的支援要求](.\.\azure-stack-manage-portals.md#quick-access-to-help-and-support)。 請改用下列連結：     
    - 針對 Azure Stack 開發套件，請使用 https://aka.ms/azurestackforum。    

- <!-- 2403291 - IS ASDK --> 您可能還無法使用系統管理員和使用者入口網站底部的水平捲軸。 如果您無法存取水平捲軸，請使用階層連結來瀏覽至入口網站中的上一個刀鋒視窗，方法是從入口網站左上角的階層連結清單中，選取您想要檢視的刀鋒視窗名稱。
  ![階層連結](media/asdk-release-notes/breadcrumb.png)

- <!-- TBD -  IS ASDK --> 刪除使用者訂用帳戶會產生孤立的資源。 因應措施是，先刪除使用者資源或整個資源群組，然後再刪除使用者訂用帳戶。

- <!-- TBD -  IS ASDK --> 您無法使用 Azure Stack 入口網站來檢視對您訂用帳戶的權限。 因應措施是，使用 PowerShell 來確認權限。

-   <!-- TBD -  IS ASDK --> 在管理員入口網站中，您可能會看到 Microsoft.Update.Admin 元件的重大警示。 警示名稱、說明及補救方式全部會顯示為：  
    - *錯誤 - 遺失 FaultType ResourceProviderTimeout 的範本。*

    您可以放心地忽略此警示。

#### <a name="health-and-monitoring"></a>健康情況和監視
- <!-- 1264761 - IS ASDK --> 您可能會看到「健康情況控制器」元件出現具有下列詳細資料的警示：  

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

  您可以放心地忽略這兩個警示。 這兩個警示會在一段時間過後自動關閉。  

#### <a name="marketplace"></a>Marketplace
- 使用者不需訂用帳戶就能瀏覽完整的市集，而且將會看到如方案和供應項目的管理項目。 對使用者而言，這些都是非功能性項目。

#### <a name="compute"></a>計算
- <!-- TBD -  IS ASDK --> 無法在入口網站中使用虛擬機器擴展集的調整設定。 您可以使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-powershell#change-the-capacity-of-a-scale-set) 作為因應措施。 由於 PowerShell 版本差異，您必須使用 `-Name` 參數，而不是 `-VMScaleSetName`。

- <!-- TBD -  IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結至 DS 系列 VM 的資料磁碟數目會不正確。 DS 系列 VM 可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD -  IS ASDK --> 當 VM 映像建立失敗時，可能會在 VM 映像計算刀鋒視窗上新增一個您無法刪除的失敗項目。

  因應措施是，建立一個具有虛擬 VHD 的新 VM 映像，您可以透過 Hyper-V 建立此 VHD (New-VHD -Path C:\dummy.vhd -Fixed -SizeBytes 1 GB)。 此程序應該可以修正防止刪除失敗項目的問題。 接著，在建立虛擬映像後的 15 分鐘之後，您便可以順利刪除它。

  您可以接著嘗試下載先前失敗的 VM 映像。

- <!-- TBD -  IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 - IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。

#### <a name="networking"></a>網路功能
- <!-- 1766332 - IS, ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 2388980 -  IS ASDK --> 在建立 VM 並與公用 IP 位址建立關聯之後，您就無法將 VM 與該 IP 位址取消關聯。 取消關聯看似可以運作，但先前指派的公用 IP 位址會繼續與原始 VM 保持關聯。

  目前，您只能將新的公用 IP 位址用於新建立的 VM。

  即使您將 IP 位址重新指派給新的 VM (通常稱為 *VIP 交換*)，還是會發生這種行為。 之後透過此 IP 位址連線的所有嘗試都會導致連線到原始 VM，而不是新的 VM。

- <!-- 2292271 - IS ASDK --> 如果供應項目和方案與租用戶訂用帳戶相關聯，而您將屬於供應項目和方案一部分的網路資源提高其配額限制，則新的限制不會套用到該訂用帳戶。 不過，新的限制會套用到配額增加後所建立的新訂用帳戶中。

  若要解決這個問題，請使用附加方案，在方案已與訂用帳戶產生關聯時，增加網路配額。 如需詳細資訊，請參閱如何[提供附加方案](.\.\azure-stack-subscribe-plan-provision-vm.md#to-make-an-add-on-plan-available)。

- <!-- 2304134 IS ASDK --> 您無法刪除有相關聯 DNS 區域資源或路由表資源的訂用帳戶。 若要成功刪除訂用帳戶，您必須先從租用戶訂用帳戶中刪除 DNS 區域和路由表資源。


- <!-- 1902460 -  IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 -  IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- TBD -  IS ASDK --> Azure Stack 不支援在部署 VM 之後將額外的網路介面新增至 VM 執行個體。 如果 VM 需要多個網路介面，必須在部署階段就定義這些網路介面。


#### <a name="sql-and-mysql"></a>SQL 和 MySQL
- <!-- TBD - ASDK --> 裝載伺服器的資料庫必須專供資源提供者和使用者工作負載使用。 您無法使用任何其他取用者 (包括「應用程式服務」) 正在使用的執行個體。

- <!-- IS, ASDK --> 當您為 SQL 和 MySQL 資源提供者建立 SKU 時，[系列] 名稱不支援特殊字元，包括空格和句點。

#### <a name="app-service"></a>App Service 方案
- <!-- TBD -  IS ASDK --> 使用者必須在於訂用帳戶中建立第一個 Azure 函式之前，先註冊儲存體資源提供者。

- <!-- TBD -  IS ASDK --> 為了向外延展基礎結構 (背景工作、管理、前端角色)，您必須如「計算」的版本資訊所述，使用 PowerShell。

#### <a name="usage"></a>使用量  
- <!-- TBD -  IS ASDK --> 使用量公用 IP 位址使用計量資料會針對每一筆記錄顯示相同的 EventDateTime 值，而不是建立記錄時顯示的 TimeDate 時間戳記。 目前，您無法使用這項資料來執行公用 IP 位址使用狀況的正確計量。

<!--
#### Identity
-->



#### <a name="downloading-azure-stack-tools-from-github"></a>從 GitHub 下載 Azure Stack 工具
- 使用 *invoke-webrequest* PowerShell Cmdlet 從 GitHub 下載 Azure Stack 工具時，您會收到下列錯誤：     
    -  *invoke-webrequest: 要求已經中止: 無法建立 SSL/TLS 的安全通道。*     

  之所以發生此錯誤，是因為新近的 GitHub 支援取代 Tlsv1 和 Tlsv1.1 密碼編譯標準 (PowerShell 的預設值)。 如需詳細資訊，請參閱[弱式密碼編譯標準移除通知](https://githubengineering.com/crypto-removal-notice/)。

<!-- #### Identity -->



