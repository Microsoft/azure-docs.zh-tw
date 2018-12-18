---
title: 在 Azure Service Fabric Mesh 應用程式中調整服務 | Microsoft Docs
description: 了解如何使用 Azure CLI，獨立調整 Service Fabric Mesh 上所執行應用程式內的服務。
services: service-fabric-mesh
documentationcenter: .net
author: rwike77
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: azure-cli
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/08/2018
ms.author: ryanwi
ms.custom: mvc, devcenter
ms.openlocfilehash: e68bcd135c33c7fd83908b8fed0fd098a698fd36
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038045"
---
# <a name="scale-services-within-an-application-running-on-service-fabric-mesh"></a>調整 Service Fabric Mesh 上所執行應用程式內的服務

本文說明如何獨立調整應用程式內的微服務。 在此範例中，視覺物件應用程式包含兩項微服務：`web` 和 `worker`。

`web` 服務是一個 ASP.NET Core 應用程式，具有會在瀏覽器中顯示三角形的網頁。 針對 `worker` 服務的每個執行個體，瀏覽器都會顯示一個三角形。

`worker` 服務會在空間中預先定義的間隔內移動三角形，並將三角形的位置傳送給 `web` 服務。 它會使用 DNS 來解析 `web` 服務的位址。

## <a name="set-up-service-fabric-mesh-cli"></a>設定 Service Fabric Mesh CLI

您可以使用 Azure Cloud Shell 或安裝在本機的 Azure CLI 來完成此工作。 請依照下列[指示](service-fabric-mesh-howto-setup-cli.md)安裝 Azure Service Fabric Mesh CLI 擴充功能模組。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 Azure 並設定您的訂用帳戶。

```azurecli-interactive
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-resource-group"></a>建立資源群組

建立要部署應用程式的資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location eastus 
```

## <a name="deploy-the-application-with-one-worker-service"></a>使用一個背景工作角色服務來部署應用程式

使用 `deployment create` 命令在資源群組中建立您的應用程式。

下列範例會使用 [mesh_rp.base.linux.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.base.linux.json)來部署 Linux 應用程式。 若要部署 Windows 應用程式，請使用 [mesh_rp.base.windows.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.base.windows.json)。 Windows 容器映像比 Linux 容器映像大，因此部署時可能需要較多時間。

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.base.linux.json --parameters "{\"location\": {\"value\": \"eastus\"}}"
```

命令應會在幾分鐘內傳回以下回應：

`visualObjectsApp has been deployed successfully on visualObjectsNetwork with public ip address <IP Address>` 

## <a name="open-the-application"></a>開啟應用程式

部署命令會傳回服務端點的公用 IP 位址。 在應用程式成功部署後，請取得服務端點的公用 IP 位址，並在瀏覽器中將其開啟。 它會顯示一個 Web 網頁，其中包含一個移動的三角形。

此應用程式的網路資源名稱為 `visualObjectsNetwork`。 您可以使用下列命令查看應用程式的相關資訊，例如其描述、位置、資源群組等：

```azurecli-interactive
az mesh network show --resource-group myResourceGroup --name visualObjectsNetwork
```

## <a name="scale-worker-service"></a>調整 `worker` 服務

使用下列命令將 `worker` 服務調整為三個執行個體。 下列範例會使用 [mesh_rp.scaleout.linux.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.scaleout.linux.json)來部署 Linux 應用程式。 若要部署 Windows 應用程式，請使用 [mesh_rp.scaleout.windows.json 範本](https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.scaleout.windows.json)。 請注意，較大的容器映像可能需要較長的部署時間。

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.scaleout.linux.json --parameters "{\"location\": {\"value\": \"eastus\"}}"
  
```

應用程式部署成功後，瀏覽器應會顯示一個網頁，其中包含三個移動的三角形。

## <a name="delete-the-resources"></a>刪除資源

請勤加刪除您已不會在 Azure 中使用的資源。 若要刪除此範例的相關資源，請使用下列命令，將資源部署所在的資源群組刪除 (這會刪除所有與該資源群組相關聯的項目)：

```azurecli-interactive
az group delete --resource-group myResourceGroup
```

## <a name="next-steps"></a>後續步驟

- 檢視 [GitHub](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/visualobjects) 上的視覺物件應用程式範例。
- 若要深入了解 Service Fabric 資源模型，請參閱 [Service Fabric Mesh 資源模型](service-fabric-mesh-service-fabric-resources.md)。
- 若要深入了解 Service Fabric Mesh，請閱讀 [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)。