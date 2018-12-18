---
title: Azure Active Directory B2C 中的使用者帳戶概觀 | Microsoft Docs
description: 了解 Azure Active Directory B2C 中的使用者帳戶。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 2354bf02b47c9fbc74dbc0dab07e30ca321279cf
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43670096"
---
# <a name="overview-of-user-accounts-in-azure-active-directory-b2c"></a>Azure Active Directory B2C 中的使用者帳戶概觀

在 Azure Active Directory (Azure AD) B2C 中，您可以使用不同類型的帳戶。 Azure Active Directory、Azure Active Directory B2B 和 Azure Active Directory B2C 會共用可用的使用者帳戶類型。

可用的帳戶類型如下：

- **公司帳戶** - 公司帳戶可存取租用戶中的資源，並可在具有系統管理員角色後管理租用戶。
- **來賓帳戶** - 來賓帳戶只能是 Microsoft 帳戶或 Azure Active Directory 使用者 (後者必須是可用來存取應用程式或管理租用戶的使用者)。 
- **取用者帳戶** - 取用者帳戶的建立方式為在 Azure AD B2C 應用程式中進行註冊原則程序，亦可使用 Azure AD 圖形 API 來建立，此帳戶類型可供已向 Azure AD B2C 註冊的應用程式使用者使用。 

## <a name="work-account"></a>公司帳戶

所有以 Azure AD 為基礎的租用戶，會有相同的公司帳戶建立方式。 若要建立公司帳戶，您可以使用[快速入門：將使用者新增至 Azure Active Directory](../active-directory/fundamentals/add-users-azure-active-directory.md) 中的資訊。 在 Azure 入口網站中使用 [新增使用者] 選項即可建立公司帳戶。

在新增公司帳戶時，您需要考慮下列組態設定：

- **名稱**和**使用者名稱** - **名稱**屬性包含使用者的姓名。 **使用者名稱**是使用者為了登入所輸入的識別碼。 使用者名稱包括完整網域。 使用者名稱的網域名稱部分必須是初始預設網域名稱「your-domain.onmicrosoft.com」，或是已驗證的非同盟[自訂網域](../active-directory/fundamentals/add-custom-domain.md)名稱，例如 contoso.com。
- **設定檔** - 帳戶會使用使用者資料設定檔來進行設定。 您可以輸入名字、姓氏、職稱和部門名稱。 您可以在建立帳戶之後編輯設定檔。
- **群組** - 您可以使用群組來執行管理工作，例如將授權或權限一次指派給數個使用者或裝置。 您可以將新的帳戶放到租用戶中的現有[群組](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。 
- **目錄角色** - 您需要指定使用者帳戶所具有的租用戶資源存取權層級。 可用的權限層級如下：

    - **使用者** - 使用者可存取受指派的資源，但無法管理多數租用戶資源。
    - **全域管理員** - 全域管理員可完整控制所有租用戶資源。
    - **受限的管理員** - 為使用者選取一或多個系統管理角色。 如需可選角色的詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](../active-directory/users-groups-roles/directory-assign-admin-roles.md)。 

### <a name="create-a-work-account"></a>建立公司帳戶

您可以使用下列資訊來建立新的公司帳戶：

- [Azure 入口網站](../active-directory/fundamentals/add-users-azure-active-directory.md)
- [Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_post_users)

### <a name="update-a-user-profile"></a>更新使用者設定檔

您可以使用下列資訊來更新使用者的設定檔：

- [Azure 入口網站](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)
- [Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_update)

### <a name="reset-a-password-for-a-user"></a>重設使用者的密碼

您可以使用下列資訊來重設使用者的密碼： 

- [Azure 入口網站](../active-directory/fundamentals/active-directory-users-reset-password-azure-portal.md)
- [Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_update)

## <a name="guest-user"></a>來賓使用者

您可以邀請外部使用者成為租用戶的來賓使用者。 將來賓使用者邀請到 Azure AD B2C 租用戶的典型案例是為了分擔管理責任。 如需使用來賓帳戶的範例，請參閱 [Azure Active Directory B2B 共同作業使用者的屬性](../active-directory/b2b/user-properties.md)。

將來賓使用者邀請到租用戶時，您必須提供收件者的電子郵件地址和描述邀請詳情的訊息。 邀請連結會將使用者帶往同意頁面，該頁面已選取 [開始使用] 按鈕，並已接受權限檢閱。 如果收件匣並未連結至電子郵件地址，使用者可以使用受邀認證移至 Microsoft 頁面以瀏覽至同意頁面。 接著，系統會強制使用者兌換邀請，方式和在電子郵件中按一下連結時相同。 例如： `https://myapps.microsoft.com/B2CTENANTNAME` 。

您也可以使用 [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/api/invitation_post) 邀請來賓使用者。

## <a name="consumer-user"></a>取用者使用者

取用者使用者可以登入 Azure AD B2C 所保護的應用程式，但無法存取 Azure 資源，例如 Azure 入口網站。  取用者使用者可以使用本機帳戶或同盟帳戶，例如 Facebook 或 Twitter。 您可以使用[註冊或登入原則](../active-directory-b2c/active-directory-b2c-reference-policies.md)來建立取用者帳戶。

您可以指定使用自訂使用者屬性建立取用者使用者帳戶時所要收集的資料。 如需詳細資訊，請參閱[在 Azure Active Directory B2C 中定義自訂屬性](../active-directory-b2c/active-directory-b2c-reference-custom-attr.md)。

您可以使用[使用 Azure AD 圖形 API](active-directory-b2c-devquickstarts-graph-dotnet.md) 中**建立取用者使用者帳戶**一節的資訊，來建立 Azure AD B2C 取用者帳戶。 您也可以使用同一篇文章中**更新取用者使用者帳戶**一節的資訊，來管理帳戶的屬性。

### <a name="migrate-consumer-user-accounts"></a>遷移取用者使用者帳戶

您可能有需要將任何識別提供者的現有取用者使用者帳戶遷移至 Azure AD B2C。 如需詳細資訊，請參閱[使用者移轉](active-directory-b2c-user-migration.md)或[遷移具有社交身分識別的使用者](active-directory-b2c-social-migration.md)。