---
title: Azure CLI 指令碼範例 - 在 App Service 方案中建立函式應用程式 | Microsoft Docs
description: Azure CLI 指令碼範例 - 在 App Service 方案中建立函式應用程式
services: functions
documentationcenter: functions
author: ggailey777
manager: jeconnoc
ms.assetid: 0e221db6-ee2d-4e16-9bf6-a456cd05b6e7
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/03/2018
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 0f0c8bbca6c93f68627308762f99fb57c8f397cf
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946099"
---
# <a name="create-a-function-app-in-an-app-service-plan"></a>在 App Service 方案中建立函式應用程式

這個 Azure Functions 範例指令碼會建立函式應用程式，這是您的函式容器。 系統會使用專用的 App Service 方案建立函式應用程式，這表示您的伺服器資源會永遠啟用。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

此指令碼會使用專用的 [App Service 方案](../functions-scale.md#app-service-plan)建立 Azure 函數應用程式。

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-app-service-plan/create-function-app-app-service-plan.sh "Create an Azure Function on an App Service plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

下表中的每個命令都會連結至命令特定的文件。 此指令碼會使用下列命令：

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立 Azure 儲存體帳戶。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create) | 建立 App Service 方案。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在 App Service 方案中建立函式應用程式。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Functions 文件](../functions-cli-samples.md)中找到其他 Azure Functions CLI 指令碼範例。
