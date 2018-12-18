---
title: 使用 Azure AD 存取權檢閱來管理已從條件式存取原則中排除的使用者 | Microsoft Docs
description: 使用 Azure Active Directory (Azure AD) 存取權檢閱來管理已從條件式存取原則中排除的使用者
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: compliance
ms.date: 09/25/2018
ms.author: rolyon
ms.reviewer: mwahl
ms.openlocfilehash: 0ae6c27091dd2938ec7410e78ae12dbe20509029
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167642"
---
# <a name="use-azure-ad-access-reviews-to-manage-users-that-have-been-excluded-from-conditional-access-policies"></a>使用 Azure AD 存取權檢閱來管理已從條件式存取原則中排除的使用者

在理想的世界中，所有使用者都會遵循存取原則來安全地存取您組織的資源。 不過，有時候會有需要您視為例外狀況的商務案例。 本文說明一些可能需要排除項目的範例，以及 IT 系統管理員如何管理這項工作，避免疏忽原則例外狀況，以及向稽核員提供使用 Active Directory (Azure AD) 存取權檢閱定期檢閱這些例外狀況的證明。

> [!NOTE]
> 需要已付費的有效 Azure AD Premium P2、Enterprise Mobility + Security E5 或試用版授權，才能使用 Azure AD 存取權檢閱。 如需詳細資訊，請參閱 [Azure Active Directory 版本](../fundamentals/active-directory-whatis.md)。

## <a name="why-would-you-exclude-users-from-policies"></a>您為何要從原則中排除使用者？

身為 IT 管理員的您可以使用 [Azure AD 條件式存取](../conditional-access/overview.md)，要求使用者使用多重要素驗證 (MFA) 進行驗證或從信任的網路或裝置登入。 在部署規劃期間，您發現這些需求無法由所有使用者滿足。 例如，從不屬於您內部網路的遠端辦公室工作的使用者，或使用不受支援舊式手機的主管。 企業要求這些使用者能夠登入並從事其工作，因此，他們會從條件式存取原則中排除。

另舉一例，您可以在條件式存取中使用[具名位置](../conditional-access/location-condition.md)來設定一組您不想允許使用者存取其租用戶的國家和區域。

![具名位置](./media/conditional-access-exclusion/named-locations.png)

不過，在某些情況下，使用者從這些已封鎖國家/地區登入可能會有合理的原因。 例如，使用者可能因為工作或個人原因而旅行。 在此範例中，用來封鎖這些國家/地區的條件式存取原則，可能有專供從原則中排除的使用者使用的雲端安全性群組。 在旅行時需要存取的使用者，可以使用 [Azure AD 自助式群組管理](../users-groups-roles/groups-self-service-management.md)將本身新增到群組。

另一個範例可能是您有條件式存取原則，可[封鎖絕大多數使用者的舊版驗證](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/07/azure-ad-conditional-access-support-for-blocking-legacy-auth-is-in-public-preview/)。 Microsoft 強烈建議您在租用戶中封鎖使用舊版通訊協定，以改善安全性狀態。 不過，如果有些使用者一定需要使用舊版驗證方法才能透過 Office 2010 或 IMAP/SMTP/POP 型用戶端存取您的資源，您則可從封鎖舊版驗證方法的原則中排除這些使用者。

## <a name="why-are-exclusions-challenging"></a>為何排除項目具有挑戰性？

在 Azure AD 中，您可以將條件式存取原則的範圍設為一組使用者。 您也可以藉由選取目錄角色、個別使用者或來賓使用者，排除其中一些使用者。 請務必記住，設定這些排除項目時，不能對這些使用者強制執行原則意圖。 如果將這些排除項目設定為一份個別使用者清單或一個傳統內部部署安全性群組，則會限制此排除清單的可見性 (使用者可能不知道它的存在)，以及 IT 系統管理員對於它的控制能力 (使用者可以加入安全性群組以略過原則）。 此外，一度有資格排除的使用者可能不再需要它，或者符合其資格。

在排除開頭，有一份略過原則的簡短使用者清單。 經過一段時間，越來越多使用者遭到排除，且清單隨之成長。 在某些時候，需要檢閱此清單，並確認仍然應該排除每個使用者。 從技術觀點來管理清單相當容易，但是由誰進行商務決策，以及如何確定全都可稽核？

不過，如果您使用 Azure AD 群組來設定條件式存取原則的排除，則可使用存取權檢閱作為補償控制項，以提高可見度，並減少具有例外狀況的使用者數目。

## <a name="how-to-create-an-exclusion-group-in-a-conditional-access-policy"></a>如何在條件式存取原則中建立排除群組

請遵循下列步驟來建立新的 Azure AD 群組，以及不會套用至該群組的條件式存取原則。

### <a name="create-an-exclusion-group"></a>建立排除群組

1. 登入 Azure 入口網站。

1. 在左側導覽列中，按一下 [Azure Active Directory]，然後按一下 [群組]。

1. 在上方功能表中，按一下 [新增群組] 以開啟群組窗格。

1. 在 [群組類型] 清單中，選取 [安全性]。 指定名稱和描述。

1. 務必將 [成員資格] 類型設定為 [已指派]。

1. 選取應成為此排除群組成員的使用者，然後按一下 [建立]。

    ![新增群組窗格](./media/conditional-access-exclusion/new-group.png)

### <a name="create-a-conditional-access-policy-that-excludes-the-group"></a>建立可排除群組的條件式存取原則

您現在可以建立使用此排除群組的條件式存取原則。

1. 在左側導覽列中，按一下 [Azure Active Directory]，然後按一下 [條件式存取] 以開啟 [原則] 刀鋒視窗。

1. 按一下 [新增原則] 以開啟 [新增] 窗格。

1. 指定名稱。

1. 在 [指派] 底下按一下 [使用者和群組]。

1. 在 [包含] 索引標籤上，選取 [所有使用者]。

1. 在 [排除] 索引標籤上，將核取記號新增至 [使用者和群組]，然後按一下 [選取排除的使用者]。

1. 選取您所建立的排除群組。

    > [!NOTE]
    > 最佳做法：建議在測試時從原則中排除至少一個系統管理員帳戶，以確保您不會被鎖定在您的租用戶外。

1. 繼續根據貴組織的需求來設定條件式存取原則。

    ![選取排除的使用者](./media/conditional-access-exclusion/select-excluded-users.png)

我們會討論您可以使用存取權檢閱來管理條件式存取原則中排除項目的兩個範例。

## <a name="example-1-access-review-for-users-accessing-from-blocked-countries"></a>範例 1：可供使用者從已封鎖的國家/地區存取的存取權檢閱

假設您有可封鎖從特定國家/地區存取的條件式存取原則。 其中包含從原則中排除的群組。 以下是會檢閱群組成員的建議存取權檢閱。

> [!NOTE]
> 需具備全域管理員或使用者帳戶管理員角色，才可建立存取權檢閱。

1. 此檢閱會每週重複出現。

2. 永遠不會結束，以確保您會讓此排除群組保持最新狀態。

3. 此群組的所有成員都在檢閱範圍內。

4. 每個使用者必須自我證明，他們仍然需要從這些已封鎖的國家/地區存取，因此他們仍必須是群組的成員。

5. 如果使用者未回應檢閱要求，他們將會從群組中自動移除，因此前往這些國家/地區時，再也無法存取租用戶。

6. 啟用郵件通知，讓使用者能收到開始和完成存取權檢閱的通知。

    ![建立存取權檢閱](./media/conditional-access-exclusion/create-access-review-1.png)

## <a name="example-2-access-review-for-users-accessing-with-legacy-authentication"></a>範例 2：可供使用者透過舊版驗證存取的存取權檢閱

假設您有一個條件式存取原則，可封鎖使用舊版驗證和較舊用戶端版本的使用者進行存取。 其中包含從原則中排除的群組。 以下是會檢閱群組成員的建議存取權檢閱。

1. 此檢閱必須是週期性檢閱。

2. 群組中的每個人都必須經過檢閱。

3. 您可以將該原則設定為列出商務單位擁有者作為選取的檢閱者。

4. 自動套用結果，並移除並未獲准繼續使用舊版驗證方法的使用者。

5. 這可能有助於啟用建議事項，讓大型群組的檢閱者得以輕鬆地進行其決策。

6. 啟用郵件通知，讓使用者能收到開始和完成存取權檢閱的通知。

    ![建立存取權檢閱](./media/conditional-access-exclusion/create-access-review-2.png)

**專業提示**：如果您有許多排除群組，因而需要建立多個存取權檢閱，Microsoft Graph 搶鮮版 (Beta) 端點中現在有一個 API，可讓您以程式設計方式建立和管理存取權檢閱。 若要開始，請參閱 [Azure AD 存取權檢閱 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/accessreviews_root)和[透過 Microsoft Graph 擷取 Azure AD 存取權檢閱的範例](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/td-p/236096)。

## <a name="access-review-results-and-audit-logs"></a>存取權檢閱結果和稽核記錄

既然您已備妥一切 (群組、條件式存取原則和存取權檢閱)，就可以開始監視和追蹤這些檢閱的結果。

1. 在 Azure 入口網站中，開啟 [存取權檢閱] 刀鋒視窗。

1. 開啟您為了管理排除群組所建立的控制項和程式。

1. 按一下 [結果]，查看有誰獲准留在清單上，以及誰已遭到移除。

    ![存取權檢閱結果](./media/conditional-access-exclusion/access-reviews-results.png)

1. 然後按一下 [稽核記錄] 以查看在此檢閱期間採取的動作。

    ![存取權檢閱稽核記錄](./media/conditional-access-exclusion/access-reviews-audit-logs.png)

IT 系統管理員知道管理您原則的排除群組有時是不可避免的。 不過，利用 Azure AD 存取權檢閱，可以更輕鬆地維護這些群組、由商務擁有者或使用者本身定期檢閱它們，以及稽核這些變更。

## <a name="next-steps"></a>後續步驟

- [使用 Azure AD 建立群組成員或應用程式存取的存取權檢閱](create-access-review.md)
- [什麼是 Azure Active Directory 中的條件式存取？](../conditional-access/overview.md)
