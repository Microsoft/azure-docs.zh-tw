---
title: 使用 Azure CLI & IoT 擴充功能來管理 IoT 中樞裝置布建服務
description: '瞭解如何使用 Azure CLI 和 IoT 擴充功能來管理 IoT 中樞裝置布建服務 (DPS) '
author: chrissie926
ms.author: menchi
ms.date: 01/17/2018
ms.topic: conceptual
ms.service: iot-dps
ms.custom: devx-track-azurecli
services: iot-dps
ms.openlocfilehash: dd0564fbb23a0695d849852fd464308cd1b5fac9
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98678938"
---
# <a name="how-to-use-azure-cli-and-the-iot-extension-to-manage-the-iot-hub-device-provisioning-service"></a>如何使用 Azure CLI 和 IoT 擴充功能來管理 IoT 中樞裝置佈建服務

[Azure CLI](/cli/azure) 是一個開放原始碼跨平台命令列工具，用來管理 Azure 資源 (例如 IoT Edge)。 Azure CLI 可在 Windows、Linux 和 macOS 上取得。 Azure CLI 可讓您管理 Azure IoT 中樞資源、裝置佈建服務執行個體，以及現成的連結中樞。

IoT 擴充功能以裝置管理和完整 IoT Edge 功能來擴充 Azure CLI 的功能。

在本教學課程中，您會先完成設定 Azure CLI 和 IoT 擴充功能的步驟。 然後您會了解如何執行 CLI 命令，以執行基本裝置佈建服務作業。 

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="prerequisites"></a>必要條件

- 需要 [Python 2.7x 或 Python 3.x](https://www.python.org/downloads/)。

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- 本文需要 2.0.70 版或更新版本的 Azure CLI。 如果您是使用 Azure Cloud Shell，就已安裝最新版本。

## <a name="basic-device-provisioning-service-operations"></a>基本裝置佈建服務作業

此範例說明如何登入您的 Azure 帳戶、建立 Azure 資源群組 (可保存 Azure 解決方案相關資源的容器)、建立 IoT 中樞、建立裝置佈建服務、列出現有的裝置佈建服務，以及使用 CLI 命令建立連結的 IoT 中樞。 

開始之前，請先完成先前所述的安裝步驟。 如果您沒有 Azure 帳戶，可以立即[建立一個免費帳戶](https://azure.microsoft.com/free/?v=17.39a)。 


### <a name="1-log-in-to-the-azure-account"></a>1. 登入 Azure 帳戶
  
```azurecli
az login
```

![螢幕擷取畫面顯示執行命令 az login 的命令提示字元視窗。](./media/how-to-manage-dps-with-cli/login.jpg)

### <a name="2-create-a-resource-group-iothubblogdemo-in-eastus"></a>2. 在 eastus 中建立資源群組 IoTHubBlogDemo

```azurecli
az group create -l eastus -n IoTHubBlogDemo
```

![建立資源群組](./media/how-to-manage-dps-with-cli/create-resource-group.jpg)


### <a name="3-create-two-device-provisioning-services"></a>3. 建立兩個裝置布建服務

```azurecli
az iot dps create --resource-group IoTHubBlogDemo --name demodps
```

![建立裝置佈建服務](./media/how-to-manage-dps-with-cli/create-dps.jpg)

```azurecli
az iot dps create --resource-group IoTHubBlogDemo --name demodps2
```

### <a name="4-list-all-the-existing-device-provisioning-services-under-this-resource-group"></a>4. 列出此資源群組之下的所有現有裝置布建服務

```azurecli
az iot dps list --resource-group IoTHubBlogDemo
```

![列出裝置佈建服務](./media/how-to-manage-dps-with-cli/list-dps.jpg)


### <a name="5-create-an-iot-hub-blogdemohub-under-the-newly-created-resource-group"></a>5. 在新建立的資源群組下建立 IoT 中樞 blogDemoHub

```azurecli
az iot hub create --name blogDemoHub --resource-group IoTHubBlogDemo
```

![建立 IoT 中樞](./media/how-to-manage-dps-with-cli/create-hub.jpg)

### <a name="6-link-one-existing-iot-hub-to-a-device-provisioning-service"></a>6. 將一個現有的 IoT 中樞連結至裝置布建服務

```azurecli
az iot dps linked-hub create --resource-group IoTHubBlogDemo --dps-name demodps --connection-string <connection string> -l westus
```

![連結中樞](./media/how-to-manage-dps-with-cli/create-hub.jpg)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

前進到下一個教學課程，以了解如何跨負載平衡中樞來佈建多個裝置。 

> [!div class="nextstepaction"]
> [跨負載平衡 IoT 中樞來佈建裝置](./tutorial-provision-multiple-hubs.md)