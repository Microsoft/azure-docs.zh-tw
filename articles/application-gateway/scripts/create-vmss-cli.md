---
title: Azure CLI 指令碼範例 - 管理網路流量 | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用應用程式閘道和虛擬機器擴展集來管理網路流量。
services: application-gateway
documentationcenter: networking
author: vhorne
manager: jpconnock
editor: tysonn
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: e2b17ef790f47a9352e3c2b744bdbb4960fe616b
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579987"
---
# <a name="manage-web-traffic-using-the-azure-cli"></a>使用 Azure CLI 管理網路流量

這個指令碼會建立應用程式閘道，它使用虛擬機器擴展集作為後端伺服器。 然後可以設定應用程式閘道來管理網路流量。 執行指令碼之後，您可以使用應用程式閘道的公用 IP 位址來進行測試。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/application-gateway/create-vmss/create-vmss.sh "Create application gateway")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、應用程式閘道和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroupAG --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az-net) | 建立虛擬網路。 |
| [az network vnet subnet create](https://docs.microsoft.com/cli/azure/network/vnet/subnet#az-network_vnet_subnet_create) | 在虛擬網路中建立子網路。 |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest) | 建立應用程式閘道的公用 IP 位址。 |
| [az network application-gateway create](https://docs.microsoft.com/cli/azure/network/application-gateway?view=azure-cli-latest) | 建立應用程式閘道。 |
| [az vmss create](https://docs.microsoft.com/cli/azure/vmss#az-vmss-create) | 建立虛擬機器擴展集。 |
| [az network public-ip show](https://docs.microsoft.com/cli/azure/network/public-ip#az-network_public_ip_show) | 取得應用程式閘道的公用 IP 位址。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure Windows 虛擬機器文件](../cli-samples.md)中找到其他應用程式閘道的 CLI 指令碼範例。
