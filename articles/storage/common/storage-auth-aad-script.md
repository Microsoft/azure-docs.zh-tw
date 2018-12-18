---
title: 在 Azure AD 身分識別下執行 Azure CLI 或 PowerShell 命令以存取 Azure 儲存體 (預覽) | Microsoft Docs
description: Azure CLI 和 PowerShell 支援使用 Azure AD 身分識別登入，以便在 Azure 儲存體容器、佇列及其資料上執行命令。 存取權杖是針對工作階段提供，用來授權呼叫作業。 權限取決於指派給 Azure AD 身分識別的角色。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/20/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 6354d89ff5a23ccb51b85737b3a842c08534683e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223605"
---
# <a name="use-an-azure-ad-identity-to-access-azure-storage-with-cli-or-powershell-preview"></a>使用 Azure AD 身分識別以使用 CLI 或 PowerShell 存取 Azure 儲存體 (預覽)

Azure 儲存體提供 Azure CLI 和 PowerShell 的預覽擴充，可讓您在 Azure Active Directory (Azure AD) 身分識別下登入並且執行指令碼命令。 Azure AD 身分識別可以是使用者、群組或應用程式服務主體，也可以是[受控服務識別](../../active-directory/managed-identities-azure-resources/overview.md)。 您可以透過角色型存取控制 (RBAC)，將存取儲存體資源的權限指派給 Azure AD 身分識別。 如需 Azure 儲存體中 RBAC 角色的詳細資訊，請參閱[使用 RBAC 管理 Azure 儲存體資料的存取權限 (預覽)](storage-auth-aad-rbac.md)。

當您使用 Azure AD 身分識別登入 Azure CLI 或 PowerShell 時，系統會傳回存取權杖，以便在該身分識別之下存取 Azure 儲存體。 然後 CLI 或 PowerShell 會自動使用該權杖，針對 Azure 儲存體對作業進行授權。 針對支援的作業，您不需要再使用命令傳遞帳戶金鑰或 SAS 權杖。

[!INCLUDE [storage-auth-aad-note-include](../../../includes/storage-auth-aad-note-include.md)]

## <a name="supported-operations"></a>支援的作業

容器和佇列上的作業支援預覽擴充。 您會呼叫哪些作業，取決於您用來登入 Azure CLI 或 PowerShell 的 Azure AD 身分識別所獲得的授權。 給予 Azure 儲存體容器或佇列的權限，是透過角色型存取控制 (RBAC) 指派的。 例如，如果將資料讀者角色指派給身分識別，則您可以執行讀取容器或佇列資料的指令碼命令。 如果將資料參與者角色指派給身分識別，則您可以執行讀取、寫入或刪除容器或佇列或其中所含資料的指令碼命令。 

如需容器或佇列上每個 Azure 儲存體作業所需權限的詳細資訊，請參閱[呼叫 REST 作業的權限](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-azure-active-directory#permissions-for-calling-rest-operations)。  

## <a name="call-cli-commands-with-an-azure-ad-identity"></a>使用 Azure AD 身分識別呼叫 CLI 命令

若要安裝 Azure CLI 的預覽擴充：

1. 請確定您已安裝 Azure CLI 2.0.32 版或更新版本。 執行 `az --version` 來檢查您安裝的版本。
2. 執行下列命令來安裝預覽擴充： 

    ```azurecli
    az extension add -n storage-preview
    ```

預覽擴充會將新的 `--auth-mode` 參數新增至支援的命令：

- 將 `--auth-mode` 參數設定為 `login`，以使用 Azure AD 身分識別登入。
- 將 `--auth-mode` 參數設定為舊版 `key` 值，以便在未針對帳戶提供驗證參數時，嘗試查詢帳戶金鑰。 

例如，若要使用 Azure AD 身分識別在 Azure CLI 中下載 Blob，請先執行 `az login`，然後將 `--auth-mode` 設定為 `login` 來呼叫命令：

```azurecli
az login
az storage blob download --account-name storagesamples --container sample-container --name myblob.txt --file myfile.txt --auth-mode login 
```

與 `--auth-mode` 參數相關聯的環境變數是 `AZURE_STORAGE_AUTH_MODE`。

## <a name="call-powershell-commands-with-an-azure-ad-identity"></a>使用 Azure AD 身分識別呼叫 PowerShell 命令

Azure PowerShell 僅支援搭配下列其中一個預覽模組使用 Azure AD 身分識別進行登入： 

- 4.4.0-preview 
- 4.4.1-preview 

若要使用 Azure AD 身分識別以 Azure PowerShell 登入：

1. 解除安裝任何先前安裝的 Azure PowerShell：

    - 使用 [設定] 底下的 [應用程式與功能]，從 Windows 移除任何先前安裝的 Azure PowerShell。
    - 從 `%Program Files%\WindowsPowerShell\Modules` 移除所有 **Azure*** 模組。

1. 確定您已安裝最新版的 PowerShellGet。 開啟 Windows PowerShell 視窗，然後執行下列命令來安裝最新版本：
 
    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```
1. 在安裝完 PowerShellGet 之後，關閉並重新開啟 PowerShell 視窗。 

1. 安裝最新版的 Azure PowerShell：

    ```powershell
    Install-Module AzureRM –Repository PSGallery –AllowClobber
    ```

1. 安裝其中一個支援 Azure AD 的 Azure 儲存體預覽模組：

    ```powershell
    Install-Module Azure.Storage –Repository PSGallery -RequiredVersion 4.4.1-preview  –AllowPrerelease –AllowClobber –Force 
    ```
1. 關閉並重新開啟 PowerShell 視窗。
1. 呼叫 [New-AzureStorageContext](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestoragecontext) Cmdlet 以建立內容，並且包含 `-UseConnectedAccount` 參數。 
1. 若要使用 Azure AD 身分識別呼叫 Cmdlet，請將新建立的內容傳遞給 Cmdlet。

下列範例顯示如何使用 Azure AD 身分識別，從 Azure PowerShell 列出容器中的 Blob。 請務必以您的值取代預留位置帳戶與容器名稱： 

```powershell
$ctx = New-AzureStorageContext -StorageAccountName storagesamples -UseConnectedAccount 
Get-AzureStorageBlob -Container sample-container -Context $ctx 
```

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 儲存體的 RBAC 角色，請參閱[使用 RBAC 管理儲存體資料的存取權限 (預覽)](storage-auth-aad-rbac.md)。
- 若要了解搭配 Azure 儲存體使用適用於 Azure 資源之受控身分識別的相關資訊，請參閱[使用適用於 Azure 資源的 Azure 受控身分識別驗證針對 Blob 與佇列的存取 (預覽)](storage-auth-aad-msi.md)。
- 若要深入了解如何從儲存體應用程式內授權容器和佇列的存取權，請參閱[使用 Azure AD 與儲存體應用程式](storage-auth-aad-app.md)。
- 如需 Azure Blob 和佇列的 Azure AD 整合詳細資訊，請參閱 Azure 儲存體小組部落格文章：[宣告適用於 Azure 儲存體的 Azure AD 驗證預覽](https://azure.microsoft.com/blog/announcing-the-preview-of-aad-authentication-for-storage/)。
