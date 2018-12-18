---
title: CLI 範例 - 建立 DNS 區域和網域名稱記錄 - Azure | Microsoft Docs
description: 此 Azure CLI 指令碼範例會示範如何建立 DNS 區域和網域名稱記錄
services: load-balancer
documentationcenter: traffic-manager
author: vhorne
manager: jeconnoc
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 04/30/2018
ms.author: victorh
ms.openlocfilehash: b8a69b44714a24c78bf4077c27b5bf5633cc56d3
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39431761"
---
# <a name="azure-cli-script-example-create-a-dns-zone-and-record"></a>Azure CLI 指令碼範例：建立 DNS 區域和記錄

此 Azure CLI 指令碼範例會建立 DNS 區域和網域名稱記錄。 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

```azurecli-interactive

# Create a resource group.
az group create \
  -n myResourceGroup \
  -l eastus

# Create a DNS zone. Substitute zone name "contoso.com" with the values for your own.

az network dns zone create \
  -g MyResourceGroup \
  -n contoso.com

# Create a DNS record. Substitute zone name "contoso.com" and IP address "1.2.3.4* with the values for your own.

az network dns record-set a add-record \
  -g MyResourceGroup \
  -z contoso.com \
  -n www \
  -a 1.2.3.4

# Get a list the DNS records in your zone
az network dns record-set list \
  -g MyResourceGroup \ 
  -z contoso.com
```

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、DNS 區域和所有相關資源。

```azurecli
az group delete -n myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器、可用性設定組、負載平衡器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az network dns zone create](/cli/azure/network/dns/zone#az-network-dns-zone-create) | 建立 Azure DNS 區域。 |
| [az network dns record-set a add-record](/cli/azure/network/dns/record-set#az-network-dns-record-set-a-add-record) | 將 A 記錄新增至 DNS 區域。 |
| [az network dns record-set list](/cli/azure/network/dns/record-set#az-network-dns-record-set-a-list) | 列出 DNS 區域中的所有 A 記錄集。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

