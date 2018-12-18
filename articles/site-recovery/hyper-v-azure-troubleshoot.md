---
title: 對 Azure Site Recovery 的 Hyper-V 至 Azure 複寫進行疑難排解 | Microsoft Docs
description: 說明如何排解使用 Azure Site Recovery 進行 Hyper-V 至 Azure 的複寫時發生的問題
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 10/10/2018
ms.author: ramamill
ms.openlocfilehash: 027107cca1872ddd504567b74c798fc00e9af214
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49067380"
---
# <a name="troubleshoot-hyper-v-to-azure-replication-and-failover"></a>對 Hyper-V 至 Azure 的複寫和容錯移轉進行疑難排解

本文說明當您使用 [Azure Site Recovery](site-recovery-overview.md) 將內部部署 Hyper-V VM 複寫至 Azure 時可能會遇到的常見問題。

## <a name="enable-protection-issues"></a>啟用保護的問題

如果當您為 Hyper-V VM 啟用保護功能時發生問題，請參閱下列建議：

1. 確認 Hyper-V 主機和 VM 符合所有的[需求和必要條件](hyper-v-azure-support-matrix.md)。
2. 如果 Hyper-V 伺服器位於 System Center Virtual Machine Manager (VMM) 雲端中，請確認您已備妥 [VMM 伺服器](hyper-v-prepare-on-premises-tutorial.md#prepare-vmm-optional)。
3. 確認已在 Hyper-V 主機上執行 Hyper-V 虛擬機器管理服務。
4. 檢查 Hyper-V-VMMS\Admin 登入虛擬機器時出現的問題。 此記錄檔位於 [應用程式及服務記錄檔] > [Microsoft] > [Windows] 中。
5. 在客體 VM 上，確認 WMI 已啟用並可供存取。
  - [了解](https://blogs.technet.microsoft.com/askperf/2007/06/22/basic-wmi-testing/)基本 WMI 測試。
  - 對 WMI 進行[疑難排解](https://aka.ms/WMiTshooting)。
  - 對 WMI 指令碼和服務的問題進行[疑難排解](https://technet.microsoft.com/library/ff406382.aspx#H22)。
6. 在客體 VM 上，確定所執行的是最新版的 Integration Services。
    - [確認](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services)您擁有最新版本。
    - 將 Integration Services [保持](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#keep-integration-services-up-to-date)為最新狀態。
    
## <a name="replication-issues"></a>複寫問題

排解初始和進行中的複寫發生的問題，如下所示：

1. 確定您執行的是[最新版本](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx)的 Site Recovery 服務。
2. 確認複寫是否已暫停：
  - 在 Hyper-V 管理員主控台中查看 VM 健康狀態。
  - 如果處於重大狀態，請以滑鼠右鍵按一下 VM > [複寫] > [檢視複寫健康情況]。
  - 如果複寫已暫停，請按一下 [繼續複寫]。
3. 確認必要服務都在執行中。 若非如此，請加以重新啟動。
    - 如果您要在不使用 VMM 的情況下複寫 Hyper-V，請確認 Hyper-V 主機正在執行下列服務：
        - 虛擬機器管理服務
        - Microsoft Azure 復原服務代理程式服務
        - Microsoft Azure Site Recovery 服務
        - WMI 提供者主機服務
    - 如果您要在環境中使用 VMM 進行複寫，請確認下列服務正在執行中：
        - 在 Hyper-V 主機上，確認虛擬機器管理服務、Microsoft Azure 復原服務代理程式和 WMI 提供者主機服務都在執行中。
        - 在 VMM 伺服器上，確定 System Center Virtual Machine Manager 服務正在執行中。
4. 檢查 Hyper-V 伺服器與 Azure 之間的連線。 若要檢查連線能力，請在 Hyper-V 主機上開啟工作管理員。 在 [效能] 索引標籤上，按一下 [開啟資源監視器]。 在 [網路] 索引標籤 > [網路活動的程序] 上，查看 cbengine.exe 是否正在傳送大量資料 (以 MB 為單位)。
5. 檢查 Hyper-V 主機是否可連線至 Azure 儲存體 Blob URL。 若要檢查主機是否可以連線，請選取並檢查 **cbengine.exe**。 檢視 [TCP 連線]，以驗證從主機到 Azure 儲存體 Blob 的連線。
6. 檢查效能問題，如下所述。
    
### <a name="performance-issues"></a>效能問題

網路頻寬限制可能會影響複寫。 請依照下列方式排解問題：

1. [檢查](https://support.microsoft.com/help/3056159/how-to-manage-on-premises-to-azure-protection-network-bandwidth-usage)您的環境中是否有頻寬或節流的條件約束。
2. 執行[部署規劃分析工具](hyper-v-deployment-planner-run.md)。
3. 執行分析工具後，請依照[頻寬](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input)和[儲存體](hyper-v-deployment-planner-analyze-report.md#vm-storage-placement-recommendation)的建議操作。
4. 檢查[資料變換限制](hyper-v-deployment-planner-analyze-report.md#azure-site-recovery-limits)。 如果您發現某個 VM 的資料變換偏高，請執行下列作業：
  - 檢查您的 VM 是否要標示為要進行重新同步處理。
  - 依照[這些步驟](https://blogs.technet.microsoft.com/virtualization/2014/02/02/hyper-v-replica-debugging-why-are-very-large-log-files-generated/)調查變換的來源。
  - 當 HRL 記錄檔超過可用磁碟空間的 50% 時，即可能發生變換。 如果是這個問題，請為發生此問題的所有 VM 佈建更多儲存空間。
  - 確認複寫並未暫停。 如果已暫停，則會繼續將變更寫入 HRL 檔案，而導致檔案大小增加。
 

## <a name="critical-replication-state-issues"></a>重大複寫狀態問題

1. 若要檢查複寫健康情況，請連線至內部部署 Hyper-V 管理員主控台，選取 VM，然後確認健康情況。

    ![複寫健康情況](media/hyper-v-azure-troubleshoot/replication-health1.png)
    

2. 按一下 [檢視複寫健康情況] 以查看詳細資料：

    - 如果複寫已暫停，請以滑鼠右鍵按一下 VM > [複寫] > [繼續複寫]。
    - 如果在 Site Recovery 中設定的 Hyper-V 主機上的 VM 移轉至相同叢集中的不同 Hyper-V 主機，或移轉至獨立機器，VM 的複寫將不受影響。 請確認新的 Hyper-V 主機符合所有必要條件，且設定於 Site Recovery 中。

## <a name="app-consistent-snapshot-issues"></a>應用程式一致快照集問題

應用程式一致快照集是 VM 內的應用程式資料的時間點快照集。 磁碟區陰影複製服務 (VSS) 可確保在建立快照集時，VM 上的應用程式處於一致狀態。  本節將詳細說明您可能會遇到的一些常見問題。

### <a name="vss-failing-inside-the-vm"></a>VM 內部的 VSS 失敗

1. 確認已安裝並執行最新版的 Integration Services。  在 Hyper-V 主機上透過提升權限的 PowerShell 提示字元執行下列命令，以檢查是否有可用更新：**get-vm  | select Name, State, IntegrationServicesState**。
2. 確認 VSS 服務正在執行，且狀態良好：
    - 若要檢查服務，請登入來賓虛擬機器。 然後，開啟管理員命令提示字元中，並執行下列命令，以檢查是否所有 VSS 寫入器都處於良好的狀態。
        - **Vssadmin list writers**
        - **Vssadmin list shadows**
        - **Vssadmin list providers**
    - 查看輸出。 如果寫入器處於失敗狀態，執行下列作業：
        - 在 VM 的應用程式事件記錄中檢查是否有 VSS 作業錯誤。
    - 嘗試重新啟動下列與失敗的寫入器相關聯的服務：
        - 磁碟區陰影複製
         - Azure Site Recovery VSS 提供者
    - 執行這項操作後，請等待幾個小時，以確認應用程式一致快照集是否已成功產生。
    - 如果仍無法解決問題，請嘗試重新啟動 VM。 這或許可解決服務處於無回應狀態的問題。
3. 確認您的 VM 中沒有動態磁碟。 應用程式一致快照集不支援此做法。 您可以在「磁碟管理」(diskmgmt.msc) 中進行此檢查。

    ![動態磁碟](media/hyper-v-azure-troubleshoot/dynamic-disk.png)
    
4. 確認您沒有連結至 VM 的 iSCSI 磁碟。 不支援此做法。
5. 確認備份服務已啟用。 請在 [Hyper-V 設定] > [Integration Services] 中確認服務已啟用。
6. 確定與建立 VSS 快照集的應用程式之間沒有衝突。 如果有多個應用程式同時嘗試建立 VSS 快照集，則可能發生衝突。 例如，如果「備份」應用程式嘗試在您的複寫原則排定要由 Site Recovery 建立快照集的時間建立 VSS 快照集，就會有衝突。   
7. 檢查 VM 是否經歷偏高的變換率：
    - 您可以在 Hyper-V 主機上使用效能計數器，測量客體 VM 的每日資料變更率。 若要測量資料變更率，請啟用以下計數器。 請在各 VM 磁碟間彙總此值 5-15 分鐘的樣本，以取得 VM 變換量。
        - 類別：「Hyper-V 虛擬儲存裝置」
        - 計數器：「每秒的寫入位元組數」</br>
        - 根據 VM 或其應用程式的忙碌程度，此資料變換率將會增加或維持在高水平。
        - 就 Site Recovery 的標準儲存體而言，平均來源磁碟資料變換量為每秒 2 MB。 [深入了解](hyper-v-deployment-planner-analyze-report.md#azure-site-recovery-limits)
    - 此外，您可以[確認儲存體的延展性目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets#scalability-targets-for-a-storage-account)。
8. 執行[部署規劃工具](hyper-v-deployment-planner-run.md)。
9. 檢閱[網路](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input)和[儲存體](hyper-v-deployment-planner-analyze-report.md#recommendations-with-available-bandwidth-as-input)的相關建議。


### <a name="vss-failing-inside-the-hyper-v-host"></a>Hyper-V 主機內部的 VSS 失敗

1. 檢查事件日誌中是否有 VSS 錯誤和建議：
    - 在 Hyper-V 主機伺服器上，從 [事件檢視器] > [應用程式及服務記錄檔] > [Microsoft] > [Windows] > [Hyper V] > [管理] 開啟 Hyper-V 管理員事件記錄。
    - 確認是否有任何事件指出應用程式一致快照集失敗。
    - 常見的錯誤是：「Hyper-V 無法產生虛擬機器 'XYZ' 的 VSS 快照集：寫入器發生非暫時性錯誤。 如果 VSS 服務沒有回應，重新啟動服務或許可解決問題」。

2. 若要產生 VM 的 VSS 快照集，請確認已在 VM 上安裝 Hyper-V Integration Services，而且已啟用備份 (VSS) Integration Service。
    - 確定 Integration Services VSS 服務/精靈正在客體上執行，並且處於 [正常] 狀態。
    - 您可以在 Hyper-V 主機上透過 **et-VMIntegrationService -VMName<VMName>-Name VSS** 命令從提升權限的 PowerShell 工作階段進行此確認。您也可以藉由登入客體 VM 取得這項資訊。 [深入了解](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services)。
    - 確定 VM 上的備份/VSS Integration Service 正在執行，且處於良好的狀態。 若非如此，請重新啟動這些服務，並結束 Hyper-V 主機伺服器上的 Hyper-V 磁碟區陰影複製要求者服務。

### <a name="common-errors"></a>常見錯誤

**錯誤碼** | **訊息** | **詳細資料**
--- | --- | ---
**0x800700EA** | 「Hyper-V 無法產生虛擬機器的 VSS 快照集：有更多可用的資料。 (0x800700EA)。 如果備份作業正在進行中，VSS 快照集可能無法產生。<br/><br/> 虛擬機器的複寫作業失敗：有更多可用的資料。」 | 檢查您的 VM 是否已啟用動態磁碟。 不支援此做法。
**0x80070032** | 「Hyper-V 磁碟區陰影複製要求者無法連線至虛擬機器 <./VMname>，因為版本不符合 Hyper-V 所預期的版本」 | 檢查是否已安裝最新的 Windows 更新。<br/><br/> [升級](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#keep-integration-services-up-to-date)至最新版的 Integration Services。



## <a name="collect-replication-logs"></a>收集複寫記錄

所有 Hyper-V 複寫事件都記錄在位於 [應用程式及服務記錄檔] > [Microsoft] > [Windows] 的 Hyper-V-VMMS\Admin 記錄中。 此外，您還可以啟用 Hyper-V 虛擬機器管理服務的分析記錄，如下所示：

1. 讓 [分析] 與 [偵錯] 記錄可在 [事件檢視器] 中檢視。 若要讓記錄檔可供使用，請在 [事件檢視器] 中按一下 [檢視] > [顯示分析與偵錯記錄檔]。 [分析] 記錄會出現在 **Hyper-V-VMMS** 下。
2. 在 [動作] 窗格中，按一下 [啟用記錄檔]。 

    ![啟用記錄](media/hyper-v-azure-troubleshoot/enable-log.png)
    
3. 啟用之後，它會在 [效能監視器] 中顯示為 [資料收集器集合工具] 下的 [事件追蹤工作階段]。 
4. 若要檢視收集到的資訊，請停用記錄以停止追蹤工作階段。 儲存記錄，然後在事件檢視器中重新將其開啟，或視需要使用其他工具加以轉換。


### <a name="event-log-locations"></a>事件記錄位置

**事件記錄** | **詳細資料** |
--- | ---
**Applications and Service Logs/Microsoft/VirtualMachineManager/Server/Admin** (VMM 伺服器) | 用來排解 VMM 問題的記錄。
**Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** (Hyper-V 主機) | 用來排解 Microsoft Azure 復原服務代理程式問題的記錄。 
**Applications and Service Logs/Microsoft/Azure Site Recovery/Provider/Operational** (Hyper-V 主機)| 用來排解 Microsoft Azure Site Recovery 服務問題的記錄。
**Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** (Hyper-V 主機) | 用來排解 Hyper-V VM 管理問題的記錄。

### <a name="log-collection-for-advanced-troubleshooting"></a>用於進階疑難排解的記錄集合

下列工具有助於進行進階疑難排解：

-   對於 VMM，請使用[支援診斷平台 (SDP) 工具](http://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx)執行 Site Recovery 記錄收集。
-   對於不具 VMM 的 Hyper-V，請[下載此工具](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab)，並在 Hyper-V 主機上執行，以收集記錄。

