---
title: 使用 Azure Log Analytics 監視 Active Directory 複寫狀態 | Microsoft Docs
description: 「Active Directory 複寫狀態」解決方案套件會定期監視您的 Active Directory 環境是否有任何複寫失敗。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 1b988972-8e01-4f83-a7f4-87f62778f91d
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 0ccd457295584f871088bc20864ef103648f1654
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128719"
---
# <a name="monitor-active-directory-replication-status-with-log-analytics"></a>使用 Log Analytics 監視 Active Directory 複寫狀態

![AD 複寫狀態符號](./media/log-analytics-ad-replication-status/ad-replication-status-symbol.png)

Active Directory 是企業 IT 環境的重要元件。 為了確保高可用性和高效能，每個網域控制站有它自己的 Active Directory 資料庫複本。 網域控制站會彼此複寫，以便將變更傳播到整個企業。 此複寫處理序中的失敗可導致整個企業發生各種問題。

「AD 複寫狀態」解決方案套件會定期監視您的 Active Directory 環境是否有任何複寫失敗。

## <a name="installing-and-configuring-the-solution"></a>安裝和設定方案
請使用下列資訊來安裝和設定方案。

* 您必須將代理程式安裝在隸屬於要評估之網域成員的網域控制站上。 或者您必須將代理程式安裝在成員之伺服器上，並設定讓代理程式將 AD 複寫資料傳送至 Log Analytics。 若要了解如何將 Windows 電腦連線到 Log Analytics，請參閱[將 Windows 電腦連線到 Log Analytics](log-analytics-windows-agent.md)。 如果您的網域控制站已屬於您要連線到 Log Analytics 的現有 System Center Operations Manager 環境，請參閱[將 Operations Manager 連線到 Log Analytics](log-analytics-om-agents.md)。
* 使用[從方案庫加入 Log Analytics 方案](log-analytics-add-solutions.md)中所述的程序，將「Active Directory 複寫狀態」解決方案加入您的 Log Analytics 工作區中。  不需要進一步的組態。

## <a name="ad-replication-status-data-collection-details"></a>AD 複寫狀態資料收集詳細資料
下表顯示 AD 複寫狀態的資料收集方法和其他資料收集方式的詳細資料。

| 平台 | 直接代理程式 | SCOM 代理程式 | Azure 儲存體 | SCOM 是否為必要項目？ | 透過管理群組傳送的 SCOM 代理程式資料 | 收集頻率 |
| --- | --- | --- | --- | --- | --- | --- |
| Windows |&#8226; |&#8226; |  |  |&#8226; |每隔五天 |

## <a name="optionally-enable-a-non-domain-controller-to-send-ad-data-to-log-analytics"></a>(選擇性) 啟用非網域控制站以將 AD 資料傳送至 Log Analytics
如果您不要將任何網域控制站直接連線到 Log Analytics，您可以使用網域中任何其他連線到 Log Analytics 的電腦來收集「AD 複寫狀態」解決方案套件的資料並讓它傳送資料。

### <a name="to-enable-a-non-domain-controller-to-send-ad-data-to-log-analytics"></a>讓非網域控制站可以將 AD 資料傳送至 Log Analytics
1. 確認電腦是您要使用 AD 複寫狀態解決方案監視的網域成員。
2. 如果尚未連線，請[將 Windows 電腦連線到 Log Analytics](log-analytics-windows-agent.md)，或[使用現有的 Operations Manager 環境將它連線到 Log Analytics](log-analytics-om-agents.md)。
3. 該該電腦上，設定下列登錄機碼︰

   * 機碼：**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Management Groups\<ManagementGroupName>\Solutions\ADReplication**
   * 值：**IsTarget**
   * 數值資料︰**true**

   > [!NOTE]
   > 在您重新啟動 Microsoft Monitoring Agent service (HealthService.exe) 後，這些變更才會生效。
   >
   >

## <a name="understanding-replication-errors"></a>了解複寫錯誤
將 AD 複寫狀態資料傳送至 Log Analytics 後，您會在 Log Analytics 中看到如下列影像所示的圖格，指出目前有多少複寫錯誤。  
![AD 複寫狀態圖格](./media/log-analytics-ad-replication-status/oms-ad-replication-tile.png)

**重大複寫錯誤** 就是達到或超過 Active Directory 樹系 75% [標記存留期](https://technet.microsoft.com/library/cc784932%28v=ws.10%29.aspx) 的錯誤。

當您按一下圖格時，您可以檢視有關錯誤的詳細資訊。
![AD 複寫狀態儀表板](./media/log-analytics-ad-replication-status/oms-ad-replication-dash.png)

### <a name="destination-server-status-and-source-server-status"></a>目的地伺服器狀態和來源伺服器狀態
這些欄位會顯示發生複寫錯誤的目的地伺服器和來源伺服器的狀態。 每個網域控制站名稱後面的數字表示該網域控制站上的複寫錯誤數目。

目的地伺服器和來源伺服器的錯誤都會顯示，因為從來源伺服器的觀點來看，有些問題比較容易排解，而從目的地伺服器的觀點來看，另一些問題則比較容易排解。

在此範例中，您可以看到許多目的地伺服器的錯誤數目大致相同，但有一部來源伺服器 (ADDC35) 具有比其他伺服器更多的錯誤。 ADDC35 上可能有些問題導致它無法將資料傳送至其複寫夥伴。 修正 ADDC35 上的問題可能會解決目的地伺服器區域中出現的許多錯誤。

### <a name="replication-error-types"></a>複寫錯誤類型
此區域可提供在您的企業中偵測到的錯誤類型相關資訊。 每個錯誤都有唯一的數字代碼和訊息，可協助您判斷錯誤的根本原因。

頂端的圓圈可讓您了解在您的環境中哪些錯誤較常和較不常出現。

會顯示多個網域控制站在何時遇到相同的複寫錯誤。 在此情況下，您可以在一個網域控制站上探索或識別解決方案，然後在受到相同錯誤影響的其他網域控制站上重複執行。

### <a name="tombstone-lifetime"></a>標記存留期
標記存留期決定已刪除的物件 (稱為標記) 會保留在 Active Directory 資料庫中多久。 當已刪除的物件超過標記存留期時，記憶體回收程序會從 Active Directory 資料庫中自動將它移除。

對大多數最新版的 Windows 而言，預設標記存留期為 180 天，但是在較舊的版本上為 60 天，而 Active Directory 系統管理員可予以明確變更。

請務必知道您是否有很接近或超過標記存留期的複寫錯誤。 如果有兩個網域控制站遭遇存留超過標記存留期的複寫錯誤，即使此複寫錯誤已經修正，這兩個網域控制站之間的複寫還是會停用。

標記留存期區域可協助您找出可能停用的位置。 [超過 100% TSL]  類別中的每個錯誤代表至少在樹系的標記存留期內，尚未在其來源與目的地伺服器之間複寫的磁碟分割。

在此情況下，只修正複寫錯誤還不夠。 您最少必須手動調查，先找出並清除停留物件，才可以重新開始複寫。 您甚至可能需要解除委任網域控制站。

除了找出已存留超過標記存留期的複寫錯誤，您也要注意任何落入 **50-75% TSL** 或 **75-100% TSL** 類別的錯誤。

這些是明顯滯留 (非暫時性) 的錯誤，因此可能需要您的介入才能解決。 好消息是它們尚未到達標記存留期。 如果您立即修正這些問題，而在到達標記存留期之前  ，可以最少的手動介入來重新開始複寫。

如先前所述，AD 複寫狀態解決方案的儀表板圖格會顯示您環境中的「重大」  複寫錯誤數目，其定義為超過 75% 標記存留期的錯誤 (包括超過 100% TSL 的錯誤)。 請努力讓此數字保持在 0。

> [!NOTE]
> 所有標記存留期百分比計算都是以 Active Directory 樹系的實際標記存留期為基礎，所以即使您已設定自訂標記存留期值，仍可確信這些百分比是精確的。
>
>

### <a name="ad-replication-status-details"></a>AD 複寫狀態詳細資料
當您按一下其中一份清單中的任何項目時，您會看到有關使用「記錄檔搜尋」的其他詳細資料。 結果會經過篩選，僅顯示該項目的相關錯誤。 例如，如果您按一下列在 [目的地伺服器狀態 (ADDC02)] 之下的第一個網域控制站，您會看到搜尋結果已篩選成顯示將該網域控制站列為目的地伺服器的錯誤︰

![搜尋結果中的 AD 複寫狀態錯誤](./media/log-analytics-ad-replication-status/oms-ad-replication-search-details.png)

從這裡，您可以進一步篩選、修改搜尋查詢等。 如需使用記錄檔搜尋的詳細資訊，請參閱 [記錄檔搜尋](log-analytics-log-searches.md)。

**HelpLink** 欄位會顯示 TechNet 頁面的 URL，其中包含該特定錯誤的其他詳細資訊。 您可以將此連結複製並貼入瀏覽器視窗，以查看疑難排解和修正此錯誤的相關資訊。

您也可以按一下 [匯出]  將結果匯出至 Excel。 匯出資料可以幫助您以任何想要的方式來視覺化複寫錯誤資料。

![Excel 中匯出的 AD 複寫狀態錯誤](./media/log-analytics-ad-replication-status/oms-ad-replication-export.png)

## <a name="ad-replication-status-faq"></a>AD 複寫狀態常見問題集
**問︰多久更新一次 AD 複寫狀態資料？**
答︰此資訊會每隔五天更新一次。

**問：是否有設定此資料更新頻率的方法？**
答：目前沒有。

**問︰我是否必須將所有網域控制站加入至我的 Log Analytics 工作區，才能查看複寫狀態？**
答︰否，只需加入單一網域控制站。 如果您的 Log Analytics 工作區中有多個網域控制站，這些網域控制站的資料都會傳送至 Log Analytics。

**問︰我不想要將任何網域控制站新增至我的 Log Analytics 工作區。仍可使用 AD 複寫狀態解決方案嗎？**
答： 會。 您可以設定要啟用此解決方案的登錄機碼值。 請參閱[讓非網域控制站可以將 AD 資料傳送至 Log Analytics](#to-enable-a-non-domain-controller-to-send-ad-data-to-oms)。

**問：負責收集資料之處理序的名稱為何？**
答︰AdvisorAssessment.exe

**問：收集資料需要花費多少時間？**
答︰資料收集時間取決於 Active Directory 環境的大小，但所需時間通常少於 15 分鐘。

**問：收集的資料類型為何？**
答：複寫資訊是透過 LDAP 收集。

**問：是否有設定資料收集時間的方法？**
答：目前沒有。

**問︰我需要哪些權限才能收集資料？**
答︰Active Directory 的一般使用者權限就足夠了。

## <a name="troubleshoot-data-collection-problems"></a>疑難排解資料收集問題
為了收集資料，「AD 複寫狀態」解決方案套件需要至少有一個網域控制站連線至您的 Log Analytics 工作區。 連接網域控制站後，會彈出訊息顯示**資料仍在收集中**。

如果您需要連接其中一個網域控制站的協助，您可以檢視 [將 Windows 電腦連接到 Log Analytics](log-analytics-windows-agent.md)文件。 或者，如果您的網域控制站已連接到現有的 System Center Operations Manager 環境，您可以檢視 [將 System Center Operations Manage 連接到 Log Analytics](log-analytics-om-agents.md)文件。

如果您不想將任何網域控制站直接連線到 Log Analytics 或連線到 System Center Operations Manager，請參閱[讓非網域控制站可以將 AD 資料傳送至 Log Analytics](#to-enable-a-non-domain-controller-to-send-ad-data-to-oms)。

## <a name="next-steps"></a>後續步驟
* 使用 [Log Analytics 中的記錄檔搜尋](log-analytics-log-searches.md) 可檢視詳細的 Active Directory 複寫狀態資料。
