---
title: 使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽) | Microsoft Docs
description: 使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 06/01/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 90868961475c2e9d0ac7d28c5d9a50c8eb281675
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525200"
---
# <a name="authenticate-access-to-azure-storage-using-azure-active-directory-preview"></a>使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)

Azure 儲存體支援使用 Azure Active Directory (Azure AD) 對 Blob 和佇列服務進行驗證和授權。 您可以藉由 Azure AD，使用角色型存取控制 (RBAC) 來授與存取權給使用者、群組或應用程式服務主體。 

使用 Azure AD 授權應用程式存取 Azure 儲存體，能比其他授權選項提供較高的安全性，也更方便使用。 雖然您可以繼續使用共用金鑰授權於應用程式，但使用 Azure AD 就不需要將帳戶存取金鑰和程式碼一起儲存。 同樣地，您可以繼續使用共用存取簽章 (SAS) 將細部存取權授與儲存體帳戶中的資源，但 Azure AD 提供類似功能，卻不必管理 SAS 權杖或擔心需要撤銷遭盜用的 SAS。

## <a name="about-the-preview"></a>關於預覽

請留意有關預覽的下列事項：

- 只有預覽版才提供 Blob 和佇列服務的 Azure AD 整合。
- 所有公用區域中的 GPv1、GPv2 及 Blob 儲存體帳戶均提供 Azure AD 整合。 
- 只支援使用 Resource Manager 部署模型建立的儲存體帳戶。 
- 即將推出在 Azure 儲存體分析記錄中支援呼叫者身分識別資訊。
- 目前支援 Azure AD 授與標準儲存體帳戶中資源的存取權。 即將推出支援進階儲存體帳戶中授與分頁 Blob 的存取權。
- Azure 儲存體支援內建與自訂 RBAC 角色。 您可以指定訂用帳戶、資源群組、儲存體帳戶或個別的容器或佇列範圍內的角色。
- 目前支援 Azure AD 整合的 Azure 儲存體用戶端程式庫包括：
    - [.NET](https://www.nuget.org/packages/WindowsAzure.Storage/9.2.0)
    - [Java](http://mvnrepository.com/artifact/com.microsoft.azure/azure-storage) (使用 7.1.x 預覽版)
    - Python
        - [Blob](https://github.com/Azure/azure-storage-python/releases/tag/v1.2.0rc1-blob)
        - [佇列](https://github.com/Azure/azure-storage-python/releases/tag/v1.2.0rc1-queue)
    - [Node.js](https://www.npmjs.com/package/azure-storage)
    - [JavaScript](https://aka.ms/downloadazurestoragejs))

> [!IMPORTANT]
> 此預覽僅適用於非生產環境。 Azure 儲存體的 Azure AD 整合宣告上市之前，無法使用生產服務等級協定 (SLA)。 如果您的案例尚未支援 Azure AD 整合，請繼續在應用程式中使用您的共用金鑰授權或 SAS 權杖。
>
> 在預覽期間，RBAC 角色指派可能需要五分鐘的時間傳播。
>
> Azure AD 與 Azure 儲存體的整合需要您使用 HTTPS 來進行 Azure 儲存體作業。

## <a name="get-started-with-azure-ad-for-storage"></a>開始使用 Azure AD 於儲存體

使用 Azure AD 與 Azure 儲存體的整合，第一個步驟就是將儲存體資料的 RBAC 角色指派給服務主體 (使用者、群組或應用程式服務主體) 或受控服務識別 (MSI)。 RBAC 角色包含容器和佇列的一般權限集合。 若要深入了解 Azure 儲存體的 RBAC 角色，請參閱[使用 RBAC 管理儲存體資料的存取權限 (預覽)](storage-auth-aad-rbac.md)。

若要使用 Azure AD 授與應用程式中的儲存體資源存取權，您需要從程式碼中要求 OAuth 2.0 存取權杖。 若要了解如何要求存取權杖，並用來授權對 Azure 儲存體的要求，請參閱[從 Azure 儲存體應用程式中使用 Azure AD 進行驗證 (預覽)](storage-auth-aad-app.md)。 如果您使用 Azure 受控服務識別 (MSI)，請參閱[從 Azure VM 受控服務識別使用 Azure AD 進行驗證 (預覽)](storage-auth-aad-msi.md)。

Azure CLI 和 PowerShell 現在支援以 Azure AD 身分識別登入。 以 Azure AD 身分識別登入之後，工作階段便會在該身分識別下執行。 若要進一步了解，請參閱[使用 Azure AD 身分識別以使用 CLI 或 PowerShell 存取 Azure 儲存體 (預覽)](storage-auth-aad-script.md)。

## <a name="next-steps"></a>後續步驟

如需 Azure Blob 和佇列的 Azure AD 整合詳細資訊，請參閱 Azure 儲存體小組部落格文章[宣布適用於 Azure 儲存體的 Azure AD 驗證預覽](https://azure.microsoft.com/blog/announcing-the-preview-of-aad-authentication-for-storage/) \(英文\)。
