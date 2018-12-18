---
title: 教學課程 - 升級 Azure Service Fabric Mesh 中所執行的應用程式 | Microsoft Docs
description: 在本教學課程中，您將了解如何升級 Service Fabric Mesh 中所執行的 Service Fabric 應用程式。
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
ms.openlocfilehash: c82e0cd5bd6a15ff33f51b4e88f68c13080f595d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967960"
---
# <a name="tutorial-upgrade-a-service-fabric-application-running-in-service-fabric-mesh"></a>教學課程：升級 Service Fabric Mesh 中所執行的 Service Fabric 應用程式

本教學課程是一個系列課程的第三部分。 您將了解如何藉由增加配置的 CPU 資源數，對[先前部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md) 的 Service Fabric 應用程式進行升級。  當您完成後，您就能以更多的 CPU 資源數目來執行 Web 前端服務。

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 變更應用程式組態
> * 升級 Service Fabric Mesh 中所執行的應用程式

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [使用範本將應用程式部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md)
> * [調整 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-scale-services.md)
> * 升級 Service Fabric Mesh 中所執行的應用程式
> * [移除應用程式](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 開啟 [Azure Cloud Shell](service-fabric-mesh-howto-setup-cli.md)，或[在本機安裝 Azure CLI 和 Service Fabric Mesh CLI](service-fabric-mesh-howto-setup-cli.md#install-the-service-fabric-mesh-cli-locally)。

## <a name="upgrade-application-configurations"></a>升級應用程式組態

將應用程式部署到 Service Fabric Mesh 的其中一個主要優點是可讓您輕鬆更新應用程式組態。  例如，您服務的 CPU 或記憶體資源數目。

本教學課程使用待辦事項清單樣本作為範例，該樣本[先前已完成部署](service-fabric-mesh-tutorial-template-deploy-app.md)，因此現在應已在執行中。 應用程式有兩個服務：WebFrontEnd 和 ToDoService。 每個服務一開始都會使用 0.5 個 CPU 資源進行部署。  若要檢視 WebFrontEnd 服務的 CPU 資源數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp
```

在應用程式資源的部署範本中，每個服務都有一個 cpu 屬性，可用於設定要求的 CPU 資源數。 應用程式可由多個服務組成，每個服務都具有唯一的 cpu 設定，這些服務會一同部署及管理。 若要增加 Web 前端服務的 CPU 資源數，請修改部署範本或參數檔案中的 cpue 值。  然後，升級應用程式。

### <a name="modify-the-deployment-template-parameters"></a>修改部署範本參數

當您範本中的值是您要在部署時變更的值，或希望可以選擇在每個部署基礎上進行變更的值 (如果您打算將此範本重新用於其他部署)，最佳做法是將值參數化。

先前，應用程式已使用 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)及 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案進行部署。

在本機上開啟 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案，並將 frontEndCpu 的值設為 1：

```json
      "frontEndCpu":{
        "value": "1"
      }
```

將您的變更儲存到參數檔案。  

frontEndCpu 參數的宣告位置位於 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)的「參數」區段：

```json
"frontEndCpu": {
    "defaultValue": "0.5",
    "type": "string",
    "metadata": {
        "description": "The CPU resources for the front end web service."
    }
}
```

WebFrontEnd 服務的 codePackages->resources->requests->cpu 屬性會參考 frontEndCpu 參數：

```json
    "services": [
          {
            "name": "WebFrontEnd",
            "properties": {
              "description": "WebFrontEnd description.",
              "osType": "Windows",
              "codePackages": [
                {
                  "name": "WebFrontEnd",
                  "image": "[parameters('frontEndImage')]",
                  ...
                  "resources": {
                    "requests": {
                      "cpu": "[parameters('frontEndCpu')]",
                      "memoryInGB": "[parameters('frontEndMemory')]"
                    }
                  },
                  "imageRegistryCredential": {
                    "server": "[parameters('registryServer')]",
                    "username": "[parameters('registryUserName')]",
                    "password": "[parameters('registryPassword')]"
                  }
                }
              ],
              ...
```

### <a name="upgrade-your-application"></a>將您的應用程式升級

在應用程式執行時，您可以透過重新部署範本及更新後的參數檔案來進行升級：

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

這將會啟動應用程式的輪流升級，且您應該會在數分鐘內看到 CPU 資源執行個體已增加。  若要檢視 WebFrontEnd 服務的 CPU 資源數，請執行下列內容：

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp
```

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 變更應用程式組態
> * 升級 Service Fabric Mesh 中所執行的應用程式

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [移除 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-template-remove-app.md)
