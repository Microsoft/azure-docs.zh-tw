---
title: 設定 Azure Service Fabric Mesh CLI | Microsoft Docs
description: 了解如何設定 Azure Service Fabric Mesh CLI。
services: service-fabric-mesh
keywords: ''
author: tylermsft
ms.author: twhitney
ms.date: 07/26/2018
ms.topic: get-started-article
ms.service: service-fabric-mesh
manager: timlt
ms.openlocfilehash: 7e8a12a215c94102f6b08262f129faebf9cfcde9
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49115619"
---
# <a name="set-up-the-service-fabric-mesh-cli"></a>設定 Service Fabric Mesh CLI
必須使用 Service Fabric Mesh 命令列介面 (CLI)，才能在 Service Fabric Mesh 中部署及管理資源。 

針對預覽版本，Azure Service Fabric Mesh CLI 是以 Azure CLI 擴充功能的形式撰寫。 您可以將其安裝在 Azure Cloud Shell 或 Azure CLI 的本機安裝上。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

## <a name="install-the-service-fabric-mesh-cli-locally"></a>本機安裝 Service Fabric Mesh CLI
如果您選擇在本機安裝和使用 CLI，則必須安裝 Azure CLI 2.0.43 版或更新版本。 執行 `az --version` 以尋找版本。 若要安裝或升級至最新版本的 CLI，請參閱[安裝 Azure CLI][azure-cli-install]。

請使用下列命令安裝 Azure Service Fabric Mesh CLI 擴充功能模組。 

```azurecli-interactive
az extension add --name mesh
```

若要更新現有的 Azure Service Fabric Mesh CLI 模組，請執行下列命令。

```azurecli-interactive
az extension update --name mesh
```

您也可以設定 [Windows 開發環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)。

[azure-cli-install]: /cli/azure/install-azure-cli
