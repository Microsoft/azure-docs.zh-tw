---
title: Azure Stack 1807 更新 | Microsoft Docs
description: 了解 Azure Stack 整合系統 1807 更新的新功能，包括已知問題和更新下載位置。
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
ms.openlocfilehash: 951c012dd22cf83741211b9601a5e7502de33be6
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393021"
---
# <a name="azure-stack-1807-update"></a>Azure Stack 1807 更新

「適用於：Azure Stack 整合系統」

本文將說明 1807 更新套件的內容。 此更新中包括此版 Azure Stack 的改良功能、修正及已知問題，以及可下載更新的位置。 已知問題分為直接與更新程序相關的問題，以及組建 (安裝後) 的相關問題。

> [!IMPORTANT]  
> 這個更新程式封裝僅適用於 Azure Stack 整合系統。 請勿將此更新套件套用至 Azure Stack 開發套件。

## <a name="build-reference"></a>建置參考

Azure Stack 1807 更新組建編號為 **1.1807.0.76**。  

### <a name="new-features"></a>新功能

此更新包含下列適用於 Azure Stack 的改良功能。

- <!-- 1658937 | ASDK, IS --> **根據預先定義的排程啟動備份** - 作為設備的 Azure Stack 現在可以自動地定期觸發基礎結構備份，無須人為介入。 若備份超過定義的保留期限，Azure Stack 也會自動清除其外部共用。 了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。

- <!-- 2496385 | ASDK, IS -->**已將資料轉送時間新增到備份時間總計中。** 了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。

-   <!-- 1702130 | ASDK, IS -->**備份外部容量現在會顯示正確的外部共用容量。** (以前是以硬式編碼將其設為 10 GB。)了解如何[使用 PowerShell 來啟用 Azure Stack 的備份](azure-stack-backup-enable-backup-powershell.md)以取得更多資訊。

- <!-- 2508488 |  IS   -->  [新增額外的縮放單位節點](azure-stack-add-scale-node.md)來**擴充容量**。

- <!-- 2753130 |  IS, ASDK   -->  **Azure Resource Manager 範本現在支援條件元素** - 您現在可以使用條件在 Azure Resource Manger 範本中部署資源。 您可以將範本設計成根據條件 (例如評估某個參數值是否存在) 來部署資源。 如需有關使用範本作為條件的資訊，請參閱 Azure 文件中的[依條件部署資源](https://docs.microsoft.com/azure/architecture/building-blocks/extending-templates/conditional-deploy)和 [Azure Resource Manager 範本的 Variables 區段](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-templates-variables)。 

   您也可以使用範本[將資源部署至多個訂用帳戶或資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-cross-resource-group-deployment)。  

- <!--2753073 | IS, ASDK -->  **Microsoft.Network API 資源版本支援已更新**，從適用於 Azure Stack 網路資源的 2015-06-15 更新成包含對 API 2017-10-01 版的支援。  此版本未包含對 2017-10-01 與 2015-06-15 之間資源版本的支援，但未來的版本中將會包含。  如需了解功能差異，請參閱 [Azure Stack 網路服務的注意事項](user/azure-stack-network-differences.md)。

- <!-- 2272116 | IS, ASDK   -->  **Azure Stack 已針對對外的 Azure Stack 基礎結構端點 (亦即針對 portal、adminportal、management 及 adminmanagement) 新增反向 DNS 查閱支援**。 這可允許從 IP 位址解析 Azure Stack 外部端點名稱。

- <!-- 2780899 |  IS, ASDK   --> **Azure Stack 現在支援為現有的 VM 新增額外的網路介面。**  藉由使用入口網站、PowerShell 及 CLI，即可使用此功能。 如需詳細資訊，請參閱 Azure 文件中的[新增或移除網路介面](https://docs.microsoft.com/azure/virtual-network/virtual-network-network-interface-vm)。 

- <!-- 2222444 | IS, ASDK   -->  **已提升網路使用量計量的精確度和彈性**。  網路使用量計量現在更精確，並將暫停的訂用帳戶、中斷期及競爭條件納入考慮。

- <!-- 2753080 | IS -->  **更新可用的通知。** 已連接的 Azure Stack 部署現在將會定期檢查受保護的端點，並判斷是否有您雲端適用的更新。 此通知會顯示在 [更新] 圖格中，就像在手動檢查並匯入新的更新後一樣。 深入了解如何[管理 Azure Stack 的更新](azure-stack-updates.md)。

- <!-- 2297790 | IS, ASDK -->  **改進 Azure Stack Syslog 用戶端 (預覽功能)**. 此用戶端允許將與 Azure Stack 基礎結構相關的稽核和記錄轉送到 Azure Stack 外的 Syslog 伺服器或安全性資訊與事件管理 (SIEM) 軟體。 Syslog 用戶端現在支援搭配純文字或 TLS 1.2 加密的 TCP 通訊協定，後者為預設設定。 您可以為 TLS 連線設定僅限伺服器的驗證或相互驗證。

  若要設定 Syslog 用戶端與 Syslog 伺服器的通訊方式 (例如通訊協定、加密及驗證)，請使用 *Set-SyslogServer* Cmdlet。 您可以從具特殊權限的端點 (PEP) 使用此 Cmdlet。

  若要為 Syslog 用戶端 TLS 1.2 相互驗證新增用戶端憑證，請使用 PEP 中的 Set-SyslogClient Cmdlet。

  在此預覽版中，您可看到的稽核和警示數量會多出許多。 

  由於此功能仍是預覽版，因此在生產環境中請勿倚賴此功能。

  如需詳細資訊，請參閱 [Azure Stack Syslog 轉送](azure-stack-integrate-security.md)。

<!-- - | ####### | IS, ASDK |  **Azure Resource Manager includes the region name.** With this release, objects retrieved from the Azure Resource Manager will now include the region name attribute. If an existing PowerShell script directly passes the object to another cmdlet, the script may produce an error and fail. This is Azure Resource Manager compliant behavior, and requires the calling client to subtract the region attribute. For more information about the Azure Resource Manager see [Azure Resource Manager Documentation](https://docs.microsoft.com/azure/azure-resource-manager/). verify 8-10 mdb -->

- <!-- TBD | IS, ASDK -->  **變更委派的提供者功能。** 從 1807 開始，為了能夠與 Azure 轉售商模型更一致，已將「委派的提供者」模型簡化，「委派的提供者」將無法建立其他「委派的提供者」，本質上就是將此模型壓平合併，而在單一層級上提供「委派的提供者」功能。 為了能夠轉換至訂用帳戶的新模型和管理，現在可在屬於相同目錄租用戶之新的或現有的「委派的提供者」訂用帳戶之間，移動使用者訂用帳戶。 您也可以將屬於「預設提供者訂用帳戶」的使用者訂用帳戶移至相同目錄租用戶中的「委派的提供者訂用帳戶」。  如需詳細資訊，請參閱 [Azure Stack 中的委派供應項目](azure-stack-delegated-provider.md)。

- <!-- 2536808 IS ASDK --> 針對使用從 Azure Marketplace 下載之映像建立的 VM，**縮短 VM 建立時間**。

- <!-- TBD | IS, ASDK -->  **提升 Azure Stack Capacity Planner 可用性**。 Azure Stack [Capacity Planner](http://aka.ms/azstackcapacityplanner) 現在提供一個在定義解決方案 SKU 時輸入 S2D 快取和 S2D 容量的簡化體驗。 1000 VM 限制已被移除。


### <a name="fixed-issues"></a>已修正的問題

- <!-- TBD | ASDK, IS --> 已對更新程序進行各種改進，使其變得更可靠。 此外，也已對基底基礎結構進行修正，將更新期間工作負載的可能停機時間縮到最短。

- <!--2292271 | ASDK, IS --> 修正問題：已修改的配額限制無法套用到現有訂用帳戶。 現在，如果「供應項目」和「方案」與使用者訂用帳戶相關聯，而您將該「供應項目」和「方案」所含網路資源的配額限制提高，則新限制不但會套用到新的訂用帳戶，也會套用到既有的訂用帳戶。

- <!-- 448955 | IS ASDK --> 您現在可以順利地對部署在 UTC+N 時區的系統查詢活動記錄。    

- <!-- 2319627 |  ASDK, IS --> 備份組態參數 (路徑/使用者名稱/密碼/加密金鑰) 的前置檢查不會再對備份組態進行不正確的設定。 (之前，如果在備份中設定了不正確的設定，當觸發備份時，備份就會發生失敗)。

- <!-- 2215948 |  ASDK, IS --> 現在，當您從外部共用手動刪除備份時，備份清單會進行重新整理。

- <!-- 2332636 | IS -->  使用 AD FS 進行部署時，此版本的更新不會再將預設提供者訂用帳戶的預設擁有者重設為內建的 **CloudAdmin** 使用者。

- <!-- 2360715 |  ASDK, IS -->  當您設定資料中心整合時，不會再從 Azure 檔案共用存取 AD FS 中繼資料檔案。 如需詳細資訊，請參閱[藉由提供同盟中繼資料檔案設定 AD FS 整合](azure-stack-integrate-identity.md#setting-up-ad-fs-integration-by-providing-federation-metadata-file)。 

- <!-- 2388980 | ASDK, IS --> 修正問題：防止使用者將先前已指派給網路介面或 Load Balancer 的現有公用 IP 位址，指派給新的網路介面或 Load Balancer。  

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [概觀] 時，[基本資訊] 窗格現在會正確地顯示所有預期的資訊。 

- <!-- 2551834 - IS, ASDK --> 當您在系統管理員或使用者入口網站中選取儲存體帳戶的 [標籤] 時，資訊現在會正確地顯示。

- <!-- TBD - IS ASDK --> 此版本的 Azure Stack 修正了導致無法從 OEM 延伸模組套件套用驅動程式更新的問題。

-   <!-- 2055809- IS ASDK --> 我們已修正導致您在 VM 建立失敗時無法從 [計算] 刀鋒視窗刪除 VM 的問題。  

- <!--  2643962 IS ASDK -->  「記憶體容量不足」的警示不再不正確地顯示。

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**。


<!-- ### Additional releases timed with this update    -->

### <a name="common-vulnerabilities-and-exposures"></a>常見的弱點與漏洞
Azure Stack 使用 Windows Server 2016 的 Server Core 安裝來裝載關鍵基礎結構。 此版本會在 Azure Stack 的基礎結構伺服器上安裝下列 Windows Server 2016 更新： 
- [CVE-2018-8206](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8206)
- [CVE-2018-8222](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8222)
- [CVE-2018-8282](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8282)
- [CVE-2018-8304](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8304)
- [CVE-2018-8307](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8307)
- [CVE-2018-8308](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8308) 
- [CVE-2018-8309](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8309)
- [CVE-2018-8313](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-8313)  

如需有關這些弱點的詳細資訊，請按一下上述連結，或參閱 Microsoft 知識庫文章 [4338814](https://support.microsoft.com/help/4338814) 和 [4345418](https://support.microsoft.com/help/4345418)。



## <a name="before-you-begin"></a>開始之前

### <a name="prerequisites"></a>必要條件

- 請先安裝 Azure Stack [1805 更新](azure-stack-update-1805.md)，然後才套用 Azure Stack 1807 更新。  沒有 1806 更新。  

- 安裝最新可用的 [1805 版更新或 Hotfix](azure-stack-update-1805.md#post-update-steps)。  
  > [!TIP]  
  > 訂閱下列 *RRS* 或 *Atom* 摘要，以掌握最新的 Azure Stack Hotfix：
  > - RRS： https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/rss … 
  > - Atom： https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/atom …


- 開始安裝此更新之前，請先執行 [Test-AzureStack](azure-stack-diagnostic-test.md) 與下列參數來驗證 Azure Stack 的狀態，並解決所發現的一切運作問題，包括所有警告和失敗。 也請檢閱作用中警示，並將所有需要採取動作的警示解決。  

  ```PowerShell
  Test-AzureStack -Include AzsControlPlane, AzsDefenderSummary, AzsHostingInfraSummary, AzsHostingInfraUtilization, AzsInfraCapacity, AzsInfraRoleSummary, AzsPortalAPISummary, AzsSFRoleSummary, AzsStampBMCSummary
  ``` 

### <a name="known-issues-with-the-update-process"></a>關於更新程序的已知問題

- <!-- 2468613 - IS -->在安裝 1807 更新期間，您可能會看到警示，標題為「錯誤 - FaultType UserAccounts.New 的範本已遺失」  您可以放心地忽略這些警示。 在此更新完成之後，這些警示會自動關閉。

- <!-- 2489559 - IS --> 在此更新安裝期間，請勿嘗試建立虛擬機器。 如需管理更新的詳細資訊，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md#plan-for-updates)。

- <!-- 2830461 - IS --> 在某些情況下，當需要注意某個更新時，可能並不會產生對應的警示。 入口網站仍然會反映正確的狀態而不受影響。

### <a name="post-update-steps"></a>更新後步驟
在安裝此更新之後，安裝任何適用的 Hotfix。 如需詳細資訊，請檢視下列知識庫文章，以及我們的[服務原則](azure-stack-servicing-policy.md)。 
- [KB 4464231 – Azure Stack Hotfix Azure Stack Hotfix 1.1807.1.78]( https://support.microsoft.com/help/4464231)

 <!-- 2933866 – IS --> 安裝此更新後，您可能會看到**改進失敗的更新安裝狀態。** 其中可能包含已修訂成反映這兩個新 STATE 類別的先前更新安裝失敗相關資訊。 新的 STATE 類別為 *PreparationFailed* 和 *InstallationFailed*。  

## <a name="known-issues-post-installation"></a>已知問題 (安裝後)

以下是此組建版本的安裝後已知問題。

### <a name="portal"></a>入口網站

- Azure Stack 技術文件會以最新版本為主。 由於各版本的入口網站會有所變更，您在使用 Azure Stack 入口網站時所看到的項目，可能與您在文件中所看到的內容不同。 

- 無法在系統管理員入口網站內，[從下拉式清單開啟新的支援要求](azure-stack-manage-portals.md#quick-access-to-help-and-support)。 針對 Azure Stack 整合式系統，應使用下列連結：[https://aka.ms/newsupportrequest](https://aka.ms/newsupportrequest)。

- <!-- 2931230 – IS  ASDK --> 無法刪除以附加方案形式新增至使用者訂用帳戶的方案 (即使您從使用者訂用帳戶中移除此方案)。 此方案會持續保留，直到參考附加方案的訂用帳戶也遭到刪除為止。 

- <!--2760466 – IS  ASDK --> 當您安裝執行此版本的新 Azure Stack 環境時，可能不會顯示指出「需要啟用」的警示。 必須[啟用](azure-stack-registration.md)，才能使用市集摘要整合。  

- <!-- TBD - IS ASDK --> 不應該使用[隨 1804 版導入的](azure-stack-update-1804.md#new-features)兩個系統管理訂用帳戶類型。 這些訂用帳戶類型為「計量訂用帳戶」和「取用訂用帳戶」。 自 1804 版起，這些訂用帳戶類型就已經會在新的 Azure Stack 環境中顯示，只是還未就緒以供使用。 請繼續使用「預設提供者」訂用帳戶類型。

- <!-- 2403291 - IS ASDK --> 您可能還無法使用系統管理員和使用者入口網站底部的水平捲軸。 如果您無法存取水平捲軸，請使用階層連結來瀏覽至入口網站中的上一個刀鋒視窗，方法是從入口網站左上角的階層連結清單中，選取您想要檢視的刀鋒視窗名稱。

  ![階層連結](media/azure-stack-update-1804/breadcrumb.png)

- <!-- TBD - IS --> 可能無法在管理員入口網站中檢視計算或儲存體資源。 之所以發生此問題，是因為在更新安裝期間發生錯誤，且誤將更新報告為成功。 如果發生此問題，請連絡 Microsoft 客戶支援服務以尋求協助。

- <!-- TBD - IS --> 您可能會在入口網站中看到空白的儀表板。 若要復原儀表板，請選取位於入口網站右上角的齒輪圖示，然後選取 [還原預設設定]。

- <!-- TBD - IS ASDK --> 刪除使用者訂用帳戶會產生孤立的資源。 因應措施是，先刪除使用者資源或整個資源群組，然後再刪除使用者訂用帳戶。

- <!-- TBD - IS ASDK --> 您無法使用 Azure Stack 入口網站來檢視對您訂用帳戶的權限。 因應措施是，使用 PowerShell 來確認權限。


### <a name="health-and-monitoring"></a>健康情況和監視
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


- <!-- 2812138 | IS --> 您可能會看到具有下列詳細資料的**儲存體**元件警示：

   - 名稱：儲存體服務內部通訊錯誤  
   - 嚴重性：嚴重  
   - 元件：儲存體  
   - 描述：將要求傳送至下列節點時，發生儲存體服務內部通訊錯誤。  

    您可以放心地忽略此警示，但必須手動關閉此警示。

- <!-- 2368581 - IS. ASDK --> 身為 Azure Stack 操作員，如果您收到記憶體不足的警示，而且租用戶虛擬機器無法部署並出現「Fabric VM 建立錯誤」，則可能表示 Azure Stack 戳記的可用記憶體不足。 請使用 [Azure Stack 容量規劃工具](https://gallery.technet.microsoft.com/Azure-Stack-Capacity-24ccd822)來充分了解可供工作負載使用的容量。


### <a name="compute"></a>計算

- <!-- 2724873 - IS --> 開始使用 PowerShell Cmdlet **Start-AzsScaleUnitNode** 或 **Stop-AzsScaleunitNode** 來管理縮放單位時，第一次嘗試啟動或停止縮放單位可能會失敗。 如果第一次執行 Cmdlet 時發生失敗，請再次執行 Cmdlet。 第二次執行應該會成功地完成此作業。 

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


- <!-- TBD - IS ASDK --> 無法在入口網站中使用虛擬機器擴展集的調整設定。 您可以使用 [Azure PowerShell](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-powershell#change-the-capacity-of-a-scale-set) 作為因應措施。 由於 PowerShell 版本差異，您必須使用 `-Name` 參數，而不是 `-VMScaleSetName`。

- <!-- TBD - IS --> 當您移至 [新增] > [計算] > [可用性設定組] 在入口網站中建立可用性設定組時，您只能以一個容錯網域和一個更新網域建立可用性設定組。 因應措施是在建立新的虛擬機器時，使用 PowerShell、CLI 或從入口網站建立可用性設定組。

- <!-- TBD - IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結至 DS 系列 VM 的資料磁碟數目會不正確。 DS 系列 VM 可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD - IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。  

- <!-- 2724961- IS ASDK --> 當您在「訂用帳戶」設定中註冊 **Microsoft.Insight** 資源提供者，並建立已啟用「客體 OS 診斷」功能的 Windows VM 時，VM 概觀頁面不會顯示計量資料。 

   若要尋找該 VM 的計量資料 (例如「CPU 百分比」圖表)，請移至 [計量] 刀鋒視窗，並顯示所有支援的 Windows VM 客體計量。

### <a name="networking"></a>網路功能  

- <!-- 1766332 - IS ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 1902460 - IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 - IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- 2702741 -  IS ASDK --> 針對使用「動態」配置方法來部署的公用 IP，不保證在發出 Stop-Deallocate 之後還能予以保留。

- <!-- 2529607 - IS ASDK --> 在 Azure Stack「祕密輪替」期間，會有一段 2 到 5 分鐘的期間無法存取「公用 IP 位址」。

-   <!-- 2664148 - IS ASDK --> 在租用戶使用 S2S VPN 通道來存取其虛擬機器的案例中，如果在已經建立閘道之後，才將內部部署子網路新增至「區域網路閘道」，租用戶就可能會遇到連線嘗試失敗的情況。 


### <a name="sql-and-mysql"></a>SQL 和 MySQL



- <!-- No Number - IS, ASDK --> 當您為 SQL 和 MySQL 資源提供者建立 SKU 時，[系列] 名稱不支援特殊字元，包括空格和句點。 

- <!-- TBD - IS --> 僅支援資源提供者在裝載 SQL 或 MySQL 的伺服器上建立項目。 在不是由資源提供者建立的主機伺服器上建立項目，可能會導致不相符的狀態。  

> [!NOTE]  
> <!-- TBD - IS --> 在更新至此版 Azure Stack 之後，您可以繼續使用先前部署的 SQL 和 MySQL 資源提供者。  建議您在新版本可供使用時更新 SQL 和 MySQL。 如同 Azure Stack，請循序套用 SQL 和 MySQL 資源提供者的更新。 例如，如果您使用 1804 版，請先套用 1805 版，再更新至 1807 版。  
>  
> 安裝此更新並不會影響使用者目前對 SQL 或 MySQL 資源提供者的使用。 
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
您可以從[這裡](https://aka.ms/azurestackupdatedownload)下載 Azure Stack 1807 更新套件。


## <a name="next-steps"></a>後續步驟
- 若要檢閱 Azure Stack 整合式系統的服務原則，以及必須如何做才能將狀態保持在支援的狀態，請參閱 [Azure Stack 服務原則](azure-stack-servicing-policy.md)。  
- 若要使用具有特殊權限的端點 (PEP) 來監視和繼續更新，請參閱[使用具有特殊權限的端點來監視 Azure Stack 中的更新](azure-stack-monitor-update.md)。  
- 如需 Azure Stack 中的更新管理概觀，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md)。  
- 如需如何使用 Azure Stack 套用更新的相關資訊，請參閱[在 Azure Stack 中套用更新](azure-stack-apply-updates.md)。  
