---
title: Azure CLI 指令碼範例 - 訂閱 Azure 訂用帳戶 | Microsoft Docs
description: Azure CLI 指令碼範例 - 訂閱 Azure 訂用帳戶
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
ms.openlocfilehash: ac02c5515f598daf4aa91879af78341969718163
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31425814"
---
# <a name="subscribe-to-events-for-an-azure-subscription-with-azure-cli"></a>使用 Azure CLI 訂閱 Azure 訂用帳戶的事件

此指令碼可針對 Azure 訂用帳戶的事件建立 Event Grid 訂用帳戶。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/event-grid/subscribe-to-azure-subscription/subscribe-to-azure-subscription.sh "Subscribe to Azure subscription")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立事件訂用帳戶。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az eventgrid event-subscription create](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create) | 建立事件格線訂用帳戶。 |


## <a name="next-steps"></a>後續步驟

* 如需查詢訂用帳戶的相關資訊，請參閱[查詢 Event Grid 訂用帳戶](../query-event-subscriptions.md)。
* 如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
