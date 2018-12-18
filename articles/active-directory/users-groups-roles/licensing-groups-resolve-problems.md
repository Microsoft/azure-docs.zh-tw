---
title: 解決 Azure Active Directory 中群組的授權問題 | Microsoft Docs
description: 當您使用 Azure Active Directory 以群組為基礎的授權時，如何識別及解決授權指派問題
services: active-directory
keywords: Azure AD 授權
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: users-groups-roles
ms.topic: article
ms.workload: identity
ms.date: 06/05/2017
ms.author: curtand
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 5d64cf71ea3a44b7539835e3616150218e8b3635
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37861091"
---
# <a name="identify-and-resolve-license-assignment-problems-for-a-group-in-azure-active-directory"></a>識別及解決 Azure Active Directory 中群組的授權指派問題

Azure Active Directory (Azure AD) 中以群組為基礎的授權會介紹使用者授權錯誤狀態的概念。 在本文章中，我們會說明使用者最後都處於此狀態的原因。

當您將授權直接指派給個別使用者，而不使用以群組為基礎的授權時，指派作業會失敗。 例如，當您在使用者系統上執行 PowerShell Cmdlet `Set-MsolUserLicense` 時，許多與商務邏輯相關的原因可能會造成此 Cmdlet 失敗。 例如，可能是授權數量不足，或無法同時指派兩個服務方案的衝突。 系統會立即向您回報此問題。

當您使用以群組為基礎的授權時，可能會發生相同錯誤，但是當 Azure AD 服務指派授權時，則會在背景中發生錯誤。 基於這個原因，無法立即向您通知這些錯誤。 而是會記錄在使用者物件上，然後透過系統管理入口網站報告。 授權使用者的原意永遠不會遺失，但會記錄為錯誤狀態，供未來調查和解決。

## <a name="how-to-find-license-assignment-errors"></a>如何找出授權指派錯誤
**找出授權指派錯誤**

   1. 若要在特定群組中尋找處於錯誤狀態的使用者，請開啟該群組的窗格。 如果有任何使用者處於錯誤狀態，[授權] 底下會出現通知。

   ![群組，錯誤通知](./media/licensing-groups-resolve-problems/group-error-notification.png)

   2. 選取這份通知，以開啟所有受影響的使用者清單。 您可以分別選取每個使用者以查看詳細資料。

   ![群組，處於錯誤狀態的使用者清單](./media/licensing-groups-resolve-problems/list-of-users-with-errors.png)

   3. 若要尋找包含至少一個錯誤的所有群組，在 [Azure Active Directory] 刀鋒視窗上選取 [授權]，然後選取 [概觀]。 值得您注意的群組會顯示資訊方塊。

   ![概觀，處於錯誤狀態的群組相關資訊](./media/licensing-groups-resolve-problems/group-errors-widget.png)

   4. 選取方塊以查看具有錯誤的所有群組的清單。 您可以選取每個群組以查看詳細資訊。

   ![概觀，具有錯誤的群組清單](./media/licensing-groups-resolve-problems/list-of-groups-with-errors.png)


下列幾節描述每個潛在問題及其解決方法。

## <a name="not-enough-licenses"></a>沒有足夠的授權

**問題：** 群組中指定的其中一項產品沒有足夠的可用授權。 您需要為產品購買更多授權，或從其他使用者或群組釋放未使用的授權。

若要查看有多少授權可用，請移至 [Azure Active Directory] > [授權] > [所有產品]。

若要查看哪些使用者及群組在取用授權，請選取產品。 在 [授權的使用者] 底下，您會看到已直接或透過一或多個群組而被指派授權的所有使用者的清單。 在 [授權的群組] 底下，您會看到已被指派該產品的所有群組。

**PowerShell：** PowerShell Cmdlet 會將此錯誤報告為 _CountViolation_。

## <a name="conflicting-service-plans"></a>衝突的服務方案

**問題：** 群組中指定的其中一個產品包含服務方案，與已透過不同產品指派給使用者的另一個服務方案相衝突。 某些服務方案會設定為不能與另一個相關的服務方案一起指派給相同使用者。

請思考一下下列範例。 使用者擁有直接指派的 Office 365 企業版 *E1* 授權，所有方案皆啟用。 使用者已新增至獲得 Office 365 企業版 *E3* 產品指派的群組。 E3 產品包含的服務方案不能與 E1 包含的方案重疊，因此，群組授權指派會失敗，而發生「衝突的服務方案」錯誤。 在此範例中，衝突的服務方案是︰

-   SharePoint Online (方案 2) 與 SharePoint Online (方案 1) 衝突。
-   Exchange Online (方案 2) 與 Exchange Online (方案 1) 衝突。

若要解決此衝突，您必須停用兩個方案。 您可以停用直接指派給使用者的 E1 授權。 就是，必須在 E3 授權中修改整個群組授權指派並停用方案。 或者，如果在 E3 授權內容中是多餘的，您可能會決定從使用者移除 E1 授權。

如何解決產品授權的衝突一律屬於系統管理員的決策。 Azure AD 不會自動解決授權衝突。

**PowerShell：** PowerShell Cmdlet 會將此錯誤報告為 _MutuallyExclusiveViolation_。

## <a name="other-products-depend-on-this-license"></a>其他產品相依於此授權

**問題：** 群組中指定的其中一個產品包含服務方案，必須在另一個產品中針對另一個服務方案啟用，才能夠運作。 當 Azure AD 嘗試移除基礎服務方案時會發生此錯誤。 比方說，從群組移除使用者時可能會發生這種情形。

若要解決這個問題，您必須確定所需的方案仍透過其他方法指派給使用者，或已停用這些使用者的相依服務。 之後，您可以適當地移除這些使用者的群組授權。

**PowerShell：** PowerShell Cmdlet 會將此錯誤報告為 _DependencyViolation_。

## <a name="usage-location-isnt-allowed"></a>不允許使用位置

**問題：** 由於當地法律和法規，無法在所有位置使用某些 Microsoft 服務。 您必須為使用者指定 [使用位置] 屬性，才可以將授權指派給使用者。 您可以在 Azure 入口網站的 [使用者] > [設定檔] > [設定] 區段之下指定此位置。

當 Azure AD 嘗試將群組授權指派給不支援其使用位置的使用者時，將會失敗並對該使用者記錄錯誤。

若要解決這個問題，請從授權群組不支援的位置中移除使用者。 或者，如果目前的使用位置值不代表實際使用者的位置，您可以進行修改，以便下一次正確指派授權 (如果支援新的位置)。

**PowerShell：** PowerShell Cmdlet 會將此錯誤報告為 _ProhibitedInUsageLocationViolation_。

> [!NOTE]
> 當 Azure AD 指派群組授權時，不具有指定之使用位置的任何使用者會繼承目錄的位置。 我們建議系統管理員先為使用者設定正確的使用位置值，再使用以群組為基礎的授權，以符合當地法規。

## <a name="what-happens-when-theres-more-than-one-product-license-on-a-group"></a>當群組中有一個以上的產品授權時，會發生什麼事？

您可以將一個以上的產品授權指派給群組。 例如，您可以將 Office 365 企業版 E3 和 Enterprise Mobility + Security 指派給群組，以便輕鬆地為使用者啟用所有已納入的服務。

Azure AD 會嘗試將群組中指定的所有授權指派給每位使用者。 如果 Azure AD 因為商務邏輯問題而無法指派其中一個產品，則也不會指派群組中的其他授權。 例如，全體的授權不足，或與使用者已啟用的其他服務發生衝突。

您可以看到未被指派授權的使用者，並查看此問題所影響的產品。

## <a name="how-do-you-manage-licenses-for-products-with-prerequisites"></a>如何為具有必要條件的產品來管理授權？

您可能擁有的某些 Microsoft Online 產品是「附加元件」。 附加元件規定使用者或群組啟用必要條件服務方案，才能指派授權給他們。 使用以群組為基礎的授權時，系統會要求必要條件和附加元件服務方案必須出現在相同的群組中。 這是為了確保新增至群組的所有使用者都可以收到完整有效的產品。 讓我們思考一下以下的範例：

「Microsoft 工作場所分析」是附加元件產品。 它包含具有相同名稱的單一服務方案。 只有在同時指派下列其中一個必要條件時，我們才能將此服務方案指派給使用者或群組：
- Exchange Online (方案 1) 
- Exchange Online (方案 2)

如果我們嘗試將此產品本身指派給群組，入口網站會傳回錯誤。 選取錯誤通知會顯示下列詳細資料：

![群組，缺少必要條件](./media/licensing-groups-resolve-problems/group-prerequisite-required.png)

選取詳細資料會顯示下列錯誤訊息：

>授權作業失敗。 請先確定群組具備必要服務，再新增或移除相依的服務。 **「Microsoft 工作場所分析」服務也需要啟用 Exchange Online (方案 2)。**

若要將此附加元件授權指派給群組，我們必須確定群組中也包含必要條件服務方案。 例如，我們可能更新已經包含完整 Office 365 E3 產品的現有群組，然後在其中新增附加元件產品。

也可以建立獨立群組，而其中只包含至少可讓附加元件運作的必要產品。 此群組可用來只授權特定的使用者才能使用附加元件產品。 在此範例中，我們已經將下列產品指派給相同的群組：
- 僅啟用 Exchange Online (方案 2) 服務方案的 Office 365 企業版 E3
- Microsoft 工作場所分析

![群組，包含必要條件](./media/licensing-groups-resolve-problems/group-addon-with-prerequisite.png)

從現在起，新增至此群組的任何使用者都會耗用 E3 產品的一個授權，以及工作場所分析產品的一個授權。 同時，這些使用者可能屬於另一個會提供完整 E3 產品的群組，但他們仍然只耗用該產品的一個授權。

> [!TIP]
> 您可以針對每個必要條件服務方案建立多個群組。 比方說，如果您對使用者同時使用 Office 365 企業版 E1 和 Office 365 企業版 E3，則可以建立兩個群組來授權 Microsoft 工作場所分析：一個使用 E1 作為必要條件，另一個使用 E3 作為必要條件。 這可讓您將附加元件散發給 E1 和 E3 使用者，而不會耗用額外的授權。

## <a name="license-assignment-fails-silently-for-a-user-due-to-duplicate-proxy-addresses-in-exchange-online"></a>使用者的授權指派會因為 Exchange Online 中 Proxy 位址重複而無訊息的失敗

如果您使用 Exchange Online，則租用戶中有些使用者可能錯誤地設定相同的 Proxy 位址值。 當以群組為基礎的授權嘗試指派授權給這類使用者時，將會失敗，但不會記錄錯誤。 在此功能的預覽版中，無法在此執行個體中記錄錯誤，我們會在「正式運作」之前解決此問題。

> [!TIP]
> 如果您發現某些使用者未收到授權，也沒有對這些使用者記錄錯誤，請先檢查他們是否有重複的 Proxy 位址。
> 若要查看是否有重複的 Proxy 位址，請針對 Exchange Online 執行下列 PowerShell Cmdlet：
```
Run Get-Recipient | where {$_.EmailAddresses -match "user@contoso.onmicrosoft.com"} | fL Name, RecipientType,emailaddresses
```
> 如需此問題的詳細資訊，請參閱 [Exchange Online 中出現「已使用此 Proxy 位址」錯誤訊息](https://support.microsoft.com/help/3042584/-proxy-address-address-is-already-being-used-error-message-in-exchange-online)。 該文章也包括[如何使用遠端 PowerShell 連線至 Exchange Online](https://technet.microsoft.com/library/jj984289.aspx)的相關資訊。

為受影響的使用者解決任何 Proxy 位址問題之後，請務必在群組上強制執行授權處理，以確保現在可以套用授權。

## <a name="how-do-you-force-license-processing-in-a-group-to-resolve-errors"></a>如何強制處理群組中的授權來解決錯誤？

根據您為了解決錯誤所採取的步驟而定，可能需要手動觸發處理群組來更新使用者狀態。

比方說，如果您移除使用者的直接授權指派來釋出一些授權，則必須觸發處理先前未能完整授權所有使用者成員的群組。 若要重新處理群組，請移至群組窗格，開啟 [授權]，然後選取工具列上的 [重新處理]按鈕。

## <a name="next-steps"></a>後續步驟

若要深入了解透過群組管理授權的其他案例，請閱讀下列各項：

* [將授權指派給 Azure Active Directory 中的群組](licensing-groups-assign.md)
* [什麼是 Azure Active Directory 中以群組為基礎的授權？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [如何將個別授權使用者移轉至 Azure Active Directory 中以群組為基礎的授權](licensing-groups-migrate-users.md)
* [Azure Active Directory 群組型授權其他案例 (英文)](licensing-group-advanced.md)
