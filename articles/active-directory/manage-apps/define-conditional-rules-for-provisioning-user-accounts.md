---
title: 使用範圍篩選條件佈建應用程式 | Microsoft Docs
description: 在支援使用者佈建自動化的應用程式中，了解如何使用範圍篩選條件來防止佈建不符合商務需求的物件。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: barbkess
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b62679332a45127d7c1a45593c162182886a7150
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719353"
---
# <a name="attribute-based-application-provisioning-with-scoping-filters"></a>含範圍篩選器的屬性型應用程式佈建
本文的目標在於說明如何使用範圍篩選條件來定義以屬性為基礎的規則，以決定將哪些使用者佈建到應用程式。

## <a name="scoping-filter-use-cases"></a>範圍篩選器使用案例

範圍篩選條件可讓 Azure Active Directory (Azure AD) 佈建服務包含或排除有屬性符合特定值的任何使用者。 例如，將使用者從 Azure AD 佈建至銷售團隊所使用的 SaaS 應用程式時，您可以指定只有 "Sales" 之 "Department" 屬性的使用者應該位於佈建範圍中。

範圍篩選條件的使用方式取決於佈建連接器的類型：

* **從 Azure AD 向外佈建到 SaaS 應用程式**。 當 Azure AD 為來源系統時，[使用者和群組指派](assign-user-or-group-access-portal.md)是決定哪些使用者要納入佈建範圍內的最常用方法。 這些指派也用於啟用單一登入，並提供單一方法來管理存取與佈建。 根據屬性值，除了指派或取代之外，範圍篩選器可以選擇性地用於篩選使用者。

    >[!TIP]
    > 您可以根據企業應用程式的指派，將 [範圍](user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) 功能表中的佈建設定底下的設定，變更為 **同步所有使用者與群組**，以停用佈建。 使用此選項再加上屬性型範圍篩選器時，可提供比使用群組型指派更快的效能。  

* **從 HCM 應用程式向內佈建到 Azure AD 和 Active Directory**。 當 [HCM 應用程式 (例如 Workday)](../saas-apps/workday-tutorial.md) 為來源系統時，範圍篩選條件是決定應該將哪些使用者從 HCM 應用程式佈建到 Active Directory 或 Azure AD 的主要方法。

根據預設，Azure AD 佈建連接器尚未設定任何屬性型範圍篩選器。 

## <a name="scoping-filter-construction"></a>範圍篩選器建構

範圍篩選條件包含一個或多個「子句」。 子句會評估每個使用者的屬性，以決定哪些使用者可以通過範圍篩選條件。 例如，您可能有一個子句規定使用者的 "State" 屬性必須等於 "New York"，因此只會將 New York 使用者佈建到應用程式。 

單一子句可定義單一屬性值的單一條件。 如果在單一範圍篩選條件中建立多個子句，則會使用 "AND" 邏輯一起評估這些子句。 也就是說，所有子句都必須評估為 "true"，才能佈建使用者。

最後，您可以針對單一應用程式建立多個範圍篩選條件。 如果有多個範圍篩選條件，則會使用 "OR" 邏輯一起評估這些篩選條件。 這表示，如果任何已設定之範圍篩選條件中的所有子句都評估為 "true"，則會佈建使用者。

Azure AD 佈建服務所處理的每個使用者或群組，一律會根據每個範圍篩選條件來個別評估。

例如，假設有下列範圍篩選條件：

![範圍篩選器](./media/define-conditional-rules-for-provisioning-user-accounts/scoping-filter.PNG) 

根據此範圍篩選條件，使用者必須滿足下列條件才能佈建：

* 他們必須住在紐約。
* 他們必須在工程部門上班。
* 他們的公司員工識別碼必須介於 1,000,000 到 2,000,000 之間。
* 他們的職稱不能是 Null 或空白。

## <a name="create-scoping-filters"></a>建立範圍篩選條件
系統會針對每個 Azure AD 使用者佈建連接器，將範圍篩選器設定為屬性對應的一部分。 下列程序假設您已經針對[其中一個支援的應用程式](../saas-apps/tutorial-list.md)設定自動佈建，而現在要新增範圍篩選條件。

### <a name="create-a-scoping-filter"></a>建立範圍篩選條件
1. 在 [Azure 入口網站](https://portal.azure.com)中，移至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] 區段。

2. 選取您已設定自動佈建的應用程式：例如，"ServiceNow"。

3. 選取 [佈建] 索引標籤。

4. 在 [對應] 區段中，選取您想要設定範圍篩選條件的對應：例如，[將 Azure Active Directory 使用者同步至 ServiceNow]。

5. 選取 [來源物件範圍] 功能表。

6. 選取 [新增範圍篩選器]。

7. 依序選取要比對的來源 [屬性名稱]、[運算子] 和 [屬性值]，來定義子句。 支援下列運算子：

   a. **EQUALS**。 如果評估的屬性完全符合輸入字串值 (區分大小寫)，子句會傳回 "true"。

   b. **NOT EQUALS**。 如果評估的屬性不符合輸入字串值 (區分大小寫)，子句會傳回 "true"。

   c. **IS TRUE**。 如果評估的屬性包含布林值 true，子句會傳回 "true"。

   d. **IS FALSE**。 如果評估的屬性包含布林值 false，子句會傳回 "true"。

   e. **IS NULL**。 如果評估的屬性是空的，子句會傳回 "true"。

   f. **IS NOT NULL**。 如果評估的屬性不是空的，子句會傳回 "true"。

   g. **REGEX MATCH**。 如果評估的屬性符合規則運算式模式，子句會傳回 "true"。 例如：\([1-9][0-9]\) 符合介於 10 到 99 之間的任何數字。

   h. **NOT REGEX MATCH**。 如果評估的屬性不符合規則運算式模式，子句會傳回 "true"。

8. 選取 [新增新範圍子句]。

9. (選擇性) 重複步驟 7-8，新增更多範圍子句。

10. 在 [範圍篩選器標題] 中，新增您範圍篩選器的名稱。

11. 選取 [確定] 。

12. 在 [範圍篩選條件] 畫面上，再次選取 [確定]。 (選擇性) 重複步驟 6-11，新增另一個範圍篩選條件。

13. 在 [屬性對應] 畫面上，選取 [儲存]。 

>[!IMPORTANT] 
> 儲存新的範圍篩選器會為應用程式觸發新的完整同步處理，其中來源系統中的所有使用者都會根據新的範圍篩選器，再次進行估。 如果應用程式中的使用者先前在佈建範圍內，但已落在範圍外，則其帳戶在應用程式中會停用或解除佈建。


## <a name="related-articles"></a>相關文章
* [SaaS 應用程式的自動化使用者佈建和解除佈建](user-provisioning.md)
* [自訂使用者佈建的屬性對應](customize-application-attributes.md)
* [撰寫屬性對應的運算式](functions-for-customizing-application-data.md)
* [帳戶佈建通知](user-provisioning.md)
* [使用 SCIM 以啟用將使用者和群組從 Azure Active Directory 自動佈建到應用程式](use-scim-to-provision-users-and-groups.md)
* [如何整合 SaaS 應用程式的教學課程清單](../saas-apps/tutorial-list.md)

