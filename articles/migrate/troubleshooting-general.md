---
title: 針對 Azure Migrate 問題進行疑難排解 | Microsoft Docs
description: 概括介紹 Azure Migrate 服務的已知問題以及常見錯誤的疑難排解訣竅。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/28/2018
ms.author: raynew
ms.openlocfilehash: 906c6e56b670dfc26b5905a453fd43a3c72086c3
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433492"
---
# <a name="troubleshoot-azure-migrate"></a>針對 Azure Migrate 疑難排解

## <a name="troubleshoot-common-errors"></a>常見問題疑難排解

[Azure Migrate](migrate-overview.md) 會評估要移轉至 Azure 的內部部署工作負載。 此文章可對 Azure Migrate 部署與使用方面的問題進行疑難排解。

### <a name="migration-project-creation-failed-with-error-requests-must-contain-user-identity-headers"></a>移轉專案建立失敗，發生「要求必須包含使用者識別標頭」錯誤。

使用者若無權存取組織的 Azure Active Directory (Azure AD) 租用戶，便會發生這種問題。 當使用者第一次被新增至 Azure AD 租用戶時，他/她會收到歡迎加入租用戶的電子郵件邀請函。 使用者必須打開電子郵件並接受邀請，才能成功加入租用戶。 如果您無法看到電子郵件，請與已有權存取該租用戶的使用者聯繫，並要求他們使用[這裡](https://docs.microsoft.com/azure/active-directory/b2b/add-users-administrator#resend-invitations-to-guest-users)指定的步驟，向您重新發送邀請。

收到電子郵件邀請函後，請打開電子郵件並按一下電子郵件中的連結來接受邀請。 完成之後，您需要登出 Azure 入口網站，然後重新登入，只是重新整理瀏覽器並不會有任何作用。 然後您可以試著建立移轉專案。

### <a name="performance-data-for-disks-and-networks-adapters-shows-as-zeros"></a>磁碟和網路介面卡的效能資料顯示為零

如果 vCenter 伺服器上的統計資料設定層級設為小於 3，可能會發生這種情況。 在層級 3 以上，vCenter 會儲存運算、儲存體和網路的虛擬機器效能歷程記錄。 層級 3 以下的話，vCenter 不會儲存儲存體和網路資料，只會儲存 CPU 和記憶體資料。 在此情況下，Azure Migrate 的效能資料會顯示為零，而且 Azure Migrate 會根據從內部部署機器收集的中繼資料來建議磁碟和網路的規模大小。

若要啟用磁碟和網路效能資料的收集功能，請將統計資料設定層級變更為 3。 然後，等待至少一天以探索並評估您的環境。

### <a name="i-installed-agents-and-used-the-dependency-visualization-to-create-groups-now-post-failover-the-machines-show-install-agent-action-instead-of-view-dependencies"></a>我已經安裝代理程式，並使用相依性視覺化建立群組。 現在，在容錯移轉後，機器會顯示「安裝代理程式」動作，而不是「檢視相依性」。
* 在已規劃或未規劃的容錯移轉後，內部部署機器都會關閉，而且對等的機器會在 Azure 中啟動。 這些機器會取得不同的 MAC 位址。 根據使用者是否選擇保留內部部署 IP 位址，這些機器可能會取得不同的 IP 位址。 如果 MAC 及 IP 位址不同，Azure Migrate 不會使內部部署機器與任何服務對應相依性資料產生關聯，而會要求使用者安裝代理程式，而不是檢視相依性。
* 在測試容錯移轉後，內部部署機器如預期保持開啟。 在 Azure 中啟動的對等機器會取得不同的 MAC 位址，而且可能會取得不同的 IP 位址。 除非使用者封鎖這些機器傳出的 Log Analytics 流量，否則 Azure Migrate 不會使內部部署機器與任何服務對應相依性資料產生關聯，而會要求使用者安裝代理程式，而不是檢視相依性。

### <a name="i-specified-an-azure-geography-while-creating-a-migration-project-how-do-i-find-out-the-exact-azure-region-where-the-discovered-metadata-would-be-stored"></a>我在建立移轉專案時指定了某個 Azure 地理位置，要如何找出將儲存所探索到中繼資料的確切 Azure 區域？

您可以前往專案 [概觀] 頁面中的 [基本資訊] 區段，以識別中繼資料的確切儲存位置。 Azure Migrate 會隨機選取該地理位置內的位置，您無法予以修改。 如果您只想在特定區域中建立專案，則可以使用 REST API 來建立移轉專案並傳遞所需的區域。

   ![專案位置](./media/troubleshooting-general/geography-location.png)

## <a name="collector-errors"></a>收集器錯誤

### <a name="deployment-of-collector-ova-failed"></a>收集器 OVA 的部署失敗

如果您使用 vSphere Web 用戶端來部署 OVA，但 OVA 只下載一部分或者由於瀏覽器的某些因素，就會發生這種錯誤。 請確認下載已完成，並使用不同的瀏覽器來部署 OVA。

### <a name="collector-is-not-able-to-connect-to-the-internet"></a>收集器無法連線到網際網路

當您使用的電腦位於 Proxy 後方時，可能會發生此問題。 如果 Proxy 需要授權認證，請務必提供授權認證。
如果您使用任何 URL 型防火牆 Proxy 控制輸出連線能力，務必將這些必要的 URL 列入允許清單：

**URL** | **用途**  
--- | ---
*.portal.azure.com | 檢查與 Azure 服務的連線能力及驗證時間同步問題時所需。
*.oneget.org | 下載以 Powershell 為基礎的 vCenter PowerCLI 模組時所需。

**因為憑證驗證失敗，所以收集器無法連線至網際網路**

如果您使用攔截 Proxy 來連線到網際網路，而且您未將 Proxy 憑證匯入到收集器 VM，也可能會發生這種情形。 您可以使用[這裡](https://docs.microsoft.com/azure/migrate/concepts-collector#internet-connectivity)詳述的步驟來匯入 Proxy 憑證。

**收集器無法使用我從入口網站複製的專案識別碼和金鑰連線到專案。**

請確定已複製並貼上正確的資訊。 若要進行疑難排解，請安裝 Microsoft Monitoring Agent (MMA) 並確認 MMA 是否可以連線至專案，如下所示：

1. 在收集器虛擬機器上，下載 [MMA](https://go.microsoft.com/fwlink/?LinkId=828603)。
2. 若要開始安裝，請按兩下下載的檔案。
3. 在設定的 [歡迎] 頁面中按 [下一步]。 在 [授權條款] 頁面上，按一下 [我同意] 以接受授權。
4. 在 [目的地資料夾] 中，保留或修改預設的安裝資料夾 > [下一步]。
5. 在 [代理程式安裝選項] 中，選取 [Azure Log Analytics] > [下一步]。
6. 按一下 [新增] 以新增 Log Analytics 工作區。 貼上您複製的專案識別碼和金鑰。 然後按 [下一步] 。
7. 確認代理程式是否可連線至專案。 如果無法連線，請檢查設定。 如果代理程式可以連線，但是收集器無法連線，請連絡支援服務。


### <a name="error-802-date-and-time-synchronization-error"></a>錯誤 802：日期和時間同步處理發生錯誤

伺服器時鐘可能與目前的時間不同步，相差到五分鐘以上。 請變更收集器虛擬機器的時間，使 之與目前的時間相符，方法如下所示：

1. 在虛擬機器上開啟系統管理命令提示字元。
2. 若要檢查時區，請執行 w32tm /tz。
3. 若要同步時間，請執行 w32tm /resync。

### <a name="vmware-powercli-installation-failed"></a>VMware PowerCLI 安裝失敗

Azure Migrate 收集器會下載 PowerCLI，然後將它安裝到設備上。 PowerCLI 安裝失敗可能是因為無法連線到 PowerCLI 存放庫的端點。 若要解決問題，請試著使用下列步驟，在收集器 VM 中手動安裝 PowerCLI：

1. 在系統管理員模式下開啟 Windows PowerShell
2. 移至目錄 C:\ProgramFiles\ProfilerService\VMWare\Scripts\
3. 執行指令碼 InstallPowerCLI.ps1

### <a name="error-unhandledexception-internal-error-occured-systemiofilenotfoundexception"></a>發生 Error UnhandledException Internal 錯誤: System.IO.FileNotFoundException

如果 VMware PowerCLI 安裝發生問題，便可能發生此問題。 請依照下列步驟來解決問題：

1. 如果您不是使用最新版的收集器設備，請[將您的收集器升級至最新版本](https://aka.ms/migrate/col/checkforupdates)，然後檢查此問題是否已解決。
2. 如果您已經有最新的收集器版本，請手動安裝 [VMware PowerCLI 6.5.2](https://www.powershellgallery.com/packages/VMware.PowerCLI/6.5.2.6268016)，然後檢查此問題是否已解決。
3. 如果上述步驟無法解決問題，請瀏覽至 C:\Program Files\ProfilerService 資料夾並移除資料夾中的 VMware.dll 和 VimService65.dll 檔案，然後在 [Windows 服務管理員] 中重新啟動 [Azure Migrate 收集器] 服務 (開啟 [執行] 並輸入 'services.msc' 以開啟 [Windows 服務管理員])。

### <a name="error-unabletoconnecttoserver"></a>Error UnableToConnectToServer

無法連線到 vCenter Server "Servername.com:9443"，因為發生錯誤：沒有任何在 https://Servername.com:9443/sdk 上進行接聽的端點可以接受該訊息。

檢查您是否正在執行最新版的收集器設備，如果不是，請將設備升級至[最新的版本](https://docs.microsoft.com/azure/migrate/concepts-collector#how-to-upgrade-collector)。

如果最新的版本仍然會發生相同的問題，有可能是收集器機器無法解析指定的 vCenter Server 名稱，或指定的連接埠錯誤。 根據預設，如果未指定連接埠，則收集器會試著連線至連接埠編號 443。

1. 試著從收集器機器偵測 Servername.com。
2. 如果步驟 1 失敗，請嘗試透過 IP 位址連線到 vCenter Server。
3. 識別連線至 vCenter 的正確連接埠號碼。
4. 最後，請檢查 vCenter 伺服器是否啟動且正在執行。

## <a name="troubleshoot-dependency-visualization-issues"></a>針對相依性視覺效果問題進行疑難排解

### <a name="i-installed-the-microsoft-monitoring-agent-mma-and-the-dependency-agent-on-my-on-premises-vms-but-the-dependencies-are-now-showing-up-in-the-azure-migrate-portal"></a>我在內部部署 VM 上安裝了 Microsoft Monitoring Agent (MMA) 和相依性代理程式，但相依性未出現在 Azure Migrate 入口網站中。

在您安裝代理程式之後，Azure Migrate 通常需要 15-30 分鐘的時間，才會在入口網站中顯示相依性。 如果您等候的時間超過 30 分鐘，請依照下列步驟進行操作，以確定 MMA 代理程式能夠與 OMS 工作區溝通：

針對 Windows VM：
1. 移至 [控制台]，然後啟動 [Microsoft Monitoring Agent]
2. 在 MMA 屬性快顯視窗中，移至 [Azure Log Analytics (OMS)] 索引標籤
3. 確定工作區的 [狀態] 為綠色。
4. 如果狀態不是綠色，請嘗試移除該工作區，然後再次新增至 MMA。
        ![MMA 狀態](./media/troubleshooting-general/mma-status.png)

針對 Linux VM，請確定 MMA 和相依性代理程式的安裝命令已成功。

### <a name="what-are-the-operating-systems-supported-by-mma"></a>MMA 支援哪些作業系統？

MMA 所支援的 Windows 作業系統清單在[這裡](https://docs.microsoft.com/azure/log-analytics/log-analytics-concept-hybrid#supported-windows-operating-systems)。
MMA 所支援的 Linux 作業系統清單在[這裡](https://docs.microsoft.com/azure/log-analytics/log-analytics-concept-hybrid#supported-linux-operating-systems)。

### <a name="what-are-the-operating-systems-supported-by-dependency-agent"></a>相依性代理程式支援哪些作業系統？

相依性代理程式所支援的 Windows 作業系統清單在[這裡](https://docs.microsoft.com/azure/monitoring/monitoring-service-map-configure#supported-windows-operating-systems)。
相依性代理程式所支援的 Linux 作業系統清單在[這裡](https://docs.microsoft.com/azure/monitoring/monitoring-service-map-configure#supported-linux-operating-systems)。

### <a name="i-am-unable-to-visualize-dependencies-in-azure-migrate-for-more-than-one-hour-duration"></a>在 Azure Migrate 中將相依性視覺化的持續時間是否無法超過一小時？
Azure Migrate 可讓您將相依性視覺化的持續時間最多為一小時。 雖然 Azure Migrate 最多可讓您返回歷程記錄中過去一個月的特定日期，但您可將相依性視覺化的持續時間最多為 1 小時。 例如，您可以使用相依性對應中的持續時間功能來檢視昨天的相依性，但只能檢視它一小時。

### <a name="i-am-unable-to-visualize-dependencies-for-groups-with-more-than-10-vms"></a>如果群組所含的 VM 超過 10 個，是否無法將其相依性視覺化？
針對所含 VM 在 10 個以內的群組，您可以[將群組的相依性視覺化](https://docs.microsoft.com/azure/migrate/how-to-create-group-dependencies)，如果您的群組所含 VM 超過 10 個，則建議您先將群組分割成較小的群組，再將相依性視覺化。

## <a name="troubleshoot-readiness-issues"></a>針對整備問題進行疑難排解

**問題** | 修正
--- | ---
不支援的開機類型 | Azure 不支援具有 EFI 開機類型的 VM。 建議您在執行移轉之前，將開機類型轉換成 BIOS。 <br/><br/>您可以使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/tutorial-migrate-on-premises-to-azure) 進行這類 Vm 的移轉，因為它會在移轉期間將 VM 的開機類型轉換成 BIOS。
有條件地支援 Windows 作業系統 | 作業系統已超過結束支援日期，且需要 [Azure 中的支援](https://aka.ms/WSosstatement)的自訂支援合約 (CSA)，請考慮在移轉至 Azure 之前升級作業系統。
不支援的 Windows 作業系統 | Azure 僅支援[選取的 Windows 作業系統版本](https://aka.ms/WSosstatement)，請考慮在移轉至 Azure 之前升級電腦作業系統。
有條件地背書 Linux 作業系統 | Azure 僅支援[選取的 Linux 作業系統版本](../virtual-machines/linux/endorsed-distros.md)，請考慮在移轉至 Azure 之前升級電腦作業系統。
未背書的 Linux 作業系統 | 電腦可能會在 Azure 中開機，但是 Azure 未提供作業系統支援，請考慮在移轉至 Azure 之前，將作業系統升級至[背書的 Linux 版本](../virtual-machines/linux/endorsed-distros.md)
作業系統不明 | 虛擬機器的作業系統在 vCenter Server 中指定為「其他」，因此，Azure Migrate 無法識別虛擬機器的 Azure 移轉整備程度。 請在移轉電腦之前，先確認 Azure [支援](https://aka.ms/azureoslist)該電腦內執行的作業系統。
不支援的作業系統位元 | 32 位元作業系統的虛擬機器可能會在 Azure 中開機，但建議在移轉至 Azure 之前，將虛擬機器的作業系統從 32 位元升級到 64 位元。
需要 Visual Studio 訂用帳戶。 | 電腦中執行的 Windows 用戶端作業系統，僅在 Visual Studio 訂用帳戶中支援。
找不到所需儲存體效能的虛擬機器。 | 機器需要的儲存體效能 (IOPS/輸送量) 超出 Azure VM 支援。 在移轉之前，降低機器的儲存體需求。
找不到所需網路效能的虛擬機器。 | 機器需要的網路效能 (傳入/傳出) 超出 Azure VM 支援。 減少機器的網路需求。
指定定價層中找不到虛擬機器。 | 如果定價層設定為「標準」，請考慮在移轉至 Azure 之前，先降級虛擬機器。 如果調整大小層為「基本」，請考慮將評估的定價層變更為「標準」。
找不到指定位置的虛擬機器。 | 在移轉之前，使用不同的目標位置。
一個或多個不適合的磁碟。 | 一或多個連結至 VM 的磁碟不符合 Azure 需求。 針對每個連結至 VM 的磁碟，請確保每個磁碟的大小小於 4 TB，如果不是，請先縮減磁碟大小，再移轉至 Azure。 請確保 Azure [受控虛擬機器磁碟](https://docs.microsoft.com/azure/azure-subscription-service-limits#storage-limits)支援每個磁碟所需的效能 (IOPS/輸送量)。   
一個或多個不適用的網路介面卡。 | 在移轉之前，從機器移除未使用的網路介面卡。
磁碟計數超過限制 | 移轉之前從機器移除未使用的磁碟。
磁碟大小超過限制 | Azure 支援大小最大 4 TB 的磁碟。 在移轉前將磁碟壓縮為小於 4 TB。
指定的位置沒有磁碟可用 | 在移轉之前，請確定磁碟已在目標位置。
沒有磁碟可當作指定的備援 | 磁碟應該使用評估設定 (預設為 LRS) 中定義的備援儲存體類型。
由於發生內部錯誤，因此無法判斷磁碟適合性 | 嘗試建立群組的新評估。
找不到具備所需核心和記憶體的虛擬機器 | Azure 找不到適當的虛擬機器類型。 在移轉之前，請減少內部部署電腦的記憶體和核心數目。
由於發生內部錯誤，無法判斷虛擬機器的適合性。 | 嘗試建立群組的新評估。
由於發生內部錯誤，因此無法判斷一個或多個磁碟的適用性。 | 嘗試建立群組的新評估。
由於發生內部錯誤，因此無法判斷一個或多個網路介面卡的適用性。 | 嘗試建立群組的新評估。


## <a name="collect-logs"></a>收集記錄

**如何在收集器 VM 上收集記錄？**

記錄預設為啟用。 記錄的位置如下所示：

- C:\Profiler\ProfilerEngineDB.sqlite
- C:\Profiler\Service.log
- C:\Profiler\WebApp.log

若要收集 Windows 的事件追蹤，請執行下列作業：

1. 在收集器虛擬機器上，開啟 PowerShell 命令視窗。
2. 執行 **Get-EventLog -LogName Application | export-csv eventlog.csv**。

**如何收集入口網站網路流量記錄？**

1. 開啟瀏覽器，然後瀏覽並登入[入口網站](https://portal.azure.com)。
2. 按 F12 開啟 Developer Tools。 如有需要，請清除 [清除瀏覽的項目] 設定。
3. 按一下 [網路] 索引標籤，並開始擷取網路流量：
 - 在 Chrome 中，選取 [保留記錄]。 記錄應該會自動啟動。 紅色圓圈表示正在擷取流量。 如果未出現，請按一下黑色圓圈啟動
 - 在 Edge/IE 中，記錄應該會自動啟動。 如果未啟動，請按一下綠色播放按鈕。
4. 嘗試重現錯誤。
5. 您在記錄時發生錯誤之後，請停止錄製，並儲存一份記錄的活動：
 - 在 Chrome 中，以滑鼠右鍵按一下 [內容另存為 HAR]。 這會壓縮記錄並匯出為 .har 檔案。
 - 在 Edge/IE 中，按一下 [匯出擷取流量] 圖示。 這會壓縮並匯出記錄。
6. 瀏覽至 [主控台] 索引標籤，檢查是否有任何警告或錯誤。 若要儲存主控台記錄：
 - 在 Chrome 中，以滑鼠右鍵按一下主控台記錄的任何位置。 選取 [另存新檔]，以匯出並壓縮記錄。
 - 在 Edge/IE 中，以滑鼠右鍵按一下錯誤，然後選取 [全部複製]。
7. 關閉 Developer Tools。

## <a name="collector-error-codes-and-recommended-actions"></a>收集器錯誤碼和建議的動作

|           |                                |                                                                               |                                                                                                       |                                                                                                                                            |
|-----------|--------------------------------|-------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| 錯誤碼 | 錯誤名稱                      | 訊息                                                                       | 可能的原因                                                                                        | 建議的動作                                                                                                                          |
| 601       | CollectorExpired               | 收集器已過期。                                                        | 收集器已過期。                                                                                    | 請下載新版收集器，然後重試一次。                                                                                      |
| 751       | UnableToConnectToServer        | 因為發生錯誤 %ErrorMessage;，所以無法連線至 vCenter Server '%Name;'     | 請查看錯誤訊息，以取得詳細資料。                                                             | 請解決問題，然後再試一次。                                                                                                           |
| 752       | InvalidvCenterEndpoint         | 伺服器 '%Name;' 並非 vCenter Server。                                  | 請提供 vCenter Server 詳細資料。                                                                       | 請使用正確的 vCenter Server 詳細資料重試作業。                                                                                   |
| 753       | InvalidLoginCredentials        | 因為發生錯誤 %ErrorMessage;，所以無法連線至 vCenter Server '%Name;' | 因為登入認證無效，所以無法連線到 vCenter Server。                             | 確認提供的登入認證正確。                                                                                    |
| 754       | NoPerfDataAvaialable           | 沒有可用的效能資料。                                               | 檢查 vCenter Server 中的統計資料等級。 該等級必須設為 3，才可取得效能資料。 | 將統計資料等級變更為 3 (5 分鐘、30 分鐘及 2 小時的持續時間)，並於等待至少一天後再進行嘗試。                   |
| 756       | NullInstanceUUID               | 發現 InstanceUUID 為 null 的電腦                                  | vCenter Server 可能有不適當的物件。                                                      | 請解決問題，然後再試一次。                                                                                                           |
| 757       | VMNotFound                     | 找不到虛擬機器                                                  | 虛擬機器可能已刪除：%VMID;                                                                | 確定於設定 vCenter 清查範圍時所選取的虛擬機器，在探索期間存在                                      |
| 758       | GetPerfDataTimeout             | VCenter 要求已逾時。訊息 %Message;                                  | vCenter Server 認證不正確                                                              | 檢查 vCenter Server 認證，確認可連線到 vCenter Server。 重試作業。 若問題持續發生，請連絡支援人員。 |
| 759       | VmwareDllNotFound              | 找不到 VMWare.Vim DLL。                                                     | PowerCLI 未正確安裝。                                                                   | 檢查 PowerCLI 是否已安裝正確。 重試作業。 若問題持續發生，請連絡支援人員。                               |
| 800       | ServiceError                   | Azure Migrate 收集器服務未執行。                               | Azure Migrate 收集器服務未執行。                                                       | 使用 services.msc 啟動服務，然後重試一次該作業。                                                                             |
| 801       | PowerCLIError                  | VMware PowerCLI 安裝失敗。                                          | VMware PowerCLI 安裝失敗。                                                                  | 重試作業。 若問題持續發生，請手動安裝並重試一次該作業。                                                   |
| 802       | TimeSyncError                  | 時間未與網際網路時間伺服器同步。                            | 時間未與網際網路時間伺服器同步。                                                    | 確定電腦上的時間已依據電腦的時區校時，然後重試一次該作業。                                 |
| 702       | OMSInvalidProjectKey           | 指定的專案機碼無效。                                                | 指定的專案機碼無效。                                                                        | 請使用正確的專案機碼重試作業。                                                                                              |
| 703       | OMSHttpRequestException        | 傳送要求時發生錯誤。 訊息 %Message;                                | 請檢查專案識別碼和機碼，並確認端點可連線。                                       | 重試作業。 如果問題持續發生，請連絡 Microsoft 支援服務。                                                                     |
| 704       | OMSHttpRequestTimeoutException | HTTP 要求已逾時。訊息 %Message;                                     | 請檢查專案識別碼和機碼，並確認端點可連線。                                       | 重試作業。 如果問題持續發生，請連絡 Microsoft 支援服務。                                                                     |
