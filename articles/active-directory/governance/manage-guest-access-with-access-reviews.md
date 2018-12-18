---
title: 使用 Azure AD 存取權檢閱來管理來賓存取權 | Microsoft Docs
description: 使用 Azure Active Directory 存取權檢閱，以作為群組成員或指派給應用程式的方式管理來賓使用者
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
ms.date: 06/21/2018
ms.author: rolyon
ms.reviewer: mwahl
ms.openlocfilehash: 7d3361b44d47fd97c14f677e5e4bbc6c1ee5ad3f
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45607638"
---
# <a name="manage-guest-access-with-azure-ad-access-reviews"></a>使用 Azure AD 存取權檢閱來管理來賓存取權


您可以搭配使用 Azure Active Directory (Azure AD) 與 [Azure AD B2B 功能](../b2b/what-is-b2b.md)，輕鬆地跨組織界限啟用共同作業。 [系統管理員](../b2b/add-users-administrator.md)或[其他使用者](../b2b/what-is-b2b.md)可邀請來自其他租用戶的來賓使用者。 這也適用於社交識別，例如 Microsoft 帳戶。

您可以輕易確保來賓使用者有適當的存取權。 您可藉由要求來賓本身或決策者參與存取權檢閱，並重新證實 (或「證明」) 來賓的存取權。 檢閱者可以根據 Azure AD 的建議，對每位使用者的持續存取需求給予其意見。 存取權檢閱完成時，您可以接著進行變更，並為不再需要存取的來賓移除存取權。

> [!NOTE]
> 本文件著重於檢閱來賓使用者的存取權。 如果您想檢閱所有使用者的存取權 (不只來賓)，請參閱[使用存取權檢閱管理使用者存取權](manage-user-access-with-access-reviews.md)。 若您想要檢閱使用者的系統管理角色 (例如全域系統管理員) 成員資格，請參閱[在 Azure AD Privileged Identity Management 中開始存取權檢閱](../privileged-identity-management/pim-how-to-start-security-review.md)。 
>
>

## <a name="prerequisites"></a>必要條件 


存取權檢閱是 Microsoft Enterprise Mobility + Security, E5 中所含之 Azure AD Premium P2 版本的一項功能。 如需詳細資訊，請參閱 [Azure Active Directory 版本](../fundamentals/active-directory-whatis.md)。 透過這項功能進行互動 (包括建立檢閱、填寫檢閱或確認其存取權) 的每個使用者，都需要授權。 

若您計劃讓來賓使用者檢閱自己的存取權，請閱讀來賓使用者授權相關資訊。 如需詳細資訊，請參閱 [Azure AD B2B 共同作業授權](../b2b/licensing-guidance.md)。

## <a name="create-and-perform-an-access-review-for-guests"></a>建立和執行來賓的存取權檢閱

首先，讓存取權檢閱出現在檢閱者的存取面板上。 以全域管理員或使用者帳戶管理員身分，移至[存取權檢閱頁面](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/)。 

Azure AD 可檢閱來賓使用者的幾個案例。

選取下列其中一個：

 - Azure AD 中的一個群組，成員包含一或多個來賓。
 - 與 Azure AD 連線的應用程式，有一或多個來賓使用者指派至此。 

然後，您可以決定要讓每個來賓都能檢閱自己的存取權，還是要讓一或多個使用者檢閱所有來賓的存取權。

 下列各節說明這些案例。

### <a name="ask-guests-to-review-their-own-membership-in-a-group"></a>要求來賓檢閱其在群組中的成員資格

使用存取權檢閱，可確定已受邀並新增至群組的使用者持續需要存取權。 您可以輕鬆地要求來賓檢閱其在該群組中的成員資格。

1. 若要開始群組的存取權檢閱，請選取檢閱只包含來賓使用者成員，並讓成員自行檢閱。 如需詳細資訊，請參閱[建立存取權檢閱](create-access-review.md)。

2. 要求每個來賓檢閱他們自己的成員資格。 根據預設，已接受邀請的每個來賓都會收到 Azure AD 的電子郵件，其中包含存取權檢閱的連結。 Azure AD 有來賓如何[檢閱其存取權](perform-access-review.md)的指示。

3. 在檢閱者提供輸入後，停止存取權檢閱並套用變更。 如需詳細資訊，請參閱[完成存取權檢閱](complete-access-review.md)。

4. 除了那些拒絕持續存取權需求的使用者外，您可能也想要移除未回應的使用者。 未回應的使用者可能不會再收到電子郵件。

5. 如果群組不是使用於存取權管理，您也可移除因未接受邀請而未獲選參與檢閱的使用者。 未接受可能是因為受邀使用者的電子郵件地址拼錯。 如果某群組是作為通訊群組清單，可能某些來賓使用者因為是連絡人物件而未獲選參與。

### <a name="ask-a-sponsor-to-review-a-guests-membership-in-a-group"></a>要求贊助者檢閱來賓在群組中的成員資格

您可以要求贊助者 (例如群組的擁有者) 檢閱來賓是否需要持續保有群組成員資格。

1. 若要開始群組的存取權檢閱，請選取檢閱只包含來賓使用者成員。 然後指定一或多個檢閱者。 如需詳細資訊，請參閱[建立存取權檢閱](create-access-review.md)。

2. 要求檢閱者提供意見。 根據預設，他們會各自收到來自 Azure AD 的電子郵件，其中的連結可連至存取面板以[執行其存取權檢閱](perform-access-review.md)。

3. 在檢閱者提供輸入後，停止存取權檢閱並套用變更。 如需詳細資訊，請參閱[完成存取權檢閱](complete-access-review.md)。

### <a name="ask-guests-to-review-their-own-access-to-an-application"></a>要求來賓檢閱其自己的應用程式存取權

使用存取權檢閱，可確保因特定應用程式而受邀的使用者持續需要存取權。 您可以輕鬆地要求來賓檢閱自己的存取權需求。

1. 若要開始應用程式的存取權檢閱，請選取檢閱只包含來賓，並讓使用者自行檢閱存取權。 如需詳細資訊，請參閱[建立存取權檢閱](create-access-review.md)。

2. 要求每個來賓檢閱其自己的應用程式存取權。 根據預設，已接受邀請的每個來賓會收到 Azure AD 的電子郵件，其中的連結可連至您組織的存取面板以執行存取權檢閱。 Azure AD 有來賓如何[檢閱其存取權](perform-access-review.md)的指示。

3. 在檢閱者提供輸入後，停止存取權檢閱並套用變更。 如需詳細資訊，請參閱[完成存取權檢閱](complete-access-review.md)。

4. 除了那些拒絕持續存取權需求的使用者外，您可能也想要移除未回應的來賓使用者。 未回應的使用者可能不會再收到電子郵件。 您也可以移除未獲選參與的來賓使用者，尤其如果他們最近未獲邀請。 這些使用者未接受邀請，因此無法存取應用程式。 

### <a name="ask-a-sponsor-to-review-a-guests-access-to-an-application"></a>要求贊助者檢閱來賓的應用程式存取權

您可以要求贊助者 (例如應用程式的擁有者) 檢閱來賓是否需要應用程式的持續存取權。

1. 若要開始應用程式的存取權檢閱，請選取檢閱只包含來賓。 然後指定一或多個使用者作為檢閱者。 如需詳細資訊，請參閱[建立存取權檢閱](create-access-review.md)。

2. 要求檢閱者提供意見。 根據預設，他們會各自收到來自 Azure AD 的電子郵件，其中的連結可連至存取面板以[執行其存取權檢閱](perform-access-review.md)。

3. 在檢閱者提供輸入後，停止存取權檢閱並套用變更。 如需詳細資訊，請參閱[完成存取權檢閱](complete-access-review.md)。

### <a name="ask-guests-to-review-their-need-for-access-in-general"></a>要求來賓檢閱其一般的存取權需求

在某些組織中，來賓可能無法得知其群組成員資格。

> [!NOTE]
> 舊版的 Azure 入口網站不允許 UserType 是「來賓」的使用者具備系統管理存取權。 在某些情況下，您目錄中的系統管理員可能已使用 PowerShell 將來賓的 UserType 值變更為「成員」。 如果您的目錄之前有過這項變更，先前的查詢可能不會包括過去具有系統管理存取權限的所有使用者。 在此情況下，您需要變更來賓的 UserType 或在群組成員資格中手動加入來賓。

1. 如果不存在適當的群組，請在 Azure AD 中建立成員是來賓的安全性群組。 比方說，您可以建立手動保留來賓成員資格的群組。 或者，您可針對 Contoso 租用戶中 UserType 屬性值為「來賓」的使用者，建立具有「Contoso 來賓」之類名稱的動態群組。  為了提高效率，請確定群組絕大多數是來賓 - 請勿選取包含不需要檢閱之使用者的群組。

2. 若要開始該群組的存取權檢閱，請選取要做為成員的檢閱者。 如需詳細資訊，請參閱[建立存取權檢閱](create-access-review.md)。

3. 要求每個來賓檢閱他們自己的成員資格。 根據預設，已接受邀請的每個來賓會收到 Azure AD 的電子郵件，其中的連結可連至您組織的存取面板以執行存取權檢閱。 Azure AD 有來賓如何[檢閱其存取權](perform-access-review.md)的指示。  未接受邀請的來賓會在檢閱結果中顯示為「未通知到」。

4. 在檢閱者提供輸入後，停止存取權檢閱。 如需詳細資訊，請參閱[完成存取權檢閱](complete-access-review.md)。

5. 移除被拒絕、未完成檢閱，或先前未接受邀請之來賓的來賓存取權。 如果某些來賓是連絡人且獲選參與檢閱，或是他們未接受邀請，您可以使用 Azure 入口網站或 PowerShell 來停用其帳戶。 如果來賓不再需要存取權、也不是連絡人，您可以使用 Azure 入口網站或 PowerShell 刪除來賓使用者物件，以從您的目錄中移除其使用者物件。

## <a name="next-steps"></a>後續步驟

[建立群組成員或應用程式存取的存取權檢閱](create-access-review.md)







