---
title: 使用 Linux VM 系統指派的受控識別來存取 Azure AD Graph API
description: 本教學課程會逐步引導您使用 Linux VM 系統指派的受控識別，以存取 Azure AD Graph API。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: daveba
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/20/2018
ms.author: daveba
ms.openlocfilehash: 1d1e0d8f5a030daadb8dab1233dee52d5485c8fb
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48237478"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-ad-graph-api"></a>教學課程：使用 Linux VM 系統指派的受控識別來存取 Azure AD Graph API

[!INCLUDE[preview-notice](~/includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Linux 虛擬機器 (VM)，以存取 Azure AD Graph API 來擷取其群組成員資格。 適用於 Azure 資源的受控識別會自動由 Azure 管理，可讓您向支援 Azure AD 驗證的服務進行驗證，而不需要將認證插入程式碼中。  

在本教學課程中，您會查詢 VM 身分識別在 Azure AD 群組中的成員資格。 群組資訊通常會用於決定授權。 在 Azure AD 中，VM 的受控識別實際上會以**服務主體**來表示。 

> [!div class="checklist"]
> * 連接至 Azure AD
> * 在 Azure AD 的群組中新增 VM 身分識別 
> * 對 VM 身分識別授與 Azure AD Graph 的存取權 
> * 使用 VM 身分識別取得存取權杖，並使用它來呼叫 Azure AD Graph

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

- [安裝最新版的 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)

- 若要對 VM 身分識別授與 Azure AD Graph 的存取權，必須在 Azure AD 中對帳戶指派**全域管理員**角色。

## <a name="connect-to-azure-ad"></a>連接至 Azure AD

您需要連線至 Azure AD 以將 VM 指派給群組，以及對 VM 授與可供擷取其群組成員資格的權限。

```cli
az login
```

## <a name="add-your-vms-identity-to-a-group-in-azure-ad"></a>在 Azure AD 的群組中新增 VM 身分識別

在Linux VM 上啟用了系統指派的受控識別時，它會在 Azure AD 中建立服務主體。  您需要將 VM 新增至群組。 請參閱下列文章，取得如何將 VM 新增至 Azure AD 中群組的指示：

- [新增群組成員](/cli/azure/ad/group/member?view=azure-cli-latest#az-ad-group-member-add)

## <a name="grant-your-vm-access-to-the-azure-ad-graph-api"></a>對 VM 授與 Azure AD Graph API 的存取權

您的程式碼可以使用適用於 Azure 資源的受控識別來取得存取權杖，以向支援 Azure AD 驗證的資源進行驗證。 Microsoft Azure AD Graph API 支援 Azure AD 驗證。 在此步驟中，您會對 VM 身分識別的服務主體授與 Azure AD Graph 的存取權，以供其查詢群組成員資格。 透過**應用程式權限**，可對服務主體授與 Microsoft 或 Azure AD Graph 的存取權。 所需授與的應用程式權限類型，取決於您要在 MS 或 Azure AD Graph 中存取的實體。

在本教學課程中，您會授與能力給 VM 身分識別，使其能夠使用 `Directory.Read.All` 應用程式權限來查詢群組成員資格。 若要授與此權限，您需要有獲得 Azure AD 中全域管理員角色的使用者帳戶。 一般來說，您可藉由瀏覽應用程式在 Azure 入口網站中的註冊，並於該處新增權限，來對應用程式授與權限。 不過，Azure 資源的受控識別不會在 Azure AD 中註冊應用程式物件，而只會註冊服務主體。 若要註冊應用程式權限，您要使用 Azure AD PowerShell 命令列工具。 

Azure AD Graph：
- 服務主體 appId (會在對應用程式授與權限時用到)：00000002-0000-0000-c000-000000000000
- 資源識別碼 (會在從 Azure 資源的受控識別要求存取權杖時用到)： https://graph.windows.net
- 權限範圍參考：[Azure AD Graph 權限參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)

### <a name="grant-application-permissions-using-curl"></a>使用 CURL 來對應用程式授與權限

1. 擷取權杖來發出 CURL 要求：

   ```cli
   az account get-access-token --resource "https://graph.windows.net/"
   ```

2. 您必須擷取並記下您 VM 的 `objectId`。 這會在後續步驟中用來對 VM 授與權限，以讀取其群組成員資格。 將 `<ACCESS TOKEN>` 取代為您在先前步驟中擷取的存取權杖。

   ```bash
   curl 'https://graph.windows.net/myorganization/servicePrincipals?$filter=startswith%28displayName%2C%27myVM%27%29&api-version=1.6' -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

3. 使用 Azure AD Graph appID (00000002-0000-0000-c000-000000000000) 擷取並記下 `odata.type: Microsoft.DirectoryServices.ServicePrincipal` 的 `objectId` 和 `Directory.Read.All`應用程式角色權限的 `id`。  將 `<ACCESS TOKEN>` 取代為您先前擷取的存取權杖。

   ```bash
   curl "https://graph.windows.net/myorganization/servicePrincipals?api-version=1.6&%24filter=appId%20eq%20'00000002-0000-0000-c000-000000000000'" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   回應：

   ```json
   "odata.metadata":"https://graph.windows.net/myorganization/$metadata#directoryObjects",
   "value":[
      {
         "odata.type":"Microsoft.DirectoryServices.ServicePrincipal",
         "objectType":"ServicePrincipal",
         "objectId":"81789304-ff96-402b-ae73-07ec0db26721",
         "deletionTimestamp":null,
         "accountEnabled":true,
         "addIns":[
         ],
         "alternativeNames":[
         ],
         "appDisplayName":"Windows Azure Active Directory",
         "appId":"00000002-0000-0000-c000-000000000000",
         "appOwnerTenantId":"f8cdef31-a31e-4b4a-93e4-5f571e91255a",
         "appRoleAssignmentRequired":false,
         "appRoles":[
            {
               "allowedMemberTypes":[
                  "Application"
               ],
               "description":"Allows the app to read data in your company or school directory, such as users, groups, and apps.",
               "displayName":"Read directory data",
               "id":"5778995a-e1bf-45b8-affa-663a9f3f4d04",
               "isEnabled":true,
               "value":"Directory.Read.All"
            },
            {
               //other appRoles values
            }
   ``` 

4. 現在，使用 Azure AD Graph API 將 VM 的服務主體讀取存取授與 Azure AD 目錄物件。  `id` 值是 `Directory.Read.All` 應用程式角色權限的值，而`resourceId` 是 `odata.type:Microsoft.DirectoryServices.ServicePrincipal` 服務主體的 `objectId` (您在上一個步驟記下的值)。

   ```bash
   curl "https://graph.windows.net/myorganization/servicePrincipals/<VM Object ID>/appRoleAssignments?api-version=1.6" -X POST -d '{"id":"5778995a-e1bf-45b8-affa-663a9f3f4d04","principalId":"<VM Object ID>","resourceId":"81789304-ff96-402b-ae73-07ec0db26721"}'-H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ``` 
 
## <a name="get-an-access-token-using-the-vms-identity-to-call-azure-ad-graph"></a>使用 VM 身分識別取得存取權杖，以便呼叫 Azure AD Graph 

若要完成這些步驟，您需要 SSH 用戶端。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../../virtual-machines/linux/mac-create-ssh-keys.md)。

1. 在入口網站中，瀏覽至 [Linux VM]，並在 [概觀] 中按一下 [連線]。  
2. 使用您所選擇的 SSH 用戶端來**連線**到 VM。 
3. 在終端機視窗中，使用 CURL 向 Azure 資源端點的本機受控識別提出要求，以取得 Azure AD Graph 的存取權杖。  
    
   ```bash
   curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://graph.windows.net' -H Metadata:true
   ```    
   此回應包含您存取 Azure AD Graph 所需的存取權杖。

   回應：

   ```bash
   {
       "access_token":"eyJ0eXAiOiJKV...",
       "expires_in":"3599",
       "expires_on":"1519622535",
       "not_before":"1519618635",
       "resource":"https://graph.windows.net",
       "token_type":"Bearer"
   }
   ```

4. 使用 VM 服務主體的物件識別碼 (您在先前步驟中擷取的值)，您便可以查詢 Azure AD Graph API 來擷取其群組成員資格。 將 `<OBJECT-ID>` 取代為您 VM 服務主體的物件識別碼，並將 `<ACCESS-TOKEN>` 取代為先前取得的存取權杖：

   ```bash
   curl 'https://graph.windows.net/myorganization/servicePrincipals/<OBJECT-ID>/getMemberGroups?api-version=1.6' -X POST -d "{\"securityEnabledOnly\": false}" -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS-TOKEN>"
   ```

   回應：

   ```bash   
   Content : {"odata.metadata":"https://graph.windows.net/myorganization/$metadata#Collection(Edm.String)","value":["<ObjectID of VM's group membership>"]}
   ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Linux VM 系統指派的受控識別，來存取 Azure AD Graph。  若要深入了解 Azure AD Graph，請參閱：

>[!div class="nextstepaction"]
>[Azure AD Graph](https://docs.microsoft.com/azure/active-directory/develop/active-directory-graph-api)