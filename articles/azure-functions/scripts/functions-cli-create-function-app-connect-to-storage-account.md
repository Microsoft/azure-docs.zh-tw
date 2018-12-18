---
title: 建立連線至 Azure 儲存體的 Azure 函式 | Microsoft Docs
description: Azure CLI 指令碼範例 - 建立連線至 Azure 儲存體的 Azure 函式
services: functions
documentationcenter: functions
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 04/20/2017
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: deddfe16e4ad3eeee099ce1d7b53b9f15cbf85d2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956674"
---
# <a name="create-a-function-app-that-connects-to-an-azure-storage-account"></a>建立可連線至 Azure 儲存體帳戶的函式應用程式

這個 Azure Functions 範例指令碼會建立函式應用程式，並將函式連線至 Azure 儲存體帳戶。 包含連線的所建立應用程式設定可以搭配[儲存體觸發程序或繫結](..\functions-bindings-storage-blob.md)使用。 

[!INCLUDE [upgrade runtime](../../../includes/functions-cli-version-note.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您在本機使用 CLI，請確定您是執行 Azure CLI 2.0 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

此範例會建立 Azure 函數應用程式，然後將儲存體連接字串新增至應用程式設定。

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-storage/create-function-app-connect-to-storage-account.sh "Integrate Function App into Azure Storage Account")]


## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，請執行下列命令來移除資源群組和所有相關資源：

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 使用位置建立資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立儲存體帳戶。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在無伺服器[取用方案](../functions-scale.md#consumption-plan)中建立函式應用程式。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Functions 文件](../functions-cli-samples.md)中找到其他 Azure Functions CLI 指令碼範例。
