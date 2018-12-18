---
title: Azure CLI 指令碼 - 取得 MongoDB 應用程式的 Azure Cosmos DB 連接字串 | Microsoft Docs
description: Azure CLI 指令碼範例 - 取得 MongoDB 應用程式的 Azure Cosmos DB 連接字串
services: cosmos-db
documentationcenter: cosmosdb
author: SnehaGunda
manager: kfile
tags: azure-service-management
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: mvc
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: cosmosdb
ms.workload: database
ms.date: 06/02/2017
ms.author: sngun
ms.openlocfilehash: 93f33e9c66ede2c18064c403345b4db9ab26e122
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973281"
---
# <a name="get-an-azure-cosmos-db-connection-string-for-mongodb-apps-using-the-azure-cli"></a>使用 Azure CLI 取得 MongoDB 應用程式的 Azure Cosmos DB 連接字串

此範例會使用 Azure CLI 取得 MongoDB 應用程式的 Azure Cosmos DB 連接字串。 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本主題會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/cosmosdb/secure-cosmosdb-get-mongodb-connection-string/secure-cosmosdb-get-mongodb-connection-string.sh?highlight=36-39 "Get Azure Cosmos DB connection string for MongoDB apps")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az cosmosdb update](https://docs.microsoft.com/cli/azure/cosmosdb#az-cosmosdb-update) | 更新 Azure Cosmos DB 帳戶。 |
| [az cosmosdb list-connection-strings](https://docs.microsoft.com/cli/azure/cosmosdb#az-cosmosdb-list-connection-strings) | 取得帳戶的連接字串。|
| [az group delete](https://docs.microsoft.com/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Cosmos DB CLI 文件](../cli-samples.md)中找到其他 Azure Cosmos DB CLI 指令碼範例。
