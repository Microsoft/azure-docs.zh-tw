---
title: 從 Azure Active Directory 的使用者體驗中隱藏應用程式 | Microsoft Docs
description: 如何從 Azure Active Directory 存取面板或 Office 365 啟動器的使用者體驗中隱藏應用程式。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/04/2018
ms.author: barbkess
ms.reviewer: asteen
ms.custom: it-pro
ms.openlocfilehash: 55f80396df4cbfe7d0a16a6a5066b68aadc0bdd3
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368916"
---
# <a name="hide-an-application-from-users-experience-in-azure-active-directory"></a>從 Azure Active Directory 的使用者體驗中隱藏應用程式

如果您不想要在使用者的存取面板或 Office 365 啟動器上顯示某個應用程式，有幾個選項可用來隱藏此應用程式磚。  下列兩個選項可用來從使用者的應用程式啟動器中隱藏應用程式。

- 從使用者存取面板和 Office 365 應用程式啟動器中隱藏第三方應用程式
- 從使用者存取面板中隱藏所有 Office 365 應用程式

隱藏應用程式之後，使用者仍然有應用程式的權限，但是在應用程式啟動器中看不到這些應用程式。 您必須具備適當的權限，才能管理企業應用程式，而且必須是目錄的全域管理員。


## <a name="hiding-an-application-from-users-end-user-experiences"></a>從使用者的使用者體驗中隱藏應用程式
根據您的情況，您可以使用下列步驟從存取面板中隱藏應用程式。

### <a name="how-do-i-hide-a-third-party-app-from-users-access-panel-and-o365-app-launchers"></a>如何從使用者的存取面板和 O365 應用程式啟動器中隱藏第三方應用程式？
請使用下列步驟，從使用者的存取面板和 Office 365 應用程式啟動器中隱藏應用程式。

1.  使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2.  選取 [所有服務]，在文字方塊中輸入 **Azure Active Directory**，然後選取 **Enter** 鍵。
3.  在 [Azure Active Directory - \*directoryname\*] 畫面上 (也就是您所管理目錄的 Azure AD 畫面上)，選取 [企業應用程式]。
![企業應用程式](./media/hide-application-from-user-portal/app1.png)
4.  在 [企業應用程式] 畫面上，選取 [所有應用程式]。 您會看到一份您可以管理的應用程式清單。
5.  在 [企業應用程式 - 所有應用程式] 畫面上，選取一個應用程式。</br>
![企業應用程式](./media/hide-application-from-user-portal/app2.png)
6.  在 ***appname*** 畫面 (亦即標題中含有所選應用程式名稱的畫面) 上，選取 [屬性]。
7.  在 [*appname* - 屬性] 畫面上，針對 [是否要向使用者顯示?] 選取 [是]。
![企業應用程式](./media/hide-application-from-user-portal/app3.png)
8.  選取 [儲存]  命令。

### <a name="how-do-i-hide-office-365-applications-from-users-access-panel"></a>如何從使用者存取面板中隱藏 Office 365 應用程式？

您可以使用下列步驟從存取面板中隱藏所有 Office 365 應用程式。 這些應用程式仍會顯示在 Office 365 入口網站中。

1.  使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2.  選取 [所有服務]，在文字方塊中輸入 **Azure Active Directory**，然後選取 **Enter** 鍵。
3.  在 [Azure Active Directory - \*directoryname\*] 畫面上 (也就是您所管理目錄的 Azure AD 畫面上)，選取 [使用者設定]。
4.  在 [使用者設定] 畫面上的 [企業應用程式] 下，針對 [使用者只能在 Office 365 入口網站中看見 Office 365 應用程式] 選取 [是]。

![企業應用程式](./media/hide-application-from-user-portal/apps4.png)

## <a name="next-steps"></a>後續步驟
* [查看我的所有群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [將使用者或群組指派給企業應用程式](assign-user-or-group-access-portal.md)
* [從企業應用程式中移除使用者或群組指派](remove-user-or-group-access-portal.md)
* [變更企業應用程式的名稱或標誌](change-name-or-logo-portal.md)

