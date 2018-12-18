---
title: 使用 Windows VM 系統指派的受控識別來存取 Azure AD Graph API
description: 本教學課程會逐步引導您使用 Windows VM 系統指派的受控識別，以存取 Azure AD Graph API。
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
ms.openlocfilehash: e208c8b52c14b99129bf3fce9ae54ba11f1c0a28
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47584297"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-ad-graph-api"></a>教學課程：使用 Windows VM 系統指派的受控識別來存取 Azure AD Graph API

[!INCLUDE[preview-notice](~/includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Windows 虛擬機器 (VM)，以存取 Microsoft Graph API 來擷取其群組成員資格。 適用於 Azure 資源的受控識別會自動由 Azure 管理，可讓您向支援 Azure AD 驗證的服務進行驗證，而不需要將認證插入程式碼中。  在本教學課程中，您會查詢 VM 身分識別在 Azure AD 群組中的成員資格。 舉例來說，群組資訊通常會用於決定授權。 在 Azure AD 中，VM 的受控識別實際上會以**服務主體**來表示。 在執行群組查詢前，請在 Azure AD 的群組中新增代表 VM 身分識別的服務主體。 您可以使用 Azure PowerShell、Azure AD PowerShell 或 Azure CLI 來執行這項工作。

> [!div class="checklist"]
> * 連接至 Azure AD
> * 在 Azure AD 的群組中新增 VM 身分識別 
> * 對 VM 身分識別授與 Azure AD Graph 的存取權 
> * 使用 VM 身分識別取得存取權杖，並使用它來呼叫 Azure AD Graph

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Windows 虛擬機器](/azure/virtual-machines/windows/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

- 若要對 VM 身分識別授與 Azure AD Graph 的存取權，必須在 Azure AD 中對帳戶指派**全域管理員**角色。

## <a name="connect-to-azure-ad"></a>連接至 Azure AD

您需要連線至 Azure AD 以將 VM 指派給群組，以及對 VM 授與可供擷取其群組成員資格的權限。

```powershell
Connect-AzureAD
```

## <a name="add-your-vm-identity-to-a-group-in-azure-ad"></a>在 Azure AD 的群組中新增 VM 身分識別

在 Windows VM 上啟用了系統指派的受控識別時，它會在 Azure AD 中建立服務主體。  現在，您需要將 VM 新增至群組。  下列範例會在 Azure AD 中建立新的群組，並將 VM 的服務主體新增至該群組：

```powershell
New-AzureADGroup -DisplayName "myGroup" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
$AzureADGroup = Get-AzureADGroup -Filter "displayName eq 'myGroup'"
$ManagedIdentitiesServicePrincipal = Get-AzureADServicePrincipal -Filter "displayName eq 'myVM'"
Add-AzureADGroupMember -ObjectId $AzureADGroup.ObjectID -RefObjectId $ManagedIdentitiesServicePrincipal.ObjectId
```
## <a name="grant-your-vm-access-to-the-azure-ad-graph-api"></a>對 VM 授與 Azure AD Graph API 的存取權

您的程式碼可以使用適用於 Azure 資源的受控識別來取得存取權杖，以向支援 Azure AD 驗證的資源進行驗證。 Microsoft Azure AD Graph API 支援 Azure AD 驗證。 在此步驟中，您會對 VM 身分識別的服務主體授與 Azure AD Graph 的存取權，以供其查詢群組成員資格。 透過**應用程式權限**，可對服務主體授與 Microsoft 或 Azure AD Graph 的存取權。 所需授與的應用程式權限類型，取決於您要在 MS 或 Azure AD Graph 中存取的實體。

在本教學課程中，您會授與能力給 VM 身分識別，使其能夠使用 ```Directory.Read.All``` 應用程式權限來查詢群組成員資格。 若要授與此權限，您需要有獲得 Azure AD 中全域管理員角色的使用者帳戶。 一般來說，您可藉由瀏覽應用程式在 Azure 入口網站中的註冊，並於該處新增權限，來對應用程式授與權限。 不過，Azure 資源的受控識別不會在 Azure AD 中註冊應用程式物件，而只會註冊服務主體。 若要註冊應用程式權限，您要使用 Azure AD PowerShell 命令列工具。 

Azure AD Graph：
- 服務主體 appId (會在對應用程式授與權限時用到)：00000002-0000-0000-c000-000000000000
- 資源識別碼 (會在從 Azure 資源的受控識別要求存取權杖時用到)： https://graph.windows.net
- 權限範圍參考：[Azure AD Graph 權限參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)

### <a name="grant-application-permissions-using-azure-ad-powershell"></a>使用 Azure AD PowerShell 來對應用程式授與權限

您需要有 Azure AD PowerShell 才能使用此選項。 如果您沒有安裝它，請[下載最新版本](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2)之後再繼續。

1. 開啟 PowerShell 視窗，並連線至 Azure AD：

   ```powershell
   Connect-AzureAD
   ```

2. 執行下列 PowerShell 命令來指派 ``Directory.Read.All`` 應用程式權限給代表 VM 身分識別的服務主體。

   ```powershell
   $ManagedIdentitiesServicePrincipal = Get-AzureADServicePrincipal -Filter "displayName eq 'myVM'"
   $GraphAppId = "00000002-0000-0000-c000-000000000000"
   $GraphServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$GraphAppId'"
   $PermissionName = "Directory.Read.All"
   $AppRole = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
   New-AzureAdServiceAppRoleAssignment -ObjectId $ManagedIdentitiesServicePrincipal.ObjectId -PrincipalId $ManagedIdentitiesServicePrincipal.ObjectId -ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole.Id
   ``` 

   最後一個命令的輸出看起來應該像這樣，且會傳回指派識別碼：
        
   `ObjectId`:`gzR5KyLAiUOTiqFhNeWZWBtK7ZKqNJxAiWYXYVHlgMs`

   `ResourceDisplayName`:`Windows Azure Active Directory`

   `PrincipalDisplayName`:`myVM` 

   如果 `New-AzureAdServiceAppRoleAssignment` 的呼叫失敗，且出現 `bad request, one or more properties are invalid` 錯誤，則表示應用程式權限可能早已指派給 VM 身分識別的服務主體。 您可以使用下列 PowerShell 命令，檢查 VM 身分識別與 Azure AD Graph 之間是否早就有應用程式權限：

   ```powershell
   $ManagedIdentitiesServicePrincipal = Get-AzureADServicePrincipal -Filter "displayName eq '<VM-NAME>'"
   $GraphAppId = "00000002-0000-0000-c000-000000000000"
   $GraphServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$GraphAppId'"
   $PermissionName = "Directory.Read.All"
   $AppRole = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
   Get-AzureADServiceAppRoleAssignment -ObjectId $GraphServicePrincipal.ObjectId | Where-Object {$_.Id -eq $AppRole.Id -and $_.PrincipalId -eq $ManagedIdentitiesServicePrincipal.ObjectId}
   ```

   您可以使用下列 PowerShell 命令，列出已授與給 Azure AD Graph 的所有應用程式權限：

   ```powershell
   $GraphAppId = "00000002-0000-0000-c000-000000000000"
   $GraphServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$GraphAppId'"
   Get-AzureADServiceAppRoleAssignment -ObjectId $GraphServicePrincipal.ObjectId
   ``` 

   您可以使用下列 PowerShell 命令，針對 VM 的 Azure AD Graph 身分識別，移除已授與給它的應用程式權限：

   ```powershell
   $ManagedIdentitiesServicePrincipal = Get-AzureADServicePrincipal -Filter "displayName eq '<VM-NAME>'"
   $GraphAppId = "00000002-0000-0000-c000-000000000000"
   $GraphServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$GraphAppId'"
   $PermissionName = "Directory.Read.All"
   $AppRole = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}   
   $ServiceAppRoleAssignment = Get-AzureADServiceAppRoleAssignment -ObjectId $GraphServicePrincipal.ObjectId | Where-Object {$_.Id -eq $AppRole.Id -and $_.PrincipalId -eq $ManagedIdentitiesServicePrincipal.ObjectId}
   Remove-AzureADServiceAppRoleAssignment -AppRoleAssignmentId $ServiceAppRoleAssignment.ObjectId -ObjectId $ManagedIdentitiesServicePrincipal.ObjectId
   ```
 
## <a name="get-an-access-token-using-the-vms-identity-to-call-azure-ad-graph"></a>使用 VM 身分識別取得存取權杖，以便呼叫 Azure AD Graph 

若要使用 VM 的系統指派受控識別對 Azure AD Graph 進行驗證，您需要從 VM 提出要求。

1. 在入口網站中，瀏覽至 [虛擬機器] 並移至您的 Windows VM，在 [概觀] 刀鋒視窗中按一下 [連線]。  
2. 輸入您建立 Windows VM 時使用的**使用者名稱**和**密碼**。
3. 現在您已經建立虛擬機器的遠端桌面連線，請在遠端工作階段中開啟 PowerShell。  
4. 使用 Powershell 的 Invoke-WebRequest，向 Azure 資源端點的本機受控識別提出要求，以取得 Azure AD Graph 的存取權杖。

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://graph.windows.net' -Method GET -Headers @{Metadata="true"}
   ```

   將來自 JSON 物件的回應轉換為 PowerShell 物件。

   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```

   擷取回應中的存取權杖。

   ```powershell
   $AccessToken = $content.access_token
   ```

5. 使用 VM 身分識別服務主體的物件識別碼 (可從先前步驟中所宣告的變數擷取此值：``$ManagedIdentitiesServicePrincipal.ObjectId``)，您便可以查詢 Azure AD Graph API 來擷取其群組成員資格。 將 <OBJECT ID> 替換為上一個步驟中的物件識別碼，並將 <ACCESS-TOKEN> 替換為先前取得的存取權杖：

   ```powershell
   Invoke-WebRequest 'https://graph.windows.net/<Tenant ID>/servicePrincipals/<VM Object ID>/getMemberGroups?api-version=1.6' -Method POST -Body '{"securityEnabledOnly":"false"}' -Headers @{Authorization="Bearer $AccessToken"} -ContentType "application/json"
   ```
   
   回應中會包含您在先前步驟中作為新增 VM 服務主體時的目的地群組 `Object ID`。

   回應：

   ```powershell   
   Content : {"odata.metadata":"https://graph.windows.net/<Tenant ID>/$metadata#Collection(Edm.String)","value":["<ObjectID of VM's group membership>"]}
   ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Windows VM 系統指派的受控識別，來存取 Azure AD Graph。  若要深入了解 Azure AD Graph，請參閱：

>[!div class="nextstepaction"]
>[Azure AD Graph](https://docs.microsoft.com/azure/active-directory/develop/active-directory-graph-api)
