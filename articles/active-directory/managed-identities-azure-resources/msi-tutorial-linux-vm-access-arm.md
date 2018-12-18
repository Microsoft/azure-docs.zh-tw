---
title: 使用 Linux VM 使用者指派的受控識別來存取 Azure Resource Manager
description: 本教學課程會逐步引導您在 Linux VM 上利用使用者指派的受控識別，以存取 Azure Resource Manager。
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
ms.date: 12/22/2017
ms.author: daveba
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 4abde91e04048d64a17f861825d1fb7779873155
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106075"
---
# <a name="tutorial-use-a-user-assigned-managed-identity-on-a-linux-vm-to-access-azure-resource-manager"></a>教學課程：在 Linux VM 上利用使用者指派的受控識別來存取 Azure Resource Manager

[!INCLUDE [preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

本教學課程說明如何建立使用者指派的受控識別，並將其指派給 Linux 虛擬機器 (VM)，然後使用該身分識別來存取 Azure Resource Manager API。 適用於 Azure 資源的受控識別會自動由 Azure 管理。 它們會啟用對支援 Azure AD 驗證之服務的驗證，而不需要在程式碼中內嵌認證。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立使用者指派的受控識別
> * 將使用者指派的受控識別指派給 Linux VM 
> * 在 Azure Resource Manager 中，將存取資源群組的權利，授予使用者指派的受控識別 
> * 利用使用者指派的受控識別來取得存取權杖，以用來呼叫 Azure Resource Manager 

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

[登入 Azure 入口網站](https://portal.azure.com)

[建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0]( /cli/azure/install-azure-cli)。

## <a name="create-a-user-assigned-managed-identity"></a>建立使用者指派的受控識別

1. 如果使用的是 CLI 主控台 (而不是 Azure Cloud Shell 工作階段)，則請從登入 Azure 開始。 根據您建立使用者指派的新受控識別時，所想要使用的 Azure 訂用帳戶，使用其相關聯的帳戶：

    ```azurecli
    az login
    ```

2. 使用 [az identity create](/cli/azure/identity#az-identity-create)，建立使用者指派的受控識別。 `-g` 參數會指定資源群組，以在其中建立使用者指派的受控識別，而 `-n` 參數會指定此資源群組的名稱。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<UAMI NAME>` 參數的值：
    
[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]


```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <UAMI NAME>
```

回應包含使用者所指派受控識別建立之後的詳細資料，與下列範例類似。 在使用者指派的受控識別中，請記下 `id` 值，下一步會用到：

```json
{
"clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
"clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>/credentials?tid=5678&oid=9012&aid=12344643-8088-4d70-9532-c3a0fdc190fz",
"id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>",
"location": "westcentralus",
"name": "<UAMI NAME>",
"principalId": "9012",
"resourceGroup": "<RESOURCE GROUP>",
"tags": {},
"tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
"type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

## <a name="assign-a-user-assigned-managed-identity-to-your-linux-vm"></a>將使用者指派的受控識別指派給 Linux VM

使用者指派的受控識別可以由多個 Azure 資源上的用戶端使用。 請使用下列命令，將使用者指派的受控識別指派給單一虛擬機器。 針對 `-IdentityID` 參數，請使用前一個步驟中所傳回的 `Id` 屬性。

使用 [az vm assign-identity](/cli/azure/vm#az-vm-assign-identity)，將使用者指派的受控識別指派給 Linux VM。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<VM NAME>` 參數的值。 至於 `--identities` 參數值，請使用前一個步驟中所傳回的 `id` 屬性。

```azurecli-interactive
az vm assign-identity -g <RESOURCE GROUP> -n <VM NAME> --identities "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>"
```

## <a name="grant-your-user-assigned-managed-identity-access-to-a-resource-group-in-azure-resource-manager"></a>在 Azure Resource Manager 中，將存取資源群組的權利，授予使用者指派的受控識別 

適用於 Azure 資源的受控識別會提供身分識別，可供程式碼用來要求存取權杖，以向支援 Azure AD 驗證的資源 API 進行驗證。 在本教學課程中，您的程式碼將存取 Azure Resource Manager API。  

您必須先將身分識別存取權授與 Azure Resource Manager 中的資源，您的程式碼才能存取 API。 在此情況下，是包含 VM 的資源群組。 將 `<SUBSCRIPTION ID>` 和 `<RESOURCE GROUP>` 的值更新為適用於環境的值。 此外，將 `<UAMI PRINCIPALID>` 換成[建立使用者指派的受控識別](#create-a-user-assigned-managed-identity)中，由 `az identity create` 命令所傳回的 `principalId` 屬性：

```azurecli-interactive
az role assignment create --assignee <UAMI PRINCIPALID> --role 'Reader' --scope "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP> "
```

回應會包含已建立的角色指派詳細資料，與下列範例類似：

```json
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Authorization/roleAssignments/b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "name": "b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "properties": {
    "principalId": "f5fdfdc1-ed84-4d48-8551-999fb9dedfbl",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}

```

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-resource-manager"></a>使用 VM 身分識別取得存取權杖，並使用它來呼叫 Resource Manager 

其餘課程要從稍早建立的 VM 繼續進行。

若要完成這些步驟，您需要 SSH 用戶端。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)中使用 SSH 用戶端。 

1. 登入 Azure [入口網站](https://portal.azure.com)。
2. 在入口網站中，瀏覽至 [虛擬機器]，並移至您的 Linux 虛擬機器，在 [概觀] 中按一下 [連線]。 複製字串以連線到您的 VM。
3. 使用您所選擇的 SSH 用戶端來連線到虛擬機器。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](~/articles/virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](~/articles/virtual-machines/linux/mac-create-ssh-keys.md)。
4. 在終端機視窗中使用 CURL，向 Azure Instance Metadata Service (IMDS) 身分識別端點提出要求來取得 Azure Resource Manager 的存取權杖。  

   用來取得存取權杖的 CURL 要求，則如以下範例所示。 請務必將 `<CLIENT ID>` 換成[建立使用者指派的受控識別](#create-a-user-assigned-managed-identity)中，由 `az identity create` 命令所傳回的 `clientId` 屬性： 
    
   ```bash
   curl -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com/&client_id=<UAMI CLIENT ID>"   
   ```
    
    > [!NOTE]
    > `resource` 參數的值必須完全符合 Azure AD 的預期。 使用 Resource Manager 資源識別碼時，必須在 URI 上包含結尾斜線。 
    
    此回應包含您存取 Azure Resource Manager 所需的存取權杖。 
    
    回應範例：  

    ```bash
    {
    "access_token":"eyJ0eXAiOi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"
    } 
    ```

5. 使用存取權杖來存取 Azure Resource Manager，並讀取資源群組的屬性 (您已向使用者指派的受控識別授予該資源群組的存取權)。 請務必將 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 取代為您先前指定的值，並將 `<ACCESS TOKEN>` 取代為上一個步驟中所傳回的權杖。

    > [!NOTE]
    > URL 區分大小寫，因此請務必使用稍早在命名資源群組時所使用的相同大小寫，而且 `resourceGroups` 中的 "G" 為大寫。  

    ```bash 
    curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>?api-version=2016-09-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```

    回應包含特定資源群組資訊，與下列範例類似： 

    ```bash
    {
    "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/DevTest",
    "name":"DevTest",
    "location":"westus",
    "properties":{"provisioningState":"Succeeded"}
    } 
    ```
    
## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何建立使用者指派的受控識別，並將其連結至 Linux 虛擬機器以存取 Azure Resource Manager API。  若要深入了解 Azure Resource Manager，請參閱：

> [!div class="nextstepaction"]
>[Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview)

