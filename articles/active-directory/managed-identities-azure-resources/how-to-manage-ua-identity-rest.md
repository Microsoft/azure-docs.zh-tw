---
title: 如何使用 REST 管理 Azure 使用者指派的受控識別
description: 逐步說明如何建立、列出和刪除使用者指派的受控識別，以進行 REST API 呼叫。
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
ms.date: 06/26/2018
ms.author: daveba
ms.openlocfilehash: dc7abd4bdec30ae870ff6add33d4b9b1c08b5bbd
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44159634"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-rest-api-calls"></a>使用 REST API 呼叫建立、列出和刪除使用者指派的受控識別

[!INCLUDE [preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

Azure 資源的受控識別能夠讓 Azure 服務向支援 Azure AD 驗證的服務進行驗證，而不需要您程式碼中的認證資料。 

在本文中，您會了解如何使用 CURL 進行 REST API 呼叫，來建立、列出和刪除使用者指派的識別。

## <a name="prerequisites"></a>必要條件

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 如果您使用的是 Windows，請安裝[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)，或使用 Azure 入口網站中的 [Azure Cloud Shell](../../cloud-shell/overview.md)。
- 如果您使用的是[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)或[Linux 散發作業系統](/cli/azure/install-azure-cli-apt?view=azure-cli-latest)，請[安裝 Azure CLI 本機主控台](/cli/azure/install-azure-cli)。
- 如果您使用的是 Azure CLI 本機主控台，請登入 Azure ，登入時請使用與您想要用於部署或擷取使用者指派的受控識別的 Azure 訂用帳戶相關聯之帳戶，搭配使用 `az login` 登入。
- 若要執行本文中的管理作業，您的帳戶需要下列角色指派：
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立、讀取 (列出)、更新和刪除使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可讀取 (列出) 使用者指派受控識別的屬性。
- 使用 `az account get-access-token` 擷取持有人存取權杖來執行下列使用者指派的受控識別的作業。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-user-assigned-managed-identity"></a>建立使用者指派的受控識別 

若要建立使用者指派的受控識別，請使用 Azure Resource Manager API 的下列 CURL 要求。 以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>`、`<USER ASSIGNED IDENTITY NAME>`、`<LOCATION>` 和 `<ACCESS TOKEN>` 值：

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

## <a name="list-user-assigned-managed-identities"></a>列出使用者指派的受控識別

若要列出使用者指派的受控識別，請使用 Azure Resource Manager API 的下列 CURL 要求。 以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 和 `<ACCESS TOKEN>` 值：

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```
## <a name="delete-a-user-assigned-managed-identity"></a>刪除使用者指派的受控識別

若要刪除使用者指派的受控識別，請使用 Azure Resource Manager API 的下列 CURL 要求。 以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 和 `<ACCESS TOKEN>` 參數：

> [!NOTE]
> 將使用者指派的受控識別從受指派的任何資源中刪除，並不會移除參考。 若要使用 CURL 從虛擬機器移除使用者指派的受控識別，請參閱[從 Azure VM 移除使用者指派的識別](qs-configure-rest-vm.md#remove-a-user-assigned identity-from-an-azure-vm)。

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

## <a name="next-steps"></a>後續步驟

如需如何使用 CURL 將使用者指派的受控識別指派給 Azure VM/VMSS 的資訊，請參閱[使用 REST API 呼叫在 Azure VM 上設定 Azure 資源的受控識別](qs-configure-rest-vm.md#user-assigned-managed-identity) (英文) 和[使用 REST API 呼叫在虛擬機器擴展集上設定 Azure 資源的受控識別](qs-configure-rest-vmss.md#user-assigned-managed-identity) \(英文\)。


