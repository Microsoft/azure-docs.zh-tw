---
title: Azure CLI 指令碼範例 - 使用 Web 伺服器記錄監視 Web 應用程式 | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用 Web 伺服器記錄監視 Web 應用程式
services: appservice
documentationcenter: appservice
author: syntaxc4
manager: erikre
editor: ''
tags: azure-service-management
ms.assetid: 0887656f-611c-4627-8247-b5cded7cef60
ms.service: app-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 12/11/2017
ms.author: cfowler
ms.custom: mvc
ms.openlocfilehash: af7d6315b53a4bbb49a97fd8904536bb59d8465a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952387"
---
# <a name="monitor-a-web-app-with-web-server-logs"></a>使用 Web 伺服器記錄監視 Web 應用程式

此範例指令碼會建立資源群組、App Service 方案和 Web 應用程式，並設定 Web 應用程式來啟用 Web 伺服器記錄。 然後下載記錄檔來進行檢閱。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，需要 Azure CLI 2.0 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/app-service/monitor-with-logs/monitor-with-logs.sh "Monitor Logs")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、Web 應用程式和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) | 建立 App Service 方案。 |
| [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) | 建立 Azure Web 應用程式。 |
| [`az webapp log config`](/cli/azure/webapp/log?view=azure-cli-latest#az-webapp-log-config) | 設定 Azure Web 應用程式會保存的記錄。 |
| [`az webapp log download`](/cli/azure/webapp/log?view=azure-cli-latest#az-webapp-log-download) | 將 Azure Web 應用程式的記錄下載到本機電腦。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure App Service 文件](../app-service-cli-samples.md)中找到其他的 App Service CLI 指令碼範例。
