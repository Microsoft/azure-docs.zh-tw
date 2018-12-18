---
title: 如何使用 Azure CLI 管理使用者指派的受控識別
description: 如何使用 Azure CLI 建立、列出和刪除使用者指派之受控識別的逐步說明。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/16/2018
ms.author: daveba
ms.openlocfilehash: 19963edc3742cc5f09ed02249d313728410711c8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973487"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-the-azure-cli"></a>使用 Azure CLI 建立、列出和刪除使用者指派的受控識別

[!INCLUDE [preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

適用於 Azure 資源的受控識別會在 Azure Active Directory 中為 Azure 服務提供受控識別。 您可以使用此身分識別來向支援 Azure AD 驗證的服務進行驗證，而不需要您程式碼中的認證。 

在本文中，您會了解如何使用 Azure CLI 建立、列出和刪除使用者指派的受控識別。

## <a name="prerequisites"></a>必要條件

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 若要執行本文中的管理作業，您的帳戶需要下列角色指派：
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立、讀取 (列出)、更新和刪除使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可讀取 (列出) 使用者指派受控識別的屬性。
- 若要執行 CLI 指令碼範例，您有三個選項：
    - 從 Azure 入口網站使用 [Azure Cloud Shell](../../cloud-shell/overview.md) (請參閱下一節)。
    - 請透過每個程式碼區塊右上角的 [立即試用] 按鈕，使用內嵌的 Azure Cloud Shell。
    - 如果您偏好使用本機 CLI 主控台，請[安裝最新版的 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0.13 或更新版本)。 使用 `az login` 登入 Azure，使用與 Azure 訂用帳戶相關聯的帳戶，而您要以此帳戶部署使用者指派受控識別。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-user-assigned-managed-identity"></a>建立使用者指派的受控識別 

若要建立使用者指派的受控識別，請使用 [az identity create](/cli/azure/identity#az-identity-create) 命令。 `-g` 參數會指定要建立使用者指派之受控識別的資源群組，而 `-n` 參數則指定其名稱。 將 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 參數取代為您自己的值：

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

 ```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>列出使用者指派的受控識別

若要列出使用者指派的受控識別，請使用 [az identity list](/cli/azure/identity#az-identity-list) 命令。 將 `<RESOURCE GROUP>` 取代為您自己的值：

```azurecli-interactive
az identity list -g <RESOURCE GROUP>
```
在 json 回應中，使用者指派的受控識別具備`"Microsoft.ManagedIdentity/userAssignedIdentities"`為索引鍵傳回的值`type`。

`"type": "Microsoft.ManagedIdentity/userAssignedIdentities"`

## <a name="delete-a-user-assigned-managed-identity"></a>刪除使用者指派的受控識別

若要刪除使用者指派的受控識別，請使用 [az identity delete](/cli/azure/identity#az-identity-delete) 命令。  -n 參數會指定其名稱，而 -g 參數會指定建立使用者指派之受控識別所在的資源群組。 將 `<USER ASSIGNED IDENTITY NAME>` 和 `<RESOURCE GROUP>` 參數取代為您自己的值：

 ```azurecli-interactive
az identity delete -n <USER ASSIGNED IDENTITY NAME> -g <RESOURCE GROUP>
```
> [!NOTE]
> 將使用者指派的受控識別從受指派的任何資源中刪除，並不會移除參考。 請使用 `az vm/vmss identity remove` 命令將其從 VM/VMSS 中移除

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 身分識別命令的完整清單，請參閱 [az identity](/cli/azure/identity)。

如需如何將使用者指派之受控識別指派至 Azure VM 的相關資訊，請參閱[使用 Azure CLI 對 Azure VM 設定 Azure 資源的受控識別](qs-configure-cli-windows-vm.md#user-assigned-managed-identity)


 
