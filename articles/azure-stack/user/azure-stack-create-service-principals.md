---
title: 為 Azure Stack 建立服務主體 | Microsoft Docs
description: 描述如何建立可以與 Azure Resource Manager 中的角色型存取控制搭配使用來管理資源存取權的服務主體。
services: azure-resource-manager
documentationcenter: na
author: sethmanheim
manager: femila
ms.assetid: 7068617b-ac5e-47b3-a1de-a18c918297b6
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/22/2018
ms.author: sethm
ms.reviewer: thoroet
ms.openlocfilehash: 891032e9f0a40b2a9df19ed25ffc19dd81507cab
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45628943"
---
# <a name="give-applications-access-to-azure-stack-resources-by-creating-service-principals"></a>建立服務主體以將應用程式存取權提供給 Azure Stack 資源

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以建立使用 Azure Resource Manager 的服務主體，以將應用程式存取權提供給 Azure Stack 資源。 服務主體可讓您使用[角色型存取控制](azure-stack-manage-permissions.md)委派特定權限。

最佳做法是您應該使用您應用程式的服務主體。 相較於使用自己的認證執行應用程式，使用服務主體更合適，因為：

* 您可以對服務主體指派不同於您自己帳戶權限的權限。 一般而言，服務主體的權限只會限制為 App 必須執行的確切權限。
* 如果您的腳色或職責變更，就不需要變更 App 的認證。
* 您可以使用憑證在執行自動指令碼時自動進行驗證。

## <a name="example-scenario"></a>範例案例

您的組態管理工具需要使用 Azure Resource Manager 來清查 Azure 資源。 您可以建立服務主體，並將它指派給讀者角色。 此角色可讓應用程式唯讀存取 Azure 資源。

## <a name="getting-started"></a>開始使用

使用本文中的步驟作為指南：

* 建立應用程式的服務主體。
* 註冊您的應用程式並建立驗證金鑰。
* 將您的應用程式指派給角色。

您針對 Azure Stack 設定 Active Directory 的方式可決定您建立服務主體的方式。 挑選下列其中一個選項：

* 為 [Azure Active Directory (Azure AD)](azure-stack-create-service-principals.md#create-service-principal-for-azure-ad) 建立服務主體。
* 為 [Active Directory Federation Services (AD FS)](azure-stack-create-service-principals.md#create-service-principal-for-ad-fs) 建立服務主體。

在 Azure AD 與 AD FS 中，將服務主體指派給角色的步驟相同。 建立服務主體之後，您可藉由將服務主體指派給角色來[委派權限](azure-stack-create-service-principals.md#assign-role-to-service-principal)。

## <a name="create-a-service-principal-for-azure-ad"></a>為 Azure AD 建立服務主體

如果 Azure Stack 使用 Azure AD 作為身分識別存放區，您可以在 Azure 入口網站中使用與 Azure 中相同的步驟建立服務主體。

>[!NOTE]
開始建立服務主體之前，請查看您有[必要的 Azure AD 權限](../../azure-resource-manager/resource-group-create-service-principal-portal.md#required-permissions)。

### <a name="create-service-principal"></a>建立服務主體

若要為您的應用程式建立服務主體：

1. 透過 [Azure 入口網站](https://portal.azure.com)登入您的 Azure 帳戶。
2. 選取 [Azure Active Directory] > [應用程式註冊] > [新增]。
3. 提供應用程式的名稱和 URL。 針對您想要建立的應用程式類型，選取 [Web 應用程式/API] 或 [原生]。 設定值之後，選取 [建立]。

### <a name="get-credentials"></a>取得認證

以程式設計方式登入時，使用應用程式的識別碼和驗證金鑰。 若要取得這些值：

1. 在 Active Directory 中，從 [應用程式註冊]選取您的應用程式。

2. 複製 [應用程式識別碼] 並儲存在您的應用程式碼中。 [範例應用程式](#sample-applications)中的應用程式會在參考 [應用程式識別碼] 時使用**用戶端識別碼**。

     ![應用程式的應用程式識別碼](./media/azure-stack-create-service-principal/image12.png)
3. 若要產生驗證金鑰，請選取 [金鑰]。

4. 提供金鑰的描述和金鑰的持續時間。 完成時，選取 [儲存]。

>[!IMPORTANT]
儲存金鑰之後，就會顯示金鑰**值**。 請記下此值，因為您之後就無法擷取金鑰。 將金鑰值儲存在應用程式可擷取的地方。

![已儲存金鑰的金鑰值警告。](./media/azure-stack-create-service-principal/image15.png)

最後一個步驟是[將您的應用程式指派給角色](azure-stack-create-service-principals.md#assign-role-to-service-principal)。

## <a name="create-service-principal-for-ad-fs"></a>為 AD FS 建立服務主體

如果您已使用 AD FS 將 Azure Stack 部署為身分識別存放區，您可以使用 PowerShell 進行下列工作：

* 建立服務主體。
* 將服務主體指派給角色。
* 使用服務主體的身分識別進行登入。

如需如何建立服務主體的詳細資訊，請參閱[為 AD FS 建立服務主體](../azure-stack-create-service-principals.md#create-service-principal-for-ad-fs)。

## <a name="assign-the-service-principal-to-a-role"></a>將服務主體指派給角色

若要存取您的訂用帳戶中的資源，您必須將應用程式指派給角色。 決定哪個角色代表應用程式的正確權限。 若要深入了解可用的角色，請參閱 [RBAC：內建角色](../../role-based-access-control/built-in-roles.md)。

>[!NOTE]
您可以在訂用帳戶、資源群組或資源的層級設定角色的範圍。 較低的範圍層級會繼承較高層級的權限。 例如，具有資源群組讀取者角色的應用程式，代表應用程式可以讀取資源群組中的任何資源。

使用下列步驟作為指南，將角色指派給服務主體。

1. 在 Azure Stack 入口網站中，瀏覽至您想要指派應用程式的範圍層級。 例如，若要在訂用帳戶範圍指派角色，請選取 [訂用帳戶]。

2. 選取應用程式所要指派至的訂用帳戶。 在此範例中，訂用帳戶是 Visual Studio Enterprise。

     ![選取 Visual Studio Enterprise 訂用帳戶以供指派](./media/azure-stack-create-service-principal/image16.png)

3. 針對訂用帳戶選取 [存取控制 (IAM)]。

     ![選取存取控制](./media/azure-stack-create-service-principal/image17.png)

4. 選取 [新增] 。

5. 選取您想要將應用程式指派給哪個角色。

6. 搜尋並選取您的應用程式。

7. 選取 [確定] 以完成角色指派。 您可以在使用者清單中看到應用程式已指派給該範圍的角色。

既然您已建立服務主體並指派角色，您的應用程式即可存取 Azure Stack 資源。

## <a name="next-steps"></a>後續步驟

[管理使用者權限](azure-stack-manage-permissions.md)
