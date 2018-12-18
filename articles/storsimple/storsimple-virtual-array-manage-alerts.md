---
title: 檢視和管理 Microsoft Azure StorSimple Virtual Array 警示 | Microsoft Docs
description: 描述 StorSimple Virtual Array 警示條件和嚴重性，以及如何使用 StorSimple Manager 服務管理警示。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 97ee25a1-0ec3-4883-9a0a-54b722598462
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/12/2018
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7d4d680e3460fbeff73c2f334c6461da7967374d
ms.sourcegitcommit: 48fce90a4ec357d2fb89183141610789003993d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2018
ms.locfileid: "27786402"
---
# <a name="use-storsimple-device-manager-to-manage-alerts-for-the-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員來管理 StorSimple Virtual Array 的警示

## <a name="overview"></a>概觀

StorSimple 裝置管理員服務中的警示功能可讓您即時檢閱並清除 StorSimple Virtual Array 的相關警示。 您可以使用 [服務摘要] 刀鋒視窗上的警示，集中監視您的 StorSimple Virtual Array 和整個 Microsoft Azure StorSimple 解決方案的健康狀態。

本教學課程說明如何設定警示通知、常見的警示狀況、警示嚴重性層級及如何檢視和追蹤警示。 此外，也包含警示快速參考表，可讓您快速找出特定的警示並適當地回應。

![警示頁面](./media/storsimple-virtual-array-manage-alerts/alerts1.png)

## <a name="configure-alert-settings"></a>設定警示設定

您可以選擇是否要透過電子郵件收到每個 StorSimple Virtual Array 的警示狀況通知。 此外，您可以在 [其他電子郵件收件者] 方塊中輸入電子郵件地址 (以分號隔開)，以識別其他警示通知收件者。

> [!NOTE]
> 您可以對每一個虛擬陣列輸入最多 20 個電子郵件地址。


啟用虛擬陣列的電子郵件通知之後，每當發生重大警示時，通知清單的成員將會收到電子郵件訊息。 訊息將會從 *storsimple-alerts-noreply@mail.windowsazure.com* 送出，並說明警示狀況。 收件者可以按一下 [取消訂閱]，將自己從電子郵件通知清單中移除。

#### <a name="to-enable-email-notification-for-alerts"></a>若要啟用警示的電子郵件通知

1. 移至您的 StorSimple 裝置管理員服務，然後在 [管理] 區段中，選取並按一下 [裝置]。 從顯示的裝置清單中，選取並按一下您的裝置。
   
    ![警示設定](./media/storsimple-virtual-array-manage-alerts/alerts2.png)
2. 這會開啟 [設定] 刀鋒視窗。 在 [裝置設定] 區段中，選取 [一般]。 這會開啟 [一般設定] 刀鋒視窗。
   
    ![警示通知組態](./media/storsimple-virtual-array-manage-alerts/alerts4.png)
3. 在 [一般設定] 刀鋒視窗中，移至 [警示設定] 區段，然後進行下列設定︰
   
   1. 在 [啟用電子郵件通知] 欄位中，選取 [是]。
   2. 如果您想要讓服務管理員和所有共同管理員收到警示通知，請在 [電子郵件服務管理員] 欄位中選取 [是]。
   3. 在 [其他電子郵件收件者] 欄位中，輸入應該收到警示通知的其他所有收件者的電子郵件地址。 以 *someone@somewhere.com* 格式輸入名稱。使用分號來分隔電子郵件地址。 您可以對每一個虛擬裝置設定最多 20 個電子郵件地址。
      
       ![警示通知組態](./media/storsimple-virtual-array-manage-alerts/alerts6.png)
   4. 若要傳送測試電子郵件通知，請按一下 [傳送測試電子郵件]。 StorSimple 裝置管理員服務在轉寄測試通知時會顯示狀態訊息。
      
       ![已傳送警示測試通知電子郵件](./media/storsimple-virtual-array-manage-alerts/alerts7.png)
      
      > [!NOTE]
      > 如果無法傳送測試通知訊息，StorSimple 裝置管理員服務會顯示適當的訊息。 按一下 [ **確定**]，等候幾分鐘，然後嘗試再次傳送測試通知訊息。
      > 
      > 
   5. 按一下頁面底部的 [儲存]  來儲存您的組態。 系統提示您進行確認時，按一下 [是] 。
      
      ![已傳送警示測試通知電子郵件](./media/storsimple-virtual-array-manage-alerts/alerts10.png)

## <a name="common-alert-conditions"></a>常見的警示狀況

StorSimple Virtual Array 會產生警示以回應各種不同的狀況。 以下是最常見的警示狀況類型：

* **連線問題** – 當傳送資料有困難時會發生這些警示。 在往返於 Azure 儲存體帳戶傳輸資料期間，或因為虛擬裝置與 StorSimple 裝置管理員服務之間沒有連線，可能會發生通訊問題。 通訊問題最難解決，因為失敗點實在太多。 在繼續進行更進階的疑難排解之前，您一定要先確認有網路連線和網際網路存取可用。 如需連接埠和防火牆設定的詳細資訊，請參閱 [StorSimple Virtual Array 系統需求](storsimple-ova-system-requirements.md)。 如需疑難排解的說明，請移至 [Test-Connection Cmdlet 的疑難排解](storsimple-troubleshoot-deployment.md)。
* **效能問題** – 當系統未以最佳方式運作時，例如負荷過重，就會造成這些警示。

此外，您也可能會看到有關安全性、更新或作業失敗的警示。

## <a name="alert-severity-levels"></a>警示嚴重性層級

視警示狀況造成的影響和是否需要回應警示而定，警示有不同的嚴重性層級。 嚴重性層級如下：

* **重大** – 這個警示是回應會影響系統成功運作的狀況。 需要採取動作以確保 StorSimple 服務不中斷。
* **警告** – 此狀況如果不解決，可能會演變成重大狀況。 您應該調查這種情況，並採取任何必要的動作來解決問題。
* **資訊** – 此警示包含適用於追蹤和管理系統的資訊。

## <a name="view-and-track-alerts"></a>檢視和追蹤警示

StorSimple 裝置管理員服務的摘要刀鋒視窗中，可讓您快速概覽虛擬裝置上的警示數目 (依嚴重性層級排列)。

![警示儀表板](./media/storsimple-virtual-array-manage-alerts/alerts14.png)

按一下嚴重性層級以開啟 [警示] 刀鋒視窗。 結果只包含符合該嚴重性層級的警示。

![限定警示類型的警示報告](./media/storsimple-virtual-array-manage-alerts/alerts15.png)

按一下清單中的警示來取得警示的其他詳細資料，包括上次報告警示的時間、裝置上發生該警示的次數，以及建議採取來解決警示的動作。

![警示清單和詳細資料](./media/storsimple-virtual-array-manage-alerts/alerts16.png)

如果您需要將資訊傳送給 Microsoft 支援服務，您可以將警示詳細資料複製到文字檔案。 依照建議解決內部部署警示狀況之後，您應該從清單中清除警示。 從清單中選取警示，然後按一下 [清除]。 若要清除多個警示，請選取每個警示，按一下除了 [警示] 資料行以外的任何資料行，然後在選取要清除的所有警示之後按一下 [清除]。

當您按一下 [ **清除**] 時，您有機會提供警示的相關註解，以及您用來解決問題的步驟。 

![警示註解](./media/storsimple-virtual-array-manage-alerts/alerts17.png)

如果新的資訊觸發另一個事件，系統會清除某些事件。 

## <a name="sort-and-review-alerts"></a>排序和檢閱警示

[警示] 刀鋒視窗最多可顯示 250 個警示。 如果超過該警示數目，則並非所有警示都會出現在預設檢視中。 您可以結合下列欄位來自訂要顯示的通知：

* **狀態** – 您可以顯示 [作用中] 或 [已清除] 警示。 作用中警示仍在系統上觸發，而已清除的警示已由系統管理員手動清除，或因為系統以新資訊更新警示狀況而以程式設計方式清除。
* **嚴重性** – 您可以顯示所有嚴重性層級 (重大、警告、資訊) 的警示，或只顯示特定嚴重性的警示，例如只有重大警示。
* **來源** – 您可以顯示所有來源的警示，或只顯示來自服務或其中一個或所有虛擬裝置的警示。
* **時間範圍** – 您可以指定 [從] 和 [到] 日期和時間戳記，以查看您感興趣的期間內的警示。

## <a name="alerts-quick-reference"></a>警示快速參考

下表列出一些您可能會遇到的 StorSimple 警示，以及其他資訊和適用的建議。 StorSimple Virtual Array 警示分為下列類別：

* [雲端連線能力警示](#cloud-connectivity-alerts)
* [組態警示](#configuration-alerts)
* [作業失敗警示](#job-failure-alerts)
* [效能警示](#performance-alerts)
* [安全性警示](#security-alerts)

### <a name="cloud-connectivity-alerts"></a>雲端連線能力警示

| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| 裝置 *<device name>* 未連接至雲端。 |指定的裝置無法連接至雲端。 |無法連接至雲端。 這可能是下列其中一個原因所造成：<ul><li>您的裝置上的網路設定可能有問題。</li><li>儲存體帳戶認證可能有問題。</li></ul>如需有關針對連線問題進行疑難排解的詳細資訊，請移至裝置的[本機 Web UI](storsimple-ova-web-ui-admin.md)。 |

### <a name="configuration-alerts"></a>組態警示

| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| 不支援的內部部署虛擬裝置組態。 |效能變慢。 |目前的組態可能會導致效能降低。 請確定您的伺服器符合最低組態需求。 如需詳細資訊，請參閱 [StorSimple Virtual Array 需求](storsimple-ova-system-requirements.md)。 |
| <裝置名稱> 上的佈建磁碟空間即將用盡。 |磁碟空間警告。 |佈建磁碟空間偏低。 若要釋放空間，請考慮將工作負載移到另一個磁碟區或共用或刪除資料。 |

### <a name="job-failure-alerts"></a>作業失敗警示

| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| 無法完成 <裝置名稱> 的備份。 |備份作業失敗。 |無法建立備份。 請考慮採用下列其中一項︰<ul><li>連線問題可能導致備份作業無法順利完成。 確定沒有連線問題。 如需有關針對連線問題進行疑難排解的詳細資訊，請移至您的虛擬裝置的[本機 Web UI](storsimple-ova-web-ui-admin.md)。</li><li>您已達到可用的儲存體限制。 若要釋放空間，請考慮刪除任何不再需要的備份。</li></ul> 解決問題、清除警示，然後重試作業。 |
| 無法完成 <裝置名稱> 的複製。 |複製作業失敗。 |無法建立複製。 請考慮採用下列其中一項︰<ul><li>您的備份清單可能無效。 重新整理清單，以便確認它仍然有效。</li><li>連線問題可能導致複製作業無法順利完成。 確定沒有連線問題。</li><li>您已達到可用的儲存體限制。 若要釋放空間，請考慮刪除任何不再需要的備份。</li></ul>解決問題、清除警示，然後重試作業。 |

### <a name="networking-alerts"></a>網路警示
| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| 無法連線到驗證服務。 |資料路徑錯誤 |無法連線到用來驗證的 URL。 請確定您的防火牆規則有包含為 StorSimple 裝置指定的 URL 模式。 如需有關 Azure 入口網站之 URL 模式的詳細資訊，請參閱 [StorSimple Virtual Array 網路需求](storsimple-ova-system-requirements.md#url-patterns-for-firewall-rules)。|

### <a name="performance-alerts"></a>效能警示

| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| 您遇到資料傳輸的非預期延遲 |慢速資料傳輸。 |當超出儲存體服務的延展性目標時，會出現節流錯誤。 儲存體服務這麼做是為了確保沒有任何用戶端或是租用戶可犧牲其他服務來使用這項服務。 如需疑難排解 Azure 儲存體帳戶問題的詳細資訊，請參閱[監視、診斷及疑難排解 Microsoft Azure 儲存體的問題](../storage/common/storage-monitoring-diagnosing-troubleshooting.md)。 |
| <裝置名稱> 上的本機保留磁碟空間即將用盡。 |回應時間變慢。 |在本機裝置上為 <裝置名稱> 佈建的總大小中，有 10% 為保留空間，而該保留空間即將用盡。 <裝置名稱> 的工作負載產生較高的變換率，或您最近可能曾經移轉過大量的資料。 這可能會導致效能降低。 請考慮使用下列其中一個方法來解決這個問題：<ul><li>增加此裝置的雲端頻寬。</li><li>降低工作負載或將工作負載移到另一個磁碟區或共用。</li></ul> |

### <a name="security-alerts"></a>安全性警示

| 警示文字 | Event | 詳細資訊 / 建議的動作 |
|:--- |:--- |:--- |
| <裝置名稱> 的密碼即將於 <數字> 天後到期。 |密碼警告。 |您的密碼即將於 <數字> 天後到期。 請考慮變更您的密碼。 如需詳細資訊，請參閱[變更 StorSimple Virtual Array 裝置系統管理員密碼](storsimple-virtual-array-change-device-admin-password.md)。 |

## <a name="next-steps"></a>後續步驟

* [深入了解 StorSimple Virtual Array](storsimple-ova-overview.md)。

