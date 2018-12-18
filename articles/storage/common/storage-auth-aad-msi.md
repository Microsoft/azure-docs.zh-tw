---
title: 使用 Azure 資源的 Azure Active Directory 受控識別來驗證 Blob 和佇列的存取權 (預覽) - Azure 儲存體 | Microsoft Docs
description: Azure Blob 和佇列儲存體支援使用 Azure 資源的受控識別來進行 Azure Active Directory 驗證。 您可以使用 Azure 資源的受控識別，從執行於 Azure 虛擬機器、函式應用程式和虛擬機器擴展集等項目中的應用程式，驗證 Blob 和佇列的存取權。 藉由使用 Azure 資源的受控識別並利用 Azure AD 驗證的功能，您可以避免在執行於雲端中的應用程式內儲存認證。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/05/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 67e0731c1f10bb635baa4e0d1a26dce0a336b555
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44090350"
---
# <a name="authenticate-access-to-blobs-and-queues-with-azure-managed-identities-for-azure-resources-preview"></a>使用 Azure 資源的 Azure 受控識別來驗證 Blob 和佇列的存取權 (預覽)

Azure Blob 和佇列儲存體支援使用 [Azure 資源的受控識別](../../active-directory/managed-identities-azure-resources/overview.md)來進行 Azure Active Directory (Azure AD) 驗證。 您可以使用 Azure 資源的受控識別，從執行於 Azure 虛擬機器 (VM)、函式應用程式和虛擬機器擴展集等項目中的應用程式，驗證 Blob 和佇列的存取權。 藉由使用 Azure 資源的受控識別並利用 Azure AD 驗證的功能，您可以避免在執行於雲端中的應用程式內儲存認證。  

若要對 Blob 容器或佇列授與受控識別的權限，請將角色型存取控制 (RBAC) 角色 (包含該資源在適當範圍內的權限) 指派給該受控識別。 如需儲存體中 RBAC 角色的詳細資訊，請參閱[使用 RBAC 管理儲存體資料的存取權限 (預覽)](storage-auth-aad-rbac.md)。 

本文說明如何從 Azure VM 使用受控識別向 Azure Blob 或佇列儲存體進行驗證。  

[!INCLUDE [storage-auth-aad-note-include](../../../includes/storage-auth-aad-note-include.md)]

## <a name="enable-managed-identities-on-a-vm"></a>在 VM 上啟用受控識別

若要能夠從 VM 使用 Azure 資源的受控識別來驗證 Blob 和佇列的存取權，您必須先在該 VM 上啟用 Azure 資源的受控識別。 若要了解如何啟用 Azure 資源的受控識別，請參閱下列其中一篇文章：

- [Azure 入口網站](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Azure Resource Manager 範本](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Azure SDK](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="get-a-managed-identity-access-token"></a>取得受控識別的存取權杖

若要使用受控識別進行驗證，應用程式或指令碼必須取得受控識別存取權杖。 若要了解如何取得存取權杖，請參閱[如何在 Azure VM 上使用 Azure 資源的受控識別來取得存取權杖](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)。

## <a name="net-code-example-create-a-block-blob"></a>.NET 程式碼範例：建立區塊 Blob

此程式碼範例假設您有受控識別存取權杖。 存取權杖可用來授權受控識別以建立區塊 Blob。

### <a name="add-references-and-using-statements"></a>新增參考並使用陳述式  

在 Visual Studio 中，安裝 Azure 儲存體用戶端程式庫的預覽版本。 在 [工具] 功能表中，依序選取 [Nuget 套件管理員] 及 [套件管理員主控台]。 將下列命令輸入到主控台：

```
Install-Package https://www.nuget.org/packages/WindowsAzure.Storage  
```

將下列 using 陳述式新增至程式碼：

```dotnet
using Microsoft.WindowsAzure.Storage.Auth;
```

### <a name="create-credentials-from-the-managed-identity-access-token"></a>從受控識別存取權杖建立認證

若要建立區塊 Blob，請使用預覽套件所提供的 **TokenCredentials** 類別。 建構 **TokenCredentials** 的新執行個體，並傳入您先前取得的受控識別存取權杖：

```dotnet
// Create storage credentials from your managed identity access token.
TokenCredential tokenCredential = new TokenCredential(accessToken);
StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

// Create a block blob using the credentials.
CloudBlockBlob blob = new CloudBlockBlob(new Uri("https://storagesamples.blob.core.windows.net/sample-container/Blob1.txt"), storageCredentials);
``` 

> [!NOTE]
> Azure AD 與 Azure 儲存體的整合需要您使用 HTTPS 來進行 Azure 儲存體作業。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 儲存體的 RBAC 角色，請參閱[使用 RBAC 管理儲存體資料的存取權限 (預覽)](storage-auth-aad-rbac.md)。
- 若要深入了解如何從儲存體應用程式內授權容器和佇列的存取權，請參閱[使用 Azure AD 與儲存體應用程式](storage-auth-aad-app.md)。
- 若要了解如何以 Azure AD 身分識別登入 Azure CLI 和 PowerShell，請參閱[使用 Azure AD 身分識別透過 CLI 或 PowerShell 來存取 Azure 儲存體 (預覽)](storage-auth-aad-script.md)。
- 如需 Azure Blob 和佇列的 Azure AD 整合詳細資訊，請參閱 Azure 儲存體小組部落格文章，[宣告適用於 Azure 儲存體的 Azure AD 驗證預覽](https://azure.microsoft.com/blog/announcing-the-preview-of-aad-authentication-for-storage/)。
