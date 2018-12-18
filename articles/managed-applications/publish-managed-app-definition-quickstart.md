---
title: 建立 Azure 受控應用程式定義 | Microsoft Docs
description: 示範如何建立 Azure 受控應用程式，以供組織成員使用。
services: managed-applications
author: tfitzmac
ms.service: managed-applications
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.date: 10/04/2018
ms.author: tomfitz
ms.openlocfilehash: 1f80d7e63d994f0e3eb3733b99afaa1b056f4686
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804901"
---
# <a name="publish-an-azure-managed-application-definition"></a>發佈 Azure 受控應用程式定義

本快速入門將簡介如何使用受控應用程式。 您可以為您組織中的使用者，將受控應用程式定義新增至內部目錄。 為簡化指示，我們已為受控應用程式建置檔案。 這些檔案皆可透過 GitHub 取得。 您可以從[建立服務目錄應用程式](publish-service-catalog-app.md)教學課程中，了解如何建置這些檔案。

當您完成時，您會有名為 **appDefinitionGroup** 的資源群組，其中具有受控應用程式定義。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-resource-group-for-definition"></a>建立定義的資源群組

資源群組中存在受控應用程式定義。 資源群組是在其中部署與管理 Azure 資源的邏輯集合。

若要建立資源群組，請使用下列命令：

```azurecli-interactive
az group create --name appDefinitionGroup --location westcentralus
```

## <a name="create-the-managed-application-definition"></a>建立受控應用程式定義

定義受控應用程式時，您會為取用者選取管理資源的使用者、群組或應用程式。 此身分識別會根據指派的角色，取得受控資源群組的權限。 一般來說，您會建立 Azure Active Directory 群組來管理資源。 不過，在本文中您會使用自己的身分識別。

若要取得您身分識別的物件識別碼，請在下列命令中提供您的使用者主體名稱：

```azurecli-interactive
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
```

接下來，您需要的角色定義識別碼，是您想要授與使用者存取權的 RBAC 內建角色。 下列命令會顯示如何取得「擁有者」角色的角色定義識別碼：

```azurecli-interactive
roleid=$(az role definition list --name Owner --query [].name --output tsv)
```

現在，建立受控應用程式定義資源。 受控應用程式只包含儲存體帳戶。

```azurecli-interactive
az managedapp definition create \
  --name "ManagedStorage" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Storage Account" \
  --description "Managed Azure Storage Account" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "https://raw.githubusercontent.com/Azure/azure-managedapp-samples/master/samples/201-managed-storage-account/managedstorage.zip"
```

命令完成時，您的資源群組中會有受控應用程式定義。 

上述範例中使用的部分參數：

* **resource-group**：資源群組的名稱，其中會建立受控應用程式定義。
* **lock-level**：放在受控資源群組的鎖定類型。 它可以避免客戶在此資源群組上執行非預期的作業。 目前唯一支援的鎖定等級是 ReadOnly。 指定 ReadOnly 時，客戶只能讀取受控資源群組中存在的資源。 獲取受控資源群組存取權的發行者身分識別免除鎖定。
* **authorizations**：描述用來授與權限給受控資源群組的主體識別碼及角色定義識別碼。 它是以 `<principalId>:<roleDefinitionId>` 格式來指定。 如果需要多個值，請在表單中指定它們：`<principalId1>:<roleDefinitionId1> <principalId2>:<roleDefinitionId2>`。 值之間以空格分隔。
* **package-file-uri**：該位置有包含必要檔案的.zip 套件。 套件必須有 **mainTemplate.json** 和 **createUiDefinition.json** 檔案。 **mainTemplate.json** 會對建立為受控應用程式一部分的 Azure 資源進行定義。 此範本與一般 Resource Manager 範本不同。 **createUiDefinition.json**會透過 入口網站為建立受控應用程式的使用者產生使用者介面。

## <a name="next-steps"></a>後續步驟

您已發佈受控應用程式定義。 現在，深入了解如何部署該定義的執行個體。

> [!div class="nextstepaction"]
> [快速入門：部署服務目錄應用程式](deploy-service-catalog-quickstart.md)