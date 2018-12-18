---
title: 為 Azure 檔案 (預覽) 啟用「透過 SMB 進行 Azure Active Directory 驗證」- Azure 儲存體
description: 了解如何透過 Azure Active Directory (Azure AD) Domain Services，為 Azure 檔案啟用「透過 SMB (伺服器訊息區) (預覽) 進行以身分識別為基礎的驗證」。 然後，您加入網域的 Windows 虛擬機器 (VM) 可以使用 Azure AD 認證存取 Azure 檔案共用。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/19/2018
ms.author: tamram
ms.openlocfilehash: 6ee80aa7b7a58e2f02ed36d3c0c4b1a0889a906f
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831461"
---
# <a name="enable-azure-active-directory-authentication-over-smb-for-azure-files-preview"></a>為 Azure 檔案 (預覽) 啟用「透過 SMB 進行 Azure Active Directory 驗證」
[!INCLUDE [storage-files-aad-auth-include](../../../includes/storage-files-aad-auth-include.md)]

如需了解為 Azure 檔案使用「透過 SMB 進行 Azure AD 驗證」的概觀，請參閱[為 Azure 檔案 (預覽) 使用「透過 SMB 進行 Azure Active Directory 驗證」的概觀](storage-files-active-directory-overview.md)。

## <a name="workflow-overview"></a>工作流程概觀
為 Azure 檔案啟用「透過 SMB 進行 Azure AD 驗證」之前，請確認您已正確設定 Azure AD 與 Azure 儲存體環境。 建議您逐步進行[必要條件](#prerequisites)，以確定您已執行所有必要步驟。 

接著，遵循下列步驟，授與對 Azure AD 認證之 Azure 檔案資源的存取權： 

1. 為您的儲存體帳戶啟用「透過 SMB 進行 Azure AD 驗證」，以使用相關聯的 Azure AD Domain Services 部署註冊儲存體帳戶。
2. 將共用的存取權限指派至 Azure AD 身分識別 (使用者、群組或服務主體)。
3. 透過 SMB 為目錄和檔案設定 NTFS 權限。
4. 從已加入網域的虛擬機器中裝載 Azure 檔案共用。

下圖說明針對 Azure 檔案啟用「透過 SMB 進行 Azure AD 驗證」的端對端工作流程。 

![此圖顯示針對 Azure 檔案使用「透過 SMB 進行 Azure AD 驗證」的工作流程](media/storage-files-active-directory-enable/azure-active-directory-over-smb-workflow.png)

## <a name="prerequisites"></a>必要條件 
1.  **選取或建立 Azure AD 租用戶。**

    您可以針對「透過 SMB 進行 Azure AD 驗證」使用全新或現有租用戶。 租用戶和您想要存取的檔案共用必須與相同的訂用帳戶相關聯。

    若要建立新的 Azure AD 租用戶，您可以[新增 Azure AD 租用戶和 Azure AD 訂用帳戶](https://docs.microsoft.com/windows/client-management/mdm/add-an-azure-ad-tenant-and-azure-ad-subscription)。 如果您有現有的 Azure AD 租用戶，但想要建立新的租用戶與 Azure 檔案搭配使用，請參閱[建立 Azure Active Directory 租用戶](https://docs.microsoft.com/rest/api/datacatalog/create-an-azure-active-directory-tenant)。

2.  **在 Azure AD 租用戶上啟用 Azure AD Domain Services。**

    若要支援使用 Azure AD 認證進行驗證，您必須為 Azure AD 租用戶啟用 Azure AD Domain Services。 如果您不是 Azure AD 租用戶的系統管理員，請連絡系統管理員，並遵循[使用 Azure 入口網站啟用 Azure Active Directory Domain Services](../../active-directory-domain-services/active-directory-ds-getting-started.md) 的逐步指導。

    通常需要大約 15 分鐘才能完成 Azure AD Domain Services 部署。 請先確認 Azure AD Domain Services 的健康情況顯示**執行中**，並已啟用密碼雜湊同步，才能繼續進行下一個步驟。

3.  **加入網域的 Azure VM 具有 Azure AD Domain Services。**

    若要使用虛擬機器的 Azure AD 認證存取檔案共用，您的虛擬機器網域必須加入至 Azure AD Domain Services。 如需如何加入虛擬機器網域的詳細資訊，請參閱[將 Windows Server 虛擬機器加入受控網域](../../active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm-portal.md)。

    > [!NOTE]
    > 只有在 Windows 7 或 Windows Server 2008 R2 以上的作業系統版本上執行之 Azure 虛擬機器，才支援對 Azure 檔案使用「透過 SMB 進行 Azure AD 驗證」。

4.  **選取或建立 Azure 檔案共用。**

    選取與 Azure AD 租用戶相同訂閱關聯的全新或現有檔案共用。 如需建立新檔案共用的資訊，請參閱 [在 Azure 檔案中建立檔案共用](storage-how-to-create-file-share.md)。 

    Azure AD 租用戶必須部署至支援透過 SMB 預覽 Azure AD 的區域。 預覽適用於所有公共區域，以下除外：美國西部、美國西部 2、美國中南部、美國東部、美國東部 2、美國中部、美國中北部、澳大利亞東部、西歐、北歐。

    為了獲得最佳效能，Microsoft 建議您的檔案共用與計畫存取共用的地方位於相同區域。

5.  **藉由使用儲存體帳戶金鑰裝載 Azure 檔案共用，驗證 Azure 檔案連線。**

    若要驗證是否已正確設定虛擬機器和檔案共用，請嘗試使用儲存體帳戶金鑰裝載檔案共用。 如需詳細資料，請參閱[裝載 Azure 檔案共用並在 Windows 中存取共用](storage-how-to-use-files-windows.md)。

## <a name="enable-azure-ad-authentication"></a>啟用 Azure AD 驗證
完成[必要條件](#prerequisites)之後，您可以啟用「透過 SMB 進行 Azure AD 驗證」。

### <a name="step-1-enable-azure-ad-authentication-over-smb-for-your-storage-account"></a>步驟 1：為您的儲存體帳戶啟用「透過 SMB 進行 Azure AD 驗證」
若要為 Azure 檔案啟用「透過 SMB 進行 Azure AD 驗證」，您可以使用 PowerShell 或 Azure CLI 的 Azure 儲存體資源提供者，在 2018 年 8 月 29 日之後建立的儲存體帳戶中設定屬性。 預覽版本不支援在 Azure 入口網站中設定屬性。 

設定此屬性會將儲存體帳戶註冊到關聯的 Azure AD Domain Services 部署。 接著，會針對儲存體帳戶中所有新的和現有的檔案共用啟用「透過 SMB 進行 Azure AD 驗證」。 

請記住，只有在將 Azure AD Domain Services 成功部署至您的 Azure AD 租用戶之後，才可以啟用「透過 SMB 進行 Azure AD 驗證」。 如需詳細資訊，請參閱[必要條件](#prerequisites)。

**Powershell**  
若要啟用「透過 SMB 進行 Azure AD 驗證」，請安裝 `AzureRM.Storage 6.0.0-preview` PowerShell 模組。 如需安裝 PowerShell 的相關資訊，請參閱[使用 PowerShellGet 在 Windows 上安裝 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。

接著，呼叫 [Set-AzureRmStorageAccount](https://docs.microsoft.com/powershell/module/azurerm.storage/set-azurermstorageaccount)，並將 **EnableAzureFilesAadIntegrationForSMB** 參數設定為 **true**。 在以下範例中，請記得以您自己的值取代預留位置值。

```powershell
# Create a new storage account
New-AzureRmStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -Location "<azure-region>" `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableAzureFilesAadIntegrationForSMB $true

# Update an existing storage account
# Supported for storage accounts created after August 29, 2018 only
Set-AzureRmStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -EnableAzureFilesAadIntegrationForSMB $true```
```

**CLI**  
若要從 Azure CLI 2.0 啟用「透過 SMB 進行 Azure AD 驗證」，請先安裝 *storage-preview* 擴充功能：

```azurecli-interactive
az extension add --name storage-preview
```

接下來，呼叫 [az storage account update](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-update)，並將 `--file-aad` 屬性設為 **true**。 在以下範例中，請記得以您自己的值取代預留位置值。

```azurecli-interactive
# Create a new storage account
az storage account create -n <storage-account-name> -g <resource-group-name> --file-aad true

# Update an existing storage account
# Supported for storage accounts created after August 29, 2018 only
az storage account update -n <storage-account-name> -g <resource-group-name> --file-aad true
```

### <a name="step-2-assign-access-permissions-to-an-identity"></a>步驟 2：將存取權限指派至身分識別 
若要使用 Azure AD 認證存取 Azure 檔案資源，身分識別 (使用者、群組或服務主體) 必須具有共用層級的必要權限。 下列逐步指引示範如何將檔案共用的讀取、寫入或刪除權限指派給身分識別。

> [!IMPORTANT]
> 完整管理控制檔案共用 (包括將角色指派給身分識別的能力) 需要使用儲存體帳戶金鑰。 Azure AD 認證不支援管理控制。 

#### <a name="step-21-define-a-custom-role"></a>步驟 2.1：定義自訂角色
若要授與共用層級權限，請定義自訂的 RBAC 角色，並將其指派給身分識別，將其範圍設定為特定檔案共用。 此程序類似於指定 Windows 共用權限，您可以從中指定給定之使用者對檔案共用的存取類型。  

下列各節顯示的範本為檔案共用提供讀取或變更權限。 若要定義自訂角色，請建立 JSON 檔案，並將適當的範本複製到該檔案。 如需定義自訂 RBAC 角色的詳細資訊，請參閱 [Azure 中的自訂角色](../../role-based-access-control/custom-roles.md)。

**適用於共用層級變更權限的角色定義**  
以下自訂角色範本提供共用層級變更權限，並授與身分識別的讀取、寫入和刪除存取的共用權限。

```json
{
  "Name": "<Custom-Role-Name>",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows for read, write and delete access to Azure File Share",
  "Actions": [
    "*"
  ],
  "NotActions": [
      "Microsoft.Authorization/*/Delete",
    "Microsoft.Authorization/*/Write",
    "Microsoft.Authorization/elevateAccess/Action"
  ],
  "DataActions": [
    "*"
  ],
  "AssignableScopes": [
        "/subscriptions/<Subscription-ID>"
  ]
}
```

**適用於共用層級讀取權限的角色定義**  
以下自訂角色範本提供共用層級讀取權限，並授與身分識別的讀取存取的共用權限。

```json
{
  "Name": "<Custom-Role-Name>",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows for read access to Azure File Share",
  "Actions": [
    "*/read"
  ],
  "DataActions": [
    "*/read"
  ],
  "AssignableScopes": [
        "/subscriptions/<Subscription-ID>"
  ]
}
```

#### <a name="step-22-create-the-custom-role-and-assign-it-to-the-target-identity"></a>步驟 2.2：建立自訂角色，並將其指派給目標身分識別
接下來，使用 PowerShell 或 Azure CLI 來建立角色，並將其指派給 Azure AD 身分識別。 

**Powershell**  
若要啟用「透過 SMB 進行 Azure AD 驗證」，請安裝 `AzureRM.Storage 6.0.0-preview` PowerShell 模組。 如需安裝 PowerShell 的相關資訊，請參閱[使用 PowerShellGet 在 Windows 上安裝 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。

下列 PowerShell 命令可建立自訂角色，並根據登入名稱將角色指派給 Azure AD 身分識別。 如需使用 PowerShell 指派 RBAC 角色的詳細資訊，請參閱[使用 RBAC 和 Azure PowerShell 管理存取](../../role-based-access-control/role-assignments-powershell.md)。

執行以下範例指令碼時，請記得使用您自己的值取代預留位置值。

```powershell
#Create a custom role based on the sample template above
New-AzureRmRoleDefinition -InputFile "<custom-role-def-json-path>"
#Get the name of the custom role
$FileShareContributorRole = Get-AzureRmRoleDefinition "<role-name>"
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshare/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzureRmRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

**CLI**  
下列 CLI 2.0 命令可建立自訂角色，並根據登入名稱將角色指派給 Azure AD 身分識別。 如需使用 Azure CLI 指派 RBAC 角色的詳細資訊，請參閱[使用 RBAC 和 Azure CLI 管理存取](../../role-based-access-control/role-assignments-cli.md)。 

執行以下範例指令碼時，請記得使用您自己的值取代預留位置值。

```azurecli-interactive
#Create a custom role based on the sample templates above
az role definition create --role-definition "<Custom-role-def-JSON-path>"
#List the custom roles
az role definition list --custom-role-only true --output json | jq '.[] | {"roleName":.roleName, "description":.description, "roleType":.roleType}'
#Assign the custom role to the target identity
az role assignment create --role "<custome-role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshare/<share-name>"
```

### <a name="step-3-configure-ntfs-permissions-over-smb"></a>步驟 3：透過 SMB 設定 NTFS 權限 
使用 RBAC 指派共用層級權限之後，必須在根、目錄或檔案層級指派適當的 NTFS 權限。 將共用層級權限視為高階閘道管理員，可決定使用者是否可存取共用，而 NTFS 權限可進行更精細的處理，以決定使用者可以在目錄或檔案層級執行哪些作業。 

Azure 檔案支援全套 NTFS 基本和進階權限。 您可以藉由裝載共用，然後執行 Windows [icacls](https://docs.microsoft.com/windows-server/administration/windows-commands/icacls) 或 [Set-ACL](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-acl) 命令，以檢視及設定 Azure 檔案共用中目錄與檔案的 NTFS 權限。 

> [!NOTE]
> 預覽版本僅支援使用 Windows 檔案總管檢視權限。 尚未支援編輯權限。

若要使用超級使用者權限設定 NTFS 權限，必須使用儲存體帳戶金鑰，從已加入網域的 VM 裝載共用。 依照下一節中的說明，從命令提示字元裝載 Azure 檔案共用，並視情況設定 NTFS 權限。

檔案共用的根目錄支援下列權限集合：

- BUILTIN\Administrators:(OI)(CI)(F)
- NT AUTHORITY\SYSTEM:(OI)(CI)(F)
- BUILTIN\Users:(RX)
- BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
- NT AUTHORITY\Authenticated Users:(OI)(CI)(M)
- NT AUTHORITY\SYSTEM:(F)
- CREATOR OWNER:(OI)(CI)(IO)(F)

#### <a name="step-31-mount-an-azure-file-share-from-the-command-prompt"></a>步驟 3.1 從命令提示字元裝載 Azure 檔案共用
使用 Windows **net use** 命令以裝載 Azure 檔案共用。 請記得使用您自己的值取代預留位置值。 如需裝載檔案共用的詳細資訊，請參閱[裝載 Azure 檔案共用並存取 Windows 中的共用](storage-how-to-use-files-windows.md)。

```
net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name>
```

#### <a name="step-32-configure-ntfs-permissions-with-icacls"></a>步驟 3.2 使用 icacls 設定 NTFS 權限
使用以下 Windows 命令，授與完整權限至檔案共用下的所有目錄和檔案，包括根目錄。 請記得使用您自己的值取代預留位置值。

```
icacls <mounted-drive-letter> /grant <user-email>:(f)
```

如需如何使用 icacls 設定 NTFS 權限，以及支援之不同類型權限的詳細資訊，請參閱 [icacls 的命令列參考](https://docs.microsoft.com/windows-server/administration/windows-commands/icacls)。

### <a name="step-4-mount-an-azure-file-share-from-a-domain-joined-vm"></a>步驟 4：從已加入網域的虛擬機器中裝載 Azure 檔案共用 
現在，您已準備好使用 Azure AD 認證，從加入網域的虛擬機器存取 Azure 檔案共用，以驗證您是否成功完成上述步驟。 首先，使用已授與權限的 Azure AD 身分識別登入虛擬機器，如下圖所示。

![螢幕擷取畫面顯示使用者驗證的 Azure AD 登入畫面](media/storage-files-active-directory-enable/azure-active-directory-authentication-dialog.png)

接著，使用下列命令，裝載 Azure 檔案共用。 請記得以您自己的值取代預留位置值。 由於您已經過驗證，因此不需要提供儲存體帳戶金鑰或 Azure AD 使用者名稱和密碼。 「透過 SMB 進行 Azure AD 驗證」支援使用 Azure AD 認證的單一登入體驗。

```
net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name>
```

您現在已成功啟用「透過 SMB 進行 Azure AD 驗證」，並指派一個自訂角色，該角色提供對 Azure AD 身分識別的檔案共用存取權。 若要向其他使用者授與對檔案共用的存取權，請遵循步驟 2 和 3 中提供的指示進行操作。

## <a name="next-steps"></a>後續步驟
如需 Azure 檔案以及使用「透過 SMB 進行 Azure AD 驗證」的詳細資訊，請參閱下列資源：

- [Azure 檔案服務簡介](storage-files-introduction.md)
- [適用於 Azure 檔案之「透過 SMB 進行 Azure AD 驗證」的概觀 (預覽)](storage-files-active-directory-overview.md)
- [常見問題集](storage-files-faq.md)