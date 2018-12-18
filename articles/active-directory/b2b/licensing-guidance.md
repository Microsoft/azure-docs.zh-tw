---
title: Azure Active Directory B2B 共同作業授權指引 | Microsoft Docs
description: Azure Active Directory B2B 共同作業不需要 Azure AD 付費授權，但您也可以取得付費功能給 B2B 來賓使用者
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 08/09/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 1d92f68bbb5e8c001594e4f78f90cb10496aaf29
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984487"
---
# <a name="azure-active-directory-b2b-collaboration-licensing-guidance"></a>Azure Active Directory B2B 共同作業授權指引

您可以使用 Azure AD B2B 共同作業功能，將來賓使用者邀請至 Azure AD 租用戶，允許它們存取 Azure AD 服務和貴組織中的其他資源。 如果您不需要讓 B2B 共同作業來賓使用者存取 Azure AD 付費功能，他們就必須獲得適當的 Azure AD 授權。 

具體而言：
* Azure AD Free 功能可供來賓使用者使用，且無需額外的授權。
* 如果您需要讓 B2B 使用者存取 Azure AD 付費功能，就必須擁有足夠的授權，以支援這些 B2B 來賓使用者。
* 具有 Azure AD 付費授權的邀請方租用戶，擁有另外五位受邀加入租用戶之 B2B 來賓使用者的 B2B 共同作業使用權。
* 擁有邀請方租用戶的客戶，必須決定多少位 B2B 共同作業使用者需要 Azure AD 付費功能。 根據您需要取得給來賓使用者的 Azure AD 付費功能而定，您擁有的 Azure AD 付費授權必須足以涵蓋相同 5:1 比例的 B2B 共同作業使用者。

會從夥伴公司將 B2B 共同作業來賓使用者新增為使用者，而非貴組織的員工或貴企業集團中不同業務部門的員工。 B2B 來賓使用者可以使用外部認證或貴組織所擁有的認證來登入，如本文中所述。 

換句話說，B2B 授權的設定方式並不是依使用者驗證，而是依貴組織使用者的關聯性。 如果這些使用者並非夥伴，就會以不同的授權條款來處理。 不會針對授權用途將它們視為 B2B 共同作業使用者，即使其 UserType 是標示為「來賓」。 它們應以一般方式獲得授權，每個使用者獲得一項授權。 這些使用者包括：
* 您的員工
* 使用外部身分識別登入的職員
* 貴企業集團中不同業務部門的員工


## <a name="licensing-examples"></a>授權範例
- 客戶想要邀請 100 位 B2B 共同作業使用者加入其 Azure AD 租用戶。 此客戶指派存取管理和佈建給所有使用者，但其中 50 位使用者還需要 MFA 和條件式存取。 此客戶必須購買 10 個 Azure AD Basic 授權及 10 個 Azure AD Premium P1 授權，才能正確地涵蓋這些 B2B 使用者。 如果客戶打算對 B2B 使用者使用「身分識別保護」功能，則他們必須有 Azure AD Premium P2 授權，才能涵蓋同樣 5:1 比例的受邀使用者。
- 客戶有 10 位目前以 Azure AD「進階 P1」授權的員工。 他們現在想邀請 60 位 B2B 使用者，而這些使用者全部都需多重要素驗證 (MFA)。 依據 5:1 授權規則，此客戶必須至少有 12 個 Azure AD Premium P1 授權，才能涵蓋全部 60 位 B2B 共同作業使用者。 由於他們已經有 10 位員工的 10 個 Premium P1 授權，因此，他們有權使用 Premium P1 功能 (例如 MFA) 來邀請 50 位 B2B 使用者。 所以，在此範例中，他們必須再購買 2 個 Premium P1 授權，才能涵蓋剩餘的 10 位 B2B 共同作業使用者。

> [!NOTE]
> 尚無法直接將授權指派給 B2B 使用者，以啟用這些 B2B 共同作業使用者權限。

擁有邀請方租用戶的客戶，必須決定多少位 B2B 共同作業使用者需要 Azure AD 付費功能。 根據您想要取得哪些 Azure AD 付費功能給來賓使用者而定，您擁有的 Azure AD 付費授權必須足以涵蓋 5:1 比例的 B2B 共同作業使用者。 

## <a name="additional-licensing-details"></a>其他授權詳細資料
- 並不需要將授權實際指派給 B2B 使用者帳戶。 系統會根據 5:1 比例來自動計算和報告授權。
- 如果租用戶中沒有 Azure AD 付費授權，則每位受邀使用者會獲得 Azure AD Free 版本所提供的權限。
- 如果 B2B 共同作業使用者已從其組織獲得 Azure AD 付費授權，他就不會取用邀請方租用戶的其中一個 B2B 共同作業授權。

## <a name="advanced-discussion-what-are-the-licensing-considerations-when-we-add-users-from-a-conglomerate-organization-as-members-using-your-apis"></a>進階討論：當我們使用您的 API 從某個集團組織新增使用者作為「成員」時，有什麼授權考量？
B2B 來賓使用者是從合作夥伴組織受邀來與主辦組織共同作業的使用者。 一般而言，任何其他情況都不符合 B2B 資格，即使是使用 B2B 功能也一樣。 讓我們來特別看看兩個案例：

1. 如果主辦組織使用取用者位址來邀請員工
  * 此情節不符合我們的授權原則，不建議使用。

2. 如果主辦組織從另一個集團組織新增使用者
  1. 在此案例中是使用 B2B API 來邀請使用者，但此案例不是傳統的 B2B。 在理想情況下，我們應該讓這些組織邀請其他組織使用者作為成員 (我們的 API 允許這麼做)。 在此情況下，必須將授權指派給這些成員，讓他們存取邀請組織中的資源。

  2. 有些組織可能會想要將另一個組織的使用者新增為「來賓」來作為一項原則。 這裡有兩個案例：
      * 集團組織已經使用 Azure AD 且受邀的使用者已在另一個組織中獲得授權：在此情況下，我們不會預期受邀使用者需依循本文件稍早所配置的 1:5 公式。 

      * 集團組織未使用 Azure AD 或沒有適當的授權：在此情況下，請依循本文件稍早所配置的 1:5 公式。

## <a name="next-steps"></a>後續步驟

請參閱下列有關 Azure AD B2B 共同作業的文章：

* [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
* [Azure Active Directory B2B 共同作業常見問題 (FAQ)](faq.md)
