---
title: 部署和監視適用於 Azure IoT Edge (CLI) 的模組 | Microsoft Docs
description: 管理在 Edge 裝置上執行的模組
keywords: ''
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 07/25/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: c94a58a19558350c3c20377ce750f6758f688c0d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998503"
---
# <a name="deploy-and-monitor-iot-edge-modules-at-scale-using-the-azure-cli"></a>使用 Azure CLI 大規模部署與監視 IoT Edge 模組

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-edge-how-to-deploy-monitor-selector.md)]

Azure IoT Edge 可讓您將分析移至 Edge，並提供雲端介面，讓您能夠管理和監視 IoT Edge 裝置，而不需實際存取每一個。 隨著物聯網解決方案越來越大且越複雜，從遠端管理裝置的能力就變得越來越重要。 Azure IoT Edge 是設計來支援您的商務目標，而不論您新增了多少個裝置。

您可以管理個別裝置，且一次一個對其部署模組。 不過，如果您想要大規模地變更裝置，可以建立 **IoT Edge 自動部署**，這是 IoT 中樞內裝置自動管理的一部分。 部署是動態程序，可讓您一次將多個模組部署到多個裝置、追蹤模組的狀態和健康情況，並視需要進行變更。 

在本文中，您會設定 Azure CLI 和 IoT 擴充功能。 然後了解如何使用可用的 CLI 命令，將模組部署到一組 IoT Edge 裝置並監視進度。

## <a name="cli-prerequisites"></a>CLI 先決條件

* Azure 訂用帳戶中的 [IoT 中樞](../iot-hub/iot-hub-create-using-cli.md)。 
* 已安裝 IoT Edge 執行階段的 [IoT Edge 裝置](how-to-register-device-cli.md)。
* 您環境中的 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 您的 Azure CLI 版本至少必須是 2.0.24 或更新版本。 使用 `az –-version` 進行驗證。 這個版本支援 az 擴充命令並引進 Knack 命令架構。 
* [適用於 Azure CLI 的 IoT 擴充功能](https://github.com/Azure/azure-iot-cli-extension) \(英文\)。

## <a name="configure-a-deployment-manifest"></a>設定部署資訊清單

部署資訊清單為 JSON 文件，說明應部署的模組、資料如何在模組之間流動，以及想要的模組對應項需要的屬性。 如需部署資訊清單的功能，以及如何建立此類清單的詳細資訊，請參閱[了解如何使用、設定以及重複使用 IoT Edge 模組](module-composition.md) (英文)。

若要使用 Azure CLI 部署模組，請在本機上將部署資訊清單儲存成 .txt 檔案。 若要執行命令以將設定套用至裝置，您會使用到下一節中的檔案路徑。 

下面以具有一個模組的基本部署資訊清單為例：

   ```json
   {
        "content": {
         "modulesContent": {
           "$edgeAgent": {
             "properties.desired": {
               "schemaVersion": "1.0",
               "runtime": {
                 "type": "docker",
                 "settings": {
                   "minDockerVersion": "v1.25",
                   "loggingOptions": "",
                   "registryCredentials": {
                     "registryName": {
                       "username": "",
                       "password": "",
                       "address": ""
                     }
                   }
               },
               "systemModules": {
                 "edgeAgent": {
                   "type": "docker",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                     "createOptions": "{}"
                   }
                 },
                 "edgeHub": {
                   "type": "docker",
                   "status": "running",
                   "restartPolicy": "always",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                     "createOptions": "{}"
                   }
                 }
               },
               "modules": {
                 "tempSensor": {
                   "version": "1.0",
                   "type": "docker",
                   "status": "running",
                   "restartPolicy": "always",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                     "createOptions": "{}"
                   }
                 }
               }
             }
           },
           "$edgeHub": {
             "properties.desired": {
               "schemaVersion": "1.0",
               "routes": {
                   "route": "FROM /* INTO $upstream"
               },
               "storeAndForwardConfiguration": {
                 "timeToLiveSecs": 7200
               }
             }
           },
           "tempSensor": {
             "properties.desired": {}
           }
         }
       }
   }
   ```


## <a name="identify-devices-using-tags"></a>使用標記識別裝置

在您可建立部署之前，必須能夠指定您要影響的裝置。 Azure IoT Edge 會使用裝置對應項中的 **tags** 來識別裝置。 每個裝置都可以有多個標記，而您可以利用任何對您解決方案有意義的方式來定義它們。 例如，如果您管理智慧建築的校園，便可以將下列標記新增至裝置：

```json
"tags":{
    "location":{
        "building": "20",
        "floor": "2"
    },
    "roomtype": "conference",
    "environment": "prod"
}
```

如需裝置對應項和標記的詳細資訊，請參閱[了解和使用 IoT 中樞的裝置對應項][lnk-device-twin]。

## <a name="create-a-deployment"></a>建立部署

透過建立一個包含部署資訊清單及其他參數的部署，您就可以將模組部署至您的目標裝置。 

使用以下命令可建立部署：

   ```cli
   az iot edge deployment create --deployment-id [deployment id] --labels [labels] --content [file path] --hub-name [hub name] --target-condition [target query] --priority [int]
   ```

* **--deployment-id** - 將建立在 IoT 中樞內的部署名稱。 為部署指定唯一的名稱，最長為 128 個小寫字母。 避免空格和下列無效字元：`& ^ [ ] { } \ | " < > /`。
* **--labels** - 新增標籤，以協助追蹤您的部署。 標籤是成對的「名稱, 值」組合，可描述您的部署。 例如，`HostPlatform, Linux` 或 `Version, 3.0.1`。
* **--content** - 部署資訊清單 JSON 的檔案名稱。 
* **--hub-name** - 將在其中建立部署的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶
* **--target-condition** - 輸入目標條件來判斷這個部署會將哪些裝置設為目標。 條件會以裝置對應項標籤或裝置對應項報告屬性為基礎，且應符合運算式格式。 例如，`tags.environment='test'` 或 `properties.reported.devicemodel='4000x'`。 
* **--priority** - 正整數。 如果兩個以上部署的目標為相同的裝置，則將會套用 [優先順序] 數值最高的部署。

## <a name="monitor-a-deployment"></a>監視部署

使用下列命令顯示部署的內容︰

   ```cli
az iot edge deployment show --deployment-id [deployment id] --hub-name [hub name]
   ```
* **--deployment-id** - 存在於 IoT 中樞的部署名稱。
* **--hub-name** - 部署存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶

在命令視窗中檢查部署。 **metrics** 屬性會列出每個中樞所評估每個計量的計數：
* **targetedCount** - 系統計量，指定 IoT 中樞內符合目標條件的裝置對應項數目。
* **appliedCount** - 系統計量，指定已在 IoT 中樞內將部署內容套用到其模組對應項的裝置數目。
* **reportedSuccessfulCount** - 裝置計量，指定從 IoT Edge 用戶端執行階段回報成功之部署中的 Edge 裝置數目。
* **reportedFailedCount** - 裝置計量，指定從 IoT Edge 用戶端執行階段回報失敗之部署中的 Edge 裝置數目。

您可以使用下列命令，顯示每個計量的裝置識別碼或物件清單：

   ```cli
az iot edge deployment show-metric --deployment-id [deployment id] --metric-id [metric id] --hub-name [hub name] 
   ```

* **--deployment-id** - 存在於 IoT 中樞的部署名稱。
* **--metric-id** - 您想要在其中查看裝置識別碼清單的計量名稱，例如 `reportedFailedCount`
* **--hub-name** - 部署存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶

## <a name="modify-a-deployment"></a>修改部署

當您修改部署時，所做的變更會立即複寫到所有目標裝置。 

如果您更新目標條件，就會發生下列更新：
* 如果裝置不符合舊的目標條件，但符合新的目標條件，而且此部署為該裝置的最高優先順序，則會將此部署套用到裝置。 
* 如果目前執行此部署的裝置不再符合目標條件，它會解除安裝此部署，並採用下一個最高優先順序的部署。 
* 如果目前執行此部署的裝置不再符合目標條件，且不符合任何其他部署的目標條件，則不會在裝置上發生任何變更。 裝置會以模組目前的狀態繼續執行它目前的模組，但不再將其當成此部署的一部分來管理。 一旦它符合任何其他部署的目標條件之後，就會解除安裝此部署，並採用新的部署。 

使用以下命令可更新部署：

   ```cli
az iot edge deployment update --deployment-id [deployment id] --hub-name [hub name] --set [property1.property2='value']
   ```
* **--deployment-id** - 存在於 IoT 中樞的部署名稱。
* **--hub-name** - 部署存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶
* **--set** - 更新部署中的屬性。 您可以更新下列屬性：
    * targetCondition - 例如 `targetCondition=tags.location.state='Oregon'`
    * 標籤 
    * 優先順序


## <a name="delete-a-deployment"></a>刪除部署

當您刪除部署時，所有裝置均會採用其下一個最高優先順序的部署。 如果您的裝置不符合任何其他部署的目標條件，則在刪除部署時不會移除模組。 

使用下列命令可刪除部署：

   ```cli
az iot edge deployment delete --deployment-id [deployment id] --hub-name [hub name] 
   ```
* **--deployment-id** - 存在於 IoT 中樞的部署名稱。
* **--hub-name** - 部署存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶

## <a name="next-steps"></a>後續步驟

深入了解[將模組部署到 Edge 裝置][lnk-deployments]。

<!-- Images -->
[1]: ./media/how-to-deploy-monitor/iot-edge-deployments.png

<!-- Links -->
[lnk-device-twin]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-portal]: https://portal.azure.com
[lnk-docker-create]: https://docs.docker.com/engine/reference/commandline/create/
[lnk-deployments]: module-deployment-monitoring.md

<!-- Anchor links -->
[anchor-monitor]: #monitor-a-deployment
