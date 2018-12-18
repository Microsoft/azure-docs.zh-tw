---
title: 了解 Azure RBAC 中的角色定義 | Microsoft Docs
description: 了解角色型存取控制 (RBAC) 中的角色定義，以對 Azure 中的資源進行微調存取管理。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/17/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: ''
ms.openlocfilehash: 416565a248fc9ef0861b5309d71fdac3b8fccc22
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39116056"
---
# <a name="understand-role-definitions"></a>了解角色定義

如果您想要嘗試了解角色的運作方式或建立自己的[自訂角色](custom-roles.md)，最好先了解如何定義角色。 本文詳細描述角色定義並提供一些範例。

## <a name="role-definition-structure"></a>角色定義結構

「角色定義」是權限集合。 有時簡稱為「角色」。 角色定義會列出可執行的作業，例如讀取、寫入和刪除。 也可能列出無法執行的作業或與基礎資料相關的作業。 角色定義的結構如下：

```
Name
Id
IsCustom
Description
Actions []
NotActions []
DataActions []
NotDataActions []
AssignableScopes []
```

用來指定作業的字串具有下列格式：

- `{Company}.{ProviderName}/{resourceType}/{action}`

作業字串的 `{action}` 部分指定您可以對資源類型執行的作業類型。 例如，您將會在 `{action}` 中看到下列子字串：

| 動作子字串    | 說明         |
| ------------------- | ------------------- |
| `*` | 此萬用字元會授與所有符合字串之作業的存取權。 |
| `read` | 啟用讀取作業 (GET)。 |
| `write` | 啟用寫入作業 (PUT、POST 和 PATCH)。 |
| `delete` | 啟用刪除作業 (DELETE)。 |

以下是 JSON 格式的[參與者](built-in-roles.md#contributor)角色定義。 `Actions` 下的萬用字元 (`*`) 作業表示指派給這個角色的主體可以執行所有動作；換句話說，它可以管理所有項目。 這包括未來 Azure 新增資源類型時所定義的動作。 `NotActions` 下的作業會從 `Actions` 扣除。 如果是[參與者](built-in-roles.md#contributor)角色，`NotActions` 會移除此角色管理資源存取權及指派資源存取權的功能。

```json
{
    "Name":  "Contributor",
    "Id":  "b24988ac-6180-42a0-ab88-20f7382dd24c",
    "IsCustom":  false,
    "Description":  "Lets you manage everything except access to resources.",
    "Actions":  [
                    "*"
                ],
    "NotActions":  [
                       "Microsoft.Authorization/*/Delete",
                       "Microsoft.Authorization/*/Write",
                       "Microsoft.Authorization/elevateAccess/Action"
                   ],
    "DataActions":  [

                    ],
    "NotDataActions":  [

                       ],
    "AssignableScopes":  [
                             "/"
                         ]
}
```

## <a name="management-and-data-operations-preview"></a>管理和資料作業 (預覽)

管理作業的角色型存取控制是在角色定義的 `Actions` 和 `NotActions` 屬性中指定。 以下是 Azure 中的一些管理作業範例：

- 管理儲存體帳戶的存取權
- 建立、更新或刪除 Blob 容器
- 刪除資源群組及其所有資源

您的資料不會繼承管理存取權。 此隔離可防止具有萬用字元 (`*`) 的角色不受限地存取您的資料。 例如，如果使用者具有訂用帳戶的[讀取者](built-in-roles.md#reader)角色，則可以檢視儲存體帳戶，但預設為無法檢視基礎資料。

在此之前，資料作業不可使用角色型存取控制。 資料作業的授權會因為資源提供者不同而有差異。 但管理作業所使用的相同角色型存取控制授權模型已延伸到資料作業 (目前處於預覽狀態)。

為支援資料作業，已新增資料屬性到角色定義結構中。 資料作業會在 `DataActions` 和 `NotDataActions` 屬性中指定。 藉由新增這些資料屬性，可繼續維持管理和資料之間的分隔。 這可避免目前具有萬用字元 (`*`) 的角色指派突然存取資料。 以下是可在 `DataActions` 和 `NotDataActions`中指定的一些資料作業：

- 讀取容器中的 Blob 清單
- 將儲存體 Blob 寫入容器中
- 刪除佇列中的訊息

以下是[儲存體 Blob 資料讀取器 (預覽)](built-in-roles.md#storage-blob-data-reader-preview) 的角色定義，其中包含 `Actions` 和 `DataActions` 屬性中的作業。 此角色可讓您讀取 Blob 容器和基礎 Blob 資料。

```json
{
    "Name":  "Storage Blob Data Reader (Preview)",
    "Id":  "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
    "IsCustom":  false,
    "Description":  "Allows for read access to Azure Storage blob containers and data",
    "Actions":  [
                    "Microsoft.Storage/storageAccounts/blobServices/containers/read"
                ],
    "NotActions":  [

                   ],
    "DataActions":  [
                        "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
                    ],
    "NotDataActions":  [

                       ],
    "AssignableScopes":  [
                             "/"
                         ]
}
```

您只能將資料作業新增至 `DataActions` 和 `NotDataActions` 屬性。 資源提供者會藉由將 `isDataAction` 屬性設為 `true`，來識別哪些作業是資料作業。 若要查看 `isDataAction` 為 `true` 的作業清單，請參閱[資源提供者作業](resource-provider-operations.md)。 在角色定義中，沒有資料作業的角色不需要有 `DataActions` 和 `NotDataActions` 屬性。

所有管理作業 API 呼叫的授權都會由 Azure Resource Manager 處理。 資料作業 API 呼叫的授權是由資源提供者或 Azure Resource Manager 處理。

### <a name="data-operations-example"></a>資料作業範例

若要進一步了解管理和資料作業如何運作，我們來看特定範例。 Alice 已在訂用帳戶範圍上獲得[擁有者](built-in-roles.md#owner)角色的指派。 Bob 已在儲存體帳戶範圍上獲得[儲存體 Blob 資料參與者 (預覽)](built-in-roles.md#storage-blob-data-contributor-preview) 角色的指派。 此範例如下圖所示。

![角色型存取控制已延伸為可支援管理和資料作業](./media/role-definitions/rbac-management-data.png)

Alice 的[擁有者](built-in-roles.md#owner)角色和 Bob 的[儲存體 Blob 資料參與者 (預覽)](built-in-roles.md#storage-blob-data-contributor-preview) 角色有下列動作：

擁有者

&nbsp;&nbsp;&nbsp;&nbsp;Actions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`*`

儲存體 Blob 資料參與者 (預覽)

&nbsp;&nbsp;&nbsp;&nbsp;Actions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/write`<br>
&nbsp;&nbsp;&nbsp;&nbsp;DataActions<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`

由於 Alice 在訂用帳戶範圍上有萬用字元 (`*`) 動作，她的權限可向下繼承，因此她可執行所有管理動作。 不過，Alice 無法執行資料作業。 例如，根據預設，Alice 無法讀取容器內的 Blob，但她可以讀取、寫入和刪除容器。

Bob 的權限僅限於[儲存體 Blob 資料參與者 (預覽)](built-in-roles.md#storage-blob-data-contributor-preview) 角色中指定的 `Actions` 和 `DataActions`。 以此角色為基礎，Bob 可以執行管理和資料作業。 例如，Bob 可以讀取、寫入和刪除指定儲存體帳戶中的容器，而且他也可以讀取、寫入和刪除 Blob。

### <a name="what-tools-support-using-rbac-for-data-operations"></a>哪些工具支援對資料作業使用 RBAC？

若要檢視及使用資料作業，您必須有正確版本的工具或 SDK：

| 工具  | 版本  |
|---------|---------|
| [Azure PowerShell](/powershell/azure/install-azurerm-ps) | 5.6.0 或更新版本 |
| [Azure CLI](/cli/azure/install-azure-cli) | 2.0.30 或更新版本 |
| [Azure for .NET](/dotnet/azure/) | 2.8.0-預覽或更新版本 |
| [Azure SDK for Go](/go/azure/azure-sdk-go-install) | 15.0.0 或更新版本 |
| [Azure for Java](/java/azure/) | 1.9.0 或更新版本 |
| [Azure for Python](/python/azure) | 0.40.0 或更新版本 |
| [Azure SDK for Ruby](https://rubygems.org/gems/azure_sdk) | 0.17.1 或更新版本 |

## <a name="actions"></a>動作

`Actions` 權限會指定角色所允許執行的管理作業。 它是識別 Azure 資源提供者的安全性實體作業的作業字串集合。 以下是可用於 `Actions` 中的一些管理作業範例。

| 作業字串    | 說明         |
| ------------------- | ------------------- |
| `*/read` | 授與所有 Azure 資源提供者的所有資源類型之讀取作業的存取權。|
| `Microsoft.Compute/*` | 授與對 Microsoft.Compute 資源提供者中所有資源類型之所有作業的存取權。|
| `Microsoft.Network/*/read` | 授與 Microsoft.Network 資源提供者的所有資源類型之讀取作業的存取權。|
| `Microsoft.Compute/virtualMachines/*` | 授與虛擬機器及其子資源類型之所有作業的存取權。|
| `microsoft.web/sites/restart/Action` | 授與重新啟動 Web 應用程式的存取權。|

## <a name="notactions"></a>NotActions

`NotActions` 權限指定從所允許 `Actions` 中排除的管理作業。 如果排除限制的作業可更輕鬆地定義您要允許的作業集合，請使用 `NotActions` 權限。 角色 (有效權限) 所授與之存取權的計算方式是將 `Actions` 作業扣除 `NotActions` 作業。

> [!NOTE]
> 如果為使用者指派會排除 `NotActions` 中作業的角色，並指派授與相同作業存取權的第二個角色，即會允許使用者執行該作業。 `NotActions` 不是拒絕規則 - 它只是一個便利的方式，可以在需要排除特定作業時建立允許的作業集合。
>

## <a name="dataactions-preview"></a>DataActions (預覽)

`DataActions` 權限會指定角色允許對物件內資料執行的管理作業。 例如，如果使用者有儲存體帳戶的讀取 Blob 資料存取權，則他們可讀取該儲存體帳戶中的 Blob。 以下是可用於 `DataActions` 中的一些資料作業範例。

| 作業字串    | 說明         |
| ------------------- | ------------------- |
| `Microsoft.Storage/storageAccounts/ blobServices/containers/blobs/read` | 傳回 Blob 或 Blob 清單。 |
| `Microsoft.Storage/storageAccounts/ blobServices/containers/blobs/write` | 傳回寫入 Blob 的結果。 |
| `Microsoft.Storage/storageAccounts/ queueServices/queues/messages/read` | 傳回訊息。 |
| `Microsoft.Storage/storageAccounts/ queueServices/queues/messages/*` | 傳回訊息或寫入或刪除訊息的結果。 |

## <a name="notdataactions-preview"></a>NotDataActions (預覽)

`NotDataActions` 權限可指定從所允許 `DataActions` 中排除的資料作業。 角色 (有效權限) 所授與之存取權的計算方式是將 `DataActions` 作業扣除 `NotDataActions` 作業。 每個資源提供者會提供個別的一組 API 來完成資料作業。

> [!NOTE]
> 如果為使用者指派的角色已在 `NotDataActions` 中排除一個資料作業，並且指派授與相同資料作業存取權的第二個角色給使用者，即會允許使用者執行該資料作業。 `NotDataActions` 不是拒絕規則 - 它只是一個便利的方式，可以在需要排除特定資料作業時建立允許的資料作業集合。
>

## <a name="assignablescopes"></a>AssignableScopes

`AssignableScopes` 屬性指定角色可指派的範圍 (管理群組 (目前處於預覽階段)、訂用帳戶、資源群組或資源)。 您可以讓角色僅指派給需要它的訂用帳戶或資源群組，不會干擾其餘訂用帳戶或資源群組的使用者體驗。 您至少必須使用一個管理群組、訂用帳戶、資源群組或資源識別碼。

內建角色的 `AssignableScopes` 設定為根目錄範圍 (`"/"`)。 根目錄範圍表示角色可指派給所有範圍。 有效的可指派範圍範例包括：

| 案例 | 範例 |
|----------|---------|
| 角色可指派給單一訂用帳戶 | `"/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e"` |
| 角色可指派給兩個訂用帳戶 | `"/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e", "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"` |
| 角色只能指派給網路資源群組 | `"/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e/resourceGroups/Network"` |
| 角色可指派給所有範圍 | `"/"` |

如需自訂角色的 `AssignableScopes` 相關資訊，請參閱[自訂角色](custom-roles.md)。

## <a name="next-steps"></a>後續步驟

* [內建角色](built-in-roles.md)
* [自訂角色](custom-roles.md)
* [Azure Resource Manager 資源提供者作業](resource-provider-operations.md)
