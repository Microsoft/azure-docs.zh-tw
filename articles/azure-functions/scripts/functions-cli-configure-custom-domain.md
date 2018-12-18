---
title: Azure CLI 指令碼範例 - 將自訂網域對應至函式應用程式 | Microsoft Docs
description: Azure CLI 指令碼範例 - 在 Azure 中將自訂網域對應至函式應用程式。
services: functions
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.assetid: d127e347-7581-47d7-b289-e0f51f2fbfbc
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/04/2018
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 5299675656ea23b85f78b3dbfdc02814ab332a8b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979044"
---
# <a name="map-a-custom-domain-to-a-function-app"></a>將自訂網域對應至函式應用程式

此範例指令碼會在 App Service 方案中建立函式應用程式，然後再將其對應至您所提供的自訂網域。 當您的函式應用程式裝載於 [App Service 方案](../functions-scale.md#app-service-plan)時，您可以使用 CNAME 或 A 記錄對應自訂網域。 若為[取用方案](../functions-scale.md#consumption-plan)中的函式應用程式，則只支援 CNAME 選項。 此範例會建立 App Service 方案，並要求 A 記錄對應網域。 

若要執行此範例指令碼，您必須已在指向您 Web 應用程式預設網域名稱的自訂網域中設定 A 記錄。 如需詳細資訊，請參閱[將現有的自訂 DNS 名稱對應至 Azure Web Apps](https://aka.ms/appservicecustomdns)。 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，您必須使用 Azure CLI 2.0 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/configure-custom-domain/configure-custom-domain.sh?highlight=3 "Map a custom domain to a function app")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令：下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立函式應用程式所需的儲存體帳戶。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create) | 建立對應自訂網域所需的 App Service 方案。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在 App Service 方案中建立函式應用程式。 |
| [az functionapp config hostname add](https://docs.microsoft.com/cli/azure/functionapp/config/hostname#az-functionapp-config-hostname-add) | 將自訂網域對應至函式應用程式。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Functions 文件](../functions-cli-samples.md)中找到其他 Functions CLI 指令碼範例。
