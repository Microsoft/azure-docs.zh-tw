---
title: Azure CLI 指令碼範例 - 調整 ACS 叢集 | Microsoft Docs
description: Azure CLI 指令碼範例 - 調整 ACS 叢集
services: container-service
documentationcenter: ''
author: neilpeterson
manager: jeconnoc
editor: ''
tags: acs, azure-container-service
keywords: Docker, 容器, 微服務, Kubernetes, DC/OS, Azure
ms.assetid: ''
ms.service: container-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/30/2017
ms.author: nepeters
ms.openlocfilehash: 1e5ca9fb44ea3ad15206f36a16e61f2865d79f5f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39427766"
---
# <a name="scale-an-azure-container-service-cluster"></a>調整 Azure Container Service 叢集

此範例會調整 Azure Container Service。 

[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

```azurecli
az acs scale --resource-group myResourceGroup --name myK8SCluster --new-agent-count 5
```

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az acs scale](/cli/azure/acs#az-acs-scale) | 調整 ACS 叢集的規模。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Container Service 文件](../cli-samples.md)中找到其他的 Azure Container Service CLI 指令碼範例。

