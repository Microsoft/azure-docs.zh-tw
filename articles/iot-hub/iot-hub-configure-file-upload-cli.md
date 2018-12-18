---
title: 使用 Azure CLI 設定檔案上傳到 IoT 中樞 | Microsoft Docs
description: 如何使用跨平台 Azure CLI 來設定檔案上傳到 Azure IoT 中樞。
author: dominicbetts
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: dobett
ms.openlocfilehash: 6cd0b657c8d0352c41e0da538396b166d633306a
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141700"
---
# <a name="configure-iot-hub-file-uploads-using-azure-cli"></a>使用 Azure CLI 來設定 IoT 中樞檔案上傳

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

若要[從裝置上傳檔案](iot-hub-devguide-file-upload.md)，您必須先將 Azure 儲存體帳戶與 IoT 中樞建立關聯。 您可以使用現有的儲存體帳戶或建立新帳戶。

若要完成此教學課程，您需要下列項目：

* 使用中的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。

* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。

* Azure IoT 中樞。 如果您沒有 IoT 中樞，您可以使用 [`az iot hub create` 命令](https://docs.microsoft.com/cli/azure/iot/hub#az-iot-hub-create)來建立一個，或[使用入口網站建立 IoT 中樞](iot-hub-create-through-portal.md)。

* Azure 儲存體帳戶。 如果您沒有 Azure 儲存體帳戶，您可以使用 [Azure CLI - 管理儲存體帳戶](../storage/common/storage-azure-cli.md#manage-storage-accounts)來建立一個，或使用入口網站[建立儲存體帳戶](../storage/common/storage-create-storage-account.md)。

## <a name="sign-in-and-set-your-azure-account"></a>登入並設定 Azure 帳戶

登入您的 Azure 帳戶並選取您的訂用帳戶。

1. 在命令提示字元中，執行[登入命令](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)：

    ```azurecli
    az login
    ```

    依照指示使用程式碼進行驗證，並透過網頁瀏覽器登入 Azure 帳戶。

2. 如果您有多個 Azure 訂用帳戶，則登入 Azure 會授予您所有與認證相關聯之 Azure 帳戶的存取權。 使用下列[命令列出 Azure 帳戶](https://docs.microsoft.com/cli/azure/account) \(英文\) 以供您使用：

    ```azurecli
    az account list
    ```

    使用下列命令，來選取您想要用來執行命令以建立 IoT 中樞的訂用帳戶。 您可以使用來自上一個命令之輸出內的訂用帳戶名稱或識別碼︰

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

## <a name="retrieve-your-storage-account-details"></a>擷取您的儲存體帳戶詳細資料

下列步驟假設您使用 [Resource Manager] 部署模型，而非 [傳統] 部署模型，建立了儲存體帳戶。

若要從裝置設定檔案上傳，您需要 Azure 儲存體帳戶的連接字串。 儲存體帳戶必須與您的 IoT 中樞位於相同的訂用帳戶中。 您也需要儲存體帳戶中 blob 容器的名稱。 使用下列命令來擷取儲存體帳戶金鑰：

```azurecli
az storage account show-connection-string --name {your storage account name} \
  --resource-group {your storage account resource group}
```

記下 **connectionString** 值。 您需要在後續步驟中用到此值。

您可以使用現有的 Blob 容器進行檔案上傳，或建立新的容器：

* 若要列出儲存體帳戶中現有的 blob 容器，使用下列命令：

    ```azurecli
    az storage container list --connection-string "{your storage account connection string}"
    ```

* 若要在儲存體帳戶中建立 blob 容器，使用下列命令：

    ```azurecli
    az storage container create --name {container name} \
      --connection-string "{your storage account connection string}"
    ```

## <a name="file-upload"></a>檔案上傳

您現在可以設定 IoT 中樞，以便能夠使用您的儲存體帳戶詳細資料[將檔案上傳到 IoT 中樞](iot-hub-devguide-file-upload.md)。

設定需要下列值：

* **儲存體容器**︰您目前 Azure 訂用帳戶內 Azure 儲存體帳戶中的 blob 容器，會與您的 IoT 中樞產生關聯。 您已在上一節中擷取了必要的儲存體帳戶資訊。 IoT 中樞會自動產生具有此 Blob 容器寫入權限的 SAS URI，以供裝置上傳檔案時使用。

* **接收已上傳檔案的通知**︰啟用或停用檔案上傳通知。

* **SAS TTL**︰這個設定是「IoT 中樞」傳回給裝置之 SAS URI 的存留時間。 預設會設為一小時。

* **檔案通知設定預設 TTL**：檔案上傳通知到期前的存留時間。 預設會設為一天。

* **檔案通知最大傳遞計數**︰IoT 中樞可嘗試傳遞檔案上傳通知的次數。 預設會設為 10。

使用下列 Azure CLI 命令來設定 IoT 中樞上的檔案上傳設定：

<!--Robinsh this is out of date, add cloud powershell -->

在 Bash 殼層中，使用：

```azurecli
az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.connectionString="{your storage account connection string}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.containerName="{your storage container name}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.sasTtlAsIso8601=PT1H0M0S

az iot hub update --name {your iot hub name} \
  --set properties.enableFileUploadNotifications=true

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.maxDeliveryCount=10

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.ttlAsIso8601=PT1H0M0S
```

您可以使用下列命令，檢閱 IoT 中樞上的檔案上傳組態︰

```azurecli
az iot hub show --name {your iot hub name}
```

## <a name="next-steps"></a>後續步驟

如需 IoT 中樞檔案上傳功能的詳細資訊，請參閱[從裝置上傳檔案](iot-hub-devguide-file-upload.md)。

遵循下列連結以深入了解如何管理 Azure IoT 中樞：

* [大量管理 IoT 裝置](iot-hub-bulk-identity-mgmt.md)
* [IoT 中樞計量](iot-hub-metrics.md)
* [作業監視](iot-hub-operations-monitoring.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [IoT 中樞開發人員指南](iot-hub-devguide.md)
* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)
* [徹底保護您的 IoT 解決方案](../iot-fundamentals/iot-security-ground-up.md)
