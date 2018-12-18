---
title: 存取 Azure Active Directory 並建立新租用戶的快速入門 | Microsoft Docs
description: 包含有關如何尋找 Azure Active Directory 以及如何為組織建立新的租用戶之步驟的快速入門。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: lizross
custom: it-pro
ms.openlocfilehash: 8ef68c8afcf61a1a11c341a679443071aece9812
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46362998"
---
# <a name="quickstart-access-azure-active-directory-to-create-a-new-tenant"></a>快速入門：存取 Azure Active Directory 以建立新的租用戶
您可以使用 Azure Active Directory (Azure AD) 入口網站執行所有的系統管理工作，包括為您的組織建立新的租用戶。 

在此快速入門中，您將了解如何存取 Azure 入口網站與 Azure Active Directory，並了解如何為組織建立基本的租用戶。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="prerequisites"></a>必要條件
在開始之前，您必須：

- 確定您的組織具有有效的 Azure AD 授權。

- 確定您是全域系統管理員。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站
使用全域系統管理員帳戶登入組織的 [Azure 入口網站](https://portal.azure.com/)。

![Azure 入口網站畫面](media/active-directory-access-create-new-tenant/azure-ad-portal.png)

## <a name="create-a-new-tenant-for-your-organization"></a>為您的組織建立新的租用戶
登入 Azure 入口網站之後，您可以為組織建立新的租用戶。 新的租用戶代表您的組織，並幫助您為內部與外部使用者管理 Microsoft 雲端服務的特定執行個體。

### <a name="to-create-a-new-tenant"></a>建立新的租用戶
1. 依序選取 [Azure Active Directory]、[建立資源] 與 [身分識別]，然後選取 [Azure Active Directory]。

    [建立目錄] 頁面隨即出現。

    ![Azure Active Directory 建立頁面](media/active-directory-access-create-new-tenant/azure-ad-create-new-tenant.png)

2.  在 [建立目錄] 頁面上，輸入下列資訊：
    
    - 在 [組織名稱] 方塊中輸入 _Contoso_。

    - 在 [初始網域名稱] 方塊中輸入 _Contoso_。

    - 保留 [國家或地區] 方塊中的 [美國] 選項。

3. 選取 [建立]。

您的新的租用戶是使用網域 contoso.onmicrosoft.com 建立。

## <a name="clean-up-resources"></a>清除資源
如果您不打算繼續使用此應用程式，可以使用下列步驟刪除租用戶：

- 選取 [Azure Active Directory]，然後在 [Contoso - 概觀] 頁面上，選取 [刪除目錄]。

    這將刪除租用戶與其相關聯的資訊。

    ![[建立目錄] 頁面](media/active-directory-access-create-new-tenant/azure-ad-delete-new-tenant.png)

## <a name="next-steps"></a>後續步驟
- 變更或新增額外的網域名稱，請參閱[如何將自訂網域名稱新增至 Azure Active Directory](add-custom-domain.md)

- 新增使用者，請參閱[新增或刪除新的使用者](add-users-azure-active-directory.md)

- 新增群組與成員，請參閱[建立基本群組並新增成員](active-directory-groups-create-azure-portal.md)

- 深入了解[使用 Privileged Identity Management 的角色型存取](../../role-based-access-control/pim-azure-resource.md)與[條件式存取](../../role-based-access-control/conditional-access-azure-management.md)，以協助您管理貴組織的應用程式與資源的存取。
