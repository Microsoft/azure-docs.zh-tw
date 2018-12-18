---
title: 教學課程 - 移除在 Azure Service Fabric Mesh 中執行的應用程式 | Microsoft Docs
description: 在本教學課程中，了解如何移除在 Service Fabric Mesh 中執行的應用程式及刪除資源。
services: service-fabric-mesh
documentationcenter: .net
author: rwike77
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/15/2018
ms.author: ryanwi
ms.custom: mvc, devcenter
ms.openlocfilehash: f366413ae5f758601dfebc2a29ff848feb756083
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960425"
---
# <a name="tutorial-remove-an-application-and-resources"></a>教學課程：移除應用程式和資源

本教學課程是一個系列的第四部分。 您將會了解如何移除[先前部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md) 的執行中應用程式。 

在本系列的第一部分中，您了解如何：

> [!div class="checklist"]
> * 刪除在 Service Fabric Mesh 中執行的應用程式
> * 刪除應用程式資源

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [使用範本將應用程式部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md)
> * [調整 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-scale-services.md)
> * [升級 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * 移除應用程式

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 開啟 [Azure Cloud Shell](service-fabric-mesh-howto-setup-cli.md)，或[在本機安裝 Azure CLI 和 Service Fabric Mesh CLI](service-fabric-mesh-howto-setup-cli.md#install-the-service-fabric-mesh-cli-locally)。

## <a name="delete-the-resource-group-and-all-the-resources"></a>刪除資源群組和所有資源

不再需要時，請刪除您所建立的所有資源。 先前，您[建立了新的資源群組](service-fabric-mesh-tutorial-template-deploy-app.md#create-a-container-registry)，用於裝載 Azure Container Registry 執行個體及 Service Fabric Mesh 應用程式資源。  您可以刪除此資源群組，同時刪除所有與相關聯的資源。

```azurecli
az group delete --resource-group myResourceGroup
```

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="individually-delete-the-resources"></a>個別刪除資源
您也可以個別刪除 ACR 執行個體、Service Fabric Mesh 應用程式及網路資源。

刪除 ACR 執行個體：

```azurecli
az acr delete --resource-group myResourceGroup --name myContainerRegistry
```

刪除 Service Fabric Mesh 應用程式：

```azurecli
az mesh app delete --resource-group myResourceGroup --name todolistapp
```

刪除網路：
```azurecli
az mesh network delete --resource-group myResourceGroup --name todolistappNetwork
```

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 刪除在 Service Fabric Mesh 中執行的應用程式
> * 刪除應用程式資源