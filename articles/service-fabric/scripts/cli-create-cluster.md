---
title: Azure CLI 指令碼部署範例
description: 使用 Azure CLI 在 Azure 中建立安全的 Service Fabric Linux 叢集。
services: service-fabric
documentationcenter: ''
author: rwike77
manager: timlt
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 01/18/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 9c2f094104172407992371b6738dd2fdf101a506
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985107"
---
# <a name="create-a-secure-service-fabric-linux-cluster-in-azure"></a>在 Azure 中建立安全的 Service Fabric Linux 叢集

此命令會建立自我簽署的憑證，然後將其加入金鑰保存庫並在本機下載憑證。  新的憑證會用來在部署叢集時保護叢集。  您也可以使用現有的憑證，而不用建立新的。  無論是哪一方法，憑證的主體名稱必須與您用來存取 Service Fabric 叢集的網域相符。 必須如此相符，才能為叢集的 HTTPS 管理端點和 Service Fabric Explorer 提供 SSL。 您無法從 CA 取得 `.cloudapp.azure.com` 網域的 SSL 憑證。 您必須為您的叢集取得自訂網域名稱。 當您向 CA 要求憑證時，憑證的主體名稱必須與用於您叢集的自訂網域名稱相符。

視需要安裝 [Azure CLI](/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。

## <a name="sample-script"></a>範例指令碼

[!code-sh[main](../../../cli_scripts/service-fabric/create-cluster/create-cluster.sh "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組、叢集和所有相關資源。

```azurecli
ResourceGroupName = "aztestclustergroup"
az group delete --name $ResourceGroupName
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az sf cluster create](https://docs.microsoft.com/cli/azure/sf/cluster?view=azure-cli-latest#az_sf_cluster_create) | 建立新的 Service Fabric 叢集。  |

## <a name="next-steps"></a>後續步驟

您可以在 [Service Fabric CLI 範例](../samples-cli.md)中找到適用於 Azure Service Fabric 的其他 Service Fabric CLI 範例。
