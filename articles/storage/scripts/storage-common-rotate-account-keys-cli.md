---
title: Azure CLI 指令碼範例 - 輪替儲存體帳戶存取金鑰 | Microsoft Docs
description: 建立 Azure 儲存體帳戶，然後擷取並輪替其帳戶存取金鑰。
services: storage
documentationcenter: na
author: tamram
manager: timlt
editor: tysonn
ms.assetid: ''
ms.custom: mvc
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: sample
ms.date: 06/22/2017
ms.author: tamram
ms.openlocfilehash: 160509a5a82b71b281d57d97e103bb4190605b7c
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29847806"
---
# <a name="create-a-storage-account-and-rotate-its-account-access-keys"></a>建立儲存體帳戶並輪替其帳戶存取金鑰

此指令碼會建立 Azure 儲存體帳戶、顯示儲存體帳戶的新存取金鑰，然後更新 (輪替) 該金鑰。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/storage/rotate-storage-account-keys/rotate-storage-account-keys.sh "Rotate storage account keys")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、儲存體帳戶和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立儲存體帳戶，並擷取和輪替其存取金鑰。 下表中的每個項目都會連結至特定命令的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | 在指定的資源群組中建立 Azure 儲存體帳戶。 |
| [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) | 顯示指定帳戶的儲存體帳戶存取金鑰。 |
| [az storage account keys renew](/cli/azure/storage/account/keys#az_storage_account_keys_renew) | 重新產生主要或次要的儲存體帳戶存取金鑰。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。

您可以在[適用於 Azure Blob 儲存體的 Azure CLI 範例](../blobs/storage-samples-blobs-cli.md)中，找到其他儲存體 CLI 指令碼範例。
