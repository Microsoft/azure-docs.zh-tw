---
title: 教學課程 - 調整 Azure Service Fabric Mesh 中所執行應用程式的大小 | Microsoft Docs
description: 在本教學課程中，了解如何調整 Service Fabric Mesh 中所執行應用程式內服務的大小。
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
ms.date: 09/18/2018
ms.author: ryanwi
ms.custom: mvc, devcenter
ms.openlocfilehash: 4d6839fea3ce0eb74cdf87396716cdc69c0cd1a0
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165976"
---
# <a name="tutorial-scale-an-application-running-in-service-fabric-mesh"></a>教學課程：調整 Service Fabric Mesh 中所執行應用程式的大小

本教學課程是一個系列的第二部分。 了解如何以手動方式調整[先前部署到 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md) 應用程式的服務執行個體數。 完成作業後，您將會有執行三個執行個體的前端服務，和執行兩個執行個體的資料服務。

在本系列的第二部分中，您將瞭解如何：

> [!div class="checklist"]
> * 設定所需要的服務執行個體數
> * 執行升級

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [使用範本將應用程式部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md)
> * 調整 Service Fabric Mesh 中所執行應用程式的大小
> * [升級 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [移除應用程式](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* [本機安裝 Azure CLI 和 Service Fabric Mesh CLI](service-fabric-mesh-howto-setup-cli.md#install-the-service-fabric-mesh-cli-locally)。

## <a name="manually-scale-your-services-in-or-out"></a>手動相應縮小或放大您的服務

將應用程式部署到 Service Fabric Mesh 的其中一個主要優點是可讓您輕鬆相應縮小或放大服務。這應用於處理您服務上不同數量的負載，或是改善可用性。

本教學課程使用待辦事項清單樣本作為範例，該樣本[先前已完成部署](service-fabric-mesh-tutorial-template-deploy-app.md)，因此現在應已在執行中。 應用程式有兩個服務：WebFrontEnd 和 ToDoService。 每個服務一開始都使用複寫計數 1 進行部署。  若要檢視執行 WebFrontEnd 服務的執行中複本數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

若要檢視執行 ToDoService 服務的執行中複本數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

在應用程式資源的部署範本中，每個服務都有一個 *replicaCount* 屬性，可用於設定您希望部署該服務的次數。 應用程式可由多個服務組成，每個服務都具有唯一的 *replicaCount* 數，這些服務會一同部署及管理。 為了調整服務複本數，請在部署範本或參數檔案中修改您欲調整的每個服務 *replicaCount* 值。  然後，升級應用程式。

### <a name="modify-the-deployment-template-parameters"></a>修改部署範本參數

當您範本中的值是您要在部署時變更的值，或希望可以選擇在每個部署基礎上進行變更的值 (如果您打算將此範本重新用於其他部署)，最佳做法是將值參數化。

先前，應用程式已使用 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)及 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案進行部署。

於本機開啟 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案，將 *frontEndReplicaCount* 值設為 3，並將 *serviceReplicaCount* 值設為 2：

```json
      "frontEndReplicaCount":{
        "value": "3"
      },
      "serviceReplicaCount":{
        "value": "2"
      }
```

將您的變更儲存到參數檔案。  *frontEndReplicaCount* 及 *ReplicaCount* 參數的宣告位置位於 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)的「參數」區段：

```json
"frontEndReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    },
    "serviceReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    }
```

WebFrontEnd 服務的 *replicaCount* 屬性會參考 *frontEndReplicaCount* 參數，ToDoService 服務的 *replicaCount* 屬性則會參考 *serviceReplicaCount* 參數：

```json
    "services": [
    {
    "name": "WebFrontEnd",
    "properties": {
        "description": "WebFrontEnd description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('frontEndReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    },
    {
    "name": "ToDoService",
    "properties": {
        "description": "ToDoService description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('serviceReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    }
],
```

修改完您的範本之後，請升級您的應用程式。

### <a name="upgrade-your-application"></a>將您的應用程式升級

在應用程式執行時，您可以透過重新部署範本及更新後的參數檔案來進行升級：

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

這將會啟動應用程式的輪流升級，且您應該會在數分鐘內看到服務執行個體增加。  若要檢視執行 WebFrontEnd 服務的執行中複本數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

若要檢視執行 ToDoService 服務的執行中複本數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 設定所需要的服務執行個體數
> * 執行升級

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [升級 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-upgrade-app.md)
