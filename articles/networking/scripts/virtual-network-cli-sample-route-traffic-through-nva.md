---
title: Azure CLI 指令碼範例 - 透過網路虛擬設備來路由傳送流量 | Microsoft Docs
description: Azure CLI 指令碼範例 - 透過防火牆網路虛擬設備來路由傳送流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: timlt
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: jdial
ms.openlocfilehash: 62077f45d96e96a7fef35cf025740849d2b99445
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42141269"
---
# <a name="route-traffic-through-a-network-virtual-appliance"></a>透過網路虛擬設備來路由傳送流量

此指令碼範例會建立一個具有前端和後端子網路的虛擬網路。 它也會建立一個已啟用 IP 轉送功能的 VM，以在兩個子網路之間路由傳送流量。 執行此指令碼之後，您可以將網路軟體 (例如防火牆應用程式) 部署到 VM。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>範例指令碼


[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.sh "Route traffic through a network virtual appliance")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立資源群組、虛擬網路及網路安全性群組。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) | 建立 Azure 虛擬網路和前端子網路。 |
| [az network subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) | 建立後端和 DMZ 子網路。 |
| [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) | 建立公用 IP 位址以從網際網路存取 VM。 |
| [az network nic create](/cli/azure/network/nic#az_network_nic_create) | 建立虛擬網路介面並為它啟用 IP 轉送功能。 |
| [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) | 建立網路安全性群組 (NSG)。 |
| [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) | 建立允許對 VM 進行 HTTP 和 HTTPS 連接埠輸入的 NSG 規則。 |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update)| 將 NSG 和路由表與子網路建立關聯。 |
| [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create)| 為所有路由建立一個路由表。 |
| [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create)| 建立路由，以透過 VM 在子網路與網際網路之間路由傳送流量。 |
| [az vm create](/cli/azure/vm#az_vm_create) | 建立虛擬機器，並將 NIC 連結到此虛擬機器。 此命令也會指定要使用的虛擬機器映像和系統管理認證。 |
| [az group delete](/cli/azure/group#az_group_delete) | 刪除資源群組及其包含的所有資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。

您可以在 [Azure 網路概觀文件](../cli-samples.md)中找到其他網路 CLI 指令碼範例