---
title: Azure CLI 指令碼範例 - 使用 App Service 建立 SignalR 服務 | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用 App Service 建立 SignalR 服務
services: signalr
documentationcenter: signalr
author: sffamily
manager: cfowler
editor: ''
tags: azure-service-management
ms.service: signalr
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: signalr
ms.date: 04/20/2018
ms.author: zhshang
ms.custom: mvc
ms.openlocfilehash: 971c618300c1aedcdfb26bfaa79660a92eada475
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978721"
---
# <a name="create-a-signalr-service-with-an-app-service"></a>使用 App Service 建立 SignalR 服務

這個範例指令碼會建立新的 Azure SignalR 服務資源，以供用來將即時內容更新推送給用戶端。 這個指令碼也會新增 Web 應用程式和 App Service 方案來裝載 ASP.NET Core Web 應用程式，以使用 SignalR Service。 Web 應用程式會透過名為 AzureSignalRConnectionString 的「應用程式設定」來設定，以連線至新的 SignalR 服務資源。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

此指令碼會使用適用於 Azure CLI 的 signalr 擴充功能。 在使用這個範例指令碼之前，請執行下列命令以安裝適用於 Azure CLI 2.0 的 signalr 擴充功能：

```azurecli-interactive
az extension add -n signalr
```

[!code-azurecli-interactive[main](../../../cli_scripts/azure-signalr/create-signalr-with-app-service/create-signalr-with-app-service.sh "Create a new Azure SignalR Service and Web App")]

記下新資源群組所產生的實際名稱。 此名稱會顯示在輸出中。 當您想要刪除所有群組資源時，就會用到該資源群組名稱。

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

下表中的每個命令都會連結至命令特定的文件。 此指令碼會使用下列命令：

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az signalr create](/cli/azure/ext/signalr/signalr#ext-signalr-az-signalr-create) | 建立 Azure SignalR 服務資源。 |
| [az signalr key list](/cli/azure/ext/signalr/signalr/key#ext-signalr-az-signalr-key-list) | 列出應用程式在使用 SignalR 推送即時內容更新時，所會使用的金鑰。 |
| [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create) | 建立用來裝載 Web 應用程式的 Azure App Service 方案。 |
| [az webapp create](/cli/azure/webapp#az-webapp-create) | 使用 App Service 主控方案建立 Azure Web 應用程式。 |
| [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) | 新增 Web 應用程式的應用程式設定。 此應用程式設定會用來儲存 SignalR 連接字串。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。

您可以在 [Azure SignalR 服務文件](../signalr-cli-samples.md)中找到其他的 Azure SignalR 服務 CLI 指令碼範例。
