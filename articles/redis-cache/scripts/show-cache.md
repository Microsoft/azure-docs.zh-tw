---
title: Azure CLI 指令碼範例 - 取得 Azure Redis 快取的詳細資料 | Microsoft Docs
description: Azure CLI 指令碼範例 - 取得 Azure Redis 快取的詳細資料
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
tags: azure-service-management
ms.assetid: 155924e6-00d5-4a8c-ba99-5189f300464a
ms.service: cache-redis
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 08/30/2017
ms.author: wesmc
ms.openlocfilehash: 00ae8e00d456d4660cadf645d4521173d254bbd7
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29846242"
---
# <a name="get-details-of-an-azure-redis-cache"></a>取得 Azure Redis 快取的詳細資料

在此案例中，您將了解關於如何擷取 Azure Redis 快取執行個體 (包括其佈建狀態) 的詳細資料。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/redis-cache/show-cache/show-cache.sh "Azure Redis Cache")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令，擷取 Azure Redis 快取執行個體的詳細資料。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az redis show](https://docs.microsoft.com/cli/azure/redis#az_redis_show) | 取得 Azure Redis Cache 執行個體的詳細資料。 |


## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Redis Cache 文件](../cli-samples.md)中找到其他 Azure Redis Cache CLI 指令碼範例。