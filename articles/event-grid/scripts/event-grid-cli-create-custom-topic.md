---
title: Azure CLI 指令碼範例 - 建立自訂主題 | Microsoft Docs
description: Azure CLI 指令碼範例 - 建立自訂主題
services: event-grid
documentationcenter: na
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/05/2018
ms.author: tomfitz
ms.openlocfilehash: 3ed21dd54cbc572df76cea5eb6440fa1eddf4fe4
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31425823"
---
# <a name="create-event-grid-custom-topic-with-azure-cli"></a>使用 Azure CLI 建立 Event Grid 自訂主題

此指令碼可建立 Event Grid 自訂主題。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/event-grid/create-custom-topic/create-custom-topic.sh "Create custom topic")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立自訂主題。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az eventgrid topic create](https://docs.microsoft.com/cli/azure/eventgrid/topic#az-eventgrid-topic-create) | 建立 Event Grid 自訂主題。 |


## <a name="next-steps"></a>後續步驟

* 如需查詢訂用帳戶的相關資訊，請參閱[查詢 Event Grid 訂用帳戶](../query-event-subscriptions.md)。
* 如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
