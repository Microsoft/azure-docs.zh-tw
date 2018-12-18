---
title: Azure AD 身分識別控管 |Microsoft Docs
description: Azure AD 身分識別控管可讓您透過適當的程序與可見性，平衡貴組織的安全性需求與員工生產力。
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
ms.reviewer: markwahl-msft
ms.openlocfilehash: 20b1c8673bfdb3b2207ed63749f79539c396642c
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167650"
---
# <a name="what-is-azure-ad-identity-governance"></a>什麼是 Azure AD 身分識別控管？

Azure Active Directory (Azure AD) 身分識別控管可讓您透過適當的程序與可見性，平衡貴組織的安全性需求與員工生產力。 它可供您確保適當使用者具有適當資源的適當存取權，並它可讓您保護、監視和稽核重要資產的存取，同時確保員工生產力。  

身分識別控管讓組織能夠對員工、商務夥伴和廠商以及服務和應用程式執行下列動作：

- 控管身分識別生命週期
- 控管存取生命週期
- 保護系統管理

具體來說，主要協助組織處理下列四個重要問題：

- 哪些使用者應該存取哪些資源？
- 使用者利用該存取權來做什麼？
- 是否有有效的組織控制項可管理存取權？
- 稽核人員可以確認這些控制項有用嗎？

## <a name="identity-lifecycle"></a>身分識別生命週期

身分識別控管可協助組織取得以下兩者之間的平衡：*生產力* - 人員可以存取資源所需的速度，例如他們何時加入我的組織？ 與*安全性* - 其存取權應該如何隨著時間變更，例如由於該人員的雇用狀態變更？  身分識別生命週期管理是身分識別控管的基礎，而大規模有效控管必須使應用程式的身分識別生命週期管理基礎結構現代化。

對許多組織來說，員工的身分識別生命週期會繫結到該使用者在 HCM (人力資本管理) 系統中的代表項目。  Azure AD Premium 會在 Active Directory 與 Azure Active Directory 中自動維護人員在 Workday 中表示的使用者身分識別，如 [Workday 輸入佈建 (預覽) 教學課程](../saas-apps/workday-inbound-tutorial.md)中所述。  Azure AD Premium 也包含 [Microsoft Identity Manager](/microsoft-identity-manager/)，其可從內部部署 HCM 系統 (例如 SAP、Oracle eBusiness 和 PeopleSoft) 匯入記錄。

有愈來愈多的案例需要與組織外部人員共同作業。 [Azure AD B2B](/azure/active-directory/b2b/) 共同作業可讓您與來賓使用者和來自任何組織的外部夥伴安全地共用貴組織的應用程式與服務，同時持續控制貴公司的資料。

## <a name="access-lifecycle"></a>存取生命週期

組織需有一個程序，可管理在建立使用者的身分識別時起初為該使用者所佈建以外的存取權。  此外，企業組織必須能夠擴充有效率地縮放，才能夠持續開發並強制執行存取原則和控制項。

一般而言，IT 會將存取核准決策委派給商務決策者。  此外，IT 可能是使用者本身。  例如，存取歐洲境內公司行銷應用程式中的機密客戶資料的使用者需要知道公司的原則。 來賓使用者可能不知道他們受邀的組織中資料的處理需求。

組織可以透過[動態群組](../users-groups-roles/groups-dynamic-membership.md)這類技術，並結合使用者佈建至 [SaaS 應用程式](../saas-apps/tutorial-list.md)或[與 SCIM 整合的應用程式](../manage-apps/use-scim-to-provision-users-and-groups.md)來自動存取生命週期程序。  組織也可以控制哪些[來賓使用者能夠存取內部部署應用程式](../b2b/hybrid-cloud-to-on-premises.md)。  接著可以利用週期性 [Azure AD 存取權檢閱](access-reviews-overview.md)，定期檢閱這些存取權限。

當使用者嘗試存取應用程式時，Azure AD 會強制執行[條件式存取](/azure/active-directory/conditional-access/)原則。 例如，條件式存取原則可以包含先顯示[使用規定](active-directory-tou.md)並[確保使用者已同意這些規定](../conditional-access/require-tou.md)，才能夠存取應用程式。

## <a name="privileged-access-lifecycle"></a>特殊權限的存取生命週期

過去，其他廠商將特殊權限的存取描述為與身分識別控管不同的功能。 不過，在 Microsoft，我們認為控管特殊權限的存取是身分識別控管的重要部分 -- 尤其是可能對組織造成與這些系統管理員權限相關聯的誤用可能性。 具有系統管理權限的員工、廠商和約聘人員都需要加以控管。

Azure AD Privileged Identity Management (PIM) 提供量身訂做的其他控制項，以保護 Azure AD、Azure 和其他 Microsoft Online Services 中資源的存取權限。  除了多重要素驗證和條件式存取，Just-In-Time 存取以及 Azure AD PIM 所提供的角色變更警示功能都會提供一組完整的控管控制項，協助保護貴公司的資源 (目錄、Office 365 和 Azure 資源角色)。 如同其他形式的存取權，組織可以使用存取權檢閱，對具有系統管理員角色的所有使用者設定週期性存取權重新確認。

## <a name="getting-started"></a>開始使用

雖然沒有適用於每位客戶的完美解決方案或建議，但下列組態指引 Microsoft 建議使用哪些基準原則來確保更安全且更有生產力的工作力。

- [身分識別與裝置存取組態](/microsoft-365/enterprise/microsoft-365-policies-configurations)
- [保護特殊權限存取](../users-groups-roles/directory-admin-roles-secure.md)


### <a name="access-reviews"></a>存取權檢閱

- [什麼是存取權檢閱？](access-reviews-overview.md)
- [使用存取權檢閱來管理使用者存取權](manage-user-access-with-access-reviews.md)
- [使用存取權檢閱來管理來賓存取權](manage-guest-access-with-access-reviews.md)
- [開始目錄角色的存取權檢閱](../privileged-identity-management/pim-how-to-start-security-review.md)

### <a name="terms-of-use"></a>使用規定

- [使用規定對我有何幫助？](active-directory-tou.md)

### <a name="privileged-identity-management"></a>Privileged Identity Management

- [什麼是 Azure AD PIM？](../privileged-identity-management/pim-configure.md)
