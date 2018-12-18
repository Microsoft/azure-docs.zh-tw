---
title: 無邀請將 B2B 共同作業使用者新增至 Azure Active Directory | Microsoft Docs
description: 您可以讓來賓使用者將其他來賓使用者加入您的 Azure AD，而不需在 Azure Active Directory B2B 共同作業中兌換邀請。
services: active-directory
documentationcenter: ''
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/21/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: e7d76138f9d3701fa7e3177227fbeb4386434126
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45983796"
---
# <a name="add-b2b-collaboration-guest-users-without-an-invitation"></a>在沒有邀請的情況下新增 B2B 共同作業來賓使用者

您現在可以送出共用應用程式的直接連結，來邀請來賓使用者。 透過此方法，除了某些特殊情況，來賓使用者已不再需要使用邀請電子郵件。 來賓使用者可以按一下應用程式連結、檢閱並接受隱私權條款，然後順暢地存取應用程式。 如需詳細資訊，請參閱 [B2B 共同作業邀請兌換](redemption-experience.md)。   

在有此新方法可用之前，若要在不需要邀請電子郵件的情況下邀請來賓使用者，您可以將邀請者 (從您的組織或從合作夥伴組織) 新增至**來賓邀請者**目錄角色，再讓邀請者透過 UI 或透過 PowerShell 將來賓使用者新增至目錄、群組或應用程式 (如果使用 PowerShell，您可以完全隱藏邀請電子郵件)。 例如︰

1. 主組織 (例如，WoodGrove) 中的使用者邀請合作夥伴組織 (例如，Sam@litware.com) 的使用者做為「來賓」。
2. 主組織中的管理員會[設定原則](delegate-invitations.md)，以允許 Sam 識別及新增來自合作夥伴組織 (Litware) 的其他使用者。 (必須將 Sam 新增至**來賓邀請者**角色)。
3. 現在，Sam 可以將來自 Litware 的其他使用者新增至 WoodGrove 目錄、群組或應用程式，而不需要兌換邀請。 若 Sam 具有 Litware 中的適當列舉權限，這會自動發生。
 
此原始方法仍能運作。 不過，行為上有些許不同。 如果您使用 PowerShell，您會發現受邀之來賓帳戶的現在狀態為 **PendingAcceptance**，而不會立即顯示 **Accepted**。 雖然處於擱置狀態，來賓使用者仍可登入並存取應用程式，而不需要按一下電子郵件邀請連結。 擱置狀態表示使用者尚未經過[同意體驗](redemption-experience.md#privacy-policy-agreement)，這是他們接受邀請組織隱私權條款的位置。 當來賓使用者第一次登入時，就會看到此同意畫面。 

如果您邀請使用者加入目錄，來賓使用者必須直接存取資源租用戶特定的 Azure 入口網站 URL (例如 https://portal.azure.com/*resourcetenant*.onmicrosoft.com) 以檢視並同意隱私權條款。

### <a name="next-steps"></a>後續步驟

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- [B2B 共同作業邀請兌換](redemption-experience.md)
- [委派 Azure Active Directory B2B 共同作業邀請](delegate-invitations.md)
- [資訊工作者如何新增 B2B 共同作業使用者？](add-users-information-worker.md)

