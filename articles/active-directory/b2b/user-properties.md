---
title: Azure Active Directory B2B 共同作業使用者的屬性 | Microsoft Docs
description: Azure Active Directory B2B 共同作業的使用者屬性可進行設定
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 5f999a17cd375a3338aa936e2f405c36f6021ebc
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984804"
---
# <a name="properties-of-an-azure-active-directory-b2b-collaboration-user"></a>Azure Active Directory B2B 共同作業使用者的屬性

Azure Active Directory (Azure AD) 企業對企業 (B2B) 共同作業使用者是 UserType = [來賓] 的使用者。 此來賓使用者通常是來自合作夥伴組織的使用者，且依預設在邀請目錄中的權限有限。

根據邀請組織的需求，Azure AD B2B 共同作業使用者可以是下列帳戶狀態之一︰

- 狀態 1：位於 Azure AD 的外部執行個體並表示為提出邀請之組織中的來賓使用者。 在此情況下，B2B 使用者會使用屬於受邀租用戶的 Azure AD 帳戶登入。 如果夥伴組織未使用 Azure AD，仍會在 Azure AD 中建立來賓使用者。 他們必須兌換其邀請，而且 Azure AD 必須確認其電子郵件地址。 此排列也稱為 Just-In-Time (JIT) 租用或「熱門」租用。

- 狀態 2：位於 Microsoft 帳戶並表示為主機組織中的來賓使用者。 在此情況下，來賓使用者會以 Microsoft 帳戶登入。 受邀使用者的社交身分識別 (google.com 或類似項目，但不是 Microsoft 帳戶) 會在供應項目兌換期間建立為 Microsoft 帳戶。

- 狀態 3：位於主機組織的內部部署 Active Directory 並與主機組織的 Azure AD 同步處理。 在此版本中，您必須使用 PowerShell 手動變更雲端中這類使用者的 UserType。

- 狀態 4：位於主機組織的 AzureAD，其 UserType = [來賓] 並具備主機組織所管理的認證。

  ![顯示邀請者的姓名縮寫](media/user-properties/redemption-diagram.png)


現在，讓我們來看看狀態 1 中 Azure AD B2B 共同作業使用者在 Azure AD 中的外觀。

### <a name="before-invitation-redemption"></a>邀請兌換之前

![供應項目兌換之前](media/user-properties/before-redemption.png)

### <a name="after-invitation-redemption"></a>邀請兌換之後

![供應項目兌換之後](media/user-properties/after-redemption.png)

## <a name="key-properties-of-the-azure-ad-b2b-collaboration-user"></a>Azure AD B2B 共同作業使用者的金鑰屬性
### <a name="usertype"></a>UserType
此屬性會指出使用者與主機租用的關聯性。 此屬性可以包含兩個值：
- 成員︰此值表示主機組織的員工，以及由組織支薪的使用者。 例如，這位使用者要能夠存取僅供內部使用的網站。 這位使用者就不會被認為是外部共同作業人員。

- 來賓：這個值表示非公司內部人員的使用者，例如外部共同作業者、合作夥伴、客戶或類似的使用者。 這類使用者不會收到執行長的內部備忘，或收到公司權益。

  > [!NOTE]
  > UserType 與使用者登入的方式、使用者的目錄角色等等無關。 此屬性只是表示使用者與主機組織的關聯性，並可讓組織強制執行此屬性的相依原則。

### <a name="source"></a>來源
此屬性指出使用者的登入方式。

- 受邀使用者︰此使用者已受邀請，但尚未兌換邀請。

- 外部 Active Directory︰此使用者位於外部組織，並使用屬於其他組織的 Azure AD 帳戶進行驗證。 這類型的登入對應於狀態 1。

- Microsoft 帳戶：此使用者位於 Microsoft 帳戶，並使用 Microsoft 帳戶進行驗證。 這類型的登入對應於狀態 2。

- Windows Server Active Directory︰此使用者已從屬於此組織的內部部署 Active Directory 登入。 這類型的登入對應於狀態 3。

- Azure Active Directory︰此使用者會使用屬於此組織的 Azure AD 帳戶進行驗證。 這類型的登入對應於狀態 4。
  > [!NOTE]
  > Source 和 UserType 是獨立的屬性。 Source 的值不代表特定的 UserType 值。

## <a name="can-azure-ad-b2b-users-be-added-as-members-instead-of-guests"></a>Azure AD B2B 使用者是否可新增為成員而非來賓？
一般而言，Azure AD B2B 使用者和來賓使用者的意義相同。 因此，Azure AD B2B 共同作業使用者依預設是新增為具有 UserType = [來賓] 的使用者。 不過，在某些情況下，合作夥伴組織是主機組織也所屬之較大組織的成員。 若是如此，主機組織可能會將合作夥伴組織中的使用者視為成員而非來賓。 使用 Azure AD B2B 邀請管理員 API 來從夥伴組織新增或邀請使用者至主機組織作為成員。

## <a name="filter-for-guest-users-in-the-directory"></a>篩選目錄中的來賓使用者

![篩選來賓使用者](media/user-properties/filter-guest-users.png)

## <a name="convert-usertype"></a>轉換 UserType
目前，使用者可以使用 PowerShell 將 UserType 從 [成員] 轉換為 [來賓]，反之亦然。 不過，UserType 屬性應該代表使用者與組織的關聯性。 因此，只有在使用者與組織的關聯性變更時，才應該變更此屬性的值。 如果使用者的關聯性變更，是否應解決使用者主體名稱 (UPN) 是否應該變更等問題？ 使用者是否應該繼續存取相同的資源？ 是否應指派信箱？ 因此，我們不建議以 PowerShell 做為不可部分完成的活動來變更 UserType 。 此外，為免使用 PowerShell 讓這個屬性成為不可變屬性，我們不建議採用此值的相依性。

## <a name="remove-guest-user-limitations"></a>移除來賓使用者限制
在某些情況下，您要提供來賓使用者更高的權限。 您可以將來賓使用者新增至任何角色，甚至移除目錄中預設的來賓使用者限制，為使用者提供與成員相同的權限。

可以關閉預設來賓使用者限制，如此公司目錄中的來賓使用者便可獲得與成員使用者相同的權限。

![移除來賓使用者限制](media/user-properties/remove-guest-limitations.png)

## <a name="can-i-make-guest-users-visible-in-the-exchange-global-address-list"></a>能否在 Exchange 全域通訊清單中顯示來賓使用者？
是。 根據預設，來賓物件不會在貴組織的全域通訊清單中顯示，但您可以使用 Azure Active Directory PowerShell 加以顯示。 如需詳細資訊，請參閱 [Office 365 群組的來賓存取](https://support.office.com/article/guest-access-in-office-365-groups-bfc7a840-868f-4fd6-a390-f347bf51aff6#PickTab=FAQ)中的**能否在 Exchange 全域通訊清單中顯示來賓物件？**。 

## <a name="next-steps"></a>後續步驟

* [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
* [B2B 共同作業使用者權杖](user-token.md)
* [B2B 共同作業使用者宣告對應](claims-mapping.md)
