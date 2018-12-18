---
title: 使用 Azure PowerShell 設定檔案上傳 | Microsoft Docs
description: 如何使用 Azure PowerShell Cmdlet 來設定 IoT 中樞，以便能夠從連接的裝置上傳檔案。 包含設定目的地 Azure 儲存體帳戶的相關資訊。
author: dominicbetts
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: dobett
ms.openlocfilehash: f468567aa67560eed972fb53669638a17778b307
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189949"
---
# <a name="configure-iot-hub-file-uploads-using-powershell"></a>使用 PowerShell 設定 IoT 中樞檔案上傳

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

若要使用 [IoT 中樞的檔案上傳功能](iot-hub-devguide-file-upload.md)，您必須先將 Azure 儲存體帳戶與您的 IoT 中樞建立關聯。 您可以使用現有的儲存體帳戶或建立新帳戶。

若要完成此教學課程，您需要下列項目：

* 使用中的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立 [免費帳戶](http://azure.microsoft.com/pricing/free-trial/) 。

* [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。

* Azure IoT 中樞。 如果您沒有 IoT 中樞，您可以使用 [New-AzureRmIoTHub Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.iothub/new-azurermiothub) 來建立一個，或使用入口網站來[建立 IoT 中樞](iot-hub-create-through-portal.md)。

* 一個 Azure 儲存體帳戶。 如果您沒有 Azure 儲存體帳戶，您可以使用 [Azure 儲存體 PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.storage/) 來建立一個，或使用入口網站來[建立儲存體帳戶](../storage/common/storage-create-storage-account.md)。

## <a name="sign-in-and-set-your-azure-account"></a>登入並設定 Azure 帳戶

登入您的 Azure 帳戶並選取您的訂用帳戶。

1. 在 PowerShell 命令提示字元中，執行 **Connect-AzureRmAccount** Cmdlet：

    ```powershell
    Connect-AzureRmAccount
    ```

2. 如果您有多個 Azure 訂用帳戶，則登入 Azure 即會為您授與和您認證相關聯之所有 Azure 訂用帳戶的存取權。 使用下列命令來列出可供您使用的 Azure 訂用帳戶：

    ```powershell
    Get-AzureRMSubscription
    ```

    使用下列命令，來選取您想要用來執行命令以管理 IoT 中樞的訂用帳戶。 您可以使用來自上一個命令之輸出內的訂用帳戶名稱或識別碼︰

    ```powershell
    Select-AzureRMSubscription `
        -SubscriptionName "{your subscription name}"
    ```

## <a name="retrieve-your-storage-account-details"></a>擷取您的儲存體帳戶詳細資料

下列步驟假設您使用 [Resource Manager] 部署模型，而非 [傳統] 部署模型，建立了儲存體帳戶。

若要從裝置設定檔案上傳，您需要 Azure 儲存體帳戶的連接字串。 儲存體帳戶必須與您的 IoT 中樞位於相同的訂用帳戶中。 您也需要儲存體帳戶中 blob 容器的名稱。 使用下列命令來擷取儲存體帳戶金鑰：

```powershell
Get-AzureRmStorageAccountKey `
  -Name {your storage account name} `
  -ResourceGroupName {your storage account resource group}
```

記下 **key1** 儲存體帳戶金鑰值。 您需要在後續步驟中用到此值。

您可以使用現有的 blob 容器進行檔案上傳，或建立新容器：

* 若要列出儲存體帳戶中現有的 blob 容器，使用下列命令：

    ```powershell
    $ctx = New-AzureStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    Get-AzureStorageContainer -Context $ctx
    ```

* 若要在儲存體帳戶中建立 blob 容器，使用下列命令：

    ```powershell
    $ctx = New-AzureStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    New-AzureStorageContainer `
        -Name {your new container name} `
        -Permission Off `
        -Context $ctx
    ```

## <a name="configure-your-iot-hub"></a>設定 IoT 中樞

您現在可以使用儲存體帳戶詳細資料來設定 IoT 中樞，以[將檔案上傳到 IoT 中樞](iot-hub-devguide-file-upload.md)。

設定需要下列值：

* **儲存體容器**︰您目前 Azure 訂用帳戶內 Azure 儲存體帳戶中的 blob 容器，會與您的 IoT 中樞產生關聯。 您已在上一節中擷取了必要的儲存體帳戶資訊。 IoT 中樞會自動產生具有此 Blob 容器寫入權限的 SAS URI，以供裝置上傳檔案時使用。

* **接收已上傳檔案的通知**︰啟用或停用檔案上傳通知。

* **SAS TTL**︰這個設定是「IoT 中樞」傳回給裝置之 SAS URI 的存留時間。 預設會設為一小時。

* **檔案通知設定預設 TTL**：檔案上傳通知到期前的存留時間。 預設會設為一天。

* **檔案通知最大傳遞計數**︰IoT 中樞可嘗試傳遞檔案上傳通知的次數。 預設會設為 10。

使用下列 PowerShell Cmdlet 來設定 IoT 中樞上的檔案上傳設定：

```powershell
Set-AzureRmIotHub `
    -ResourceGroupName "{your iot hub resource group}" `
    -Name "{your iot hub name}" `
    -FileUploadNotificationTtl "01:00:00" `
    -FileUploadSasUriTtl "01:00:00" `
    -EnableFileUploadNotifications $true `
    -FileUploadStorageConnectionString "DefaultEndpointsProtocol=https;AccountName={your storage account name};AccountKey={your storage account key};EndpointSuffix=core.windows.net" `
    -FileUploadContainerName "{your blob container name}" `
    -FileUploadNotificationMaxDeliveryCount 10
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