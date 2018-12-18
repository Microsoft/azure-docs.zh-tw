---
title: 使用 Azure AD，為群組成員或可存取應用程式的使用者完成存取權檢閱 | Microsoft Docs
description: 了解如何在 Azure Active Directory 中，完成可存取應用程式之群組成員或使用者的存取權檢閱。
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
ms.date: 05/02/2018
ms.author: rolyon
ms.reviewer: mwahl
ms.openlocfilehash: 72b560aa742fb509b55fe5e27cb87b9a224bce7e
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45606878"
---
# <a name="complete-an-access-review-of-members-of-a-group-or-users-access-to-an-application-in-azure-ad"></a>在 Azure AD 中，為群組成員或可存取應用程式的使用者完成存取權檢閱

系統管理員可以使用 Azure Active Directory (Azure AD) 為群組成員或指派給應用程式的或使用者[建立存取權檢閱](create-access-review.md)。 Azure AD 會自動傳送電子郵件給檢閱者，提示其檢閱存取權。 如果使用者未收到電子郵件，您可以傳送[檢閱存取權](perform-access-review.md)的指示給他們。 (請注意，受指派為檢閱者但尚未接受邀請的來賓，將不會收到一封存取權檢閱所發出的電子郵件，因為他們在檢閱之前必須先接受邀請)。存取權檢閱期間結束後，或如果系統管理員停止存取權檢閱，請依照本文的步驟來查看並套用結果。

## <a name="view-an-access-review-in-the-azure-portal"></a>在 Azure 入口網站中檢視存取權檢閱

1. 移至[存取權檢閱頁面](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/)，選取 [程式集]，並選取包含存取權檢閱控制項的程式。

2. 選取 [管理]，並選取存取權檢閱控制項。 如果程式中有許多控制項，您可以篩選特定類型的控制項，並依其狀態排序。 您也可以依據存取權檢閱控制項的名稱或原先建立它之擁有者的顯示名稱來搜尋。 

## <a name="stop-a-review-that-hasnt-finished"></a>停止尚未完成的檢閱

如果檢閱尚未到達排程的結束日期，系統管理員可以選取 [停止] 以提早結束檢閱。 在您停止檢閱之後，就無法再檢閱使用者。 檢閱停止後即無法重新開始。

## <a name="apply-the-changes"></a>套用變更 

存取權檢閱完成之後，因為它觸達結束日期，或是系統管理員手動將它停止，而檢閱未設定自動套用，您可以選取 [套用] 以手動套用變更。 檢閱的結果是藉由更新群組或應用程式實作。 若在檢閱中拒絕使用者的存取權，當系統管理員選取此選項時，Azure AD 會移除其成員資格或應用程式指派。 

存取權檢閱完成之後，會設定自動套用，然後檢閱的狀態會從 [已完成] 歷經中繼狀態最後變更為 [已套用] 狀態。 您應該會看到遭到拒絕的使用者 (若有) 在幾分鐘內從資源群組成員資格或應用程式指派中移除。

已設定的自動套用檢閱，或選取 [套用] 並不會影響源自內部部署目錄中的群組，或動態群組。 如果您想要變更源自於內部部署的群組，請下載結果，並將那些變更套用至該目錄中的群組圖像。

## <a name="download-the-results-of-the-review"></a>下載檢閱的結果

若要擷取檢閱的結果，請選取 [核准]，然後選取 [下載]。 可在 Excel 中，或開啟 UTF-8 編碼 CSV 檔案的其他程式中檢視產生的 CSV 檔案。

## <a name="optional-delete-a-review"></a>選擇性：刪除檢閱
如果您對檢閱不再感興趣，可以將它刪除。 選取 [刪除] 以從 Azure AD 移除檢閱。

> [!IMPORTANT]
> 刪除執行前不會收到警告，因此請務必確定您想要刪除檢閱。
> 
> 

## <a name="next-steps"></a>後續步驟

- [使用 Azure AD 存取權檢閱來管理使用者存取權](manage-user-access-with-access-reviews.md)
- [使用 Azure AD 存取權檢閱來管理來賓存取權](manage-guest-access-with-access-reviews.md)
- [管理 Azure AD 存取權檢閱的程式和控制項](manage-programs-controls.md)
- [建立群組成員或應用程式存取的存取權檢閱](create-access-review.md)
- [在 Azure AD 管理角色中建立使用者的存取權檢閱](../privileged-identity-management/pim-how-to-start-security-review.md)
