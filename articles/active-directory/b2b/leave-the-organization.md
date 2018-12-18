---
title: 以來賓使用者的身分從組織離職 - Azure Active Directory | Microsoft Docs
description: 說明 Azure AD B2B 來賓使用者如何使用 [存取面板] 從組織離職。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: cea882bd1ba2ba12d34690fb47ec1afd6edf5c4c
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982165"
---
# <a name="leave-an-organization-as-a-guest-user"></a>以來賓使用者的身分從組織離職

Azure Active Directory (Azure AD) B2B 使用者如果不再需要組織的應用程式，或不需要與組織保持關聯，隨時皆可決定從組織離職。 使用者可以自行從組織離職，不需要聯絡系統管理員。

## <a name="leave-an-organization"></a>從組織離職

若要從組織離職，請以使用者身分登入 [存取面板](https://myapps.microsoft.com)，接著執行下列動作：

1. 如果您尚未登入打算離開的組織，請在右上角選取您的名稱，然後按一下要離開的組織。
2. 在右上角選取您的名稱。
3. 選取 [組織] 旁邊的設定圖示 (齒輪)。
 
   ![顯示 [存取面板] 中使用者設定的螢幕擷取畫面](media/leave-the-organization/UserSettings.png) 

3. 在 [組織] 下方找到要離開的組織，然後選取 [從組織離職]。

   ![顯示使用者介面中 [從組織離職] 選項的螢幕擷取畫面](media/leave-the-organization/LeaveOrg.png)

4. 系統要求您確認時，請選取 [離開]。 

## <a name="account-removal"></a>帳戶移除

使用者從組織離職後，其帳戶將在目錄中「虛刪除」。 預設情況下，使用者物件將移至 Azure AD 中的 [刪除的使用者] 區域，但在 30 天內不會永久刪除。 透過虛刪除的作法，若使用者在 30 天內提出要求，系統管理員便可還原使用者帳戶 (包括群組和權限)。

如有需要，租用戶系統管理員在 30 天內隨時可永久刪除帳戶。 作法：

1. 在 [[Azure 入口網站]](https://portal.azure.com) 中，選取 **[Azure Active Directory]**。
2. 在 [管理] 底下選取 [使用者]。
3. 選取 [刪除的使用者]。
4. 選取已刪除使用者旁邊的核取方塊，然後選取 [永久刪除]。

若永久刪除使用者，此動作將無法回復。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>後續步驟

- 如需 Azure AD B2B 的概觀，請參閱[何謂 Azure AD B2B 共同作業？](what-is-b2b.md)



