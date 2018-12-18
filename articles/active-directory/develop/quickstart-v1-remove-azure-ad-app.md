---
title: 從 Azure Active Directory 移除應用程式
description: 了解如何從 Azure Active Directory (Azure AD) 移除應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: lenalepa, sureshja
ms.openlocfilehash: 2cd77ee6df52940a800733209b7e384d72d98103
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962125"
---
# <a name="quickstart-remove-an-application-from-azure-active-directory"></a>快速入門：從 Azure Active Directory 移除應用程式

已向 Azure Active Directory (Azure AD) 註冊應用程式的企業開發人員和軟體即服務 (SaaS) 提供者，可能需要從其 Azure AD 租用戶移除應用程式的註冊。

在此快速入門中，您將了解如何：

* [移除您的組織所編寫的應用程式](#removing-an-application-authored-by-your-organization)
* [移除其他組織授權的多租用戶應用程式](#removing-a-multi-tenant-application-authorized-by-another-organization)

## <a name="prerequisites"></a>必要條件

若要開始，您需要有一個已有應用程式向其註冊的 Azure AD 租用戶。

* 如果您還沒有租用戶，[了解如何取得租用戶](quickstart-create-new-tenant.md)。
* 若要了解如何新增並註冊應用程式到您的租用戶，請參閱[新增應用程式到 Azure AD](quickstart-v1-integrate-apps-with-azure-ad.md)。

## <a name="removing-an-application-authored-by-your-organization"></a>移除您的組織所編寫的應用程式

您組織已註冊的應用程式會顯示在租用戶主要 [應用程式註冊] 頁面的 [我的應用程式] 篩選器底下。 這些應用程式是您透過 Azure 入口網站手動註冊的應用程式，或透過 PowerShell 或 Microsoft Graph API 以程式設計方式註冊的應用程式。 更具體來說，這些應用程式會由您租用戶中的應用程式與服務主體物件來表示。 如需這些物件的詳細資訊，請參閱[應用程式物件和服務主體物件](app-objects-and-service-principals.md)。

### <a name="to-remove-a-single-tenant-application-from-your-directory"></a>從您的目錄移除單一租用戶應用程式

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 如果您的帳戶可讓您存取多個租用戶，請在右上角選取您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
1. 在左側導覽窗格中，依序選取 [Azure Active Directory] 服務和 [應用程式註冊]，然後尋找並選取您要設定的應用程式。
    這會將您帶往應用程式的主要註冊頁面，以開啟應用程式的 [設定] 頁面。
1. 在應用程式的主要註冊頁面上，選取 [刪除]。
1. 選取 [是] 以確認要刪除應用程式。

### <a name="to-remove-a-multi-tenant-application-from-its-home-directory"></a>從主目錄移除多租用戶應用程式

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 如果您的帳戶可讓您存取多個租用戶，請在右上角選取您的帳戶，並將您的入口網站工作階段設定為想要的 Azure AD 租用戶。
1. 在左側導覽窗格中，依序選取 [Azure Active Directory] 服務和 [應用程式註冊]，然後尋找並選取您要設定的應用程式。
    這會將您帶往應用程式的主要註冊頁面，以開啟應用程式的 [設定] 頁面。
1. 在 [設定] 頁面上選擇 [屬性]，然後將 [多重租用戶] 切換開關變更為 [否]，以便先將您的應用程式變更為單一租用戶，然後選取 [儲存]。
    應用程式的服務主體物件則仍留在已同意應用程式之所有組織的租用戶中。
1. 在應用程式的主要註冊頁面上，選取 [刪除]。
1. 選取 [是] 以確認要刪除應用程式。

## <a name="removing-a-multi-tenant-application-authorized-by-another-organization"></a>移除其他組織授權的多租用戶應用程式

在租用戶的主要 [應用程式註冊] 頁面上的 [所有應用程式] 篩選器 (不包括 [我的應用程式] 註冊) 底下顯示的應用程式子集，都是多租用戶應用程式。

就技術上來說，這些多租用戶應用程式是來自其他租用戶，且已在同意程序期間註冊到您的租用戶中。 更具體來說，它們僅由您租用戶中的服務主體物件表示，而沒有對應的應用程式物件。 如需有關應用程式與服務主體物件之差異的詳細資訊，請參閱 [Azure AD 中的應用程式和服務主體物件](app-objects-and-service-principals.md)。

若要移除多租用戶應用程式對您目錄的存取權 (在授與同意之後)，公司系統管理員必須移除應用程式的服務主體。 系統管理員必須擁有全域系統管理員存取權，才能透過 Azure 入口網站移除或使用 [Azure AD PowerShell Cmdlet](http://go.microsoft.com/fwlink/?LinkId=294151) 來移除存取權。

## <a name="next-steps"></a>後續步驟

針對使用 Azure AD v1.0 端點的應用程式，深入了解這些其他相關的應用程式管理快速入門：

- [將應用程式新增到 Azure AD](quickstart-v1-integrate-apps-with-azure-ad.md)
- [在 Azure AD 中更新應用程式](quickstart-v1-update-azure-ad-app.md)
