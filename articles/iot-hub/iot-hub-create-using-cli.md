---
title: 使用 Azure CLI 建立 IoT 中樞 | Microsoft Docs
description: 如何使用 Azure CLI 建立 Azure IoT 中樞。
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: robinsh
ms.openlocfilehash: 90830c4e27e90af6c9d77509844696f64e4909f0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46994763"
---
# <a name="create-an-iot-hub-using-the-azure-cli"></a>使用 Azure CLI 建立 IoT 中樞

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

本文將說明如何使用 Azure CLI 建立 IoT 中樞。

## <a name="prerequisites"></a>必要條件

若要完成此操作說明，您需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-and-set-your-azure-account"></a>登入並設定 Azure 帳戶

如果您是在本機執行 Azure CLI，而不是使用 Cloud Shell，則必須登入您的 Azure 帳戶。

在命令提示字元中，執行[登入命令](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli)：

    ```azurecli
    az login
    ```

依照指示使用程式碼進行驗證，並透過網頁瀏覽器登入 Azure 帳戶。

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

使用 Azure CLI 建立資源群組，然後新增 IoT 中樞。

1. 在建立 IoT 中樞時，必須將其建立在資源群組內。 使用現有的資源群組，或執行下列[命令來建立資源群組](https://docs.microsoft.com/cli/azure/resource) (英文)：
    
   ```azurecli
   az group create --name {your resource group name} --location westus
   ```

   > [!TIP]
   > 上一個範例會建立位於美國西部位置的資源群組。 您可以執行下列命令，以檢視可用位置清單： 
   >
   >``` bash
   >az account list-locations -o table
   >```
   >

2. 使用 IoT 中樞的全域唯一名稱，在資源群組中執行下列[命令建立 IoT 中樞](https://docs.microsoft.com/cli/azure/iot/hub#az-iot-hub-create) (英文)：
    
   ```azurecli
   az iot hub create --name {your iot hub name} \
      --resource-group {your resource group name} --sku S1
   ```

   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]


上一個命令會在您付費使用的 S1 定價層中建立 IoT 中樞。 如需詳細資訊，請參閱 [Azure IoT 中樞價格](https://azure.microsoft.com/pricing/details/iot-hub/)。

## <a name="remove-an-iot-hub"></a>移除 IoT 中樞

您可以使用 Azure CLI 來[刪除個別資源](https://docs.microsoft.com/cli/azure/resource) (英文，例如 IoT 中樞)，或刪除資源群組和其所有資源 (包括任何 IoT 中樞)。

若要[刪除 IoT 中樞](https://docs.microsoft.com/cli/azure/iot/hub#az-iot-hub-delete) (英文)，請執行下列命令︰

```azurecli
az iot hub delete --name {your iot hub name} -\
  -resource-group {your resource group name}
```

若要[刪除資源群組](https://docs.microsoft.com/cli/azure/group#az-group-delete) (英文) 及其所有資源，請執行下列命令︰

```azurecli
az group delete --name {your resource group name}
```

## <a name="next-steps"></a>後續步驟

若要深入了解如何使用 IoT 中樞，請參閱以下文章︰

* [IoT 中樞開發人員指南](iot-hub-devguide.md)
* [使用 Azure 入口網站管理 IoT 中樞](iot-hub-create-through-portal.md)
