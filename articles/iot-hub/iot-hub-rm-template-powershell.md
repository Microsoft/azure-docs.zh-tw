---
title: 使用範本建立 Azure IoT 中樞 (PowerShell) | Microsoft Docs
description: 如何在 PowerShell 中使用 Azure Resource Manager 範本建立 IoT 中樞。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: dobett
ms.openlocfilehash: cdc53aeb0ecbea5714a8587c8d03ab6aa292394c
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34634993"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-powershell"></a>使用 Azure Resource Manager 範本建立 IoT 中樞 (PowerShell)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

您可以使用 Azure 資源管理員，以程式設計方式建立和管理 Azure IoT 中樞。 本教學課程示範如何使用 Azure Resource Manager 範本以 PowerShell 建立 IoT 中樞。

> [!NOTE]
> Azure 建立和處理資源的部署模型有兩種：[Azure Resource Manager](../azure-resource-manager/resource-manager-deployment-model.md) 和傳統。 本文涵蓋使用 Azure Resource Manager 部署模型的部分。

若要完成此教學課程，您需要下列項目：

* 使用中的 Azure 帳戶。 <br/>如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。
* [Azure PowerShell 1.0][lnk-powershell-install] 或更新版本。

> [!TIP]
> [搭配使用 Azure PowerShell 與 Azure Resource Manager][lnk-powershell-arm] 一文提供有關如何使用 PowerShell 和 Azure Resource Manager 範本來建立 Azure 資源的詳細資訊。

## <a name="connect-to-your-azure-subscription"></a>連接到 Azure 訂用帳戶

在 PowerShell 命令提示字元中，輸入下列命令來登入您的 Azure 訂用帳戶：

```powershell
Connect-AzureRmAccount
```

如果您有多個 Azure 訂用帳戶，則登入 Azure 即會為您授與和您認證相關聯之所有 Azure 訂用帳戶的存取權。 使用下列命令來列出可供您使用的 Azure 訂用帳戶：

```powershell
Get-AzureRMSubscription
```

使用下列命令，選取您想要用來執行命令以建立 IoT 中樞的訂用帳戶。 您可以使用來自上一個命令之輸出內的訂用帳戶名稱或識別碼︰

```powershell
Select-AzureRMSubscription `
    -SubscriptionName "{your subscription name}"
```

您可以使用下列命令來探索可部署 IoT 中樞的位置和目前支援的 API 版本：

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).Locations
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).ApiVersions
```

在 IoT 中樞支援的其中一個位置，使用下列命令建立資源群組來包含您的 IoT 中樞。 這個範例會建立一個稱為 **MyIoTRG1**的資源群組：

```powershell
New-AzureRmResourceGroup -Name MyIoTRG1 -Location "East US"
```

## <a name="submit-a-template-to-create-an-iot-hub"></a>提交範本，以建立 IoT 中樞

使用 JSON 範本在資源群組中建立 IoT 中樞。 您也可以使用 Azure Resource Manager 範本來對現有的 IoT 中樞進行變更。

1. 使用文字編輯器來建立名為 **template.json** 的 Azure Resource Manager 範本，其中包含下列資源定義來建立新的標準 IoT 中樞。 這個範例會在「美國東部」區域新增「IoT 中樞」、在「事件中樞」相容端點上建立兩個取用者群組 (**cg1** 和 **cg2**)，並使用 **2016-02-03** API 版本。 這個範本也要求您在一個名為 **hubName**的參數中傳入 IoT 中樞名稱。 如需目前支援「IoT 中樞」的位置清單，請參閱 [Azure 狀態][lnk-status]。

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": {
          "type": "string"
        }
      },
      "resources": [
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs",
        "name": "[parameters('hubName')]",
        "location": "East US",
        "sku": {
          "name": "S1",
          "tier": "Standard",
          "capacity": 1
        },
        "properties": {
          "location": "East US"
        }
      },
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs/eventhubEndpoints/ConsumerGroups",
        "name": "[concat(parameters('hubName'), '/events/cg1')]",
        "dependsOn": [
          "[concat('Microsoft.Devices/Iothubs/', parameters('hubName'))]"
        ]
      },
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs/eventhubEndpoints/ConsumerGroups",
        "name": "[concat(parameters('hubName'), '/events/cg2')]",
        "dependsOn": [
          "[concat('Microsoft.Devices/Iothubs/', parameters('hubName'))]"
        ]
      }
      ],
      "outputs": {
        "hubKeys": {
          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
          "type": "object"
        }
      }
    }
    ```

2. 將 Azure Resource Manager 範本檔案儲存在您的本機電腦上。 這個範例假設您將它儲存在名為 **c:\templates** 的資料夾中。

3. 執行下列命令來部署新的 IoT 中樞，並傳遞 IoT 中樞的名稱做為參數。 在此範例中，IoT 中樞的名稱是 `abcmyiothub`。 您的 IoT 中樞名稱必須是全域唯一：

    ```powershell
    New-AzureRmResourceGroupDeployment -ResourceGroupName MyIoTRG1 -TemplateFile C:\templates\template.json -hubName abcmyiothub
    ```
  [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

4. 輸出會顯示您建立的 IoT 中樞的金鑰。

5. 若要確認您的應用程式已新增新的 IoT 中樞，請前往 [Azure 入口網站][lnk-azure-portal] 並檢視您的資源清單。 或者，使用 **Get-AzureRmResource** PowerShell Cmdlet。

> [!NOTE]
> 此範例應用程式會加入您付費的「S1 標準 IoT 中樞」。 您可透過 [Azure 入口網站][lnk-azure-portal]刪除此 IoT 中樞，或在完成時，使用 **Remove-AzureRmResource** PowerShell Cmdlet。

## <a name="next-steps"></a>後續步驟

現在您已使用 Azure Resource Manager 範本搭配 PowerShell 來部署 IoT 中樞，您可以再進一步探索：

* 閱讀 [IoT 中樞資源提供者 REST API][lnk-rest-api] 功能的相關資訊。
* 如需 Azure Resource Manager 功能的詳細資訊，請參閱 [Azure Resource Manager 概觀][lnk-azure-rm-overview]。

若要深入了解如何開發 IoT 中樞，請參閱以下文章︰

* [C SDK 簡介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azure/install-azurerm-ps
[lnk-rest-api]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md
[lnk-powershell-arm]: ../azure-resource-manager/powershell-azure-resource-manager.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
