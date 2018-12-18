---
title: 啟用 Azure Multi-Factor Authentication 試驗
description: 在本教學課程中，您將為試驗使用者群組啟用 Azure AD Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 098973e2ece3477ec87b154c0304c4ca7e0246d1
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39163265"
---
# <a name="tutorial-complete-an-azure-multi-factor-authentication-pilot-roll-out"></a>教學課程：完成 Azure Multi-Factor Authentication 試驗推出

在本教學課程中，您將在登入 Azure 入口網站時逐步設定啟用 Azure Multi-Factor Authentication (Azure MFA) 的條件式存取原則。 此原則會部署至特定的試驗使用者群組，並進行測試。 相較於傳統施行的方法，使用條件式存取的 Azure MFA 部署可為組織和系統管理員提供極大的彈性。

> [!div class="checklist"]
> * 啟用 Azure Multi-Factor Authentication
> * 測試 Azure Multi-Factor Authentication

## <a name="prerequisites"></a>必要條件

* 可運作且至少已啟用試用版授權的 Azure AD 租用戶。
* 具有全域系統管理員權限的帳戶。
* 您已知其測試用密碼的非系統管理員測試使用者；如果您需要建立使用者，請參閱[快速入門：將新的使用者新增至 Azure Active Directory](../add-users-azure-active-directory.md) 一文。
* 該名非系統管理員使用者所屬的測試用試驗群組；如果您需要建立群組，請參閱[在 Azure Active Directory 中建立群組和新增使用者](../active-directory-groups-create-azure-portal.md)一文。

## <a name="enable-azure-multi-factor-authentication"></a>啟用 Azure Multi-Factor Authentication

1. 使用全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 [Azure Active Directory]、[條件式存取]
1. 選取 [新增原則]
1. 將您的原則命名為 **MFA 試驗**
1. 在 [使用者和群組] 下方，選取 [選取使用者和群組] 選項按鈕
    * 選取您在本文的先決條件一節中建立的試驗群組
    * 按一下 [完成]
1. 在 [雲端應用程式] 下方，選取 [選取應用程式] 選項按鈕
    * Azure 入口網站的雲端應用程式是 **Microsoft Azure 管理**
    * 按一下 [選取]
    * 按一下 [完成]
1. 略過 [條件] 區段
1. 在 [授與] 下方，確定已選取 [授與存取權] 選項按鈕
    * 核取 [需要多重要素驗證] 的方塊
    * 按一下 [選取]
1. 略過 [工作階段] 區段
1. 將 [啟用原則] 切換為 [開啟]
1. 按一下 [建立] 

## <a name="test-azure-multi-factor-authentication"></a>測試 Azure Multi-Factor Authentication

若要證明您的條件式存取原則有效用，您必須對資源進行應該不需要 MFA 的測試登入，然後再對 Azure 入口網站進行需要 MFA 的測試登入。

1. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com)。
   * 使用在本文的先決條件一節中建立的測試使用者進行登入，並留意系統應該不會要求您完成 MFA。
   * 關閉瀏覽器視窗。
2. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://portal.azure.com](https://portal.azure.com)。
   * 使用在本文的先決條件一節中建立的測試使用者進行登入，並留意您現在應該必須註冊並使用 Azure Multi-Factor Authentication。
   * 關閉瀏覽器視窗。

## <a name="clean-up-resources"></a>清除資源

如果您決定不再使用您在本教學課程中設定的功能，請進行下列變更。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 [Azure Active Directory]、[條件式存取]。
1. 選取您先前建立的條件式存取原則。
1. 按一下 [刪除] 。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已啟用 Azure Multi-Factor Authentication。 請繼續進行下一個教學課程，以了解如何將 Azure Identity Protection 整合到自助式密碼重設和 Multi-Factor Authentication 體驗中。

> [!div class="nextstepaction"]
> [在登入時評估風險](tutorial-risk-based-sspr-mfa.md)