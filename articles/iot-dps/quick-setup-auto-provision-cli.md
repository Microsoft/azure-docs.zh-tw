---
title: 使用 Azure 入口網站設定裝置佈建 | Microsoft Docs
description: Azure 快速入門 - 使用 Azure CLI 設定 Azure IoT 中樞裝置佈建服務
author: wesmc7777
ms.author: wesmc
ms.date: 02/26/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: cf2e108aa7cab6be2996cb535d27d597e462617c
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39626534"
---
# <a name="set-up-the-iot-hub-device-provisioning-service-with-azure-cli"></a>使用 Azure CLI 設定 IoT 中樞裝置佈建服務

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門詳細說明如何使用 Azure CLI 來建立 IoT 中樞和 IoT 中樞裝置佈建服務，以及將這兩項服務連結在一起。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!IMPORTANT]
> 您在本快速入門中建立的 IoT 中樞與佈建服務均可作為 DNS 端點公開探索。 如果您決定變更這些資源所用的名稱，請務必避免使用任何敏感性資訊。
>


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 westus 位置建立名為 my-sample-resource-group 的資源群組。

```azurecli-interactive 
az group create --name my-sample-resource-group --location westus
```

> [!TIP]
> 此範例會建立位於美國西部位置的資源群組。 您可以執行命令 `az account list-locations -o table`，以檢視可用位置清單。
>
>

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

使用 [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create) 命令建立 IoT 中樞。

下列範例會在 westus 位置建立名為 my-sample-hub 的 IoT 中樞。  

```azurecli-interactive 
az iot hub create --name my-sample-hub --resource-group my-sample-resource-group --location westus
```

## <a name="create-a-provisioning-service"></a>建立佈建服務

使用 [az iot dps create](/cli/azure/iot/dps#az-iot-dps-create) 命令建立佈建服務。 

下列範例會在 westus 位置建立名為 my-sample-dps 的佈建服務。  

```azurecli-interactive 
az iot dps create --name my-sample-dps --resource-group my-sample-resource-group --location westus
```

> [!TIP]
> 此範例會在美國西部位置建立佈建服務。 執行 `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` 命令，或移至 [Azure 狀態](https://azure.microsoft.com/status/)頁面並搜尋「裝置佈建服務」，即可檢視可用的位置清單。 在命令中，可指定一個字或多字格式的位置；例如：uswest、West US、WEST US 等等。此值不區分大小寫。 如果您使用多字格式來指定位置，此值會用引號括住；例如 `-- location "West US"`。
>


## <a name="get-the-connection-string-for-the-iot-hub"></a>取得 IoT 中樞的連接字串

您需要 IoT 中樞的連接字串，才能將它與裝置佈建服務連結。 使用 [az iot hub show-connection-string](/cli/azure/iot/hub#az-iot-hub-show-connection-string) 命令取得連接字串，並使用其輸出來設定將在連結兩項資源時使用的變數。 

下列範例會將 hubConnectionString 變數設定為連接字串的值，作為中樞之 iothubowner 原則的主索引鍵。 您可以使用 `--policy-name` 參數來指定不同的原則。 此命令會使用 Azure CLI [查詢](/cli/azure/query-azure-cli)和[輸出](/cli/azure/format-output-azure-cli#tsv-output-format)選項，從命令輸出中擷取連接字串。

```azurecli-interactive 
hubConnectionString=$(az iot hub show-connection-string --name my-sample-hub --key primary --query connectionString -o tsv)
```

您可以使用 `echo` 命令來查看連接字串。

```azurecli-interactive 
echo $hubConnectionString
```

> [!NOTE]
> 這兩個命令均適用於在 Bash 下執行的主機。 如果您使用本機 Windows/CMD shell 或 PowerShell 主機，您需要修改命令才能使用該環境的正確語法。
>

## <a name="link-the-iot-hub-and-the-provisioning-service"></a>連結 IoT 中樞與佈建服務

使用 [az iot dps linked-hub create](/cli/azure/iot/dps/linked-hub#az-iot-dps-linked-hub-create) 命令來連結 IoT 中樞與佈建服務。 

下列範例會在 westus 位置建立名為 my-sample-hub 的 IoT 中樞，以及建立名為 my-sample-dps 的裝置佈建服務。 它會使用上一個步驟的 hubConnectionString 變數中儲存之 my-sample-hub 的連接字串。

```azurecli-interactive 
az iot dps linked-hub create --dps-name my-sample-dps --resource-group my-sample-resource-group --connection-string $hubConnectionString --location westus
```

## <a name="verify-the-provisioning-service"></a>驗證佈建服務

使用 [az iot dps show](/cli/azure/iot/dps#az-iot-dps-show) 命令取得佈建服務的詳細資料。

下列範例會取得名為 my-sample-dps 的佈建服務詳細資料。 連結的 IoT 中樞會顯示在 properties.iotHubs 集合中。

```azurecli-interactive
az iot dps show --name my-sample-dps
```

## <a name="clean-up-resources"></a>清除資源

此集合中的其他快速入門會以本快速入門為基礎。 如果您打算繼續進行後續的快速入門或教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續執行，可以使用下列命令來刪除佈建服務、IoT 中樞或資源群組及其所有資源。

若要刪除佈建服務，請執行 [az iot dps delete](/cli/azure/iot/dps#az-iot-dps-delete) 命令：

```azurecli-interactive
az iot dps delete --name my-sample-dps --resource-group my-sample-resource-group
```
若要刪除 IoT 中樞，請執行 [az iot hub delete](/cli/azure/iot/hub#az-iot-hub-delete) 命令：

```azurecli-interactive
az iot hub delete --name my-sample-hub --resource-group my-sample-resource-group
```

若要刪除資源群組和其所有資源，請執行 [az group delete](/cli/azure/group#az-group-delete) 命令：

```azurecli-interactive
az group delete --name my-sample-resource-group
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 IoT 中樞和裝置佈建服務執行個體，並已連結這兩個資源。 若要深入了解如何使用這項設定來佈建模擬裝置，請繼續進行建立模擬裝置的快速入門。

> [!div class="nextstepaction"]
> [建立模擬裝置的快速入門](./quick-create-simulated-device.md)

