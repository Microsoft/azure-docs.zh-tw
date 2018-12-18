---
title: 在 Azure Active Directory B2C 中移轉具社交身分識別的使用者 | Microsoft Docs
description: 討論使用圖形 API 將具社交身分識別的使用者移轉至 Azure AD B2C 的核心概念。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/03/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: b9378face28b4d053dcd5f01b8f87126457cf339
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445138"
---
# <a name="azure-active-directory-b2c-migrate-users-with-social-identities"></a>Azure Active Directory B2C：遷移具社交識別的使用者
當您計劃將識別提供者遷移到 Azure AD B2C 時，您可能也需要遷移具社交識別的使用者。 本文會說明如何將現有的社交識別帳戶 (例如：Facebook、LinkedIn、Microsoft 與 Google 帳戶) 遷移至 Azure AD B2C。 本文也適用於同盟身分識別，不過這些移轉較不常見。

## <a name="prerequisites"></a>先決條件
本文延續使用者移轉文章，內容著重在社交識別移轉。 請在開始前先閱讀[使用者移轉](active-directory-b2c-user-migration.md)。

## <a name="social-identities-migration-introduction"></a>社交識別移轉簡介

* 在 Azure AD B2C 中，**本機帳戶**登入名稱 (使用者名稱或電子郵件地址) 會儲存在使用者記錄的 `signInNames` 集合中。 `signInNames` 包含一或多個會指定使用者登入名稱的 `signInName` 記錄。 每個登入名稱都必須是租用戶中的唯一名稱。

* **社交帳戶**的身分識別會儲存在 `userIdentities` 集合中。 此項目會指定 `issuer` (識別提供者名稱，例如 facebook.com) 和 `issuerUserId` (這是簽發者的唯一使用者識別碼)。 `userIdentities` 屬性包含一或多個 UserIdentity 記錄，該記錄會指定來自社交識別提供者的社交帳戶類型和唯一使用者識別碼。

* **結合本機帳戶與社交識別**。 如前所述，本機帳戶登入名稱和社交帳戶身分識別會儲存在不同的屬性中。 `signInNames` 用於本機帳戶，`userIdentities` 則用於社交帳戶。 單一 Azure AD B2C 帳戶可以是純本機帳戶、純社交帳戶，或在一個使用者記錄中結合本機帳戶與社交識別。 此行為可讓您管理單一帳戶，而使用者又可使用本機帳戶認證或使用社交識別來登入。

* `UserIdentity` 類型 - 包含 Azure AD B2C 租用戶中社交帳戶使用者的身分識別相關資訊：
    * `issuer` 以字串表示、發出了使用者識別碼的識別提供者，例如 facebook.com。
    * `issuerUserId` 社交識別提供者所使用的唯一使用者識別碼 (採用 base64 格式)。

    ```JSON
    "userIdentities": [{
            "issuer": "Facebook.com",
            "issuerUserId": "MTIzNDU2Nzg5MA=="
        }
    ]
    ```

* 根據識別提供者而定，**社交使用者識別碼**是每個應用程式指定使用者或開發帳戶的唯一值。 請使用社交提供者先前所指派的相同應用程式識別碼設定 Azure AD B2C 原則。 或在相同開發帳戶內的另一個應用程式。

## <a name="use-graph-api-to-migrate-users"></a>使用圖形 API 來遷移使用者
您可以透過[圖形 API](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-devquickstarts-graph-dotnet) 建立 Azure AD B2C 使用者帳戶。 若要與圖形 API 通訊，您必須先獲得含有系統管理權限的服務帳戶。 在 Azure AD 中，您要註冊應用程式並向 Azure AD 驗證。 應用程式的認證包括：應用程式識別碼和應用程式祕密。 應用程式會以本身 (而非使用者的身分) 呼叫圖形 API。 請依照[使用者移轉](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-user-migration#step-1-use-graph-api-to-migrate-users)一文中步驟 1 的指示來進行。

## <a name="required-properties"></a>必要屬性
下列清單會顯示您在建立使用者時的必要屬性。
* **accountEnabled** - true
* **displayName** - 要在使用者通訊錄中顯示的名稱。
* **passwordProfile** - 使用者的密碼設定檔。 

> [!NOTE]
> 僅針對社交帳戶 (不含本機帳戶認證)，您仍然必須指定密碼。 Azure AD B2C 會忽略您為社交帳戶指定的密碼。

* **userPrincipalName** - 使用者主體名稱 (someuser@contoso.com)。 使用者主體名稱必須包含租用戶的其中一個已驗證網域。 若要指定 UPN，請產生新的 GUID 值，並與 `@` 和您的租用戶名稱串連。
* **mailNickname** - 使用者的郵件別名。 這個值可與您用於 userPrincipalName 的識別碼相同。 
* **signInNames** - 一或多個指定使用者登入名稱的 SignInName 記錄。 在公司/租用戶中，每個登入名稱必須是唯一的。 僅針對社交帳戶，這個屬性可以保持空白。
* **userIdentities** - 一或多個 UserIdentity 記錄，該記錄會指定來自社交識別提供者的社交帳戶類型和唯一使用者識別碼。
* [選用] **otherMails** - 僅針對社交帳戶，此為使用者的電子郵件地址 

如需詳細資訊，請參閱：[圖形 API 參考](https://msdn.microsoft.com/library/azure/ad/graph/api/users-operations#CreateLocalAccountUser)

## <a name="migrate-social-account-only"></a>遷移社交帳戶 (僅限此帳戶)
若要僅建立不含本機帳戶認證的社交帳戶。 請將 HTTPS POST 要求傳送至圖形 API。 要求本文包含要建立之社交帳戶使用者的屬性。 您最少必須指定必要的屬性。 


**POST**  https://graph.windows.net/tenant-name.onmicrosoft.com/users

提交下列表單資料： 

```JSON
{
    "objectId": null,
    "accountEnabled": true,
    "mailNickname": "c8c3d3b8-60cf-4c76-9aa7-eb3235b190c8",
    "signInNames": [],
    "creationType": null,
    "displayName": "Sara Bell",
    "givenName": "Sara",
    "surname": "Bell",
    "passwordProfile": {
        "password": "Test1234",
        "forceChangePasswordNextLogin": false
    },
    "passwordPolicies": null,
    "userIdentities": [{
            "issuer": "Facebook.com",
            "issuerUserId": "MTIzNDU2Nzg5MA=="
        }
    ],
    "otherMails": ["sara@live.com"],
    "userPrincipalName": "c8c3d3b8-60cf-4c76-9aa7-eb3235b190c8@tenant-name.onmicrosoft.com"
}
```
## <a name="migrate-social-account-with-local-account"></a>遷移具本機帳戶的社交帳戶
若要建立具社交識別的合併本機帳戶。 請將 HTTPS POST 要求傳送至圖形 API。 要求本文包含要建立之社交帳戶使用者的屬性。 您最少必須指定必要的屬性。 

**POST**  https://graph.windows.net/tenant-name.onmicrosoft.com/users

提交下列表單資料： 

```JSON
{
    "objectId": null,
    "accountEnabled": true,
    "mailNickname": "5164db16-3eee-4629-bfda-dcc3326790e9",
    "signInNames": [{
            "type": "emailAddress",
            "value": "david@contoso.com"
        }
    ],
    "creationType": "LocalAccount",
    "displayName": "David Hor",
    "givenName": "David",
    "surname": "Hor",
    "passwordProfile": {
        "password": "1234567",
        "forceChangePasswordNextLogin": false
    },
    "passwordPolicies": "DisablePasswordExpiration,DisableStrongPassword",
    "userIdentities": [{
            "issuer": "contoso.com",
            "issuerUserId": "ZGF2aWRAY29udG9zby5jb20="
        }
    ],
    "otherMails": [],
    "userPrincipalName": "5164db16-3eee-4629-bfda-dcc3326790e9@tenant-name.onmicrosoft.com"
}
```

## <a name="frequently-asked-questions"></a>常見問題集
### <a name="how-can-i-know-the-issuer-name"></a>我要如何知道簽發者名稱？
簽發者名稱 (或識別提供者名稱) 會設定於您的原則中。 如果您不知道要在 `issuer` 中指定的值，請遵循此程序：
1. 使用其中一個社交帳戶進行登入
2. 從 JWT 權杖複製 `sub` 值。 `sub` 通常會包含 Azure AD B2C 中的使用者物件識別碼。 或從 Azure 入口網站開啟使用者的屬性，並複製物件識別碼。
3. 開啟 [Azure AD Graph 總管](https://graphexplorer.azurewebsites.net)
4. 使用您的系統管理員來登入。 N
5. 執行下列 GET 要求。 將 userObjectId 取代為您所複製的使用者識別碼。 **GET** https://graph.windows.net/tenant-name.onmicrosoft.com/users/userObjectId
6. 在 Azure AD B2C 所傳回的 JSON 內找到 `userIdentities` 元素。
7. [選用] 您也可以將 `issuerUserId` 值解碼。

> [!NOTE]
> 使用 B2C 租用戶本機的 B2C 租用戶系統管理員帳戶。 帳戶名稱語法是 admin@tenant-name.onmicrosoft.com。

### <a name="is-it-possible-to-add-social-identity-to-an-existing-local-account"></a>是否能將社交識別新增至現有的本機帳戶？
是。 建立本機帳戶後，您就可以新增社交識別。 執行 HTTPS PATCH 要求。 將 userObjectId 取代為您要更新的使用者識別碼。 

**PATCH** https://graph.windows.net/tenant-name.onmicrosoft.com/users/userObjectId

提交下列表單資料： 

```JSON
{
    "userIdentities": [
        {
            "issuer": "Facebook.com",
            "issuerUserId": "MTIzNDU2Nzg5MA=="
        }
    ]
}
```

### <a name="is-it-possible-to-add-multiple-social-identities"></a>是否可以新增多個社交識別？
是。 您可以為單一 Azure AD B2C 帳戶新增多個社交識別。 執行 HTTPS PATCH 要求。 將 userObjectId 取代為使用者識別碼。 

**PATCH** https://graph.windows.net/tenant-name.onmicrosoft.com/users/userObjectId

提交下列表單資料： 

```JSON
{
    "userIdentities": [
        {
            "issuer": "google.com",
            "issuerUserId": "MjQzMjE2NTc4NTQ="
        },
        {
            "issuer": "facebook.com",
            "issuerUserId": "MTIzNDU2Nzg5MA=="
        }
    ]
}
```

## <a name="optional-user-migration-application-sample"></a>[選用] 使用者移轉應用程式範例
[下載及執行應用程式範例 V2](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-user-migration)。 應用程式範例 V2 會使用包含虛擬使用者資料的 JSON 檔案，其中包括本機帳戶、社交帳戶，以及單一帳戶中的本機和社交識別。  若要編輯 JSON 檔案，請開啟 `AADB2C.UserMigration.sln` Visual Studio 方案。 在 `AADB2C.UserMigration` 專案中，開啟 `UsersData.json` 檔案。 此檔案包含使用者實體的清單。 每個使用者實體都有下列屬性：
* **signInName** - 僅針對本機帳戶，此為用來登入的電子郵件地址
* **displayName** - 使用者的顯示名稱
* **firstName** - 使用者的名字
* **lastName** - 使用者的姓氏
* **password** 僅針對本機帳戶，此為使用者的密碼 (可以空白)
* **issuer** - 僅針對社交帳戶，此為識別提供者名稱
* **issuerUserId** - 僅針對社交帳戶，此為社交識別提供者所使用的唯一使用者識別碼。 此值應採用純文字格式。 應用程式範例會將此值編碼為 base64。
* **email** 僅針對社交帳戶 (未合併)，此為使用者的電子郵件地址

```JSON
{
  "userType": "emailAddress",
  "Users": [
    {
      // Local account only
      "signInName": "James@contoso.com",
      "displayName": "James Martin",
      "firstName": "James",
      "lastName": "Martin",
      "password": "Pass!w0rd"
    },
    {
      // Social account only
      "issuer": "Facebook.com",
      "issuerUserId": "1234567890",
      "email": "sara@contoso.com",
      "displayName": "Sara Bell",
      "firstName": "Sara",
      "lastName": "Bell"
    },
    {
      // Combine local account with social identity
      "signInName": "david@contoso.com",
      "issuer": "Facebook.com",
      "issuerUserId": "0987654321",
      "displayName": "David Hor",
      "firstName": "David",
      "lastName": "Hor",
      "password": "Pass!w0rd"
    }
  ]
}
```

> [!NOTE]
> 如果您沒有使用您的資料更新此範例中的 UsersData.json 檔案，您可以使用本機帳戶認證範例來登入，但不要使用社交帳戶範例。 若要遷移您的社交帳戶，請提供實際資料。

如需如何使用應用程式範例的詳細資訊，請參閱 [Azure Active Directory B2C：使用者移轉](active-directory-b2c-user-migration.md)
