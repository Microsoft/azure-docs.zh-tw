---
title: Azure CLI 指令碼範例：路由傳送流量以達到應用程式的高可用性 | Microsoft Docs
description: Azure CLI 指令碼範例：路由傳送流量以達到應用程式的高可用性
services: traffic-manager
documentationcenter: traffic-manager
author: asudbring
manager: KumudD
ms.service: traffic-manager
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: traffic-manager
ms.date: 06/26/2018
ms.author: allensu
ms.openlocfilehash: 7103b11b7ee268acbddd8b402e1be1d44074f54d
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98234064"
---
# <a name="route-traffic-for-high-availability-of-applications---azure-cli"></a>路由傳送流量以達到應用程式的高可用性-Azure CLI

這個指令碼會建立一個資源群組、兩個 App Service 方案、兩個 Web 應用程式、一個流量管理員設定檔和兩個流量管理員端點。 流量管理員會將流量導向主要區域中的應用程式，然後當主要區域中的應用程式無法使用時，將流量導向至次要區域。 在執行指令碼之前，您必須將 MyWebApp、MyWebAppL1 和 MyWebAppL2 值都變更為整個 Azure 中的唯一值。 在執行指令碼之後，您可以透過 URL mywebapp.trafficmanager.net 來存取主要區域中的應用程式。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/traffic-manager/direct-traffic-for-increased-application-availability/direct-traffic-for-increased-application-availability.sh "Route traffic for high availability")]


## <a name="clean-up-deployment"></a>清除部署 

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組、App Service 應用程式和所有相關資源。

```azurecli
az group delete --name myResourceGroup1 --yes
az group delete --name myResourceGroup2 --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、Web 應用程式、流量管理員設定檔和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [az group create](/cli/azure/group) | 建立用來存放所有資源的資源群組。 |
| [az appservice plan create](/cli/azure/appservice/plan) | 建立 App Service 方案。 這就像是 Azure Web 應用程式的伺服器陣列。 |
| [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) | 在 App Service 方案內建立 Azure Web 應用程式。 |
| [az network traffic-manager profile create](/cli/azure/network/traffic-manager/profile) | 建立 Azure 流量管理員設定檔。 |
| [az network traffic-manager endpoint create](/cli/azure/network/traffic-manager/endpoint) | 新增端點至 Azure 流量管理員設定檔。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。

您可以在 [Azure 網路文件](../cli-samples.md)中找到其他的 App Service CLI 指令碼範例。