---
title: 使用 Azure IoT 中樞大規模設定和監視 IoT 裝置 (CLI) | Microsoft Docs
description: 使用 Azure IoT 中樞自動裝置設定來將設定指派給多個裝置
author: ChrisGMsft
manager: bruz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/13/2018
ms.author: chrisgre
ms.openlocfilehash: 6535eee93b0fcaa2c0888fc6b647811c21e564d9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971904"
---
# <a name="configure-and-monitor-iot-devices-at-scale-using-the-azure-cli"></a>使用 Azure CLI 大規模設定和監視 IoT 裝置

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-hub-auto-device-config-selector.md)]

Azure IoT 中樞中的自動裝置管理，可將大批裝置在其生命週期全程中的許多重複且複雜的管理工作自動化。 使用自動裝置管理時，您可以根據裝置的屬性，以特定的裝置集合作為目標來定義所需的設定，並讓 IoT 中樞在這些裝置進入範圍內時更新裝置。  執行此動作的方式是使用自動裝置設定，這也可讓您對完成與合規性進行摘要處理、處理合併和衝突，以及階段式地推出設定。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

自動裝置設定的運作方式，是以所需的屬性更新一組裝置對應項，並根據裝置對應項報告屬性來報告摘要。  它推出名為*設定*的新類別和 JSON 文件，其共有三部分：

* **目標條件**會定義要更新的裝置對應項範圍。 目標條件會以查詢的方式指定於裝置對應項標籤和/或報告屬性上。

* **目標內容**會定義要在目標裝置對應項中新增或更新的所需屬性。 內容包含了一個路徑，連往所要變更屬性的區段。

* **計量**會定義各種設定狀態 (例如 **Success**、**Progress** 及 **Error**) 的摘要計數。 自訂計量會以查詢的形式指定於裝置對應項報告屬性上。  系統計量是測量對應項更新狀態的預設計量，例如作為目標的裝置對應項數目，以及成功更新的對應項數目。 

## <a name="cli-prerequisites"></a>CLI 先決條件

* Azure 訂用帳戶中的 [IoT 中樞](../iot-hub/iot-hub-create-using-cli.md)。 
* 您環境中的 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 您的 Azure CLI 版本至少必須是 2.0.24 或更新版本。 使用 `az –-version` 進行驗證。 這個版本支援 az 擴充命令並引進 Knack 命令架構。 
* [適用於 Azure CLI 的 IoT 擴充功能](https://github.com/Azure/azure-iot-cli-extension) \(英文\)。

## <a name="implement-device-twins-to-configure-devices"></a>實作裝置對應項來設定裝置

自動裝置設定需要使用裝置對應項，以同步處理雲端和裝置之間的狀態。  如需使用裝置對應項的指引，請參閱[了解和使用 IoT 中樞的裝置對應項](iot-hub-devguide-device-twins.md)。

## <a name="identify-devices-using-tags"></a>使用標記識別裝置

在您可建立設定之前，必須指定所要影響的裝置。 Azure IoT 中樞會使用裝置對應項中的標籤來識別裝置。 每個裝置都可以有多個標記，而您可以利用任何對您解決方案有意義的方式來定義它們。 例如，如果您管理的裝置散置在不同位置，可以將下列標籤新增至裝置對應項：

```json
"tags": {
    "location": {
        "state": "Washington",
        "city": "Tacoma"
    }
},
```

## <a name="define-the-target-content-and-metrics"></a>定義目標內容和計量

目標內容和計量查詢是指定為 JSON 文件，它說明要設定的裝置對應項所需屬性和要測量的報告屬性。  若要使用 Azure CLI 建立自動裝置組態，請在本機上將目標內容和計量儲存為 .txt 檔案。 當您執行命令以將組態套用至您的裝置時，稍後會在下一節中使用檔案路徑。 

以下是基本目標內容範例：

```json
{
  "content": {
    "deviceContent": {
      "properties.desired.chillerWaterSettings": {
        "temperature": 38,
        "pressure": 78
      }
    }
}
```

以下是計量查詢的範例：

```json
{
  "queries": {
    "Compliant": "select deviceId from devices where configurations.[[chillersettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='current'",
    "Error": "select deviceId from devices where configurations.[[chillersettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='error'",
    "Pending": "select deviceId from devices where configurations.[[chillersettingswashington]].status = 'Applied' AND properties.reported.chillerWaterSettings.status='pending'"
  }
}
```

## <a name="create-a-configuration"></a>建立設定

您可以建立由目標內容和計量組成的組態，來設定目標裝置。 

使用以下命令建立組態：

```cli
   az iot hub configuration create --config-id [configuration id] \
     --labels [labels] --content [file path] --hub-name [hub name] \
     --target-condition [target query] --priority [int] \
     --metrics [metric queries]
```

* --**config-id** - 將會在 IoT 中樞建立的組態名稱。 為設定指定唯一的名稱，最長為 128 個小寫字母。 避免空格和下列無效字元：`& ^ [ ] { } \ | " < > /`。

* --**labels** - 新增標籤以協助追蹤您的組態。 標籤是成對的「名稱, 值」組合，可描述您的部署。 例如，`HostPlatform, Linux` 或 `Version, 3.0.1`。

* --**content** - 要設為對應項所需屬性的目標內容內嵌 JSON 或檔案路徑。 

* --**hub-name** - 要在其中建立組態的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶

* --**target-condition** - 輸入目標條件來判斷這個組態會將哪些裝置設為目標。 條件會以裝置對應項標籤或裝置對應項所需屬性為基礎，且應符合運算式格式。 例如，`tags.environment='test'` 或 `properties.desired.devicemodel='4000x'`。 

* --**priority** - 正整數。 在兩個以上組態目標為相同裝置的事件中，將會套用具有優先順序最高數值的組態。

* --**metrics** - 計量查詢的檔案路徑。 計量能提供在套用設定內容後，裝置所回報各種狀態的摘要計數。 例如，您可以建立擱置設定變更的計量、錯誤的計量，以及成功設定變更的計量。 

## <a name="monitor-a-configuration"></a>監視設定

使用下列命令來顯示組態的內容︰

```cli
az iot hub configuration show --config-id [configuration id] \
  --hub-name [hub name]
```

* --**config-id** - 存在於 IoT 中樞的組態名稱。

* --**hub-name** - 組態存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需訂用帳戶

在命令視窗中檢查組態。 **metrics** 屬性會列出每個中樞評估的每個計量計數：

* **targetedCount** - 系統計量，指定 IoT 中樞中符合目標條件的裝置對應項數目。

* **appliedCount** - 系統計量，指定已套用目標內容的裝置數目。

* **您的自訂計量** - 您定義的任何計量會視為使用者計量。

您可以使用下列命令，顯示每個計量的裝置識別碼或物件清單：

```cli
az iot hub configuration show-metric --config-id [configuration id] \
   --metric-id [metric id] --hub-name [hub name] --metric-type [type] 
```

* --**config-id** - 存在於 IoT 中樞的部署名稱。

* --**metric-id** - 您想要在其中查看裝置識別碼清單的計量名稱，例如 `appliedCount`。

* --**hub-name** - 部署存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶。

* --**metric-type** - 計量類型可以是 `system` 或 `user`。  系統計量是 `targetedCount` 和 `appliedCount`。 其他所有計量是使用者計量。

## <a name="modify-a-configuration"></a>修改設定

當您修改設定時，所做的變更會立即複寫到所有目標裝置。 

如果您更新目標條件，就會發生下列更新：

* 如果裝置對應項不符合舊的目標條件，但符合新的目標條件，而且此設定為該裝置對應項的最高優先順序設定，則會將此設定套用到該裝置對應項。 

* 如果裝置對應項不再符合目標條件，系統會將設定中的設定移除，並以優先順序次高的設定來修改該裝置對應項。 

* 如果目前執行此設定的裝置對應項不再符合目標條件，也不符合任何其他設定的目標條件，則系統會將設定中的設定移除，且不會對該對應項進行任何其他變更。 

使用以下命令更新組態：

```cli
az iot hub configuration update --config-id [configuration id] \
   --hub-name [hub name] --set [property1.property2='value']
```

* --**config-id** - 存在於 IoT 中樞的組態名稱。

* --**hub-name** - 組態存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶。

* --**set** - 更新組態中的屬性。 您可以更新下列屬性：

    * targetCondition - 例如 `targetCondition=tags.location.state='Oregon'`

    * 標籤 

    * 優先順序

## <a name="delete-a-configuration"></a>刪除設定

當您刪除設定時，任何裝置對應項都會採用其優先順序次高的設定。 如果裝置對應項不符合任何其他設定的目標條件，則不會套用任何其他設定。 

使用以下命令刪除組態：

```cli
az iot hub configuration delete --config-id [configuration id] \
   --hub-name [hub name] 
```
* --**config-id** - 存在於 IoT 中樞的組態名稱。

* --**hub-name** - 組態存在於其中的 IoT 中樞名稱。 中樞必須在目前訂用帳戶中。 使用 `az account set -s [subscription name]` 命令切換到所需的訂用帳戶。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何大規模設定和監視 IoT 裝置。 遵循下列連結以深入了解如何管理 Azure IoT 中樞：

* [管理大量的 IoT 中樞裝置身分識別](iot-hub-bulk-identity-mgmt.md)
* [IoT 中樞計量](iot-hub-metrics.md)
* [作業監視](iot-hub-operations-monitoring.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [IoT 中樞開發人員指南](iot-hub-devguide.md)
* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)

若要探索使用 IoT 中樞裝置佈建服務進行 Just-In-Time 自動佈建，請參閱： 

* [Azure IoT 中樞裝置佈建服務](/azure/iot-dps)
