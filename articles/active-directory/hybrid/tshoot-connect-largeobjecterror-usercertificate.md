---
title: Azure AD Connect - userCertificate 屬性所造成的 LargeObject 錯誤 | Microsoft Docs
description: 本主題針對 userCertificate 屬性所造成的 LargeObject 錯誤提供補救步驟。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 146ad5b3-74d9-4a83-b9e8-0973a19828d9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 0882976df898d36f1d5a5ff06e0de5c747613719
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306067"
---
# <a name="azure-ad-connect-sync-handling-largeobject-errors-caused-by-usercertificate-attribute"></a>Azure AD Connect 同步︰處理 userCertificate 屬性所造成的 LargeObject 錯誤 | Microsoft Docs

Azure AD 會在 **userCertificate** 屬性上強制執行最大限制 **15** 個憑證值。 如果 Azure AD Connect 匯出至 Azure AD 的物件有 15 個以上的值，Azure AD 會傳回 **LargeObject** 錯誤，訊息為︰

>*「佈建的物件太大。請修剪此物件的屬性值數量。下次同步處理循環會重試這項作業...」*

其他 AD 屬性也可能造成 LargeObject 錯誤。 若要確認確實是由 userCertificate 屬性所造成，您需要在內部部署 AD 或 [Synchronization Service Manager Metaverse 搜尋](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-service-manager-ui-mvsearch)中驗證物件。

若要取得租用戶中發生 LargeObject 錯誤的物件清單，請使用下列其中一種方法︰

 * 如果已啟用您的租用戶透過 Azure AD Connect Health 進行同步，您可以參考已提供的[同步處理錯誤報告](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health-sync#object-level-synchronization-error-report-preview)。
 
 * 在每個同步週期結束時會傳送目錄同步處理錯誤的通知電子郵件，其中提供一份發生 LargeObject 錯誤的物件清單。 
 * 如果您按一下最新的「匯出至 Azure AD」作業，[Synchronization Service Manager Operations [作業] 索引標籤](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-service-manager-ui-operations)會顯示發生 LargeObject 錯誤的物件清單。
 
## <a name="mitigation-options"></a>緩和選項
在解決 LargeObject 錯誤之前，同一個物件的其他屬性變更無法匯出至 Azure AD。 若要解決錯誤，您可以考慮下列選項︰

 * 將 Azure AD Connect 升級至 1.1.524.0 組建版本或更新版本。 在 Azure AD Connect 1.1.524.0 組建版本中，現成的同步處理規則已更新成如果屬性有 15 個以上的值，則不匯出屬性 userCertificate 和 userSMIMECertificate。 如需如何升級 Azure AD Connect 的詳細資訊，請參閱 [Azure AD Connect：從舊版升級到最新版本](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-upgrade-previous-version)一文。

 * 在 Azure AD Connect 中實作**輸出同步規則**，以匯出 **Null 值，而不是有 15 個以上憑證值之物件的實際值**。 對於有 15 個值以上的物件，如果您不需要將任何憑證值匯出至 Azure AD，此選項就很適合。 如需有關如何實作此同步規則的詳細資訊，請參閱下一節：[實作同步規則以限制 userCertificate 屬性的匯出](#implementing-sync-rule-to-limit-export-of-usercertificate-attribute)。

 * 在內部部署 AD 物件上，移除您的組織不再使用的值，以減少憑證值數量 (15 個或更少)。 如果由於過期或未使用的憑證而造成屬性膨脹，這是很適合的作法。 您可以使用[這裡提供的 PowerShell 指令碼](https://gallery.technet.microsoft.com/Remove-Expired-Certificates-0517e34f)，協助您在內部部署 AD 中尋找、備份和刪除過期的憑證。 刪除憑證之前，建議您向組織的公開金鑰基礎結構管理員查證。

 * 設定 Azure AD Connect 將 userCertificate 屬性排除，而不要匯出至 Azure AD。 一般而言，我們不建議這個選項，因為 Microsoft Online Services 可能使用此屬性來啟用特定案例。 特別是：
    * Exchange Online 和 Outlook 用戶端會使用 User 物件的 userCertificate 屬性，將郵件簽署和加密。 若要深入了解此功能，請參閱[為郵件簽章和加密的 S/MIME](https://technet.microsoft.com/library/dn626158(v=exchg.150).aspx) 一文。

    * Azure AD 會使用 Computer 物件的 userCertificate 屬性，以允許加入網域的 Windows 10 內部部署裝置連線至 Azure AD。 若要深入了解此功能，請參閱[將已加入網域的裝置連線至 Azure AD 以體驗 Windows 10](https://docs.microsoft.com/azure/active-directory/active-directory-azureadjoin-devices-group-policy) 一文。

## <a name="implementing-sync-rule-to-limit-export-of-usercertificate-attribute"></a>實作同步規則以限制 userCertificate 屬性的匯出
若要解決 userCertificate 屬性所造成的 LargeObject 錯誤，您可以在 Azure AD Connect 中實作輸出同步規則，以匯出 **Null 值，而不是有 15 個以上憑證值之物件的實際值**。 本節說明對 **User** 物件實作同步規則所需的步驟。 這些步驟可以調整為適合 **Contact** 和 **Computer** 物件。

> [!IMPORTANT]
> 匯出 Null 值會移除先前成功匯出至 Azure AD 的憑證值。

步驟可以摘要如下︰

1. 停用同步排程器，並確認沒有任何同步處理在進行中。
3. 尋找 userCertificate 屬性的現有輸出同步規則。
4. 建立所需的輸出同步規則。
5. 在發生 LargeObject 錯誤的現有物件上驗證新的同步規則。
6. 將新的同步規則套用至發生 LargeObject 錯誤的其餘物件。
7. 確認沒有非預期的變更在等候匯出至 Azure AD。
8. 將變更匯出至 Azure AD。
9. 重新啟用同步排程器。

### <a name="step-1-disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>步驟 1. 停用同步排程器，並確認沒有任何同步處理在進行中
請確定您在實作新的同步規則時不會發生同步處理，以避免非預期的變更匯出至 Azure AD。 若要停用內建的同步排程器︰
1. 在 Azure AD Connect 伺服器上啟動 PowerShell 工作階段。

2. 執行 Cmdlet 以停用排定的同步處理︰`Set-ADSyncScheduler -SyncCycleEnabled $false`

> [!Note]
> 上述步驟僅適用於已內建排程器之較新版本 (1.1.xxx.x) 的 Azure AD Connect。 如果您使用較舊版本 (1.0.xxx.x) 的 Azure AD Connect (使用 Windows 工作排程器)，或使用您自己的自訂排程器 (不常見) 觸發定期同步處理，則必須視情況停用它們。

3. 移至 [開始] → [同步處理服務] 來啟動 **Synchronization Service Manager**。

4. 移至 [作業] 索引標籤，確認沒有任何作業是「進行中」狀態。

### <a name="step-2-find-the-existing-outbound-sync-rule-for-usercertificate-attribute"></a>步驟 2. 尋找 userCertificate 屬性的現有輸出同步規則
應該有一個現有的同步規則已啟用，且設定為將 User 物件的 userCertificate 屬性匯出至 Azure AD。 找出此同步規則，查明其**優先順序**和**範圍設定篩選**設定︰

1. 移至 [開始] → [同步處理規則編輯器] 來啟動**同步處理規則編輯器**。

2. 使用下列值來設定搜尋篩選條件：

    | 屬性 | 值 |
    | --- | --- |
    | 方向 |**輸出** |
    | MV 物件類型 |**人員** |
    | 連接器 |Azure AD 連接器的名稱 |
    | 連接器物件類型 |**user** |
    | MV 屬性 |**userCertificate** |

3. 如果您使用 OOB (全新) 同步規則讓 Azure AD 連接器匯出 User 物件的 userCertficiate 屬性，您應該會取回「輸出至 AAD – 使用者 ExchangeOnline」規則。
4. 記下此同步規則的**優先順序**值。
5. 選取同步規則，然後按一下 [編輯]。
6. 在 [確認編輯保留規則] 快顯對話方塊中，按一下 [否]。 (別擔心，我們完全不會變更此同步規則)。
7. 在編輯畫面中，選取 [範圍設定篩選] 索引標籤。
8. 記下的範圍設定篩選組態。 如果您使用 OOB 同步規則，應該只有**一個範圍設定篩選群組，含有兩個子句**，包括︰

    | 屬性 | 運算子 | 值 |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | 使用者 |
    | cloudMastered | NOTEQUAL | True |

### <a name="step-3-create-the-outbound-sync-rule-required"></a>步驟 3. 建立所需的輸出同步規則
新的同步規則必須具有相同的**範圍設定篩選**，以及比現有同步規則**更高的優先順序**。 這可確保新的同步規則會套用至現有同步規則的同一組物件，並覆寫 userCertificate 屬性的現有同步規則。 若要刪除同步規則：
1. 在 [同步處理規則編輯器] 中，按一下 [新增規則] 按鈕。
2. 在 [描述] 索引標籤下，提供下列設定︰

    | 屬性 | 值 | 詳細資料 |
    | --- | --- | --- |
    | 名稱 | 提供名稱 | 例如，「輸出至 AAD – userCertificate 的自訂覆寫」 |
    | 說明 | 提供描述 | 例如，「如果 userCertificate 屬性有 15 個以上的值，匯出 NULL」。 |
    | 連線系統 | 選取 Azure AD 連接器 |
    | 連線系統物件類型 | **user** | |
    | Metaverse 物件類型 | **person** | |
    | 連結類型 | **Join** | |
    | 優先順序 | 選擇 1-99 之間的數字 | 選擇的數字不能被任何現有的同步規則使用，而且比現有同步規則的值更小 (因此，優先順序較高)。 |

3. 移至 [範圍設定篩選] 索引標籤，實作與現有同步規則相同的範圍設定篩選。
4. 略過 [聯結規則] 索引標籤。
5. 移至 [轉換] 索引標籤，使用下列設定新增轉換︰

    | 屬性 | 值 |
    | --- | --- |
    | 流程類型 |**運算式** |
    | 目標屬性 |**userCertificate** |
    | 來源屬性 |「使用下列運算式」：`IIF(IsNullOrEmpty([userCertificate]), NULL, IIF((Count([userCertificate])> 15),AuthoritativeNull,[userCertificate]))` |
    
6. 按一下 [新增] 按鈕建立同步規則。

### <a name="step-4-verify-the-new-sync-rule-on-an-existing-object-with-largeobject-error"></a>步驟 4. 在發生 LargeObject 錯誤的現有物件上驗證新的同步規則
這是為了先在發生 LargeObject 錯誤的現有 AD 物件上，確認同步規則可以正確運作，然後再套用至其他物件︰
1. 移至 Synchronization Service Manager 中的 [作業] 索引標籤。
2. 選取最新的「匯出至 Azure AD」作業，然後按一下其中一個發生 LargeObject 錯誤的物件。
3.  在 [連接器空間物件屬性] 快顯畫面中，按一下 [預覽] 按鈕。
4. 在 [預覽] 快顯畫面中，選取 [完整同步處理]，然後按一下 [認可預覽]。
5. 關閉 [預覽] 畫面和 [連接器空間物件屬性] 畫面。
6. 移至 Synchronization Service Manager 中的 [連接器] 索引標籤。
7. 以滑鼠右鍵按一下 [Azure AD 連接器]，然後選取 [執行...]。
8. 在 [執行連接器] 快顯視窗中，選取 [匯出] 步驟，然後按一下 [確定]。
9. 等候「匯出至 Azure AD」完成，並確認此特定物件不再發生 LargeObject 錯誤。

### <a name="step-5-apply-the-new-sync-rule-to-remaining-objects-with-largeobject-error"></a>步驟 5。 將新的同步規則套用至發生 LargeObject 錯誤的其餘物件
新增同步規則之後，您需要在 AD 連接器上執行完整同步處理步驟︰
1. 移至 Synchronization Service Manager 中的 [連接器] 索引標籤。
2. 以滑鼠右鍵按一下 [AD 連接器]，然後選取 [執行...]。
3. 在 [執行連接器] 快顯視窗中，選取 [完整同步處理] 步驟，然後按一下 [確定]。
4. 等候「完整同步處理」步驟完成。
5. 如果您有多個 AD 連接器，請對剩餘的 AD 連接器重複上述步驟。 通常，如果您有多個內部部署目錄，則需要多個連接器。

### <a name="step-6-verify-there-are-no-unexpected-changes-waiting-to-be-exported-to-azure-ad"></a>步驟 6. 確認沒有非預期的變更在等候匯出至 Azure AD
1. 移至 Synchronization Service Manager 中的 [連接器] 索引標籤。
2. 以滑鼠右鍵按一下 [Azure AD 連接器]，然後選取 [搜尋連接器空間]。
3. 在 [搜尋連接器空間] 快顯視窗中：
    1. 將 [範圍] 設定為 [暫止匯出]。
    2. 將 3 個核取方塊全部勾選，包括 [新增]、[修改] 和 [刪除]。
    3. 按一下 [搜尋] 按鈕，以傳回所有在等候將變更匯出至 Azure AD 的物件。
    4. 請確認沒有任何非預期的變更。 若要檢查給定物件的變更，請按兩下物件。

### <a name="step-7-export-the-changes-to-azure-ad"></a>步驟 7. 將變更匯出至 Azure AD
若要將變更匯出至 Azure AD：
1. 移至 Synchronization Service Manager 中的 [連接器] 索引標籤。
2. 以滑鼠右鍵按一下 [Azure AD 連接器]，然後選取 [執行...]。
4. 在 [執行連接器] 快顯視窗中，選取 [匯出] 步驟，然後按一下 [確定]。
5. 等候「匯出至 Azure AD」完成，並確認不再有 LargeObject 錯誤。

### <a name="step-8-re-enable-sync-scheduler"></a>步驟 8。 重新啟用同步排程器
現在已解決問題，請重新啟用內建的同步排程器︰
1. 啟動 PowerShell 工作階段。
2. 執行 Cmdlet 以重新啟用排定的同步處理︰`Set-ADSyncScheduler -SyncCycleEnabled $true`

> [!Note]
> 上述步驟僅適用於已內建排程器之較新版本 (1.1.xxx.x) 的 Azure AD Connect。 如果您使用較舊版本 (1.0.xxx.x) 的 Azure AD Connect (使用 Windows 工作排程器)，或使用您自己的自訂排程器 (不常見) 觸發定期同步處理，則必須視情況停用它們。

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

