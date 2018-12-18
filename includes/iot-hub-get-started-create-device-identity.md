---
title: 包含檔案
description: 包含檔案
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 09/07/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: e80033d696de1b83da43fc27e5be9eca3b3f8757
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44169111"
---
## <a name="create-a-device-identity"></a>建立裝置識別

在本節中，您可以針對本教學課程使用 Azure CLI 建立裝置身分識別。 Azure CLI 會預先安裝在 [Azure Cloud Shell](https://docs.microsoft.com/zure/cloud-shell/overview)中，或是您可以將其[安裝在本機上](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。 裝置識別碼會區分大小寫。

1. 在您使用 Azure CLI 安裝 IoT 擴充功能的命令列環境中，執行下列命令：

    ```cmd/sh
    az extension add --name azure-cli-iot-ext
    ```

1. 如果您在本機執行 Azure CLI，請使用下列命令來登入您的 Azure 帳戶 (如果您使用 Cloud Shell，您會自動登入，而且不需要執行此命令)：

    ```cmd/sh
    az login
    ```

1. 最後，建立稱為 `myDeviceId` 的新裝置身分識別，並使用下列命令擷取裝置連接字串：

    ```cmd/sh
    az iot hub device-identity create --device-id myDeviceId --hub-name {Your IoT Hub name}
    az iot hub device-identity show-connection-string --device-id myDeviceId --hub-name {Your IoT Hub name} -o table
    ```

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

記下結果中的裝置連接字串。 裝置應用程式會使用此裝置連接字串，以裝置的形式連接到您的 IoT 中樞。

<!-- images and links -->
