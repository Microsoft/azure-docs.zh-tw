---
title: 使用 RBAC 管理容器和佇列的存取權限 (預覽版) - Azure 儲存體 | Microsoft Docs
description: 使用角色型存取控制 (RBA)，將存取 Blob 和佇列資料的角色指派給使用者、群組、應用程式服務主體或受控服務識別。 Azure 儲存體支援讓內建和自訂角色來取得容器和佇列的存取權限。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/07/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: c26eee05a59a10036138f81086b3f6a9e0de6d5d
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45737289"
---
# <a name="manage-access-rights-to-azure-storage-data-with-rbac-preview"></a>使用 RBAC 來管理 Azure 儲存體資料的存取權限 (預覽)

Azure Active Directory (Azure AD) 會透過[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview)，來授與存取受保護資源的權限。 Azure 儲存體會定義一組內建的 RBAC 角色，其中包含一般用來存取容器或佇列的權限集合。 當 RBAC 角色指派給 Azure AD 身分識別時，該身分識別會根據指定的範圍，獲得存取這些資源的權限。 存取權的範圍可以包括訂用帳戶、資源群組、儲存體帳戶或個別的容器或佇列層級。 您可以使用 Azure 入口網站、Azure 命令列工具或 Azure 管理 API 來指派 Azure 儲存體資源的存取權限。 

Azure AD 身分識別可以是使用者、群組或應用程式服務主體，也可以是 Azure 資源的受控識別。 安全性主體可以是使用者、群組或應用程式服務主體。 [Azure 資源的受控識別](../../active-directory/managed-identities-azure-resources/overview.md)是自動受控識別，用來在 Azure 虛擬機器中執行的應用程式、函式應用程式和虛擬機器擴展集等之中執行驗證。 如需 Azure AD 中的身分識別概觀，請參閱[了解 Azure 身分識別解決方案](https://docs.microsoft.com/azure/active-directory/understand-azure-identity-solutions)。

## <a name="rbac-roles-for-blobs-and-queues"></a>適用於 Blob 和佇列的 RBAC 角色

Azure 儲存體支援內建與自訂 RBAC 角色。 Azure 儲存體提供下列可與 Azure AD 搭配使用的內建 RBAC 角色：

- [儲存體 Blob 資料參與者 (預覽)](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor-preview)
- [儲存體 Blob 資料讀者 (預覽)](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-reader-preview)
- [儲存體佇列資料參與者 (預覽)](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-queue-data-contributor-preview)
- [儲存體佇列資料讀者 (預覽)](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-queue-data-reader-preview)

如需有關如何為 Azure 儲存體定義內建角色的詳細資訊，請參閱[了解角色定義](https://docs.microsoft.com/azure/role-based-access-control/role-definitions#management-and-data-operations-preview)。

您也可以定義能與容器和佇列搭配使用的自訂角色。 如需詳細資訊，請參閱[建立 Azure 角色型存取控制的自訂角色](https://docs.microsoft.com/azure/role-based-access-control/custom-roles)。 

[!INCLUDE [storage-auth-aad-note-include](../../../includes/storage-auth-aad-note-include.md)]

## <a name="assign-a-role-to-a-security-principal"></a>將角色指派給安全性主體

將 RBAC 角色指派給 Azure 身分識別，可授與存取您儲存體帳戶中容器或佇列的權限。 您可以將角色指派範圍限定在儲存體帳戶或特定容器或佇列。 下表摘要說明內建角色在不同範圍中授與的存取權限： 

|                                 |     Blob 資料參與者                                                 |     Blob 資料讀者                                                |     佇列資料參與者                                  |     佇列資料讀者                                 |
|---------------------------------|------------------------------------------------------------------------------|------------------------------------------------------------------------|----------------------------------------------------------------|----------------------------------------------------------|
|    範圍限定在訂用帳戶       |    訂用帳戶中所有容器和 Blob 的讀取/寫入存取       |    訂用帳戶中所有容器和 Blob 的讀取存取       |    訂用帳戶中所有佇列的讀取/寫入存取       |    訂用帳戶中所有佇列的讀取存取         |
|    範圍限定在資源群組     |    資源群組中所有容器和 Blob 的讀取/寫入存取     |    資源群組中所有容器和 Blob 的讀取存取     |    資源群組中所有佇列的讀取/寫入存取     |    資源群組中所有佇列的讀取存取     |
|    範圍限定在儲存體帳戶    |    儲存體帳戶中所有容器和 Blob 的讀取/寫入存取    |    儲存體帳戶中所有容器和 Blob 的讀取存取    |    儲存體帳戶中所有佇列的讀取/寫入存取    |    儲存體帳戶中所有佇列的讀取存取    |
|    範圍限定在容器/佇列    |    指定容器和其 Blob 的讀取/寫入存取              |    指定容器和其 Blob 的讀取存取              |    指定佇列的讀取/寫入存取                  |    指定佇列的讀取存取                    |

> [!NOTE]
> 身為 Azure 儲存體帳戶擁有者的您，並不會自動收到指派的資料存取權限。 您必須明確地將 Azure 儲存體的 RBAC 角色指派給自己。 您可以在訂用帳戶、資源群組、儲存體帳戶或容器/佇列層級上指派此角色。

如需呼叫 Azure 儲存體作業所需權限的詳細資訊，請參閱[呼叫 REST 作業的權限](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-azure-active-directory#permissions-for-calling-rest-operations)。

下列章節將示範如何對儲存體帳戶或個別容器範圍指派角色。

### <a name="assign-a-role-scoped-to-the-storage-account-in-the-azure-portal"></a>在 Azure 入口網站中對儲存體帳戶範圍指派角色

在 Azure 入口網站中指派內建角色，以授與儲存體帳戶中所有容器或佇列的存取權：

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的儲存體帳戶。
2. 選取儲存體帳戶，然後選取 [存取控制 (IAM)]，以顯示帳戶的存取控制設定。 按一下 [新增] 按鈕即可新增角色。

    ![顯示儲存體存取控制設定的螢幕擷取畫面](media/storage-auth-aad-rbac/portal-access-control.png)

3. 在 [新增權限] 視窗中，選取要指派給 Azure AD 身分識別的角色。 然後搜尋以找出您要對其指派該角色的身分識別。 如下圖所示，指派給使用者的是**儲存體 Blob 資料讀者 (預覽)** 角色。

    ![顯示如何指派 RBAC 角色的螢幕擷取畫面](media/storage-auth-aad-rbac/add-rbac-role.png)

4. 按一下 [檔案] 。 您對其指派角色的身分識別會出現在該角色下方。 如下圖所示，新增的使用者現在具有讀取所有儲存體帳戶中所有 Blob 資料的權限。

    ![顯示指派至角色的使用者清單螢幕擷取畫面](media/storage-auth-aad-rbac/account-scoped-role.png)

### <a name="assign-a-role-scoped-to-a-container-or-queue-in-the-azure-portal"></a>在 Azure 入口網站中對容器或佇列範圍指派角色

對容器或佇列範圍指派內建角色的步驟很類似。 這裡顯示的程序會對容器範圍指派角色，但是您可以依照相同的步驟，對佇列範圍指派角色： 

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的儲存體帳戶，並顯示帳戶的**概觀**。
2. 選取 [Blob 服務] 下方的 [瀏覽 Blob]。 
3. 找出您要為其指派角色的容器，並顯示容器的設定。 
4. 選取 [存取控制 (IAM)]，以顯示容器的存取控制設定。
5. 在 [新增權限] 視窗中，選取要指派給 Azure AD 身分識別的角色。 然後搜尋以找出您要對其指派該角色的身分識別。
6. 按一下 [檔案] 。 您對其指派角色的身分識別會出現在該角色下方。 如下圖所示，新增的使用者現在有權對名為 sample-container 容器讀取資料。

    ![顯示指派至角色的使用者清單螢幕擷取畫面](media/storage-auth-aad-rbac/container-scoped-role.png)

## <a name="next-steps"></a>後續步驟

- 若要深入了解 RBAC，請參閱[開始使用角色型存取控制](../../role-based-access-control/overview.md)。
- 若要了解如何使用 Azure PowerShell、Azure CLI 或 REST API 來指派和管理 RBAC 角色指派，請參閱下列文章：
    - [使用 Azure PowerShell 管理角色型存取控制 (RBAC)](../../role-based-access-control/role-assignments-powershell.md)
    - [使用 Azure CLI 管理角色型存取控制 (RBAC)](../../role-based-access-control/role-assignments-cli.md)
    - [使用 REST API 管理角色型存取控制 (RBAC)](../../role-based-access-control/role-assignments-rest.md)
- 若要深入了解如何從儲存體應用程式內授權容器和佇列的存取權，請參閱[使用 Azure AD 與 Azure 儲存體應用程式](storage-auth-aad-app.md)。
- 如需 Azure 容器和佇列的 Azure AD 整合詳細資訊，請參閱 Azure 儲存體小組部落格文章，[宣告適用於 Azure 儲存體的 Azure AD 驗證預覽](https://azure.microsoft.com/blog/announcing-the-preview-of-aad-authentication-for-storage/)。
- 
