---
title: Azure Active Directory B2C 中的使用者帳戶概觀
description: 瞭解 Azure Active Directory B2C 中可使用的使用者帳戶類型。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/05/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: c3e6c1d6bfa83ef238ef38b25b189510cf142a38
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98661080"
---
# <a name="overview-of-user-accounts-in-azure-active-directory-b2c"></a>Azure Active Directory B2C 中的使用者帳戶概觀

在 Azure Active Directory B2C (Azure AD B2C) 中，可以建立數種類型的帳戶。 在可使用的使用者帳戶類型中，Azure Active Directory、Active Directory B2B 和 Active Directory B2C 共用。

可用的帳戶類型如下：

- **公司帳戶** - 公司帳戶可存取租用戶中的資源，並可在具有系統管理員角色後管理租用戶。
- **來賓帳戶** - 來賓帳戶只能是 Microsoft 帳戶或 Azure Active Directory 使用者 (後者必須是可用來存取應用程式或管理租用戶的使用者)。
- 取用 **者帳戶**-您已向 Azure AD B2C 註冊之應用程式的使用者會使用取用者帳戶。 您可以建立取用者帳戶：
  - 使用者在 Azure AD B2C 應用程式中進行註冊使用者流程
  - 使用 Microsoft Graph API
  - 使用 Azure 入口網站

## <a name="work-account"></a>公司帳戶

所有以 Azure AD 為基礎的租用戶，會有相同的公司帳戶建立方式。 若要建立公司帳戶，您可以使用[快速入門：將使用者新增至 Azure Active Directory](../active-directory/fundamentals/add-users-azure-active-directory.md) 中的資訊。 在 Azure 入口網站中使用 [新增使用者] 選項即可建立公司帳戶。

在新增公司帳戶時，您需要考慮下列組態設定：

- **名稱** 和 **使用者名稱** - **名稱** 屬性包含使用者的姓名。 **使用者名稱** 是使用者為了登入所輸入的識別碼。 使用者名稱包括完整網域。 使用者名稱的網域名稱部分必須是初始預設網域名稱「your-domain.onmicrosoft.com」，或是已驗證的非同盟[自訂網域](../active-directory/fundamentals/add-custom-domain.md)名稱，例如 contoso.com。
- **設定檔** - 帳戶會使用使用者資料設定檔來進行設定。 您可以輸入名字、姓氏、職稱和部門名稱。 您可以在建立帳戶之後編輯設定檔。
- **群組** -使用群組來執行管理工作，例如將授權或許可權一次指派給許多使用者或裝置。 您可以將新的帳戶放到租用戶中的現有[群組](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。
- **目錄角色** - 您需要指定使用者帳戶所具有的租用戶資源存取權層級。 可用的權限層級如下：

    - **使用者** - 使用者可存取受指派的資源，但無法管理多數租用戶資源。
    - **全域管理員** - 全域管理員可完整控制所有租用戶資源。
    - **受限的管理員** - 為使用者選取一或多個系統管理角色。 如需可選角色的詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](../active-directory/roles/permissions-reference.md)。

### <a name="create-a-work-account"></a>建立公司帳戶

您可以使用下列資訊來建立新的公司帳戶：

- [Azure 入口網站](../active-directory/fundamentals/add-users-azure-active-directory.md)
- [Microsoft Graph](/graph/api/user-post-users)

### <a name="update-a-user-profile"></a>更新使用者設定檔

您可以使用下列資訊來更新使用者的設定檔：

- [Azure 入口網站](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

### <a name="reset-a-password-for-a-user"></a>重設使用者的密碼

您可以使用下列資訊來重設使用者的密碼：

- [Azure 入口網站](../active-directory/fundamentals/active-directory-users-reset-password-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

## <a name="guest-user"></a>來賓使用者

您可以邀請外部使用者成為租用戶的來賓使用者。 將來賓使用者邀請到 Azure AD B2C 租用戶的典型案例是為了分擔管理責任。 如需使用來賓帳戶的範例，請參閱 [Azure Active Directory B2B 共同作業使用者的屬性](../active-directory/external-identities/user-properties.md)。

將來賓使用者邀請到租用戶時，您必須提供收件者的電子郵件地址和描述邀請詳情的訊息。 邀請連結會將使用者帶到同意頁面。 如果收件匣並未連結至電子郵件地址，使用者可以使用受邀認證移至 Microsoft 頁面以瀏覽至同意頁面。 接著，系統會強制使用者兌換邀請，方式和在電子郵件中按一下連結時相同。 例如：`https://myapps.microsoft.com/B2CTENANTNAME`。

您也可以使用 [Microsoft Graph API](/graph/api/invitation-post?view=graph-rest-beta) 邀請來賓使用者。

## <a name="consumer-user"></a>取用者使用者

取用者使用者可以登入 Azure AD B2C 所保護的應用程式，但無法存取 Azure 資源，例如 Azure 入口網站。 取用者使用者可以使用本機帳戶或同盟帳戶，例如 Facebook 或 Twitter。 取用者帳戶的建立方式是使用 [註冊或登入使用者流程](user-flow-overview.md)、使用 Microsoft Graph API，或使用 Azure 入口網站。

您可以指定建立取用者使用者帳戶時所收集的資料。 如需詳細資訊，請參閱 [新增使用者屬性和自訂使用者輸入](configure-user-input.md)。

如需管理取用者帳戶的詳細資訊，請參閱 [使用 Microsoft Graph 管理 Azure AD B2C 的使用者帳戶](manage-user-accounts-graph-api.md)。

### <a name="migrate-consumer-user-accounts"></a>遷移取用者使用者帳戶

您可能有需要將任何識別提供者的現有取用者使用者帳戶遷移至 Azure AD B2C。 如需詳細資訊，請參閱 [將使用者遷移至 Azure AD B2C](user-migration.md)。
