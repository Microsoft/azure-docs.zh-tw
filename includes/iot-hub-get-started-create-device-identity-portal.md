---
title: 包含檔案
description: 包含檔案
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 05/17/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: ffd5da239f8e271a8c9b2aaf3f6d5fd9f885c79c
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49400282"
---
## <a name="create-a-device-identity"></a>建立裝置識別

在本節中，您會使用 [Azure 入口網站](https://portal.azure.com)，在 IoT 中樞的身分識別登錄中建立裝置身分識別。 裝置無法連線到 IoT 中樞，除非它在身分識別登錄中具有項目。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南](../articles/iot-hub/iot-hub-devguide-identity-registry.md)中的＜身分識別登錄＞一節 使用入口網站中的 [IoT 裝置] 面板，為您的裝置產生唯一的裝置識別碼和金鑰，用來向 IoT 中樞識別該裝置。 裝置識別碼會區分大小寫。

1. 登入 [Azure 入口網站](https://portal.azure.com)

2. 選取 [所有資源] 並尋找您的 IoT 中樞資源。

3. 當您的 IoT 中樞資源開啟時，按一下 [IoT 裝置] 工具，然後按一下頂端的 [新增]。 

    ![在入口網站中建立裝置識別](./media/iot-hub-get-started-create-device-identity-portal/create-identity-portal.png)

4. 提供新裝置的名稱 (例如 **myDeviceId**)，然後按一下 [儲存]。 此動作會為您的 IoT 中樞建立新的裝置身分識別。

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

   ![新增裝置](./media/iot-hub-get-started-create-device-identity-portal/create-a-device.png)

5. 在裝置清單中，按一下新建立的裝置，並複製 [連接字串---主索引鍵] 以供日後使用。

    ![裝置連接字串](./media/iot-hub-get-started-create-device-identity-portal/device-details.png)

> [!NOTE]
> IoT 中樞身分識別登錄只會儲存裝置身分識別，以啟用對 IoT 中樞的安全存取。 它會儲存裝置識別碼和金鑰來作為安全性認證，以及啟用或停用旗標，讓您用來停用個別裝置的存取。 如果您的應用程式需要儲存其他裝置特定的中繼資料，它應該使用應用程式專用的存放區。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南](../articles/iot-hub/iot-hub-devguide-identity-registry.md)。
