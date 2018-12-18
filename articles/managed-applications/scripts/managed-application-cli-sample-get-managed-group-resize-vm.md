---
title: Azure CLI 指令碼範例 - 取得受控資源群組並調整 VM 的大小 | Microsoft Docs
description: Azure CLI 指令碼範例 - 取得受控資源群組並調整 VM 的大小
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/25/2017
ms.author: tomfitz
ms.openlocfilehash: 85d58538e15881308ee1f645f7ddd12ec27c94de
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423407"
---
# <a name="get-resources-in-a-managed-resource-group-and-resize-vms-with-azure-cli"></a>使用 Azure CLI 取得受控資源群組中的資源並調整 VM 大小

此指令碼會從受控資源群組擷取資源，並調整該資源群組中 VM 的大小。


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/managed-applications/get-application/get-application.sh "Get application")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來部署受控應用程式。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az managedapp list](https://docs.microsoft.com/cli/azure/managedapp#az-managedapp-list) | 列出受控應用程式。 提供查詢值以聚焦在結果上。 |
| [az resource list](https://docs.microsoft.com/cli/azure/resource#az-resource-list) | 列出資源。 提供用來聚焦結果的資源群組名稱。 |
| [az vm resize](https://docs.microsoft.com/cli/azure/vm#az-vm-resize) | 更新虛擬機器的大小。 |


## <a name="next-steps"></a>後續步驟

* 如需受控應用程式的簡介，請參閱 [Azure 受控應用程式概觀](../overview.md)。
* 如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
