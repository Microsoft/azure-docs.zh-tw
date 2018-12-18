---
title: Azure 資訊安全中心的檔案完整性監視 (預覽) | Microsoft Docs
description: " 了解如何在 Azure 資訊安全中心啟用檔案完整性監視。 "
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/2018
ms.author: terrylan
ms.openlocfilehash: 722a4fd11f35f04ed22d73638f07d15c49ea3c26
ms.sourcegitcommit: e14229bb94d61172046335972cfb1a708c8a97a5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/14/2018
ms.locfileid: "34161831"
---
# <a name="file-integrity-monitoring-in-azure-security-center-preview"></a>Azure 資訊安全中心的檔案完整性監視 (預覽)
了解如何利用此逐步解說，在 Azure 資訊安全中心設定檔案完整性監視 (FIM)。

## <a name="what-is-fim-in-security-center"></a>何謂資訊安全中心的 FIM？
檔案完整性監視 (FIM) 也稱為變更監視，它會檢查作業系統、應用程式軟體等檔案和登錄中是否有可能表示攻擊的變更。 它使用比較方法來判斷檔案的目前狀態是否不同於上次掃描的檔案。 您可以利用這項比較，來判斷對您的檔案所做的修改為有效或可疑。

資訊安全中心的檔案完整性監視會驗證 Windows 檔案、Windows 登錄和 Linux 檔案的完整性。 您可以啟用 FIM 來選取要監視的檔案。 資訊安全中心會監視啟用 FIM 的檔案中是否有類似如下的活動：

- 檔案和登錄建立和移除
- 檔案修改 (檔案大小、存取控制清單和內容雜湊的變更)
- 登錄修改 (大小、存取控制清單、類型和內容的變更)

資訊安全中心會建議要監視的實體，您可以輕鬆地對其啟用 FIM。 您也可以定義自己的 FIM 原則或要監視的實體。 本逐步解說會示範做法。

> [!NOTE]
> 檔案完整性監視 (FIM) 功能適用於 Windows 和 Linux 電腦及 VM，並可在資訊安全中心的標準層使用。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。
FIM 會將資料上傳到 Log Analytics 工作區。 根據您上傳的資料量，需要支付資料費用。 請參閱 [Log Analytics 定價](https://azure.microsoft.com/pricing/details/log-analytics/)以深入了解。
>
>

> [!NOTE]
> FIM 使用 Azure 變更追蹤解決方案來追蹤及識別您環境中的變更。 啟用檔案完整性監視時，您會有解決方案類型的**變更追蹤**資源。 如果您移除**變更追蹤**資源，您會停用資訊安全中心的檔案完整性監視功能。
>
>

## <a name="which-files-should-i-monitor"></a>我應該監視哪些檔案？
選擇要監視的檔案時，您應該考慮對系統和應用程式重要的檔案。 請考慮選擇不會未經計劃就變更的檔案。 選擇應用程式或作業系統經常變更的檔案 (例如記錄檔和文字檔) 會造成許多干擾而難以識別攻擊。

資訊安全中心會根據已知的攻擊模式 (包括檔案和登錄變更)，建議預設應該監視的檔案。

## <a name="using-file-integrity-monitoring"></a>使用檔案完整性監視
1. 開啟 [資訊安全中心] 儀表板。
2. 在左窗格的 [進階雲端防禦] 下，選取 [檔案完整性監視]。
![資訊安全中心儀表板][1]

[檔案完整性監視] 會隨即開啟。
  ![資訊安全中心儀表板][2]

每個工作區會提供下列資訊：

- 過去一週發生的變更總數 (如果工作區上未啟用 FIM，您可能會看到虛線 "-")
- 向工作區報告的電腦和 VM 總數
- 工作區的地理位置
- 工作區依據的 Azure 訂用帳戶

工作區也可能會顯示下列按鈕：

- ![啟用圖示][3] 表示工作區未啟用 FIM。 選取工作區可讓您啟用該工作區下所有電腦的 FIM。
- ![升級計劃圖示][4] 表示工作區或訂用帳戶未在 資訊安全中心的標準層下執行。 若要使用 FIM 功能，您的訂用帳戶必須執行標準層。  選取工作區可讓您升級為標準層。 若要深入了解標準層及如何升級，請參閱[升級為資訊安全中心標準層以增強安全性](security-center-pricing.md)。
- 空白 (沒有任何按鈕) 表示已啟用工作區上的 FIM。

在 [檔案完整性監視] 下，您可以選取工作區來啟用該工作區的 FIM、檢視該工作區的 [檔案完整性監視] 儀表板，或將工作區[升級](security-center-pricing.md)為標準層。

## <a name="enable-fim"></a>啟用 FIM
啟用工作區上的 FIM：

1. 在 [檔案完整性監視] 下，選取具有 [啟用] 按鈕的工作區。
2. [啟用檔案完整性監視] 會隨即開啟並顯示工作區下的 Windows 和 Linux 電腦數目。

   ![啟用檔案完整性監視][5]

   也會列出 Windows 和 Linux 的建議設定。  展開 [Windows 檔案]、[登錄] 和 [Linux 檔案]，以查看完整的建議項目清單。

3. 取消核取您不想要套用 FIM 的任何建議實體。
4. 選取 [Apply file integrity monitoring] \(套用檔案完整性監視\) 以啟用 FIM。

> [!NOTE]
> 您可以隨時變更設定。 請參閱下面的[編輯受監視的實體](security-center-file-integrity-monitoring.md#edit-monitored-items)以深入了解。
>
>

## <a name="view-the-fim-dashboard"></a>檢視 FIM 儀表板
啟用 FIM 的工作區會顯示 [檔案完整性監視] 儀表板。 當您啟用工作區上的 FIM 之後，或在 [檔案完整性監視] 視窗中選取已啟用 FIM 的工作區時，FIM 儀表板會隨即開啟。

![[檔案完整性監視] 儀表板][6]

工作區的 FIM 儀表板會顯示下列資訊：

- 連線到工作區的電腦總數
- 所選時段內發生的變更總數
- 變更類型的細目 (檔案、登錄)
- 變更類別的細目 (已修改、已新增、已移除)

選取儀表板頂端的 [篩選] 可讓您套用要查看變更的時段。

![時段篩選][7]

[電腦] 索引標籤 (如上所示) 會列出向此工作區報告的所有電腦。 針對每部電腦，儀表板會列出：

- 所選時段內發生的變更總計
- 檔案變更或登錄變更時的變更總計細目

當您在搜尋欄位中輸入電腦名稱，或選取 [電腦] 索引標籤下所列的電腦時，[記錄搜尋] 會隨即開啟。[記錄搜尋] 會顯示電腦在所選時段內的所有變更。 您可以展開某項變更，以取得詳細資訊。

![記錄搜尋][8]

[變更] 索引標籤 (如下所示) 會列出工作區在所選時段內的所有變更。 針對已變更的每個實體，儀表板會列出：

- 發生變更的電腦
- 變更的類型 (登錄或檔案)
- 變更的類別 (已修改、已新增、已移除)
- 變更的日期和時間

![工作區的變更][9]

當您在搜尋欄位中輸入變更，或選取 [變更] 索引標籤下所列的實體時，[變更詳細資料] 會隨即開啟。

![變更詳細資料][10]

## <a name="edit-monitored-entities"></a>編輯受監視的實體

1. 返回 [檔案完整性監視] 儀表板，然後選取 [設定]。

  ![設定][11]

  [工作區設定] 會隨即開啟並顯示三個索引標籤：[Windows 登錄]、[Windows 檔案] 和 [Linux 檔案]。 每個索引標籤會列出您可以在該類別中編輯的實體。 針對每個列出的實體，資訊安全中心會識別 FIM 已啟用 (True) 或未啟用 (False)。  編輯實體可讓您啟用或停用 FIM。

  ![工作區設定][12]

2. 選取 identityprotection。 在此範例中，已選取 [Windows 登錄] 下的項目。 [Edit for Change Tracking] \(編輯變更追蹤項目\) 會隨即開啟。

  ![編輯或變更追蹤][13]

在 [Edit for Change Tracking] \(編輯變更追蹤項目\) 下，您可以：

- 啟用 (True) 或停用 (False) 的檔案完整性監視
- 提供或變更實體名稱
- 提供或變更值或路徑
- 刪除實體、捨棄變更或儲存變更

## <a name="add-a-new-entity-to-monitor"></a>新增要監視的實體
1. 返回 [檔案完整性監視] 儀表板，然後選取頂端的 [設定]。 [工作區設定] 會隨即開啟。
2. 在 [工作區設定] 下，選取您要新增之實體類型的索引標籤：[Windows 登錄]、[Windows 檔案] 或 [Linux 檔案]。 在此範例中，已選取 [Linux 檔案]。

  ![新增要監視的項目][14]

3. 選取 [新增] 。 [Add for Change Tracking] \(新增變更追蹤項目\) 會隨即開啟。

  ![輸入所要求的資訊][15]

4. 在 [新增] 頁面上，鍵入所要求的資訊，然後選取 [儲存]。

## <a name="disable-monitored-entities"></a>停用受監視的實體
1. 返回 [檔案完整性監視] 儀表板。
2. 選取目前啟用 FIM 的工作區。 如果缺少 [啟用] 按鈕或 [升級計劃] 按鈕，則工作區已啟用 FIM。

  ![選取啟用 FIM 的工作區][16]

3. 在 [檔案完整性監視] 下，選取 [設定]。

  ![選取 [設定]][17]

4. 在 [工作區設定] 下，選取 [已啟用] 設定為 True 的群組。

  ![工作區設定][18]

5. 在 [Edit for Change Tracking] \(編輯變更追蹤項目\) 視窗下，將 [已啟用] 設定為 False。

  ![將 [已啟用] 設定為 False][19]

6. 選取 [ **儲存**]。

## <a name="disable-fim"></a>停用 FIM
您可以停用 FIM。 FIM 使用 Azure 變更追蹤解決方案來追蹤及識別您環境中的變更。 您可以停用 FIM，從所選工作區移除變更追蹤解決方案。

1. 若要停用 FIM，請返回 [檔案完整性監視] 儀表板。
2. 選取工作區。
3. 在 [檔案完整性監視] 下，選取 [停用]。

  ![停用 FIM][20]

4. 選取 [移除] 以停用。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用資訊安全中心的檔案完整性監視 (FIM)。 如要深入了解資訊安全中心，請參閱下列主題：

* [設定安全性原則](security-center-policies.md) - 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理安全性建議](security-center-recommendations.md) - 了解建議如何協助保護您的 Azure 資源。
* [安全性健康狀態監視](security-center-monitoring.md) - 了解如何監視 Azure 資源的健康清況。
* [管理及回應安全性警訊](security-center-managing-and-responding-alerts.md) - 了解如何管理及回應安全性警訊。
* [監視合作夥伴解決方案](security-center-partner-solutions.md) - 了解如何監視合作夥伴解決方案的健全狀態。
* [資訊安全中心常見問題集](security-center-faq.md) - 尋找有關使用服務的常見問題集。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)-- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-file-integrity-monitoring/security-center-dashboard.png
[2]: ./media/security-center-file-integrity-monitoring/file-integrity-monitoring.png
[3]: ./media/security-center-file-integrity-monitoring/enable.png
[4]: ./media/security-center-file-integrity-monitoring/upgrade-plan.png
[5]: ./media/security-center-file-integrity-monitoring/enable-fim.png
[6]: ./media/security-center-file-integrity-monitoring/fim-dashboard.png
[7]: ./media/security-center-file-integrity-monitoring/filter.png
[8]: ./media/security-center-file-integrity-monitoring/log-search.png
[9]: ./media/security-center-file-integrity-monitoring/changes-tab.png
[10]: ./media/security-center-file-integrity-monitoring/change-details.png
[11]: ./media/security-center-file-integrity-monitoring/fim-dashboard-settings.png
[12]: ./media/security-center-file-integrity-monitoring/workspace-config.png
[13]: ./media/security-center-file-integrity-monitoring/edit.png
[14]: ./media/security-center-file-integrity-monitoring/add.png
[15]: ./media/security-center-file-integrity-monitoring/add-item.png
[16]: ./media/security-center-file-integrity-monitoring/fim-dashboard-disable.png
[17]: ./media/security-center-file-integrity-monitoring/fim-dashboard-settings-disabled.png
[18]: ./media/security-center-file-integrity-monitoring/workspace-config-disable.png
[19]: ./media/security-center-file-integrity-monitoring/edit-disable.png
[20]: ./media/security-center-file-integrity-monitoring/disable-fim.png
