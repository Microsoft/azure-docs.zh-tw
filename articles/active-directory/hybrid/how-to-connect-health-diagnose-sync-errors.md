---
title: Azure AD Connect Health - 診斷重複屬性同步處理錯誤 | Microsoft Docs
description: 本文件說明重複屬性同步處理錯誤的診斷程序，和直接從 Azure 入口網站修正孤立物件案例的可行方式。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: maheshu
editor: billmath
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/11/2018
ms.author: zhiweiw
ms.openlocfilehash: 2d525056f42b35c573943a4db281c8a74605a4c5
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310607"
---
# <a name="diagnose-and-remediate-duplicated-attribute-sync-errors"></a>對重複屬性同步處理錯誤進行診斷和修復

## <a name="overview"></a>概觀
為了更進一步地點出同步處理錯誤，Azure Active Directory (Azure AD) Connect Health 導入了自助服務修復功能。 此功能會對重複屬性同步處理錯誤進行疑難排解，並修正來自 Azure AD 的孤立物件。
診斷功能具有下列功效：
- 可提供診斷程序，來縮小重複屬性同步處理錯誤的範圍。 此外，還可提供特定的修正方法。
- 只要一個步驟，即可對 Azure AD 中特有的案例套用修正方法，以解決錯誤。
- 無須升級或設定即可啟用這項功能。
如需 Azure AD 的詳細資訊，請參閱[身分識別同步處理和重複屬性恢復功能](https://aka.ms/dupattributeresdocs)。

## <a name="problems"></a>問題
### <a name="a-common-scenario"></a>常見案例
發生 **QuarantinedAttributeValueMustBeUnique** 和 **AttributeValueMustBeUnique** 同步處理錯誤時，您常會在 Azure AD 中發現 **UserPrincipalName** 或 **Proxy 位址**衝突。 您可以從內部部署端更新衝突的來源物件，藉以解決同步處理錯誤。 同步處理錯誤會在下一次同步處理後獲得解決。例如，下圖指出兩個使用者的 **UserPrincipalName** 有所衝突。 兩者都是 **Joe.J@contoso.com**。 有衝突的物件會在 Azure AD 中隔離。

![診斷同步處理錯誤的常見案例](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

### <a name="orphaned-object-scenario"></a>孤立物件案例
有時候，您可能會發現現有的使用者失去**來源錨點**。 這表示內部部署 Active Directory 刪除了來源物件。 但刪除信號的變更卻一直未同步處理至 Azure AD。 之所以遺失此變更，成因可能是同步處理引擎有問題，或網域已移轉。 當相同的物件還原或重新建立時，理論上從**來源錨點**進行同步處理的使用者，應該是現有的使用者。 

如果現有使用者是僅限雲端的物件，您也將發現衝突的使用者會同步處理至 Azure AD。 在同步處理時，使用者與現有物件無法相符。 **來源錨點**無法直接重新對應。 深入了解[現有知識庫](https://support.microsoft.com/help/2647098)。 

例如，假設 Azure AD 中的現有物件保有 Joe 的授權。 隨後，有使用不同**來源錨點**進行同步處理的物件以重複的屬性狀態出現在 Azure AD 中。 Joe 在內部部署 Active Directory 中的變更，將無法套用至 Joe 在 Azure AD 中的原始使用者 (現有的使用者)。  

![診斷同步處理錯誤的孤立物件案例](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

## <a name="diagnostic-and-troubleshooting-steps-in-connect-health"></a>Connect Health 中的診斷和疑難排解步驟 
診斷功能支援具有下列重複屬性的使用者物件：

| 屬性名稱 | 同步處理錯誤類型|
| ------------------ | -----------------|
| UserPrincipalName | QuarantinedAttributeValueMustBeUnique 或 AttributeValueMustBeUnique | 
| ProxyAddresses | QuarantinedAttributeValueMustBeUnique 或 AttributeValueMustBeUnique | 
| SipProxyAddress | AttributeValueMustBeUnique | 
| OnPremiseSecurityIdentifier |  AttributeValueMustBeUnique |

>[!IMPORTANT]
> 若要存取這項功能，必須有 RBAC 設定中的**全域管理員**權限或**參與者**權限。
>

在 Azure 入口網站中遵循下列步驟，以縮減同步處理錯誤詳細資料，並提供更明確的解決方案：

![同步處理錯誤的診斷步驟](./media/how-to-connect-health-diagnose-sync-errors/IIdFixSteps.png)

在 Azure 入口網站中，執行若干步驟以找出可修正的特定案例：  
1.  檢查 [診斷狀態] 資料行。 狀態會顯示是否有辦法直接從 Azure Active Directory 修正同步處理錯誤。 換句話說，就是有疑難排解流程可供縮小錯誤案例範圍，從而有可能加以修正。
| 狀態 | 這代表什麼意思？ |
| ------------------ | -----------------|
| 未啟動 | 您並未瀏覽此診斷程序。 根據診斷結果，或許可直接從入口網站修正同步處理錯誤。 |
| 需要手動修正 | 錯誤不符合從入口網站修正的條件。 衝突的物件類型不是使用者，或者，您已執行過診斷步驟，但沒有可從入口網站執行的修正解決方案。 若是後者，從內部部署端修正也是其中一個解決方案。 [深入了解內部部署修正](https://support.microsoft.com/help/2647098)。 | 
| 待同步 | 已套用修正方法。 入口網站正在等候下一個同步處理週期將錯誤清除。 |
  >[!IMPORTANT]
  > 在每個同步處理週期結束後，都會重設診斷狀態資料行。 
  >

2.  選取錯誤詳細資料底下的 [診斷] 按鈕。 您會回答幾個問題，並找出同步處理錯誤的詳細資料。 問題答案可協助找出孤立物件案例。

3.  如果在診斷結束時出現 [關閉] 按鈕，即表示根據您的答案，並沒有可從入口網站執行的快速修正。 請參閱最後一個步驟中顯示的解決方案。 從內部部署修正仍會是解決方案。 選取 [關閉] 按鈕。 目前同步處理錯誤的狀態會切換為 [需要手動修正]。 在目前的同步處理週期內，狀態會保持不變。

4.  發現孤立物件案例後，您即可直接從入口網站修正重複屬性同步處理錯誤。 若要觸發此程序，請選取 [套用修正] 按鈕。 目前同步處理錯誤的狀態會更新為 [待同步]。

5.  錯誤應會在下一個同步處理週期之後從清單中移除。

## <a name="how-to-answer-the-diagnosis-questions"></a>如何回答診斷問題 
### <a name="does-the-user-exist-in-your-on-premises-active-directory"></a>使用者是否存在於您的內部部署 Active Directory 中？

此問題嘗試從內部部署 Active Directory 中找出現有使用者的來源物件。  
1.  檢查 Azure Active Directory 中是否有物件具備提供的 **UserPrincipalName**。 如果沒有，請回答 [否]。
2.  如果有，請檢查物件是否仍在同步處理範圍內。  
  - 使用 DN 在 Azure AD 連接器空間中搜尋。
  - 如果發現該物件的狀態為 [待新增]，請回答 [否]。 Azure AD Connect 無法將物件連線至正確的 Azure AD 物件。
  - 如果找不到物件，請回答 [是]。

在這些範例中，問題會嘗試識別 **Joe Jackson** 是否仍存在於內部部署 Active Directory 中。
在**常見的案例**中，**Joe Johnson** 和 **Joe Jackson** 這兩個使用者會出現在內部部署 Active Directory 中。 隔離的物件是兩個不同的使用者。

![診斷同步處理錯誤的常見案例](./media/how-to-connect-health-diagnose-sync-errors/IIdFixCommonCase.png)

在**孤立物件案例**中，只有一個使用者 (**Joe Johnson**) 會出現在內部部署 Active Directory 中：

![診斷同步處理錯誤孤立物件「使用者是否存在」案例](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)

### <a name="do-both-of-these-accounts-belong-to-the-same-user"></a>這兩個帳戶是否屬於同一個使用者？
此問題會在 Azure AD 中檢查後續發生衝突的使用者與現有的使用者物件，以確認兩者是否屬於相同的使用者。  
1.  衝突的物件是同步處理至 Azure Active Directory 的新物件。 比較物件的屬性：  
  - 顯示名稱
  - 使用者主體名稱
  - 物件識別碼
2.  如果 Azure AD 無法比較這兩者，請檢查 Active Directory 中是否有物件具備提供的 **UserPrincipalNames**。 如果兩者均發現，請回答 [否]。

在下列範例中，兩個物件會屬於相同的使用者 **Joe Johnson**。

![診斷同步處理錯誤孤立物件「相同使用者」案例](./media/how-to-connect-health-diagnose-sync-errors/IIdFixOrphanedCase.png)


## <a name="what-happens-after-the-fix-is-applied-in-the-orphaned-object-scenario"></a>在孤立物件案例中套用修正後的情況
根據上述問題的回答，您會在可從 Azure AD 修正時看到 [套用修正] 按鈕。 在此情況下，內部部署物件將會與非預期的 Azure AD 物件同步處理。 這兩個物件會使用**來源錨點**進行對應。 **套用修正**變更會採取下列步驟或類似步驟：
1. 為 Azure AD 中的正確物件更新**來源錨點**。
2. 刪除 Azure AD 中有衝突的物件 (如果有的話)。

![在修正後診斷同步處理錯誤](./media/how-to-connect-health-diagnose-sync-errors/IIdFixAfterFix.png)

>[!IMPORTANT]
> **套用修正**變更只會套用至孤立物件案例。
>

完成前述步驟後，使用者即可存取連結至現有物件的原始資源。 清單檢視中的 [診斷狀態] 值會更新為 [待同步]。同步處理錯誤會在下一次同步處理後獲得解決。Connect Health 不會再於清單檢視中顯示已解決的同步處理錯誤。

## <a name="failures-and-error-messages"></a>失敗和錯誤訊息
**具衝突屬性的使用者在 Azure Active Directory 中未確實刪除。請先確認使用者已確實刪除後再重試。**  
應先清除 Azure AD 中具有衝突屬性的使用者，才能套用修正程式。 請參閱[如何在 Azure AD 中永久刪除使用者](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-users-restore)，再重試此修正程式。 進入虛刪除狀態 30 天後，也會自動永久刪除使用者。 

**不支援對租用戶中的雲端式使用者更新來源錨點。**  
Azure AD 中的雲端式使用者不應擁有來源錨點。 在此情況下，不支援更新來源錨點。 必須從內部部署環境手動修正。 

## <a name="faq"></a>常見問題集
**問：** 如果**套用修正**執行失敗，會發生什麼狀況？  
**答：** 如果執行失敗，可能表示 Azure AD Connect 正在執行匯出錯誤。 請重新整理入口網站頁面，並在下一個同步處理完成後重試。預設的同步處理週期為 30 分鐘。 


**問：** 如果**現有的物件**是要刪除的物件，將會如何？  
**答：** 如果應刪除**現有的物件**，則不會在此程序中變更**來源錨點**。 您通常可以從內部部署 Active Directory 修正此錯誤。 


**問：** 使用者需要哪些權限才能套用修正？  
**答：** RBAC 設定中的**全域管理員**或**參與者**有權存取診斷和疑難排解程序。


**問：** 我需要針對這項功能設定 Azure AD Connect 或更新 Azure AD Connect Health 代理程式嗎？  
**答：** 不需要，診斷程序是完整的雲端功能。


**問：** 如果現有的物件遭到虛刪除，診斷程序是否會將物件再次還原為使用中的物件？  
**答：** 否，修正並不會更新**來源錨點**以外的物件屬性。
