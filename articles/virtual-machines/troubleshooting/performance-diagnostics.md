---
title: Azure 虛擬機器的效能診斷| Microsoft Docs
description: 介紹適用於 Windows 的 Azure 效能診斷。
services: virtual-machines-windows'
documentationcenter: ''
author: anandhms
manager: cshepard
editor: przlplx
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 9/20/2018
ms.author: anandh
ms.openlocfilehash: 67778a269aa962b3184bf55c692154570e8357ba
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387379"
---
# <a name="performance-diagnostics-for-azure-virtual-machines"></a>Azure 虛擬機器的效能診斷

效能診斷工具可協助您針對可能影響 Windows 虛擬機器 (VM) 的效能問題進行疑難排解。 支援的疑難排解案例包括對已知問題和最佳做法的快速檢查，以及牽涉到 VM 效能變慢或是高 CPU、磁碟空間或記憶體使用量的複雜問題。 

您可以直接從 Azure 入口網站執行效能診斷，並同時在此檢閱見解以及有關各種記錄、豐富設定和診斷資料的報告。 建議您先執行效能診斷並檢閱見解和診斷資料，再連絡 Microsoft 支援人員。

> [!NOTE]
> 目前只有安裝 .NET SDK 4.5 版或更新版本的 Windows VM 才支援效能診斷。 如需在傳統 VM 上執行效能診斷的步驟，請參閱 [Azure 效能診斷 VM 延伸模組](performance-diagnostics-vm-extension.md)。

### <a name="supported-operating-systems"></a>支援的作業系統
Windows 10、Windows 8、Windows 8 Enterprise、Windows 8 Pro、Windows 8.1、Windows Server 2016、Windows Server 2012、Windows Server 2012 Datacenter、Windows Server 2012 R2、Windows Server 2012 R2 Datacenter、Windows Server 2012 R2 Standard、Windows Server 2012 Standard、Windows Server 2008 R2、Windows Server 2008 R2 Datacenter、Windows Server 2008 R2 Enterprise、Windows Server 2008 R2 Foundation、Windows Server 2008 R2 SP1、Windows Server 2008 R2 Standard。

## <a name="install-and-run-performance-diagnostics-on-your-vm"></a>在您的 VM 上安裝並執行效能診斷
效能診斷會安裝 VM 延伸模組，以執行名為 [PerfInsights](https://aka.ms/perfinsights) 的診斷工具。 若要安裝並執行效能診斷，請遵循下列步驟進行：
1.  在命令的左欄中，選取 [虛擬機器]。
1.  從 VM 名稱清單，選取您想要對其執行診斷的 VM。
1.  在命令的右欄中，選取 [效能診斷]。

    ![Azure 入口網站的螢幕擷取畫面，其中已醒目提示 [安裝效能診斷] 按鈕](media/performance-diagnostics/performance-diagnostics-install.png)

    > [!NOTE]
    > 在此螢幕擷取畫面中，會隱藏 VM 名稱的刀鋒視窗。
1. 選取儲存體帳戶 (選擇性)

    如果您想要使用單一儲存體帳戶來儲存多個 VM 的效能診斷結果，您可以按一下工具列中的 [設定] 按鈕來選取儲存體帳戶。 選取儲存體帳戶之後，按一下 [確定] 按鈕。

    如果您未指定儲存體帳戶，預設會建立新的儲存體帳戶。

    ![[效能診斷] 刀鋒視窗的螢幕擷取畫面，其中已醒目提示 [設定] 工具列按鈕](media/performance-diagnostics/settings-button.png)

    ![從 [效能診斷設定] 刀鋒視窗選取儲存體帳戶的螢幕擷取畫面](media/performance-diagnostics/select-storage-account.png)

1. 選取 [安裝效能診斷] 按鈕。
1. 如果您想要在安裝完成之後執行診斷，請選取 [執行診斷] 核取方塊。 如果您進行這項選取，您將能夠選擇效能分析案例和相關選項。

    ![效能診斷安裝按鈕的螢幕擷取畫面](media/performance-diagnostics/install-diagnostics-button.png)

## <a name="select-an-analysis-scenario-to-run"></a>選取要執行的分析案例

Azure 入口網站提供下列分析案例。 請根據您的效能問題，選取一個分析。 視分析需要，選取持續時間和追蹤選項。

* **快速效能分析**  
    檢查是否有已知問題、分析最佳做法，並收集診斷資料。 此分析需要幾分鐘的時間來執行。 [深入了解](https://aka.ms/perfinsights/quick)

* **效能分析**  
    包括快速效能分析中的所有檢查，並監視高資源耗用量。 使用此版本可針對一般效能問題 (例如高 CPU、記憶體和磁碟使用量) 進行疑難排解。 此分析需要 30 秒到 15 分鐘的時間，視選取的持續時間而定。 [深入了解](https://aka.ms/perfinsights/vmslow) 
    
* **進階效能分析**  
    包括效能分析中的所有檢查，並收集一或多個追蹤，如下列各節所列示。 使用此案例可針對需要額外追蹤的複雜問題進行疑難排解。 根據所選取的 VM 大小和追蹤選項，執行此案例的時間越長，診斷輸出的整體大小會越大。 此分析需要 30 秒到 15 分鐘的時間來執行，視選取的持續時間而定。 [深入了解](https://aka.ms/perfinsights/advanced) 
    
* **Azure 檔案服務分析**  
    包括效能分析中的所有檢查，並擷取網路追蹤和 SMB 計數器。 使用此案例可針對 Azure 檔案服務效能進行疑難排解。 此分析需要 30 秒到 15 分鐘的時間來執行，視選取的持續時間而定。 [深入了解](https://aka.ms/perfinsights/azurefiles)


![[效能診斷] 刀鋒視窗中的 [執行診斷] 窗格螢幕擷取畫面](media/performance-diagnostics/run-diagnostics-pane.png)

### <a name="provide-symptoms-optional"></a>提供徵狀 (選擇性)
從清單選取任何預先選取的徵狀，或新增新的徵狀。 這可協助我們改善未來的分析。 

### <a name="provide-support-request-number-if-available-optional"></a>若有的話，請提供申請支援號碼 (選擇性)
如果您與 Microsoft 支援工程師合作開發現有的支援票證，請提供支援票證號碼。 

### <a name="review-the-privacy-policy-and-legal-terms-and-select-the-check-box-to-acknowledge-required"></a>檢閱隱私權原則和法律條款，然後選取核取方塊進行確認 (必要)
若要執行診斷，您必須同意法律條款並接受隱私權原則。

### <a name="select-ok-to-run-the-diagnostics"></a>選取 [確定] 以執行診斷 
當效能診斷開始安裝時，會顯示一則通知。 安裝完成之後，您會看到通知指出安裝成功。 接著會在指定的期間內執行選取的分析。 這是重現效能問題的適當時機，以便擷取正確時間的診斷資料。 

分析完成之後，會將下列項目上傳至 Azure 資料表，以及指定儲存體帳戶中的二進位大型物件 (BLOB) 容器：

*   關於執行的所有見解及相關資訊
*   包含記錄檔的輸出壓縮 (.zip) 檔 (名為 **PerformanceDiagnostics_yyyy-MM-dd_hh-mm-ss-fff.zip**)
*   HTML 報告

上傳之後，Azure 入口網站中會列出新的診斷報告。

![[效能診斷] 刀鋒視窗中的 [診斷報告] 清單螢幕擷取畫面](media/performance-diagnostics/diagnostics-report-list.png)

## <a name="how-to-change-performance-diagnostics-settings"></a>如何變更效能診斷設定
使用 [設定] 工具列按鈕來變更儲存體帳戶，其中可儲存診斷見解和輸出。 您可以將同一個儲存體帳戶用於多個使用效能診斷的 VM。 當您變更儲存體帳戶時，不會刪除舊的報告和見解。 不過，這些項目不會再顯示於 [診斷報告] 清單中。 

## <a name="review-insights-and-performance-diagnostics-report"></a>檢閱見解和效能診斷報告
每個診斷執行都會包含見解和建議清單、受影響的資源、記錄檔，以及收集到的其他豐富診斷資訊，加上離線檢視的報告。 如需收集到的所有診斷資料完整清單，請參閱 [What kind of information is collected by PerfInsights?](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-use-perfinsights#what-kind-of-information-is-collected-by-perfinsights) (PerfInsights 收集何種資訊？) 

### <a name="select-a-performance-diagnostics-report"></a>選取效能診斷報告
您可以使用 [診斷報告] 清單，尋找所有執行的診斷報告。 此清單包含所使用的分析詳細資料、所發現的見解及其影響層級。 選取一列以檢視更多詳細資料。

![從 [效能診斷] 刀鋒視窗選取診斷報告的螢幕擷取畫面](media/performance-diagnostics/select-report.png)

### <a name="review-a-performance-diagnostics-report"></a>檢閱效能診斷報告
每個效能診斷報告可能包含數個見解，並指出影響層級為 [高]、[中] 或 [低]。 每個見解也會包含建議，以協助減輕問題。 見解會分組以便篩選。 

影響層級代表潛在的效能問題，取決於設定錯誤、已知問題或其他使用者所回報的問題等因素。 您可能不會遇到所列的一或多個問題。 例如，您可能在同一個資料磁碟上有 SQL 記錄檔和資料庫檔案。 如果資料庫使用量很高，此狀況可能會有很高的瓶頸和其他效能問題；如果使用量很低，您可能不會注意到問題。

![效能診斷報告概觀刀鋒視窗的螢幕擷取畫面](media/performance-diagnostics/performance-diagnostics-report-overview.png)

### <a name="reviewing-performance-diagnostics-insights-and-recommendations"></a>檢閱效能診斷見解和建議
您可以選取一個見解，以檢視有關受影響的資源、建議的風險降低和參考連結等更多詳細資料。 

![效能診斷見解詳細資料的螢幕擷取畫面](media/performance-diagnostics/insight-detail.png)

### <a name="download-and-review-the-full-performance-diagnostics-report"></a>下載並檢閱完整的效能診斷報告
您可以使用 [下載報告] 按鈕來下載 HTML 報告，其中包含其他豐富診斷資訊，例如儲存體和網路設定、效能計數器、追蹤、處理序清單和記錄。 其內容取決於選取的分析。 如需進階疑難排解，此報告可能包含有關高 CPU 使用量、高磁碟使用量和過度耗用記憶體之處理序的其他資訊和互動式圖表。 如需效能診斷報告的詳細資訊，請參閱 [Review diagnostics report](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-use-perfinsights#review-the-diagnostics-report) (檢閱診斷報告)。

## <a name="manage-performance-diagnostics-reports"></a>管理效能診斷報告
您可以使用 [刪除報告] 按鈕，刪除一或多個效能診斷報告。

## <a name="how-to-uninstall-performance-diagnostics"></a>如何解除安裝效能診斷
您可以從 VM 解除安裝效能診斷。 此動作會移除 VM 延伸模組，但不會影響儲存體帳戶中的任何診斷資料。 

![[效能診斷] 刀鋒視窗工具列的螢幕擷取畫面，其中已醒目提示 [解除安裝] 按鈕](media/performance-diagnostics/uninstal-button.png)

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="where-is-the-diagnostics-data-from-my-vm-stored"></a>我 VM 中的診斷資料儲存在何處？ 
所有效能診斷見解和報告都會儲存在您自己的儲存體帳戶中。 見解會儲存在 Azure 資料表中。 報告壓縮檔會儲存在名為 azdiagextnresults 的二進位大型物件 (BLOB) 容器中。

您可以使用工具列上的 [設定] 按鈕來檢視儲存體帳戶資訊。 

### <a name="how-do-i-share-this-data-with-microsoft-customer-support"></a>如何與 Microsoft 客戶支援共用這項資料？ 
您可以以多種方式來與 Microsoft 共用診斷報告。

**選項 1：** 自動共用最近的報告  
當您開啟與 Microsoft 的支援票證時，請務必共用效能診斷報告。 如果您選擇在執行診斷時與 Microsoft 共用這項資訊 (藉由選取 [我同意與 Microsoft 共用診斷資訊] 核取方塊)，Microsoft 將能夠在執行日期起 30 天內，使用輸出 ZIP 檔案的 SAS 連結從您的儲存體帳戶存取報告。 只會提供最近的報告給支援工程師。 

**選項 2：** 產生診斷報告壓縮檔的共用存取簽章  
您可以使用共用存取簽章來分享報告壓縮檔的連結。 若要這樣做，請遵循下列步驟： 
1.  在 Azure 入口網站中，瀏覽至儲存診斷資料的儲存體帳戶。
1.  在 [Blob 服務] 區段下，選取 [Blob]。 
1.  選取 **azdiagextnresults** 容器。
1.  選取您要共用的效能診斷輸出壓縮檔。
1.  在 [產生 SAS] 索引標籤上，選取共用準則。 
1.  按一下 [產生 Blob SAS 權杖和 URL]。
1.  複製 **Blob SAS URL** 並與支援工程師共用。 

**選項 3：** 從儲存體帳戶下載報告

您也可以使用選項 2 中的步驟 1-4 來尋找效能診斷報告壓縮檔。 選取以下載檔案，然後透過電子郵件共用，或詢問支援工程師有關上傳檔案的指示。  

### <a name="how-do-i-capture-the-diagnostics-data-at-the-correct-time"></a>如何擷取正確時間的診斷資料？
每個效能診斷執行具有兩個階段： 
1.  安裝或更新效能診斷 VM 延伸模組。
1.  在指定的期間內執行診斷。

目前沒有簡單的方法可確切知道 VM 延伸模組安裝何時完成。 一般而言，這需要 45 秒到 1 分鐘的時間來安裝 VM 延伸模組。 安裝 VM 延伸模組之後，您可以執行重現步驟，讓效能診斷擷取一組正確的資料以供疑難排解。 

## <a name="next-steps"></a>後續步驟
檢閱效能診斷見解和報告之後，如果您仍無法判斷問題的原因並需要更多協助，您可以開啟與 Microsoft 客戶支援的支援票證。 

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
