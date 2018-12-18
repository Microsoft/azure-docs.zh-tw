---
title: PIM 中的安全性精靈 -Azure | Microsoft Docs
description: 說明第一次使用 Azure AD Privileged Identity Management (PIM) 時出現的安全性精靈。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.component: pim
ms.date: 02/27/2017
ms.author: rolyon
ms.custom: pim ; H1Hack27Feb2017
ms.openlocfilehash: 178a4c5e978075f2a59b22a1cccf462138527964
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189074"
---
# <a name="security-wizard-in-pim"></a>PIM 中的安全性精靈
如果您是貴組織中執行 Azure Privileged Identity Management (PIM) 的第一個人，您就會看到精靈。 精靈會協助您了解特殊權限身分識別的安全性風險，以及如何使用 PIM 來降低這些風險。 如果您想要稍後再進行變更，就不需要在精靈中對現有的角色指派進行任何變更。

## <a name="what-to-expect"></a>未來展望
在貴組織開始使用 PIM 之前，所有的角色指派都是永久的︰即使使用者目前不需要其權限，他們都一直保有這些角色。  精靈的第一個步驟會顯示高特殊權限角色的清單，以及這些角色中目前有多少使用者。 如果不清楚一或多個角色，您可以向內切入某個特定角色來深入了解使用者。

精靈的第二個步驟會讓您有機會變更系統管理員的角色指派。  

> [!WARNING]
> 您必須至少有一個全域系統管理員，這一點很重要，而且要有多個具備組織帳戶 (而非 Microsoft 帳戶) 的特殊權限角色管理員。 如果只有一個特殊權限角色管理員，若該帳戶遭到刪除，組織就無法管理 PIM。
> 此外，如果使用者擁有 Microsoft 帳戶 (他們用來登入 Skype 和 Outlook.com 這類 Microsoft 服務的帳戶)，請將角色指派設為永久。 如果您打算要求必須執行 MFA 才能啟用該角色，該使用者將會遭到封鎖。
> 
> 

進行變更之後，將不會再次顯示精靈。 下次當您或其他特殊權限角色管理員使用 PIM 時，您將會看到 PIM 儀表板。  

* 如果您想要從角色新增或移除使用者，或將指派從永久變更為符合資格，請參閱 [如何新增或移除使用者的角色](pim-how-to-add-role-to-user.md)。
* 如果您想要讓更多使用者存取管理 PIM，請參閱 [How to give access to manage Azure AD Privileged Identity Management (如何授與存取權以管理 Azure AD Privileged Identity Management)](pim-how-to-give-access-to-pim.md)。

## <a name="next-steps"></a>後續步驟

- [開始使用 PIM](pim-getting-started.md)
- [在 PIM 中指派 Azure AD 目錄角色](pim-how-to-add-role-to-user.md)
- [授與其他系統管理員存取權以管理 PIM](pim-how-to-give-access-to-pim.md)
