---
title: Azure Stack 1808 更新 | Microsoft Docs
description: 了解 Azure Stack 整合系統 1808 更新的新功能，包括已知問題和更新下載位置。
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
ms.date: 09/28/2018
ms.author: sethm
ms.reviewer: justini
ms.openlocfilehash: 7278f784316fdbd7170bb69b9414911b8f5b3823
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452678"
---
# <a name="azure-stack-1808-update"></a>Azure Stack 1808 更新

「適用於：Azure Stack 整合系統」

本文將說明 1808 更新套件的內容。 此更新套件包括此版 Azure Stack 的改良功能、修正及已知問題。 本文也包含連結，讓您下載更新。 已知問題分為直接與更新程序相關的問題，以及組建 (安裝後) 的相關問題。

> [!IMPORTANT]  
> 這個更新程式封裝僅適用於 Azure Stack 整合系統。 請勿將此更新套件套用至 Azure Stack 開發套件。

## <a name="build-reference"></a>建置參考

Azure Stack 1808 更新組建編號為 **1.1808.0.97**。  

### <a name="new-features"></a>新功能

此更新包含下列適用於 Azure Stack 的改良功能。

- <!--  2682594   | IS  -->  **所有 Azure Stack 環境現在會使用國際標準時間 (UTC) 時區格式。**  所有記錄資料和相關資訊現在會以 UTC 格式顯示。 如果您是從未使用 UTC 安裝的舊版本更新，您的環境會更新為使用 UTC。 

- <!-- 2437250  | IS  ASDK --> **支援受控磁碟。** 您現在可以在 Azure Stack 虛擬機器和虛擬機器擴展集中使用受控磁碟。 如需詳細資訊，請參閱 [Azure Stack 受控磁碟：差異與注意事項](/azure/azure-stack/user/azure-stack-managed-disk-considerations)。

- <!-- 2563799  | IS  ASDK -->  **Azure 監視器**。 和 Azure 上的 Azure 監視器一樣，Azure Stack 上的 Azure 監視器也可為大部分服務提供基底層級的基礎結構計量和記錄。 如需詳細資訊，請參閱 [Azure Stack 上的 Azure 監視器](/azure/azure-stack/user/azure-stack-metrics-azure-data)。

- <!-- 2487932| IS -->  **準備延伸主機**。 您可以使用延伸主機來減少所需的 TCP/IP 通訊埠數目，從而協助保護 Azure Stack。 您可以藉由 1808 更新來準備 Azure Stack，以便使用延伸主機。 如需詳細資訊，請參閱 [Azure Stack 的延伸主機準備](/azure/azure-stack/azure-stack-extension-host-prepare)。

- <!-- IS --> **虛擬機器擴展集的資源庫項目現已內建**。  虛擬機器擴展集資源庫項目現在可於使用者和系統管理員入口網站中取用，不需要下載。  如果您升級至 1808，則可在升級完成時使用此項目。  

- <!-- IS, ASDK --> **虛擬機器擴展集縮放**.  您可以使用入口網站來[縮放虛擬機器擴展集](azure-stack-compute-add-scalesets.md#scale-a-virtual-machine-scale-set) (VMSS)。    

- <!-- 2489570 | IS ASDK--> **支援自訂 IPSec/IKE 原則組態**，適用於 [Azure Stack 中的 VPN 閘道](/azure/azure-stack/azure-stack-vpn-gateway-about-vpn-gateways)。

- <!-- | IS ASDK--> **Kubernetes Marketplace 項目**。 您現在可以使用 [Kubernetes Marketplace 項目](azure-stack-solution-template-kubernetes-cluster-add.md)來部署 Kubernetes 叢集。 使用者可以選取 Kubernetes 項目並且填入幾個參數，以將 Kubernetes 叢集部署至 Azure Stack。 範本的目的是讓使用者以少數幾個步驟輕鬆設定開發/測試 Kubernetes 部署。

- <!-- | IS ASDK--> **區塊鏈範本**。 您現在可以在 Azure Stack 上執行[以太坊聯盟部署](azure-stack-ethereum.md)。 您可以在 [Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates)中找到三個新範本。 這些範本可以讓使用者僅須具備最少的 Azure 和以太坊知識，即可部署及設定多成員聯盟的以太坊網路。 範本的目的是讓使用者以少數幾個步驟輕鬆設定開發/測試區塊鏈部署。

- <!-- | IS ASDK--> **API 版本設定檔 2017-03-09-profile 已更新為 2018-03-01-hybrid**。 API 設定檔會指定 Azure 資源提供者和 Azure REST 端點的 API 版本。 如需設定檔的詳細資訊，請參閱[管理 Azure Stack 中的 API 版本設定檔](/azure/azure-stack/user/azure-stack-version-profiles)。

 ### <a name="fixed-issues"></a>已修正的問題
- <!-- IS ASDK--> 我們已修正在入口網站中建立可用性設定組時，設定組中只有 1 個容錯網域和 1 個更新網域的問題。 

- <!-- IS ASDK --> 現在可以在入口網站中使用虛擬機器擴展集的調整設定。  

- <!-- 2494144- IS, ASDK --> 已解決選取 VM 大小以進行部署時，某些 F 系列虛擬機器無法顯示的問題。 

- <!-- IS, ASDK --> 改善建立虛擬機器時的效能，並且更妥善地使用基礎儲存體。

- 針對效能、穩定性、安全性和 Azure Stack 所使用的作業系統提供了**多項修正**。


### <a name="changes"></a>變更
- <!-- 1697698  | IS, ASDK --> 使用者入口網站儀表板中的「快速入門教學課程」現在會連結至線上 Azure Stack 文件中的相關文章。

- <!-- 2515955   | IS ,ASDK--> Azure Stack 系統管理員和使用者入口網站中的「更多服務」已取代為「所有服務」。 您現在可以使用「所有服務」作為在 Azure Stack 入口網站中瀏覽的替代方法，就像是在 Azure 入口網站中執行的動作一樣。

- <!-- TBD | IS, ASDK --> Azure Stack 系統管理員和使用者入口網站中的「+ 新增」已取代為「+ 建立資源」。  您現在可以使用「+ 建立資源」作為在 Azure Stack 入口網站中瀏覽的替代方法，就像是在 Azure 入口網站中執行的動作一樣。  

- <!--  TBD – IS, ASDK --> 透過入口網站[建立虛擬機器擴展集](azure-stack-compute-add-scalesets.md) (VMSS) 時，無法再使用「基本 A」虛擬機器大小。 若要建立具有此大小的 VMSS，請使用 PowerShell 或範本。  

### <a name="common-vulnerabilities-and-exposures"></a>常見的弱點與漏洞

此更新會安裝下列更新：  

- [CVE-2018-0952](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-0952)
- [CVE-2018-8200](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8200)
- [CVE-2018-8204](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8204)
- [CVE-2018-8253](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8253)
- [CVE-2018-8339](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8339)
- [CVE-2018-8340](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8340)
- [CVE-2018-8341](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8341)
- [CVE-2018-8343](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8343)
- [CVE-2018-8344](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8344)
- [CVE-2018-8345](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8345)
- [CVE-2018-8347](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8347)
- [CVE-2018-8348](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8348)
- [CVE-2018-8349](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8349)
- [CVE-2018-8394](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8394)
- [CVE-2018-8398](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8398)
- [CVE-2018-8401](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8401)
- [CVE-2018-8404](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8404)
- [CVE-2018-8405](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8405)
- [CVE-2018-8406](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2018-8406)  

如需有關這些弱點的詳細資訊，請按一下上述連結，或參閱 Microsoft 知識庫文章 [4343887](https://support.microsoft.com/help/4343887)。

此更新也包含讓推測性執行旁路弱點 (也稱為 L1 終端機錯誤 (L1TF)) 的風險降低，如 [Microsoft 資訊安全諮詢 ADV180018](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180018) 中所述。  

### <a name="prerequisites"></a>必要條件

- 請先安裝 Azure Stack [1807 更新](azure-stack-update-1807.md)，然後才套用 Azure Stack 1808 更新。 

- 安裝最新可用的 [1807 版更新或 Hotfix](azure-stack-update-1807.md#post-update-steps)。  
  > [!TIP]  
  > 訂閱下列 *RRS* 或 *Atom* 摘要，以掌握最新的 Azure Stack Hotfix：
  > - RRS： https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/rss … 
  > - Atom： https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/atom …


- 開始安裝此更新之前，請先執行 [Test-AzureStack](azure-stack-diagnostic-test.md) 與下列參數來驗證 Azure Stack 的狀態，並解決所發現的一切運作問題，包括所有警告和失敗。 也請檢閱作用中警示，並將所有需要採取動作的警示解決。  

  ```PowerShell
  Test-AzureStack -Include AzsControlPlane, AzsDefenderSummary, AzsHostingInfraSummary, AzsHostingInfraUtilization, AzsInfraCapacity, AzsInfraRoleSummary, AzsPortalAPISummary, AzsSFRoleSummary, AzsStampBMCSummary
  ```   

### <a name="known-issues-with-the-update-process"></a>關於更新程序的已知問題

- 當您在 1808 更新之後執行 [Test-AzureStack](azure-stack-diagnostic-test.md) 時，來自基礎板管理控制器 (BMC) 的警告訊息隨即顯示。 您可以放心地忽略此警告。

- <!-- 2468613 - IS -->在安裝 1807 更新期間，您可能會看到警示，標題為「錯誤 - FaultType UserAccounts.New 的範本已遺失」  您可以放心地忽略這些警示。 在此更新完成之後，這些警示會自動關閉。

- <!-- 2489559 - IS --> 在此更新安裝期間，請勿嘗試建立虛擬機器。 如需管理更新的詳細資訊，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md#plan-for-updates)。

- <!-- 2830461 - IS --> 在某些情況下，當需要注意某個更新時，可能並不會產生對應的警示。 入口網站仍然會反映正確的狀態而不受影響。

### <a name="post-update-steps"></a>更新後步驟
在安裝此更新之後，安裝任何適用的 Hotfix。 如需詳細資訊，請檢視下列知識庫文章，以及我們的[服務原則](azure-stack-servicing-policy.md)。 
- [KB 4464226 – Azure Stack Hotfix Azure Stack Hotfix 1.1808.1.99]( https://support.microsoft.com/help/4464226)


## <a name="known-issues-post-installation"></a>已知問題 (安裝後)

以下是此組建版本的安裝後已知問題。

### <a name="portal"></a>入口網站

- Azure Stack 技術文件會以最新版本為主。 由於各版本的入口網站會有所變更，您在使用 Azure Stack 入口網站時所看到的項目，可能與您在文件中所看到的內容不同。 

- <!-- TBD - IS ASDK --> 您可能會在入口網站中看到空白的儀表板。 若要復原儀表板，請按一下 [編輯儀表板]，然後按一下滑鼠右鍵並選取 [重設為預設狀態]。

- <!-- 2930718 - IS ASDK --> 在系統管理員入口網站中存取任何使用者訂用帳戶的詳細資料時，在關閉刀鋒視窗之後按一下 [最近]，使用者訂用帳戶名稱未顯示。

- <!-- 3060156 - IS ASDK --> 在系統管理員和使用者入口網站中，按一下入口網站設定，然後選取 [刪除所有設定和私人儀表板]，並未如預期運作。 錯誤通知隨即顯示。 

- <!-- 2930799 - IS ASDK --> 在系統管理員和使用者入口網站中，[所有服務] 底下的 [DDoS 保護計劃] 資產未正確列出。 實際上，它無法在 Azure Stack 中使用。 如果您嘗試建立它，則會顯示錯誤，表示入口網站無法建立 Marketplace 項目。 

- <!-- 2930820 - IS ASDK --> 在系統管理員和使用者入口網站中，如果您搜尋 "Docker"，項目未正確傳回。 實際上，它無法在 Azure Stack 中使用。 如果您嘗試建立它，則會顯示具有錯誤指示的刀鋒視窗。 

- <!-- 2967387 – IS, ASDK --> 您用來登入 Azure Stack 系統管理員或使用者入口網站的帳戶，顯示為**無法辨識的使用者**。 當帳戶未指定「名字」或「姓氏」時，就會發生這種情形。 若要解決此問題，請編輯使用者帳戶，提供「名字」或「姓氏」。 您必須登出，然後再登入回入口網站。 

-  <!--  2873083 - IS ASDK --> 當您使用入口網站來建立虛擬機器擴展集 (VMSS) 時，如果使用 Internet Explorer，[執行個體大小] 下拉式清單無法正確載入。 若要解決此問題，請在使用入口網站來建立 VMSS 時使用另一個瀏覽器。  

- <!-- 2931230 – IS  ASDK --> 無法刪除以附加方案形式新增至使用者訂用帳戶的方案 (即使您從使用者訂用帳戶中移除此方案)。 此方案會持續保留，直到參考附加方案的訂用帳戶也遭到刪除為止。 

- <!--2760466 – IS  ASDK --> 當您安裝執行此版本的新 Azure Stack 環境時，可能不會顯示指出「需要啟用」的警示。 必須[啟用](azure-stack-registration.md)，才能使用市集摘要整合。  

- <!-- TBD - IS ASDK --> 不應該使用[隨 1804 版導入的](azure-stack-update-1804.md#new-features)兩個系統管理訂用帳戶類型。 這些訂用帳戶類型為「計量訂用帳戶」和「取用訂用帳戶」。 自 1804 版起，這些訂用帳戶類型就已經會在新的 Azure Stack 環境中顯示，只是還未就緒以供使用。 請繼續使用「預設提供者」訂用帳戶類型。

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

- <!-- 3090289 – IS, ASDK --> 套用 1808 更新之後，您在部署具有受控磁碟的 VM 時可能會遇到下列問題：

   1. 如果訂用帳戶是在 1808 更新之前建立的，部署具有受控磁碟的 VM 可能會失敗，且會有內部錯誤訊息。 若要解決此錯誤，請針對每個訂用帳戶遵循下列步驟：
      1. 在租用戶入口網站中，移至 [訂用帳戶] 並尋找訂用帳戶。 按一下 [資源提供者]，按一下 [Microsoft.Compute]，然後按一下 [重新註冊]。
      2. 在相同的訂用帳戶底下，移至 [存取控制 (IAM)]，並確認 [Azure Stack - 受控磁碟] 已列出。
   2. 如果您已設定多租用戶環境，則在與來賓目錄相關聯的訂用帳戶中部署 VM 可能會失敗，且會有內部錯誤訊息。 若要解決此錯誤，請依照下列步驟執行︰
      1. 套用 [1808 Azure Stack Hotfix](https://support.microsoft.com/help/4465859)。
      2. 依照[這篇文章](azure-stack-enable-multitenancy.md#registering-azure-stack-with-the-guest-directory)中的步驟，重新設定您的每個來賓目錄。


- <!-- 2869209 – IS, ASDK --> 當使用 [**Add-AzsPlatformImage** Cmdlet](https://docs.microsoft.com/powershell/module/azs.compute.admin/add-azsplatformimage?view=azurestackps-1.4.0) 時，您必須使用 **-OsUri** 參數作為要上傳磁碟的儲存體帳戶 URI。 如果您使用磁碟的本機路徑，此 Cmdlet 就會失敗，且具有下列錯誤：「長時間執行的作業失敗，狀態為「失敗」」。 

- <!--  2966665 – IS, ASDK --> 將 SSD 資料磁碟連結至進階大小的受控磁碟虛擬機器 (DS、DSv2、Fs、Fs_V2) 失敗且有錯誤：「無法更新虛擬機器 ‘vmname’ 的磁碟。錯誤：因為 VM 大小 (Standard_DS/Ds_V2/FS/Fs_v2) 不支援儲存體帳戶類型 ‘Premium_LRS’，所以要求的作業無法執行」

   若要解決此問題，請使用 Standard_LRS 資料磁碟，而不是 Premium_LRS 磁碟。 使用 Standard_LRS 資料磁碟不會變更 IOP 或帳單成本。 

- <!--  2795678 – IS, ASDK --> 當您使用入口網站以進階 VM 大小 (DS、Ds_v2、FS、FSv2) 建立虛擬機器 (VM) 時，會在標準儲存體帳戶中建立 VM。 在標準儲存體帳戶中建立不會影響功能、IOP 或帳單。 

   您可以放心地忽略警告，該警告指出：「您已選擇在支援進階磁碟的大小上使用標準磁碟。*這樣可能會影響作業系統效能，並不建議使用。請考慮改為使用進階儲存體 (SSD)。」*

- <!-- 2967447 - IS, ASDK --> 虛擬機器擴展集 (VMSS) 建立體驗會提供 CentOS 架構 7.2 作為部署選項。 因為該映像在 Azure Stack 上無法使用，所以請為您的部署選取其他作業系統，或使用 ARM 範本指定另一個 CentOS 映像，但操作員必須在部署之前已從市集下載該印象。  

- <!-- 2724873 - IS --> 開始使用 PowerShell Cmdlet **Start-AzsScaleUnitNode** 或 **Stop-AzsScaleunitNode** 來管理縮放單位時，第一次嘗試啟動或停止縮放單位可能會失敗。 如果第一次執行 Cmdlet 時發生失敗，請再次執行 Cmdlet。 第二次執行應該會成功地完成此作業。 

- <!-- TBD - IS ASDK --> 當您在 Azure Stack 使用者入口網站上建立虛擬機器時，入口網站所顯示可連結至 DS 系列 VM 的資料磁碟數目會不正確。 DS 系列 VM 可容納與 Azure 設定數目一樣多的資料磁碟。

- <!-- TBD - IS ASDK --> 如果您使用多租用戶環境，在不屬於預設租用戶的訂用帳戶中部署具有受控磁碟的 VM 會失敗，且具有下列錯誤：「資源作業已完成，但是終端佈建狀態為「失敗」。」

- <!-- TBD - IS ASDK --> 如果在 VM 部署上佈建延伸模組所花費的時間太長，使用者應該讓佈建逾時，而不是嘗試停止程序來將 VM 解除配置或刪除。  

- <!-- 1662991 IS ASDK --> Azure Stack 不支援 Linux VM 診斷。 當您部署啟用了 VM 診斷的 Linux VM 時，部署會失敗。 如果您透過診斷設定啟用 Linux VM 基本計量，部署也會失敗。  

- <!-- 2724961- IS ASDK --> 當您在「訂用帳戶」設定中註冊 **Microsoft.Insight** 資源提供者，並建立一個已啟用「客體 OS 診斷」功能的 Windows VM 時，VM 概觀頁面中的「CPU 百分比」圖表將無法顯示計量資料。

   若要尋找該 VM 的「CPU 百分比」圖表，請移至 [計量] 刀鋒視窗並顯示所有支援的 Windows VM 客體計量。



### <a name="networking"></a>網路功能  

- <!-- 1766332 - IS ASDK --> 如果您在 [網路] 下按一下 [建立 VPN 閘道] 來設定 VPN 連線，系統就會將 [原則式] 列為 VPN 類型。 請勿選取此選項。 Azure Stack 只支援 [路由式] 選項。

- <!-- 1902460 - IS ASDK --> Azure Stack 支援每一 IP 位址有一個「區域網路閘道」。 這適用於所有租用戶訂用帳戶。 建立第一個區域網路閘道連線之後，後續若嘗試使用相同的 IP 位址來建立區域網路閘道資源，就會被封鎖。

- <!-- 16309153 - IS ASDK --> 在 DNS 伺服器設定為 [自動] 的已建立虛擬網路上，變更成自訂 DNS 伺服器時失敗。 更新的設定並未推送至該 Vnet 中的 VM。

- <!-- 2702741 -  IS ASDK --> 針對使用「動態」配置方法來部署的公用 IP，不保證在發出 Stop-Deallocate 之後還能予以保留。

- <!-- 2529607 - IS ASDK --> 在 Azure Stack「祕密輪替」期間，會有一段 2 到 5 分鐘的期間無法存取「公用 IP 位址」。

-   <!-- 2664148 - IS ASDK --> 在租用戶使用 S2S VPN 通道來存取其虛擬機器的案例中，如果在已經建立閘道之後，才將內部部署子網路新增至「區域網路閘道」，租用戶就可能會遇到連線嘗試失敗的情況。 


<!-- ### SQL and MySQL-->


### <a name="app-service"></a>App Service 方案

- <!-- 2352906 - IS ASDK --> 使用者必須在於訂用帳戶中建立第一個 Azure 函式之前，先註冊儲存體資源提供者。

- <!-- 2489178 - IS ASDK --> 為了向外延展基礎結構 (背景工作、管理、前端角色)，您必須如「計算」的版本資訊所述，使用 PowerShell。



### <a name="usage"></a>使用量  
- <!-- TBD - IS ASDK --> 使用量公用 IP 位址使用計量資料會針對每一筆記錄顯示相同的 EventDateTime 值，而不是建立記錄時顯示的 TimeDate 時間戳記。 目前，您無法使用這項資料來執行公用 IP 位址使用狀況的正確計量。


<!-- #### Identity -->
<!-- #### Marketplace -->


## <a name="download-the-update"></a>下載更新
您可以從[這裡](https://aka.ms/azurestackupdatedownload)下載 Azure Stack 1808 更新套件。
  

## <a name="next-steps"></a>後續步驟
- 若要檢閱 Azure Stack 整合式系統的服務原則，以及必須如何做才能將狀態保持在支援的狀態，請參閱 [Azure Stack 服務原則](azure-stack-servicing-policy.md)。  
- 若要使用具有特殊權限的端點 (PEP) 來監視和繼續更新，請參閱[使用具有特殊權限的端點來監視 Azure Stack 中的更新](azure-stack-monitor-update.md)。  
- 如需 Azure Stack 中的更新管理概觀，請參閱[在 Azure Stack 中管理更新概觀](azure-stack-updates.md)。  
- 如需如何使用 Azure Stack 套用更新的相關資訊，請參閱[在 Azure Stack 中套用更新](azure-stack-apply-updates.md)。  
