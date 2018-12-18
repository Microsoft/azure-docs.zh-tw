---
title: 包含檔案
description: 包含檔案
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 6fab0258b0c0e2f9b31358075a8f7a5be0228a5e
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44095690"
---
[!INCLUDE [resource group intro text](resource-group.md)]

在 Cloud Shell 中，使用 [`az group create`](/cli/azure/group?view=azure-cli-latest#az_group_create) 命令來建立資源群組。 下列範例會在「西歐」位置建立名為 myResourceGroup 的資源群組。 若要查看**基本**層中 Linux 上之 App Service 的所有支援位置，請執行 [`az appservice list-locations --sku B1 --linux-workers-enabled`](/cli/azure/appservice?view=azure-cli-latest#az_appservice_list_locations) 命令。

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

您通常會在附近的區域中建立資源群組和資源。 

當命令完成時，JSON 輸出會顯示資源群組屬性。