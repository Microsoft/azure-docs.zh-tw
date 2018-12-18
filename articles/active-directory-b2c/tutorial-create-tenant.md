---
title: 教學課程 - 建立 Azure Active Directory B2C 租用戶 | Microsoft Docs
description: 了解如何使用 Azure 入口網站建立 Azure Active Directory B2C 租用戶，以準備註冊應用程式。
services: B2C
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/26/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 7571e5f4d95320ab92fa3b69b0ea1f05ff9c771f
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408397"
---
# <a name="tutorial-create-an-azure-active-directory-b2c-tenant"></a>教學課程：建立 Azure Active Directory B2C 租用戶

應用程式必須先在您管理的租用戶中完成註冊，才可以與 Azure Active Directory (Azure AD) B2C 互動。

在此文章中，您將了解：

> [!div class="checklist"]
> * 建立 Azure AD B2C 租用戶
> * 將您的租用戶連結至您的訂用帳戶

您會在下一個教學課程中了解如何註冊應用程式。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-an-azure-ad-b2c-tenant"></a>建立 Azure AD B2C 租用戶

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含它的目錄，以確定您使用的是包含您訂用帳戶的目錄。 這個目錄與將包含您 Azure AD B2C 租用戶的目錄不同。

    ![切換至訂用帳戶目錄](./media/tutorial-create-tenant/switch-directory-subscription.png)

3. 選擇 Azure 入口網站左上角的 [建立資源]。
4. 搜尋並選取 [Active Directory B2C]，然後按一下 [建立]。
5. 選擇 [建立新的 Azure AD B2C 租用戶]、輸入組織名稱和初始網域名稱 (用於租用戶名稱中)、選取國家/地區 (之後就無法變更)，然後按一下 [建立]。

    ![建立租用戶](./media/tutorial-create-tenant/create-tenant.png)

    在此範例中，租用戶名稱是 contoso0926Tenant.onmicrosoft.com

6. 在 [建立新的 B2C 租用戶或連結至現有租用戶] 頁面上，選擇 [連結現有 Azure AD B2C 租用戶至我的 Azure 訂用帳戶]、選取您建立的租用戶、選取您的訂用帳戶、按一下 [新建] 並輸入將包含租用戶的資源群組名稱、選取位置，然後按一下 [建立]。
7. 若要開始使用您的新租用戶，按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含它的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。

    ![切換至租用戶目錄](./media/tutorial-create-tenant/switch-directories.png)

## <a name="next-steps"></a>後續步驟

在此文章中，您已了解如何：

> [!div class="checklist"]
> * 建立 Azure AD B2C 租用戶
> * 將您的租用戶連結至您的訂用帳戶

> [!div class="nextstepaction"]
> [讓 Web 應用程式使用帳戶進行驗證](active-directory-b2c-tutorials-web-app.md)