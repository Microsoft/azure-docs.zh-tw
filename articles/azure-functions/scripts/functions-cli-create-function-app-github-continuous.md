---
title: 在 Azure 中建立從 GitHub 部署的函式 | Microsoft Docs
description: 建立函數應用程式，並使用 Azure Functions 從 GitHub 存放庫部署函式程式碼。
services: functions
ms.service: azure-functions
keywords: ''
ms.devlang: azurecli
author: ggailey777
ms.author: glenga
ms.date: 07/03/2018
ms.topic: sample
ms.custom: mvc
ms.openlocfilehash: 2915ff498b53b8af8fdbf401c1c75583517c2f75
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956757"
---
# <a name="create-a-function-app-in-azure-that-is-deployed-from-github"></a>在 Azure 中建立從 GitHub 部署的函式應用程式

此 Azure Functions 範例指令碼會使用[取用方案](../functions-scale.md#consumption-plan)建立函式應用程式及其相關資源。 此指令碼也會您的函式程式碼，以便從 GitHub 存放庫持續進行部署。 

[!INCLUDE [upgrade runtime](../../../includes/functions-cli-version-note.md)]

在此範例中，您需要：

* 一個含有函式程式碼的 GitHub 存放庫 (您必須有此存放庫的系統管理權限)。
* 您 GitHub 帳戶的[個人存取權杖 (PAT)](https://help.github.com/articles/creating-an-access-token-for-command-line-use)。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您不是在本機使用 Azure CLI，則必須安裝和使用 2.0 版或更新版本。 若要判斷 Azure CLI 版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

此指令碼範例會建立 Azure 函式應用程式，並從 GitHub 部署函式程式碼。

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/deploy-function-app-with-function-github-continuous/deploy-function-app-with-function-github-continuous.sh?highlight=3-4 "Azure Service")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

下表中的每個命令都會連結至命令特定的文件。 此指令碼會使用下列命令：

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立函式應用程式所需的儲存體帳戶。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在無伺服器[取用方案](../functions-scale.md#consumption-plan)中建立函式應用程式，並與 Git 或 Mercurial 存放庫產生關聯。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Functions 文件](../functions-cli-samples.md)中找到其他 Azure Functions CLI 指令碼範例。
