---
title: 使用 Azure Resource Manager 範本設定裝置佈建 | Microsoft Docs
description: Azure 快速入門 - 使用範本設定 Azure IoT 中樞裝置佈建服務
author: wesmc7777
ms.author: wesmc
ms.date: 06/18/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 45558e25f2f61090cfb3ddb5c382170ab40792d0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999035"
---
# <a name="set-up-the-iot-hub-device-provisioning-service-with-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本設定 IoT 中樞裝置佈建服務

您可以使用 [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)，以程式設計方式設定佈建裝置所需的 Azure 雲端資源。 下列步驟示範如何使用 Azure Resource Manager 範本來建立 IoT 中樞、新的 IoT 中樞裝置佈建服務，以及將這兩項服務連結在一起。 本快速入門使用 [Azure CLI](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli) 來執行建立資源群組及部署範本所需的程式設計步驟，但您可以輕鬆使用 [Azure 入口網站](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal)、[PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)、.NET、Ruby 或其他程式設計語言來執行這些步驟及部署您的範本。 


## <a name="prerequisites"></a>必要條件

- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
- 此快速入門需要您在本機執行 Azure CLI。 您必須安裝 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。


## <a name="sign-in-to-azure-and-create-a-resource-group"></a>登入 Azure 並建立資源群組

登入您的 Azure 帳戶並選取您的訂用帳戶。

1. 在命令提示字元中，執行[登入命令][lnk-login-command]：
    
    ```azurecli
    az login
    ```

    依照指示使用程式碼進行驗證，並透過網頁瀏覽器登入 Azure 帳戶。

2. 如果您有多個 Azure 訂用帳戶，則登入 Azure 會授予您所有與認證相關聯之 Azure 帳戶的存取權。 使用下列[命令來列出可供您使用的 Azure 帳戶][lnk-az-account-command]︰
    
    ```azurecli
    az account list 
    ```

    使用下列命令，選取您想要用來執行命令以建立 IoT 中樞的訂用帳戶。 您可以使用來自上一個命令之輸出內的訂用帳戶名稱或識別碼︰

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

3. 當您建立 IoT 中樞和佈建服務之類的 Azure 雲端資源時，您可在資源群組中建立它們。 使用現有的資源群組，或執行下列[命令來建立資源群組][lnk-az-resource-command]：
    
    ```azurecli
     az group create --name {your resource group name} --location westus
    ```

    > [!TIP]
    > 上一個範例會建立位於美國西部位置的資源群組。 您可以執行命令 `az account list-locations -o table`，以檢視可用位置清單。
    >
    >

## <a name="create-a-resource-manager-template"></a>建立 Resource Manager 範本

使用 JSON 範本在資源群組中建立佈建服務和連結的 IoT 中樞。 您也可以使用 Azure Resource Manager 範本來對現有的佈建服務或 IoT 中樞進行變更。

1. 使用文字編輯器來建立名為 **template.json** 的 Azure Resource Manager 範本，其中包含下列基本架構內容。 

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {},
       "variables": {},
       "resources": []
   }
   ```

2. 以下列內容取代 **parameters** 區段。 parameters 區段會指定可從其他檔案傳入的參數。 這個區段會指定要建立之 IoT 中樞和佈建服務的名稱。 它也會指定 IoT 中樞與佈建服務的位置。 其值會受限於支援 IoT 中樞和佈建服務的 Azure 區域。 如需裝置佈建服務的支援位置清單，您可以執行下列命令 `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` 或移至 [Azure 狀態](https://azure.microsoft.com/status/)頁面並搜尋「裝置佈建服務」。

   ```json
    "parameters": {
        "iotHubName": {
            "type": "string"
        },
        "provisioningServiceName": {
            "type": "string"
        },
        "hubLocation": {
            "type": "string",
            "allowedValues": [
                "eastus",
                "westus",
                "westeurope",
                "northeurope",
                "southeastasia",
                "eastasia"
            ]
        }
    },

   ```

3. 以下列內容取代 **variables** 區段。 這個區段會定義稍後用來建構 IoT 中樞連接字串的值，這是連結佈建服務與 IoT 中樞所需的項目。 
 
   ```json
    "variables": {        
        "iotHubResourceId": "[resourceId('Microsoft.Devices/Iothubs', parameters('iotHubName'))]",
        "iotHubKeyName": "iothubowner",
        "iotHubKeyResource": "[resourceId('Microsoft.Devices/Iothubs/Iothubkeys', parameters('iotHubName'), variables('iotHubKeyName'))]"
    },

   ```

4. 若要建立 IoT 中樞，請將下列幾行新增至 **resources** 集合。 JSON 會指定建立 IoT 中樞所需的最小屬性。 **name** 和 **location** 屬性會當作參數傳遞。 若要深入了解您可以針對範本中的 IoT 中樞指定的屬性，請參閱 [Microsoft.Devices/IotHubs 範本參考](https://docs.microsoft.com/azure/templates/microsoft.devices/iothubs)。

   ```json
        {
            "apiVersion": "2017-07-01",
            "type": "Microsoft.Devices/IotHubs",
            "name": "[parameters('iotHubName')]",
            "location": "[parameters('hubLocation')]",
            "sku": {
                "name": "S1",
                "capacity": 1
            },
            "tags": {
            },
            "properties": {
            }            
        },

   ``` 

5. 若要建立佈建服務，請將下列幾行新增至 **resources** 集合中的 IoT 中樞規格之後。 佈建服務的 **name** 和 **location** 是傳入的參數。 指定 IoT 中樞以連結至 **iotHubs** 集合中的佈建服務。 您至少必須指定每個已連結 IoT 中樞的 **connectionString** 和 **location** 屬性。 您也可以設定每個 IoT 中樞的屬性 (例如 **allocationWeight** 和 **applyAllocationPolicy**)，以及佈建服務本身的屬性 (例如 **allocationPolicy** 和 **authorizationPolicies**)。 若要深入了解，請參閱 [Microsoft.Devices/provisioningServices 範本參考](https://docs.microsoft.com/azure/templates/microsoft.devices/provisioningservices)。

   **dependsOn** 屬性用來確保 Resource Manager 會先建立 IoT 中樞，再建立佈建服務。 此範本需要 IoT 中樞的連接字串，以指定其與佈建服務的連結，因此必須先建立中樞與其索引鍵。 此範本使用 **concat** 和 **listkeys** 之類的函式來建立連接字串。 若要深入了解，請參閱 [Azure Resource Manager 範本函式](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-functions)。

   ```json
        {
            "type": "Microsoft.Devices/provisioningServices",
            "sku": {
                "name": "S1",
                "capacity": 1
            },
            "name": "[parameters('provisioningServiceName')]",
            "apiVersion": "2017-11-15",
            "location": "[parameters('hubLocation')]",
            "tags": {},
            "properties": {
                "iotHubs": [
                    {
                        "connectionString": "[concat('HostName=', reference(variables('iotHubResourceId')).hostName, ';SharedAccessKeyName=', variables('iotHubKeyName'), ';SharedAccessKey=', listkeys(variables('iotHubKeyResource'), '2017-07-01').primaryKey)]",
                        "location": "[parameters('hubLocation')]",
                        "name": "[concat(parameters('iotHubName'),'.azure-devices.net')]"
                    }
                ]
            },
            "dependsOn": ["[parameters('iotHubName')]"]
        }

   ```

6. 儲存範本檔案。 完成的範本看起來應該如下所示：

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
           "iotHubName": {
               "type": "string"
           },
           "provisioningServiceName": {
               "type": "string"
           },
           "hubLocation": {
               "type": "string",
               "allowedValues": [
                   "eastus",
                   "westus",
                   "westeurope",
                   "northeurope",
                   "southeastasia",
                   "eastasia"
               ]
           }
       },
       "variables": {        
           "iotHubResourceId": "[resourceId('Microsoft.Devices/Iothubs', parameters('iotHubName'))]",
           "iotHubKeyName": "iothubowner",
           "iotHubKeyResource": "[resourceId('Microsoft.Devices/Iothubs/Iothubkeys', parameters('iotHubName'), variables('iotHubKeyName'))]"
       },
       "resources": [
           {
               "apiVersion": "2017-07-01",
               "type": "Microsoft.Devices/IotHubs",
               "name": "[parameters('iotHubName')]",
               "location": "[parameters('hubLocation')]",
               "sku": {
                   "name": "S1",
                   "capacity": 1
               },
               "tags": {
               },
               "properties": {
               }            
           },
           {
               "type": "Microsoft.Devices/provisioningServices",
               "sku": {
                   "name": "S1",
                   "capacity": 1
               },
               "name": "[parameters('provisioningServiceName')]",
               "apiVersion": "2017-11-15",
               "location": "[parameters('hubLocation')]",
               "tags": {},
               "properties": {
                   "iotHubs": [
                       {
                           "connectionString": "[concat('HostName=', reference(variables('iotHubResourceId')).hostName, ';SharedAccessKeyName=', variables('iotHubKeyName'), ';SharedAccessKey=', listkeys(variables('iotHubKeyResource'), '2017-07-01').primaryKey)]",
                           "location": "[parameters('hubLocation')]",
                           "name": "[concat(parameters('iotHubName'),'.azure-devices.net')]"
                       }
                   ]
               },
               "dependsOn": ["[parameters('iotHubName')]"]
           }
       ]
   }
   ```

## <a name="create-a-resource-manager-parameter-file"></a>建立 Resource Manager 參數檔案

您在最後一個步驟中定義的範本會使用參數來指定 IoT 中樞的名稱、佈建服務的名稱，以及其建立位置 (Azure 地區)。 您可將這些參數傳入個別檔案中。 如此一來，您可以針對多個部署重複使用相同的範本。 若要建立參數檔案，請依照下列步驟執行︰

1. 使用文字編輯器來建立名為 **parameters.json** 的 Azure Resource Manager 範本，其中包含下列基本架構內容： 

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {}
       }
   }
   ```

2. 將 **iotHubName** 值新增至 parameters 區段。 如果您變更此名稱，請確保它會遵循 IoT 中樞的適當命名慣例。 其長度應該是 3-50 個字元，而且只能包含大寫或小寫英數字元或連字號 ('-')。 

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
    }
   
   ```

3. 將 **provisioningServiceName** 值新增至 parameters 區段。 如果您變更此名稱，請確保它會遵循 IoT 中樞裝置佈建服務的適當命名慣例。 其長度應該是 3-64 個字元，而且只能包含大寫或小寫英數字元或連字號 ('-')。

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
        "provisioningServiceName": {
            "value": "my-sample-provisioning-service"
        },
    }

   ```

4. 將 **hubLocation** 值新增至 parameters 區段。 此值也會指定 IoT 中樞與佈建服務的位置。 此值必須對應到範本檔案的參數定義中 **allowedValues** 集合中所指定的其中一個位置。 此集合會將其值限制於支援 IoT 中樞和佈建服務的 Azure 位置。 如需裝置佈建服務的支援位置清單，您可以執行下列命令 `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` 或移至 [Azure 狀態](https://azure.microsoft.com/status/)頁面並搜尋「裝置佈建服務」。

   ```json
    "parameters": {
        "iotHubName": {
            "value": "my-sample-iot-hub"
        },
        "provisioningServiceName": {
            "value": "my-sample-provisioning-service"
        },
        "hubLocation": {
            "value": "westus"
        }
    }

   ```

5. 儲存檔案。 


> [!IMPORTANT]
> IoT 中樞與佈建服務均可作為 DNS 端點公開探索，所以在命名時務必避免使用任何敏感性資訊。
>

## <a name="deploy-the-template"></a>部署範本

請使用下列 Azure CLI 命令來部署您的範本，並確認部署。

1. 若要部署範本，請執行下列[命令以開始部署](https://docs.microsoft.com/cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create)：
    
    ```azurecli
     az group deployment create -g {your resource group name} --template-file template.json --parameters @parameters.json
    ```

   在輸出中尋找已設為 "Succeeded" 的 **provisioningState** 屬性。 

   ![佈建輸出](./media/quick-setup-auto-provision-rm/output.png) 


2. 若要確認您的部署，請執行下列[命令以列出資源](https://docs.microsoft.com/cli/azure/resource?view=azure-cli-latest#az-resource-list)，並在輸出中尋找新的佈建服務和 IoT 中樞：

    ```azurecli
     az resource list -g {your resource group name}
    ```


## <a name="clean-up-resources"></a>清除資源

此集合中的其他快速入門會以本快速入門為基礎。 如果您打算繼續進行後續的快速入門或教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續執行，可以使用 Azure CLI 來[刪除個別資源][lnk-az-resource-command] (例如 IoT 中樞或佈建服務)，或刪除資源群組及其所有資源。

若要刪除佈建服務，請執行下列命令：

```azurecli
az iot hub delete --name {your provisioning service name} --resource-group {your resource group name}
```
若要刪除 IoT 中樞，請執行下列命令︰

```azurecli
az iot hub delete --name {your iot hub name} --resource-group {your resource group name}
```

若要刪除資源群組和其所有資源，請執行下列命令︰

```azurecli
az group delete --name {your resource group name}
```

您也可以使用 Azure 入口網站、PowerShell 或 REST API，或使用針對 Azure Resource Manager 或 IoT 中樞裝置佈建服務發佈的支援平台 SDK，刪除資源群組和個別的資源。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 IoT 中樞和裝置佈建服務執行個體，並已連結這兩個資源。 若要深入了解如何使用這項設定來佈建模擬裝置，請繼續進行建立模擬裝置的快速入門。

> [!div class="nextstepaction"]
> [建立模擬裝置的快速入門](./quick-create-simulated-device.md)


<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-CLI-install]: https://docs.microsoft.com/cli/azure/install-az-cli2
[lnk-login-command]: https://docs.microsoft.com/cli/azure/get-started-with-az-cli2
[lnk-az-account-command]: https://docs.microsoft.com/cli/azure/account
[lnk-az-register-command]: https://docs.microsoft.com/cli/azure/provider
[lnk-az-addcomponent-command]: https://docs.microsoft.com/cli/azure/component
[lnk-az-resource-command]: https://docs.microsoft.com/cli/azure/resource
[lnk-az-iot-command]: https://docs.microsoft.com/cli/azure/iot
[lnk-iot-pricing]: https://azure.microsoft.com/pricing/details/iot-hub/
[lnk-devguide]: iot-hub-devguide.md
[lnk-portal]: iot-hub-create-through-portal.md 
