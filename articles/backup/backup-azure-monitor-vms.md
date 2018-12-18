---
title: 監視 Azure 虛擬機器的備份警示
description: 監視來自 Azure 虛擬機器備份作業的事件與警示。 根據警示傳送電子郵件。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 02/23/2018
ms.author: markgal
ms.openlocfilehash: 3783014738ec4e8f185531773b1259dc63e7f49f
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606302"
---
# <a name="monitor-alerts-for-azure-virtual-machine-backups"></a>監視 Azure 虛擬機器備份的警示
警示是來自已達到或超過事件閾值之服務的回應。 了解何時出現問題，可能對於維持低商務成本很重要。 警示通常不會依照排程發生，因此在警示發生之後盡早得知將會有所幫助。 例如，當備份或還原作業失敗時，警示會在失敗後五分鐘內發生。 在保存庫儀表板中，[備份警示] 圖格會顯示嚴重和警告層級的事件。 在 [備份警示] 設定中，您可以檢視所有事件。 但是，如果在您處理不同問題時發生警示，您該怎麼辦？ 如果您不知道何時發生警示，可能會有點不便，或可能危及資料。 若要確定正確的人員會留意警示 (當它發生時)，請設定服務以透過電子郵件傳送警示通知。 如需設定電子郵件通知的詳細資訊，請參閱 [設定通知](backup-azure-monitor-vms.md#configure-notifications)。

## <a name="how-do-i-find-information-about-the-alerts"></a>如何找到警示的相關資訊？
若要檢視擲回警示的事件相關資訊，您必須開啟 [備份警示] 區段。 開啟 [備份警示] 區段的方法有兩種︰從保存庫儀表板中的 [備份警示] 圖格，或從 [警示和事件] 區段。

若要從 [備份警示] 圖格開啟 [備份警示] 刀鋒視窗︰

* 在保存庫儀表板的 [備份警示] 圖格上，按一下 [嚴重] 或 [警告] 以檢視該嚴重性層級的作業事件。

    ![備份警示圖格](./media/backup-azure-monitor-vms/backup-alerts-tile.png)

從 [警示和事件] 區段開啟 [備份警示] 刀鋒視窗：

1. 在保存庫儀表板中，按一下 [所有設定] 。 ![所有設定按鈕](./media/backup-azure-monitor-vms/all-settings-button.png)
2. 在 [設定] 刀鋒視窗上，按一下 [警示和事件]。 ![警示和事件按鈕](./media/backup-azure-monitor-vms/alerts-and-events-button.png)
3. 在 [警示與事件] 刀鋒視窗上，按一下 [備份警示]。 ![備份警示按鈕](./media/backup-azure-monitor-vms/backup-alerts.png)

    [備份警示] 區段會開啟並顯示篩選後的警示。

    ![備份警示圖格](./media/backup-azure-monitor-vms/backup-alerts-critical.png)
4. 若要檢視特定警示的詳細資訊，請從事件清單中按一下警示，以開啟它的 [詳細資料] 區段。

    ![事件詳細資料](./media/backup-azure-monitor-vms/audit-logs-event-detail.png)

    若要自訂清單中顯示的屬性，請參閱 [檢視其他事件屬性](backup-azure-monitor-vms.md#view-additional-event-attributes)

## <a name="configure-notifications"></a>設定通知
 您可以設定服務以針對過去一小時發生的警示，或在特定類型的事件發生時，傳送電子郵件通知。

設定警示的電子郵件通知

1. 在 [備份警示] 功能表上，按一下 [設定通知] 

    ![備份警示功能表](./media/backup-azure-monitor-vms/backup-alerts-menu.png)

    [設定通知] 區段隨即開啟。

    ![設定通知刀鋒視窗](./media/backup-azure-monitor-vms/configure-notifications.png)
2. 在 [設定通知] 區段上，針對電子郵件通知按一下 [開啟]。

    [收件者] 和 [嚴重性] 對話方塊旁邊有星號，因為該資訊是必要的。 提供至少一個電子郵件地址，然後選取至少一個嚴重性。
3. 在 [收件者 (電子郵件)]  對話方塊中，輸入接收通知者的電子郵件地址。 使用格式︰username@domainname.com。 用分號分 (;) 分隔多個電子郵件位址。
4. 在 [通知] 區域中，選擇 [每個警示] 以在指定的警示發生時傳送通知，或選擇 [每小時摘要] 以傳送過去一小時的摘要。
5. 在 [嚴重性]  對話方塊中，選擇您要觸發電子郵件通知的一或多個層級。
6. 按一下 [檔案] 。

   ### <a name="what-alert-types-are-available-for-azure-iaas-vm-backup"></a>有哪些警示類型可供 Azure IaaS VM 備份使用？
   | 警示層級 | 傳送的警示 |
   | --- | --- |
   | 重要 | 適用於備份失敗、復原失敗 |
   | 警告 | 適用於成功但有警告的備份作業 (例如：建立快照集時的一些寫入失敗) |
   | 資訊 | 目前沒有適用於 Azure VM 備份的資訊警示 |

### <a name="are-there-situations-where-email-isnt-sent-even-if-notifications-are-configured"></a>會有即使已設定通知卻不寄送電子郵件的情況嗎？
即使已正確設定通知，仍會有不寄送警示的情況。 在下列情況下將不會寄送電子郵件通知，以避免警示雜訊︰

* 如果通知設定為 [每小時摘要]，而且在一小時內引發警示並加以解決，
* 作業便會取消。
* 備份作業會觸發然後失敗，且另一個備份作業正在進行中。
* 啟用 Resource Manager 功能之 VM 的排程備份作業會啟動，但 VM 將不再存在。

## <a name="using-activity-logs-to-get-notifications-for-successful-backups"></a>使用活動記錄來取得成功備份的通知

如果您想要在備份成功之後收到通知，您可以使用建置於保存庫之[活動記錄](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit)上的警示。

### <a name="login-into-azure-portal"></a>登入 Azure 入口網站
登入 Azure 入口網站並移至相關的 Azure 復原服務保存庫，然後按一下屬性中的 [活動記錄] 區段。

### <a name="identify-appropriate-log"></a>識別適當的記錄

套用下圖中顯示的篩選，以確認您是否正在接收成功備份的活動記錄。 據此變更時間範圍以檢視記錄。

![活動記錄檔](./media/backup-azure-monitor-vms/activity-logs-identify.png)

您可以按一下 [JSON] 區段來取得更多詳細資料，並將它複製並貼至文字編輯器加以檢視。 它應該會顯示保存庫詳細資料和觸發活動記錄的項目 (也就是備份項目)。

然後按一下 [新增活動記錄警示] 來產生所有這類記錄的警示。

### <a name="add-activity-log-alert"></a>新增活動記錄警示

按一下 [新增活動記錄警示] 將會為您顯示如下的畫面

![活動記錄警示](./media/backup-azure-monitor-vms/activity-logs-alerts-successful.png)
    
訂用帳戶與資源群組均會用來儲存警示。 準則將會預先填入。 確定所有值都與您的需求相關。

如果是成功備份，就會將 [等級] 標示為 [資訊]，並將 [狀態] 標示為 [成功]。

如果您選取上方的 [資源]，則在記錄到該資源或保存庫的活動記錄時，將會產生警示。 如果您想要將規則套用到所有保存庫，請將 [資源] 保留空白。

### <a name="define-action-on-alert-firing"></a>定義警示引發的動作

使用 [動作群組] 來定義產生警示時的動作。 您可以按一下 [動作類型] 來深入了解可用的動作，例如，電子郵件/SMS/與 ITSM 整合等等。

![活動記錄動作群組](./media/backup-azure-monitor-vms/activity-logs-alerts-action-group.png)


當您按一下 [確定] 之後，系統將會產生活動記錄警示，而針對成功備份所記錄的後續活動記錄將會引發如動作群組中所定義的動作。

### <a name="limitations-on-alerts"></a>警示的限制
以事件為基礎的警示受到下列限制：

1. 在復原服務保存庫中的所有虛擬機器上觸發警示。 您無法針對復原服務保存庫中的部份虛擬機器自訂警示。
2. "alerts-noreply@mail.windowsazure.com" 會傳送警示。 目前您無法修改電子郵件寄件者。

## <a name="next-steps"></a>後續步驟
如需從復原點重新建立虛擬機器的詳細資訊，請參閱 [還原 Azure VM](backup-azure-arm-restore-vms.md)。

如需保護虛擬機器的詳細資訊，請參閱 [搶先目睹︰將 VM 備份至復原服務保存庫](backup-azure-vms-first-look-arm.md)。 

在[管理 Azure 虛擬機器備份](backup-azure-manage-vms.md)一文中，深入了解 VM 備份的管理工作。
