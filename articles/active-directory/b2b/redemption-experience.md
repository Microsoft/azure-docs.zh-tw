---
title: B2B 共同作業中的邀請兌換 - Azure Active Directory | Microsoft Docs
description: 描述使用者的 Azure AD B2B 共同作業邀請兌換經驗，包括隱私權條款的協議。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 7819ed3e18092e8b7bdf52225e7025b4b6d8146a
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45981876"
---
# <a name="azure-active-directory-b2b-collaboration-invitation-redemption"></a>Azure Active Directory B2B 共同作業邀請兌換

若要透過 Azure Active Directory (Azure AD) B2B 共同作業與夥伴組織的使用者共同作業，您可以邀請來賓使用者存取共用應用程式。 透過使用者介面將來賓使用者新增至目錄之後，或透過 PowerShell 邀請使用者之後，來賓使用者必須經歷第一次同意程序，在該程序中同意[隱私權條款](#privacy-policy-agreement)。 此程序會以下列任一種方式進行：

- 來賓邀請者會送出共用應用程式的直接連結。 受邀者可按一下連結進行登入、接受隱私權條款，然後順暢地存取共用資源。 (來賓使用者仍會收到具有兌換 URL 的邀請電子郵件，但是除了某些特殊情況，不再需要使用邀請電子郵件。)  
- 來賓使用者收到邀請電子郵件，並按一下兌換 URL。 在第一次登入時，系統會提示他們接受隱私權條款。

## <a name="redemption-through-a-direct-link"></a>透過直接連結兌換

來賓邀請者可以送出共用應用程式的直接連結，來邀請來賓使用者。 對來賓使用者而言，兌換經驗與登入共用應用程式一樣簡單。 他們可以按一下應用程式的連結、檢閱並接受隱私權條款，然後順暢地存取應用程式。 在大部分情況下，來賓使用者不再需要按一下電子郵件邀請中的兌換 URL。

如果您透過使用者介面邀請來賓使用者，或在 PowerShell 邀請過程中選擇傳送邀請電子郵件，則受邀的使用者還是會收到邀請電子郵件。 此電子郵件在下列特殊情況中很有用：

- 使用者沒有 Azure AD 帳戶或 Microsoft 帳戶 (MSA)。 在此情況下，使用者必須先建立 MSA 再按一下連結；或是使用邀請電子郵件中的兌換 URL。 兌換程序會自動提示使用者建立 MSA。
- 受邀的使用者物件有時可能因為與連絡人物件 (例如 Outlook 連絡人物件) 衝突，而沒有電子郵件地址。 在此情況下，使用者必須按一下邀請電子郵件中的兌換 URL。
- 使用者可以使用受邀的電子郵件地址別名登入。 (別名是與電子郵件帳戶相關聯的其他電子郵件位址。)在此情況下，使用者必須按一下邀請電子郵件中的兌換 URL。

如果這些特殊情況對您的組織很重要，建議您使用仍會傳送邀請電子郵件的方法來邀請使用者。 此外，如果使用者不屬於其中一種特殊情況，仍可按一下邀請電子郵件中的 URL 來進行存取。

## <a name="redemption-through-the-invitation-email"></a>透過邀請電子郵件兌換

如果透過傳送邀請電子郵件的方法獲得邀請，使用者也可以透過邀請電子郵件兌換邀請。 受邀的使用者可以按一下電子郵件中的兌換 URL，然後檢閱並接受隱私權條款。 以下更詳細地說明此程序：

1.  獲得邀請之後，受邀者會透過 **Microsoft 邀請**所傳送的電子郵件收到邀請。
2.  受邀者在電子郵件中選取 [開始使用]。
3.  如果受邀者沒有 Azure AD 帳戶或 MSA，系統會提示他們建立 MSA。
4.  系統會將受邀者重新導向至 [檢閱權限] 畫面，他們可在其中檢閱邀請組織的隱私權聲明並接受條款。

## <a name="privacy-policy-agreement"></a>隱私權原則協議

在任何來賓使用者第一次登入以存取夥伴組織中的資源之後，他們會看到 [檢閱權限] 畫面。 在這裡，他們可以檢閱邀請組織的隱私權聲明。 使用者必須根據邀請組織的隱私權原則來接受其資訊的使用方式，才能繼續執行。

![顯示 [存取面板] 中使用者設定的螢幕擷取畫面](media/redemption-experience/ConsentScreen.png) 

有關身為租用戶管理員的您如何連結至貴組織隱私權聲明的詳細資訊，請參閱[做法：在 Azure Active Directory 中新增貴組織的隱私權資訊](https://aka.ms/adprivacystatement)。

## <a name="next-steps"></a>後續步驟

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- [在 Azure 入口網站中新增 Azure Active Directory B2B 共同作業使用者](add-users-administrator.md)
- [資訊工作者如何將 B2B 共同作業使用者新增到 Azure Active Directory？](add-users-information-worker.md)
- [使用 PowerShell 新增 Azure Active Directory B2B 共同作業使用者](customize-invitation-api.md#powershell)
- [以來賓使用者的身分從組織離職](leave-the-organization.md)