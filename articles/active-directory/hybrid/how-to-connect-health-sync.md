---
title: 使用 Azure AD Connect Health 進行同步處理 | Microsoft Docs
description: 這是 Azure AD Connect Health 頁面，其中討論如何監視 Azure AD Connect 同步處理。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: mtillman
ms.assetid: 1dfbeaba-bda2-4f68-ac89-1dbfaf5b4015
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/18/2017
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8ca3c0524873e4c7a1326095eb1cc28f63f59a27
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47431928"
---
# <a name="monitor-azure-ad-connect-sync-with-azure-ad-connect-health"></a>使用 Azure AD Connect Health 監視 Azure AD Connect 同步處理
下列文件適用於使用 Azure AD Connect Health 來監視 Azure AD Connect (同步處理)。  如需使用 Azure AD Connect Health 來監視 AD FS 的詳細資訊，請參閱 [在 AD FS 使用 Azure AD Connect Health](how-to-connect-health-adfs.md)。 此外，如需使用 Azure AD Connect Health 來監視 Active Directory 網域服務的詳細資訊，請參閱 [在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)。

![適用於同步處理的 Azure AD Connect Health](./media/how-to-connect-health-sync/syncsnapshot.png)

## <a name="alerts-for-azure-ad-connect-health-for-sync"></a>適用於同步處理的 Azure AD Connect Health 警示
＜適用於同步處理的 Azure AD Connect Health 警示＞小節將為您提供作用中警示的清單。 每個警示都包含相關資訊、解決步驟，以及相關文件的連結。 選取作用中或已解決的警示，您將會看到一個包含額外資訊的新刀鋒視窗，以及解決警示可以採取的步驟，和其他文件的連結。 您也可以檢視過去已解決的警示的歷史資料。

選取警示，將會為您提供其他資訊，以及解決警示可以採取的步驟，和其他文件的連結。

![Azure AD Connect 同步處理錯誤](./media/how-to-connect-health-sync/alert.png)

### <a name="limited-evaluation-of-alerts"></a>有限的警示評估
如果 Azure AD Connect 不使用預設組態 (比方說，如果 [屬性篩選] 從預設組態變更為自訂組態)，則 Azure AD Connect Health 代理程式不會上傳 Azure AD Connect 相關的錯誤事件。

這會限制服務的警示評估。 您會在 Azure 入口網站中您的服務之下，看到指出這種情況的橫幅。

![適用於同步處理的 Azure AD Connect Health](./media/how-to-connect-health-sync/banner.png)

您可以按一下 [設定] 並允許 Azure AD Connect Health 代理程式上傳所有的錯誤記錄檔，加以變更。

![適用於同步處理的 Azure AD Connect Health](./media/how-to-connect-health-sync/banner2.png)

## <a name="sync-insight"></a>同步處理深入了解
系統管理員通常想要了解將變更同步至 Azure AD 所花的時間，以及所發生的變更數量。 此功能使用下列圖形，輕鬆地呈現這項資訊：   

* 同步處理作業的延遲
* 物件變更趨勢

### <a name="sync-latency"></a>同步處理延遲
這項功能提供連接器同步處理作業 (匯入、匯出等) 延遲的圖形化趨勢。  這提供快速且輕鬆的方式，讓您了解不僅僅作業的延遲 (如果您會發生大量變更也很好)，還提供一個方式來偵測延遲中可能需要進一步調查的異常行為。

![同步處理延遲](./media/how-to-connect-health-sync/synclatency02.png)

根據預設，只會顯示 Azure AD 連接器「匯出」作業的延遲。  若要查看連接器上的更多作業，或檢視其他連接器的作業，請以滑鼠右鍵按一下圖表，然後選取 [編輯圖表]，或按一下 [編輯延遲圖表] 按鈕，然後選擇特定作業和連接器。

### <a name="sync-object-changes"></a>同步處理物件的變更
這項功能提供正在評估並匯出至 Azure AD 的變更數的圖形化趨勢。  現在，嘗試從同步處理記錄檔收集此資訊並不容易。  圖表不僅可讓您以更簡單的方式監視您的環境中發生的變更數，同時提供正在發生的失敗的視覺化檢視。

![同步處理延遲](./media/how-to-connect-health-sync/syncobjectchanges02.png)

## <a name="object-level-synchronization-error-report-preview"></a>物件層級同步處理錯誤報告 (預覽)
在 Windows Server AD 與 Azure AD 之間使用 Azure AD Connect 同步處理身分識別資料時，針對可能發生的同步處理錯誤，這項功能提供相關的報告。

* 此報告涵蓋同步處理用戶端所記錄的錯誤 (Azure AD Connect 1.1.281.0 版或更高版本)
* 它包含同步處理引擎上執行的最後一個同步處理作業所發生的錯誤 (Azure AD Connector 上的「匯出」)。
* 用於同步處理的 Azure AD Connect Health 代理程式必須有指向所需端點的輸出連線，此報告才會包含最新的資料。
* 此報告**每隔 30 分鐘更新一次**，使用的是用於同步處理的 Azure AD Connect Health 代理程式所上傳的資料。它提供下列重要功能

  * 錯誤分類
  * 依各類別之錯誤列出物件
  * 集中記錄有關錯誤的所有資料
  * 並排比較由於衝突而發生錯誤的物件
  * 將錯誤報告下載為 CVS (即將推出)

### <a name="categorization-of-errors"></a>錯誤分類
此報告將現有的同步處理錯誤分成下列類別︰

| 類別 | 說明 |
| --- | --- |
| 重複的屬性 |當 Azure AD Connect 嘗試在 Azure AD 中以一或多個重複的屬性值建立或更新物件時發生錯誤，這些屬性在租用戶中必須是唯一的，例如 proxyAddresses、UserPrincipalName。 |
| 資料不符 |當大致相符無法比對物件時發生錯誤，導致同步處理錯誤。 |
| 資料驗證失敗 |由於無效資料而導致錯誤，例如重要屬性 (例如 UserPrincipalName) 中有不支援的字元；在寫入 Azure AD 之前未能通過驗證的格式錯誤。 |
| 同盟網域變更 | 當帳戶使用不同同盟網域時發生的錯誤。 |
| 大型屬性 |當一或多個屬性大於允許的大小、長度或計數時發生錯誤。 |
| 其他 |無法歸入上述類別的其他所有錯誤。 根據意見，此類別將會進一步分成子類別。 |

![同步處理錯誤報告摘要](./media/how-to-connect-health-sync/errorreport01.png)
![同步處理錯誤報告類別](./media/how-to-connect-health-sync/SyncErrorByTypes.PNG)

### <a name="list-of-objects-with-error-per-category"></a>依各類別之錯誤列出物件
深入每個類別將可找到所發生的錯誤歸入該類別的物件清單。
![同步處理錯誤報告清單](./media/how-to-connect-health-sync/errorreport03.png)

### <a name="error-details"></a>錯誤詳細資料
每個錯誤的詳細檢視中提供下列資料

* 已醒目提示衝突的屬性
* 所涉及之「AD 物件」的識別項
* 所涉及之「Azure AD 物件」的識別項 (視情況)
* 錯誤描述及如何修正

![同步處理錯誤報告詳細資料](./media/how-to-connect-health-sync/duplicateAttributeSyncError.png)

### <a name="download-the-error-report-as-csv"></a>將錯誤報告下載為 CVS
選取 [匯出] 按鈕，即可下載包含所有錯誤詳細資料的 CSV 檔案。

### <a name="diagnose-and-remediate-sync-errors"></a>對同步錯誤進行診斷和修復 
對於涉及使用者來源錨點更新的特定重複屬性同步錯誤案例，您可以直接從入口網站加以修正。 深入了解[對重複的屬性同步錯誤進行診斷和修復](how-to-connect-health-diagnose-sync-errors.md)

## <a name="related-links"></a>相關連結
* [針對同步處理期間的錯誤進行疑難排解](tshoot-connect-sync-errors.md)
* [重複屬性恢復功能](how-to-connect-syncservice-duplicate-attribute-resiliency.md)
* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Azure AD Connect Health 代理程式安裝](how-to-connect-health-agent-install.md)
* [Azure AD Connect Health 操作](how-to-connect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](how-to-connect-health-adfs.md)
* [在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)
* [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](reference-connect-health-version-history.md)
