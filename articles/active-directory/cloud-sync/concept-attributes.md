---
title: 瞭解 Azure AD 架構和自訂表格達式
description: 本文描述 Azure AD 架構、布建代理程式流程的屬性，以及自訂表格達式。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/18/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 750bc2f4f535238dbb757afc8b615fa71063d74d
ms.sourcegitcommit: 8a74ab1beba4522367aef8cb39c92c1147d5ec13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98613439"
---
# <a name="understand-the-azure-ad-schema"></a>瞭解 Azure AD 架構
Azure Active Directory (Azure AD) 中的物件（例如任何目錄）是一種程式設計的高階資料結構，代表使用者、群組和連絡人等專案。 當您在 Azure AD 中建立新的使用者或連絡人時，您要建立該物件的新實例。 這些實例可以根據其屬性來區分。

Azure AD 中的屬性是負責在 Azure AD 中儲存物件實例相關資訊的元素。

Azure AD 架構會定義可在專案中使用屬性的規則、這些屬性可能具有的數值型別，以及使用者可能與這些值互動的方式。 

Azure AD 有兩種類型的屬性：
- **內建屬性**：由 Azure AD 架構預先定義的屬性。 這些屬性會提供不同的用法，而且可能會或可能無法存取。
- **目錄延伸** 模組：提供的屬性，可讓您自訂 Azure AD 供您自己使用。 例如，如果您已使用特定屬性擴充您的內部部署 Active Directory，並且想要傳送該屬性，則可以使用其中一個提供的自訂屬性。 

## <a name="attributes-and-expressions"></a>屬性和運算式
當物件（例如使用者）布建至 Azure AD 時，會建立使用者物件的新實例。 這項建立包含該物件的屬性（也稱為屬性）。 一開始，新建立的物件會將其屬性設定為由同步處理規則決定的值。 這些屬性接著會透過雲端布建代理程式保持最新狀態。

![物件布建](media/concept-attributes/attribute-1.png)

例如，使用者可能是行銷部門的一部分。 其 Azure AD 部門屬性一開始會在布建時建立，而值會設定為 Marketing。 六個月後，如果它們變更為 Sales，其內部部署 Active Directory 部門屬性就會變更為 Sales。 這項變更會同步處理至 Azure AD，並反映在其 Azure AD 使用者物件中。

屬性同步處理可能是直接的，其中 Azure AD 中的值會直接設定為內部部署屬性的值。 或者，程式設計運算式可能會處理同步處理。 必須進行某些邏輯或判斷來擴展值的情況下，才需要程式設計運算式。

例如，如果您有郵件屬性 " john.smith@contoso.com "，而且需要移除 " @contoso.com " 部分，而且只將值「john smith」，則會使用如下的內容：

`Replace([mail], "@contoso.com", , ,"", ,)`  

**範例輸入/輸出：** <br>

* **輸入** (mail)："john.smith@contoso.com"
* **輸出**： "john smith"

如需如何撰寫自訂表格達式和語法的詳細資訊，請參閱 [在 Azure Active Directory 中撰寫屬性對應的運算式](../app-provisioning/functions-for-customizing-application-data.md)。

下表列出常見的屬性，以及它們如何同步處理至 Azure AD。


|內部部署 Active Directory|對應類型|Azure AD|
|-----|-----|-----|
|cn|直接|commonName
|countryCode|直接|countryCode|
|displayName|直接|displayName|
|givenName|運算式|givenName|
|objectGUID|直接|sourceAnchorBinary|  
|userprincipalName|直接|userPrincipalName|
|ProxyAdress|直接|ProxyAddress|

## <a name="view-the-schema"></a>查看架構
> [!WARNING]
> 雲端同步設定會建立服務主體。 服務主體會顯示在 Azure 入口網站中。 您不應該使用 Azure 入口網站中的服務主體體驗來修改屬性對應。  不支援此連結方式。

若要查看架構並進行驗證，請遵循下列步驟。

1.  移至 [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)。
1.  使用您的全域管理員帳戶登入。
1.  選取左側的 [**修改許可權**]，並確定已 *同意*[**全部**]。
1.  執行查詢 `https://graph.microsoft.com/beta/serviceprincipals/?$filter=startswith(Displayname,'Active')` 。 此查詢會傳回已篩選的服務主體清單。
1.  找 `"appDisplayName": "Active Directory to Azure Active Directory Provisioning"` 出並記下的值 `"id"` 。
    ```
    "value": [
            {
                "id": "00d41b14-7958-45ad-9d75-d52fa29e02a1",
                "deletedDateTime": null,
                "accountEnabled": true,
                "appDisplayName": "Active Directory to Azure Active Directory Provisioning",
                "appId": "1a4721b3-e57f-4451-ae87-ef078703ec94",
                "applicationTemplateId": null,
                "appOwnerOrganizationId": "47df5bb7-e6bc-4256-afb0-dd8c8e3c1ce8",
                "appRoleAssignmentRequired": false,
                "displayName": "Active Directory to Azure Active Directory Provisioning",
                "errorUrl": null,
                "homepage": "https://account.activedirectory.windowsazure.com:444/applications/default.aspx?metadata=AD2AADProvisioning|ISV9.1|primary|z",
                "loginUrl": null,
                "logoutUrl": null,
                "notificationEmailAddresses": [],
                "preferredSingleSignOnMode": null,
                "preferredTokenSigningKeyEndDateTime": null,
                "preferredTokenSigningKeyThumbprint": null,
                "publisherName": "Active Directory Application Registry",
                "replyUrls": [],
                "samlMetadataUrl": null,
                "samlSingleSignOnSettings": null,
                "servicePrincipalNames": [
                    "http://adapplicationregistry.onmicrosoft.com/adprovisioningtoaad/primary",
                    "1a4721b3-e57f-4451-ae87-ef078703ec94"
                ],
                "signInAudience": "AzureADMultipleOrgs",
                "tags": [
                    "WindowsAzureActiveDirectoryIntegratedApp"
                ],
                "addIns": [],
                "api": {
                    "resourceSpecificApplicationPermissions": []
                },
                "appRoles": [
                    {
                        "allowedMemberTypes": [
                            "User"
                        ],
                        "description": "msiam_access",
                        "displayName": "msiam_access",
                        "id": "a0326856-1f51-4311-8ae7-a034d168eedf",
                        "isEnabled": true,
                        "origin": "Application",
                        "value": null
                    }
                ],
                "info": {
                    "termsOfServiceUrl": null,
                    "supportUrl": null,
                    "privacyStatementUrl": null,
                    "marketingUrl": null,
                    "logoUrl": null
                },
                "keyCredentials": [],
                "publishedPermissionScopes": [
                    {
                        "adminConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on behalf of the signed-in user.",
                        "adminConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "id": "d40ed463-646c-4efe-bb3e-3fa7d0006688",
                        "isEnabled": true,
                        "type": "User",
                        "userConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on your behalf.",
                        "userConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "value": "user_impersonation"
                    }
                ],
                "passwordCredentials": []
            },
    ```
1. `{Service Principal id}`以您的值取代，然後執行查詢 `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal id}/synchronization/jobs/` 。
1. 找 `"id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976"` 出並記下的值 `"id"` 。
    ```
    {
                "id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976",
                "templateId": "AD2AADProvisioning",
                "schedule": {
                    "expiration": null,
                    "interval": "PT2M",
                    "state": "Active"
                },
                "status": {
                    "countSuccessiveCompleteFailures": 0,
                    "escrowsPruned": false,
                    "code": "Active",
                    "lastSuccessfulExecutionWithExports": null,
                    "quarantine": null,
                    "steadyStateFirstAchievedTime": "2019-11-08T15:48:05.7360238Z",
                    "steadyStateLastAchievedTime": "2019-11-20T16:17:24.7957721Z",
                    "troubleshootingUrl": "",
                    "lastExecution": {
                        "activityIdentifier": "2dea06a7-2960-420d-931e-f6c807ebda24",
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 15,
                        "countEscrowedRaw": 15,
                        "countExported": 0,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "2019-11-20T16:15:21.116098Z",
                        "timeEnded": "2019-11-20T16:17:24.7488681Z"
                    },
                    "lastSuccessfulExecution": {
                        "activityIdentifier": null,
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 0,
                        "countEscrowedRaw": 0,
                        "countExported": 5,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "0001-01-01T00:00:00Z",
                        "timeEnded": "2019-11-20T14:09:46.8855027Z"
                    },
                    "progress": [],
                    "synchronizedEntryCountByType": [
                        {
                            "key": "group to Group",
                            "value": 33
                        },
                        {
                            "key": "user to User",
                            "value": 3
                        }
                    ]
                },
                "synchronizationJobSettings": [
                    {
                        "name": "Domain",
                        "value": "{\"DomainFQDN\":\"contoso.com\",\"DomainNetBios\":\"CONTOSO\",\"ForestFQDN\":\"contoso.com\",\"ForestNetBios\":\"CONTOSO\"}"
                    },
                    {
                        "name": "DomainFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "DomainNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "ForestFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "ForestNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "QuarantineTooManyDeletesThreshold",
                        "value": "500"
                    }
                ]
            }
    ```
1. 現在執行查詢 `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal Id}/synchronization/jobs/{AD2AAD Provisioning id}/schema` 。
 
    範例： https://graph.microsoft.com/beta/serviceprincipals/653c0018-51f4-4736-a3a3-94da5dcb6862/synchronization/jobs/AD2AADProvisioning.e9287a7367e444c88dc67a531c36d8ec/schema

   以您自己的值取代 `{Service Principal Id}` 和 `{AD2ADD Provisioning Id}` 。

1. 此查詢會傳回架構。

   ![傳回的架構](media/concept-attributes/schema-1.png)
 
## <a name="next-steps"></a>後續步驟

- [什麼是佈建？](what-is-provisioning.md)
- [什麼是雲端同步 Azure AD Connect？](what-is-cloud-sync.md)