---
title: 在 Azure Active Directory 中停用企業應用程式的使用者登入 | Microsoft Docs
description: 如何在 Azure Active Directory 中停用企業應用程式，讓任何使用者都無法登入它
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
ms.date: 08/28/2017
ms.author: barbkess
ms.reviewer: asteen
ms.custom: it-pro
ms.openlocfilehash: 39e926a392cbd87eff23e25d9708792ec7c40a2c
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368706"
---
# <a name="disable-user-sign-ins-for-an-enterprise-app-in-azure-active-directory"></a>在 Azure Active Directory 中停用企業應用程式的使用者登入
在 Azure Active Directory (Azure AD) 中，您可以輕鬆停用企業應用程式，讓任何使用者都無法登入它。 您必須具備適當的權限，才能管理企業應用程式，而且必須是目錄的全域管理員。

## <a name="how-do-i-disable-user-sign-ins"></a>如何停用使用者登入？
1. 使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2. 選取 [所有服務]，在文字方塊中輸入 **Azure Active Directory**，然後選取 **Enter** 鍵。
3. 在 [Azure Active Directory -  directoryname] 窗格 (也就是您所管理之目錄的 Azure AD 窗格) 上，選取 [企業應用程式]。

    ![開啟企業應用程式](./media/disable-user-sign-in-portal/open-enterprise-apps.png)
4. 在 [企業應用程式] 窗格上，選取 [所有應用程式]。 您會看到一份您可以管理的應用程式清單。
5. 在 [企業應用程式 - 所有應用程式]  窗格上，選取一個應用程式。
6. 在 [appname] 窗格 (亦即標題中含有所選應用程式之名稱的窗格) 上，選取 [屬性]。

    ![選取所有應用程式命令](./media/disable-user-sign-in-portal/select-app.png)
7. 在 [appname - 屬性] 窗格上，針對 [為使用者啟用登入?] 選取 [否]。
8. 選取 [儲存]  命令。

## <a name="next-steps"></a>後續步驟
* [查看我的所有群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [將使用者或群組指派給企業應用程式](assign-user-or-group-access-portal.md)
* [從企業應用程式中移除使用者或群組指派](remove-user-or-group-access-portal.md)
* [變更企業應用程式的名稱或標誌](change-name-or-logo-portal.md)
