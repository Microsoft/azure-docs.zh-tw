---
title: 在 Azure 中建立從 Azure DevOps 部署的函式 | Microsoft Docs
description: 建立函式應用程式和從 Azure DevOps 部署函式程式碼
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.date: 07/03/2018
ms.topic: sample
ms.service: azure-functions
ms.custom: mvc
ms.openlocfilehash: 680115f64b16059c415d295cf7c6a15301de5a3f
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166095"
---
# <a name="create-a-function-app-and-deploy-function-code-from-azure-devops"></a>建立函式應用程式和從 Azure DevOps 部署函式程式碼

本主題示範如何使用 Azure Functions，以使用[使用情況方案](../functions-scale.md#consumption-plan)來建立[無伺服器](https://azure.microsoft.com/solutions/serverless/)函數應用程式。 函式應用程式 (即您函式的容器) 是從 Azure DevOps 存放庫持續部署。 

[!INCLUDE [upgrade runtime](../../../includes/functions-cli-version-note.md)]

若要完成本主題，您必須具備：

* 包含函式應用程式專案且您具有系統管理權限的 Azure DevOps 存放庫。
* 存取 Azure DevOps 存放庫的[個人存取權杖 (PAT)](https://docs.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您不是在本機使用 Azure CLI，則必須安裝和使用 2.0 版或更新版本。 若要判斷 Azure CLI 版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

此指令碼範例會建立 Azure 函式應用程式，並從 Azure DevOps 部署函式程式碼。

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/deploy-function-app-with-function-vsts/deploy-function-app-with-function-vsts.sh?highlight=3-4 "Azure Service")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、儲存體帳戶、函數應用程式和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立函式應用程式所需的儲存體帳戶。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在無伺服器[取用方案](../functions-scale.md#consumption-plan)中建立函式應用程式。 |
| [az functionapp deployment source config](https://docs.microsoft.com/cli/azure/functionapp/deployment/source#az-functionapp-deployment-source-config) | 將函式應用程式關聯到 Git 或 Mercurial 存放庫。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Functions 文件](../functions-cli-samples.md)中找到其他 Azure Functions CLI 指令碼範例。
