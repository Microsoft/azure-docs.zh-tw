---
title: Azure AD Connect Health 代理程式安裝 | Microsoft Docs
description: 這是 Azure AD Connect Health 頁面，其中說明 AD FS 與同步處理的代理程式安裝。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: mtillman
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/18/2017
ms.author: billmath
ms.openlocfilehash: c57e6d3e35994bea99e15f37ed0fb6aa2d108f74
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46303922"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Azure AD Connect Health 代理程式安裝
本文件會逐步引導您安裝和設定 Azure AD Connect Health 代理程式。 您可以從 [這裡](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent)下載代理程式。

## <a name="requirements"></a>需求
下表是使用 Azure AD Connect Health 的需求清單。

| 需求 | 說明 |
| --- | --- |
| Azure AD Premium |Azure AD Connect Health 是 Azure AD Premium 的一個功能，而且需要 Azure AD Premium。 </br></br>如需詳細資訊，請參閱[開始使用 Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) </br>若要開始使用 30 天免費試用版，請參閱[開始使用試用版](https://azure.microsoft.com/trial/get-started-active-directory/)。 |
| 您必須是 Azure AD 的全域系統管理員，才能開始使用 Azure AD Connect Health |依預設，只有全域系統管理員可以安裝和設定 Health 代理程式，以便開始使用、存取入口網站，以及在 Azure AD Connect Health 內執行任何作業。 如需詳細資訊，請參閱[管理您的 Azure AD 目錄](../fundamentals/active-directory-administer.md)。 <br><br> 使用角色型存取控制，您可以允許貴組織中的其他使用者存取 Azure AD Connect Health。 如需詳細資訊，請參閱[適用於 Azure AD Connect Health 的角色型存取控制](how-to-connect-health-operations.md#manage-access-with-role-based-access-control)。 </br></br>**重要：** 在安裝代理程式時使用的帳戶必須是工作或學校帳戶。 不能是 Microsoft 帳戶。 如需詳細資訊，請參閱[以組織身分註冊 Azure](../fundamentals/sign-up-organization.md) |
| Azure AD Connect Health 代理程式安裝在每部目標伺服器上 | Azure AD Connect Health 會要求在目標伺服器上安裝及設定 Health 代理程式，才能接收資料及提供監視和分析功能。 </br></br>例如，若要從 AD FS 基礎結構取得資料，代理程式必須安裝於 AD FS 及 Web 應用程式 Proxy 伺服器上。 同樣地，若要取得內部部署 AD DS 基礎結構的相關資料，代理程式必須安裝在網域控制站上。 </br></br> |
| Azure 服務端點的輸出連線 | 在安裝期間和執行階段，代理程式需要連線至 Azure AD Connect Health 服務端點。 如果使用防火牆封鎖輸出連線，請確定已將下列端點新增至允許清單。 請參閱[輸出連線端點](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) | 
|以 IP 位址為基礎的輸出連線 | 如需防火牆上以 IP 位址為基礎的篩選，請參閱 [Azure IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。|
| 已篩選或停用輸出流量的 SSL 檢查 | 如果網路層的輸出流量有 SSL 檢查或終止，代理程式註冊步驟或資料上傳作業可能會失敗。 深入了解[如何設定 SSL 檢查](https://technet.microsoft.com/library/ee796230.aspx) |
| 防火牆連接埠 (位於執行代理程式的伺服器上) |為了讓代理程式能與 Azure AD Health 服務端點進行通訊，代理程式要求開啟下列防火牆連接埠。</br></br><li>TCP 通訊埠 443</li><li>TCP 通訊埠 5671</li> </br>深入了解[啟用防火牆連接埠](https://technet.microsoft.com/library/ms345310(v=sql.100).aspx) |
| 如果啟用 IE 增強式安全性，則允許下列網站 |如果啟用 IE 增強式安全性，則在即將安裝代理程式的伺服器上必須允許下列網站。</br></br><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>Azure Active Directory 所信任適用於您組織的同盟伺服器。 例如：https:\//sts.contoso.com</li> 深入了解[如何設定 IE](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing) |
| 確定已安裝 PowerShell 4.0 版或更新版本 | <li>Windows Server 2008 R2 隨附了 PowerShell 2.0 版，它對代理程式來說並不足夠。  如以下的 [Windows Server 2008 R2 伺服器上的代理程式安裝](#agent-installation-on-windows-server-2008-r2-servers)中所述，更新 PowerShell。</li><li>Windows Server 2012 隨附了 PowerShell 3.0 版，它對代理程式來說並不足夠。  [更新](http://www.microsoft.com/download/details.aspx?id=40855) Windows Management Framework。</li><li>Windows Server 2012 R2 和更新版本隨附了足夠的 PowerShell 最新版本。</li>|
|停用 FIPS|Azure AD Connect Health 代理程式不支援 FIPS。|

### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Azure 服務端點的輸出連線
 在安裝期間和執行階段，代理程式需要連線至 Azure AD Connect Health 服務端點。 如果使用防火牆封鎖輸出連線，請確定已將下列端點新增至允許清單。 深入了解[檢查輸出連線](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections)  
 
| 網域環境 | 必要 Azure 服務端點 |
| --- | --- |
| 一般公用 | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>&#42;.queue.core.windows.net </li><li>&#42;.servicebus.windows.net - Port: 5671 </li><li>&#42;.table.core.windows.net </li><li>&#42;.adhybridhealth.azure.com/</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *此端點在註冊期間僅用於探索目的。</li> | 
| Azure Germany | <li>&#42;.blob.core.cloudapi.de </li><li>&#42;.queue.core.cloudapi.de </li><li>&#42;.servicebus.cloudapi.de </li><li>&#42;.table.core.cloudapi.de </li><li>&#42;.aadconnecthealth.microsoftazure.de </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>https:\//www.office.de *此端點在註冊期間僅用於探索目的。</li> |
| Azure Government | <li>&#42;.blob.core.usgovcloudapi.net </li><li>&#42;.queue.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.table.core.usgovcloudapi.net </li><li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *此端點在註冊期間僅用於探索目的。</li> |  
 

## <a name="download-and-install-the-azure-ad-connect-health-agent"></a>下載和安裝 Azure AD Connect Health 代理程式
* 確定您符合 Azure AD Connect Health 的[需求](how-to-connect-health-agent-install.md#requirements)。
* 開始使用適用於 AD FS 的 Azure AD Connect Health
    * [下載適用於 AD FS 的 Azure AD Connect Health 代理程式](http://go.microsoft.com/fwlink/?LinkID=518973)
    * [請參閱安裝指示](#installing-the-azure-ad-connect-health-agent-for-ad-fs)。
* 開始使用適用於同步處理的 Azure AD Connect Health
    * [下載並安裝最新版的 Azure AD Connect](http://go.microsoft.com/fwlink/?linkid=615771)。 適用於同步處理的健康狀態代理程式將會隨著 Azure AD Connect 安裝 (1.0.9125.0 或更高版本) 一起安裝。
* 開始使用適用於 AD DS 的 Azure AD Connect Health
    * [下載適用於 AD DS 的 Azure AD Connect Health 代理程式](http://go.microsoft.com/fwlink/?LinkID=820540)。
    * [請參閱安裝指示](#installing-the-azure-ad-connect-health-agent-for-ad-ds)。

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-fs"></a>安裝適用於 AD FS 的 Azure AD Connect Health 代理程式
> [!NOTE]
> AD FS 伺服器應不同於您的同步伺服器。 請勿將 AD FS 代理程式安裝到同步伺服器上。
>
  
安裝之前，請確定您的 AD FS 伺服器主機名稱是唯一的，且不存在於 AD FS 服務中。   
若要啟動代理程式安裝，請按兩下您所下載的 .exe 檔案。 在第一個畫面上，按一下 [安裝]。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/install1.png)

一旦安裝完成之後，按一下 [立即設定]。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/install2.png)

這會啟動 PowerShell 視窗以起始代理程式註冊程序。 出現提示時，使用有權執行代理程式註冊的 Azure AD 帳戶進行登入。 依預設﹐全域系統管理帳戶具有存取權。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/install3.png)

登入後，PowerShell 將會繼續。 一旦完成之後，您就可以關閉 PowerShell，且設定已完成。

此時，應該會自動啟動代理程式服務，讓代理程式以安全的方式將必要資料上傳至雲端服務。

如果您還未符合前幾節所述的所有必要條件，PowerShell 視窗中會出現警告。 請務必在安裝代理程式之前，先完成[需求](how-to-connect-health-agent-install.md#requirements)。 下列螢幕擷取畫面是這些錯誤的其中一個範例。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/install4.png)

若要確認已安裝代理程式，請在伺服器上尋找下列服務。 如果您已完成設定，這些服務應已在執行中。 否則會停止，直到設定完成為止。

* Azure AD Connect Health AD FS 診斷服務
* Azure AD Connect Health AD FS Insights 服務
* Azure AD Connect Health AD FS 監視服務

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/install5.png)

### <a name="agent-installation-on-windows-server-2008-r2-servers"></a>Windows Server 2008 R2 伺服器上的代理程式安裝
適用於 Windows Server 2008 R2 伺服器的步驟：

1. 請確定伺服器在 Service Pack 1 或更高版本上執行。
2. 關閉 IE ESC 以安裝代理程式：
3. 在安裝 AD Health 代理程式之前，先在每部伺服器上安裝 Windows PowerShell 4.0。 安裝 Windows PowerShell 4.0：
   * 使用下列連結下載離線安裝程式，以安裝 [Microsoft.NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=40779) 。
   * 安裝 PowerShell ISE (從 Windows 功能)
   * 安裝 [Windows Management Framework 4.0。](https://www.microsoft.com/download/details.aspx?id=40855)
   * 在伺服器上安裝 Internet Explorer 10 或更新版本。 (Health 服務使用 Azure 系統管理員認證進行驗證時所需)。
4. 如需有關在 Windows Server 2008 R2 上安裝 Windows PowerShell 4.0 的詳細資訊，請參閱[這裡](http://social.technet.microsoft.com/wiki/contents/articles/20623.step-by-step-upgrading-the-powershell-version-4-on-2008-r2.aspx)的 Wiki 文章。

### <a name="enable-auditing-for-ad-fs"></a>啟用 AD FS 的稽核
> [!NOTE]
> 這一節只適用於 AD FS 伺服器。 您不必在 Web 應用程式 Proxy 伺服器上執行下列步驟。
>

若要讓使用情況分析功能收集並分析資料，Azure AD Connect Health 代理程式需要 AD FS 稽核記錄檔中的資訊。 預設不會啟用這些記錄檔。 使用下列程序啟用 AD FS 稽核，並在 AD FS 伺服器上找出 AD FS 稽核記錄檔。

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2008-r2"></a>在 Windows Server 2008 R2 上啟用 AD FS 的稽核
1. 按一下 [開始]，依序指向 [程式集] 和 [系統管理工具]，然後按一下 [本機安全性原則]。
2. 瀏覽至 **Security Settings\Local Policies\User Rights Assignment** 資料夾，然後按兩下 [產生安全性稽核]。
3. 在 [本機安全性設定]  索引標籤上，確認已列出 AD FS 2.0 服務帳戶。 如果不存在，按一下 [新增使用者或群組]，並將其新增至清單中，然後按一下 [確定]。
4. 若要啟用稽核，請使用提高的權限開啟命令提示字元，然後執行下列命令：<code>auditpol.exe /set /subcategory:"Application Generated" /failure:enable /success:enable</code>
5. 關閉 [本機安全性原則]。
<br>   -- **只有主要 AD FS 伺服器才需要執行下列步驟。** -- </br>
6. 開啟 [AD FS 管理] 嵌入式管理單元。 若要開啟 [AD FS 管理] 嵌入式管理單元，按一下 [開始]，依序指向 [程式集] 和 [系統管理工具]，然後按一下 [AD FS 2.0 管理]。
7. 在 [動作] 窗格中，按一下 [編輯同盟服務屬性]。
8. 在 [同盟服務屬性] 對話方塊中，按一下 [事件] 索引標籤。
9. 選取 [成功稽核] 和 [失敗稽核] 核取方塊。
10. 按一下 [確定]。

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>在 Windows Server 2012 R2 上啟用 AD FS 的稽核
1. 在 [開始] 畫面上開啟 [伺服器管理員]，或在桌面上的工作列中開啟 [伺服器管理員]，以開啟 [本機安全性原則]，然後按一下 [工具/本機安全性原則]。
2. 瀏覽至 **Security Settings\Local Policies\User Rights Assignment** 資料夾，然後按兩下 [產生安全性稽核]。
3. 在 [本機安全性設定]  索引標籤上，確認已列出 AD FS 服務帳戶。 如果不存在，按一下 [新增使用者或群組]，並將其新增至清單中，然後按一下 [確定]。
4. 若要啟用稽核，請使用提高的權限開啟命令提示字元，然後執行下列命令：```auditpol.exe /set /subcategory:"Application Generated" /failure:enable /success:enable```。
5. 關閉 [本機安全性原則]。
<br>   -- **只有主要 AD FS 伺服器才需要執行下列步驟。** -- </br>
6. 開啟 [AD FS 管理] 嵌入式管理單元 (在 [伺服器管理員] 中按一下 [工具]，然後選取 [AD FS 管理])。
7. 在 [動作] 窗格中，按一下 [編輯同盟服務屬性]。
8. 在 [同盟服務屬性] 對話方塊中，按一下 [事件] 索引標籤。
9. 選取 [成功稽核] 和 [失敗稽核] 核取方塊，然後按一下 [確定]。

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>在 Windows Server 2016 上啟用 AD FS 的稽核
1. 在 [開始] 畫面上開啟 [伺服器管理員]，或在桌面上的工作列中開啟 [伺服器管理員]，以開啟 [本機安全性原則]，然後按一下 [工具/本機安全性原則]。
2. 瀏覽至 **Security Settings\Local Policies\User Rights Assignment** 資料夾，然後按兩下 [產生安全性稽核]。
3. 在 [本機安全性設定]  索引標籤上，確認已列出 AD FS 服務帳戶。 如果不存在，按一下 [新增使用者或群組]，並將 AD FS 服務帳戶新增至清單，然後按一下 [確定]。
4. 若要啟用稽核，請使用提高的權限開啟命令提示字元，然後執行下列命令：<code>auditpol.exe /set /subcategory:"Application Generated" /failure:enable /success:enable.</code>
5. 關閉 [本機安全性原則]。
<br>   -- **只有主要 AD FS 伺服器才需要執行下列步驟。** -- </br>
6. 開啟 [AD FS 管理] 嵌入式管理單元 (在 [伺服器管理員] 中按一下 [工具]，然後選取 [AD FS 管理])。
7. 在 [動作] 窗格中，按一下 [編輯同盟服務屬性]。
8. 在 [同盟服務屬性] 對話方塊中，按一下 [事件] 索引標籤。
9. 選取 [成功稽核] 和 [失敗稽核] 核取方塊，然後按一下 [確定]。 預設會啟用此功能。
10. 開啟 PowerShell 視窗並執行下列命令：```Set-AdfsProperties -AuditLevel Verbose```。

請注意，預設會啟用「基本」稽核層級。 深入了解 [Windows Server 2016 中的 AD FS 稽核增強功能](https://docs.microsoft.com/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server)


#### <a name="to-locate-the-ad-fs-audit-logs"></a>找出 AD FS 稽核記錄檔
1. 開啟 [事件檢視器] 。
2. 移至 [Windows 記錄]，然後選取 [安全性] 。
3. 按一下右側的 [篩選目前的記錄檔] 。
4. 在 [事件來源] 下，選取 [AD FS 稽核] 。

    以及稽核記錄的快速[常見問題集附註](reference-connect-health-faq.md#operations-questions)。

![AD FS 稽核記錄](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> 群組原則可以停用 AD FS 稽核。 如果已停用 AD FS 稽核，則無法使用有關登入活動的使用量分析。 確定您沒有可停用 AD FS 稽核的群組原則。
>


## <a name="installing-the-azure-ad-connect-health-agent-for-sync"></a>安裝適用於同步處理的 Azure AD Connect Health 代理程式
> [!NOTE]
> 同步伺服器應不同於您的 AD FS 伺服器。 請勿將同步代理程式安裝到 AD FS 伺服器上。
>

適用於同步處理的 Azure AD Connect Health 代理程式會自動安裝在 Azure AD Connect 的最新組建中。 如果要使用適用於同步處理的 Azure AD Connect，您必須下載最新版的 Azure AD Connect 並加以安裝。 您可以在 [此處](http://www.microsoft.com/download/details.aspx?id=47594)下載最新版本。

若要確認已安裝代理程式，請在伺服器上尋找下列服務。 如果您已完成設定，這些服務應已在執行中。 否則會停止，直到設定完成為止。

* Azure AD Connect Health Sync Insights 服務
* Azure AD Connect Health Sync Monitoring 服務

![驗證適用於同步處理的 Azure AD Connect Health](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> 請記住，使用 Azure AD Connect Health 需要 Azure AD Premium。 如果沒有 Azure AD Premium，您就無法完成 Azure 入口網站中的組態。 如需詳細資訊，請參閱[需求頁面](how-to-connect-health-agent-install.md#requirements)。
>
>

## <a name="manual-azure-ad-connect-health-for-sync-registration"></a>手動 Azure AD Connect Health for Sync 註冊
如果 Azure AD Connect Health for Sync 代理程式在成功安裝 Azure AD Connect 後註冊失敗，您可以使用下列 PowerShell 命令來手動註冊代理程式。

> [!IMPORTANT]
> 只有代理程式在安裝 Azure AD Connect 之後註冊失敗時，才需要使用此 PowerShell 命令。
>
>

只有 Health 代理程式註冊失敗 (即使在成功安裝及設定 Azure AD Connect 之後) 時，才需要下列 PowerShell 命令。 Azure AD Connect Health 服務會在代理程式註冊成功後啟動。

您可以使用下列 PowerShell 命令，手動註冊 Azure AD Connect Health for Sync 代理程式︰

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $false -StagingMode $false`

此命令採用下列參數：

* AttributeFiltering：$true (預設值) - 如果 Azure AD Connect 並未同步處理預設屬性集，且已自訂為使用已篩選的屬性集。 否則為 $false。
* StagingMode：$false (預設值) - 如果 Azure AD Connect 伺服器不在預備模式。如果伺服器已設定要在預備模式則為 $true。

當系統提示您進行驗證時，應該使用與用來設定 Azure AD Connect 相同的全域系統管理帳戶 (例如 admin@domain.onmicrosoft.com)。

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-ds"></a>安裝適用於 AD DS 的 Azure AD Connect Health 代理程式
若要啟動代理程式安裝，請按兩下您所下載的 .exe 檔案。 在第一個畫面上，按一下 [安裝]。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

一旦安裝完成之後，按一下 [立即設定]。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

命令提示字元隨即啟動，然後再啟動可執行 PowerShell that executes Register-AzureADConnectHealthADDSAgent 的特定 PowerShell。 當系統提示您登入 Azure 時，繼續進行並登入。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

登入後，PowerShell 將會繼續。 一旦完成之後，您就可以關閉 PowerShell，且設定已完成。

此時，應該會自動啟動服務，以便讓代理程式監視並收集資料。 如果您還未符合前幾節所述的所有必要條件，PowerShell 視窗中會出現警告。 請務必在安裝代理程式之前，先完成[需求](how-to-connect-health-agent-install.md#requirements)。 下列螢幕擷取畫面是這些錯誤的其中一個範例。

![驗證適用於 AD DS 的 Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

若要確認已安裝代理程式，請在網域控制器上尋找下列服務。

* Azure AD Connect Health AD DS Insights 服務
* Azure AD Connect Health AD DS 監視服務

如果您已完成設定，這些服務應已在執行中。 否則會停止，直到設定完成為止。

![驗證 Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)


## <a name="agent-registration-using-powershell"></a>使用 PowerShell 進行代理程式註冊
安裝適當的代理程式 setup.exe 之後﹐您可以根據角色使用下列 PowerShell 命令來執行代理程式註冊步驟。 開啟 PowerShell 視窗並執行適當的命令︰

```
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

這些命令接受「認證」做為參數，進而以非互動方式或在 Server Core 機器上完成註冊。
* 您可以在當作參數傳遞的 PowerShell 變數中擷取認證。
* 您可以提供任何有權註冊代理程式但未啟用 MFA 的 Azure AD 身分識別。
* 預設全域系統管理員有權執行代理程式註冊。 您也可以允許其他權限較小的身分識別執行此步驟。 深入了解[角色型存取控制](how-to-connect-health-operations.md#manage-access-with-role-based-access-control)。

```
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>設定 Azure AD Connect Health 代理程式使用 HTTP Proxy
您可以設定 Azure AD Connect Health 代理程式使用 HTTP Proxy。

> [!NOTE]
> * 不支援使用 "Netsh WinHttp set ProxyServerAddress"，因為代理程式使用 System.Net (而不是 Microsoft Windows HTTP 服務) 提出 Web 要求。
> * 設定的 Http Proxy 地址用來當做傳遞加密 Https 訊息。
> * 不支援已驗證的 Proxy (使用 HTTPBasic)。
>
>

### <a name="change-health-agent-proxy-configuration"></a>變更健康情況代理程式 Proxy 組態
您有下列選項來設定 Azure AD Connect Health 代理程式使用 HTTP Proxy。

> [!NOTE]
> 所有的 Azure AD Connect Health 代理程式服務都必須重新啟動，才會更新 Proxy 設定。 執行以下命令：<br>
> Restart-Service AdHealth*
>
>

#### <a name="import-existing-proxy-settings"></a>匯入現有的 Proxy 設定
##### <a name="import-from-internet-explorer"></a>從 Internet Explorer 匯入
可以匯入 Internet Explorer HTTP Proxy 設定，以供 Azure AD Connect Health 代理程式使用。 在每部執行 Health 代理程式的伺服器上，執行下列 PowerShell 命令︰

    Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings

##### <a name="import-from-winhttp"></a>從 WinHTTP 匯入
可以匯入 WinHTTP Proxy 設定，以供 Azure AD Connect Health 代理程式使用。 在每部執行 Health 代理程式的伺服器上，執行下列 PowerShell 命令︰

    Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp

#### <a name="specify-proxy-addresses-manually"></a>以手動方式指定 Proxy 位址
您可以執行下列 PowerShell 命令，以在每部執行 Health 代理程式的伺服器上手動指定 Proxy 伺服器：

    Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port

範例：*Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443*

* 「位址」可以是可解析的 DNS 伺服器名稱或 IPv4 位址
* 「連接埠」可以省略。 如果省略，則會選擇 443 做為預設連接埠。

#### <a name="clear-existing-proxy-configuration"></a>清除現有的 Proxy 設定
您可以執行下列命令來清除現有的 Proxy 設定：

    Set-AzureAdConnectHealthProxySettings -NoProxy


### <a name="read-current-proxy-settings"></a>讀取目前的 Proxy 設定
您可以執行下列命令來讀取目前設定的 Proxy 設定：

    Get-AzureAdConnectHealthProxySettings


## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>測試對 Azure AD Connect Health 服務的連線
有可能會發生導致 Azure AD Connect Health 代理程式與 Azure AD Connect Health 服務連線中斷的問題。 這些包括網路問題、權限問題或各種其他原因。

如果代理程式無法將資料傳送給 Azure AD Connect Health 服務達 2 小時以上，入口網站中的下列警示會指出：「Health 服務資料不是最新的資料」。 您可以執行下列 PowerShell 命令，以確認受影響的 Azure AD Connect Health 代理程式是否能夠將資料上傳到 Azure AD Connect Health 服務：

    Test-AzureADConnectHealthConnectivity -Role ADFS

角色參數目前可接受下列值：

* ADFS
* Sync
* ADDS

> [!NOTE]
> 若要使用連線工具，您必須先完成代理程式註冊。 如果您無法完成代理程式註冊，請確定您已符合 Azure AD Connect Health 的所有[需求](how-to-connect-health-agent-install.md#requirements)。 這項連線測試預設是在代理程式註冊期間執行。
>
>

## <a name="related-links"></a>相關連結
* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Azure AD Connect Health 操作](how-to-connect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](how-to-connect-health-adfs.md)
* [使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)
* [在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)
* [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](reference-connect-health-version-history.md)
